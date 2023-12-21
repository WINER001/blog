---
title: 深入解析fabric的peer命令(三)chaincodeInvokeOrQuery方法
date: 2023/6/3 10:24
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/66.jpg
---

### 

# 兜兜转转，来到了fabric/internal/peer/common/common.go



## chaincode.go调用的是chaincodeInvokeOrQuery方法



### 1.chaincodeInvokeOrQuery(Command,bool,ChaincodeCmdFactory)方法源码

```go
func chaincodeInvokeOrQuery(cmd *cobra.Command, invoke bool, cf *ChaincodeCmdFactory) (err error) {
	spec, err := getChaincodeSpec(cmd)
	if err != nil {
		return err
	}

	// call with empty txid to ensure production code generates a txid.
	// otherwise, tests can explicitly set their own txid
	txID := ""

	proposalResp, err := ChaincodeInvokeOrQuery(
		spec,
		channelID,
		txID,
		invoke,
		cf.Signer,
		cf.Certificate,
		cf.EndorserClients,
		cf.DeliverClients,
		cf.BroadcastClient,
	)
	if err != nil {
		return errors.Errorf("%s - proposal response: %v", err, proposalResp)
	}

	if invoke {
		logger.Debugf("ESCC invoke result: %v", proposalResp)
		pRespPayload, err := protoutil.UnmarshalProposalResponsePayload(proposalResp.Payload)
		if err != nil {
			return errors.WithMessage(err, "error while unmarshalling proposal response payload")
		}
		ca, err := protoutil.UnmarshalChaincodeAction(pRespPayload.Extension)
		if err != nil {
			return errors.WithMessage(err, "error while unmarshalling chaincode action")
		}
		if proposalResp.Endorsement == nil {
			return errors.Errorf("endorsement failure during invoke. response: %v", proposalResp.Response)
		}
		logger.Infof("Chaincode invoke successful. result: %v", ca.Response)
	} else {
		if proposalResp == nil {
			return errors.New("error during query: received nil proposal response")
		}
		if proposalResp.Endorsement == nil {
			return errors.Errorf("endorsement failure during query. response: %v", proposalResp.Response)
		}

		if chaincodeQueryRaw && chaincodeQueryHex {
			return fmt.Errorf("options --raw (-r) and --hex (-x) are not compatible")
		}
		if chaincodeQueryRaw {
			fmt.Println(proposalResp.Response.Payload)
			return nil
		}
		if chaincodeQueryHex {
			fmt.Printf("%x\n", proposalResp.Response.Payload)
			return nil
		}
		fmt.Println(string(proposalResp.Response.Payload))
	}
	return nil
}
```



#### 通过源码可以看到调用了ChaincodeInvokeOrQuery（spec,channelID,txID,invoke,cf...）方法，返回是proposalResp，

#### 然后如果是query指令，就返回fmt.Println(string(proposalResp.Response.Payload))

#### 如果是invoke指令，就返回logger.Infof("Chaincode invoke successful. result: %v", ca.Response)



#### 看来，还得继续深入ChaincodeInvokeOrQuery方法。



### 2.ChaincodeInvokeOrQuery（spec,channelID,txID,invoke,cf...）方法源码

```go
// ChaincodeInvokeOrQuery invokes or queries the chaincode. If successful, the
// INVOKE form prints the ProposalResponse to STDOUT, and the QUERY form prints
// the query result on STDOUT. A command-line flag (-r, --raw) determines
// whether the query result is output as raw bytes, or as a printable string.
// The printable form is optionally (-x, --hex) a hexadecimal representation
// of the query response. If the query response is NIL, nothing is output.
//
// NOTE - Query will likely go away as all interactions with the endorser are
// Proposal and ProposalResponses
func ChaincodeInvokeOrQuery(
	spec *pb.ChaincodeSpec,
	cID string,
	txID string,
	invoke bool,
	signer identity.SignerSerializer,
	certificate tls.Certificate,
	endorserClients []pb.EndorserClient,
	deliverClients []pb.DeliverClient,
	bc common.BroadcastClient,
) (*pb.ProposalResponse, error) {
	// Build the ChaincodeInvocationSpec message
	invocation := &pb.ChaincodeInvocationSpec{ChaincodeSpec: spec}

	creator, err := signer.Serialize()
	if err != nil {
		return nil, errors.WithMessage(err, "error serializing identity")
	}

	funcName := "invoke"
	if !invoke {
		funcName = "query"
	}

	// extract the transient field if it exists
	var tMap map[string][]byte
	if transient != "" {
		if err := json.Unmarshal([]byte(transient), &tMap); err != nil {
			return nil, errors.Wrap(err, "error parsing transient string")
		}
	}

	prop, txid, err := protoutil.CreateChaincodeProposalWithTxIDAndTransient(pcommon.HeaderType_ENDORSER_TRANSACTION, cID, invocation, creator, txID, tMap)
	if err != nil {
		return nil, errors.WithMessagef(err, "error creating proposal for %s", funcName)
	}

	signedProp, err := protoutil.GetSignedProposal(prop, signer)
	if err != nil {
		return nil, errors.WithMessagef(err, "error creating signed proposal for %s", funcName)
	}

	responses, err := processProposals(endorserClients, signedProp)
	if err != nil {
		return nil, errors.WithMessagef(err, "error endorsing %s", funcName)
	}

	if len(responses) == 0 {
		// this should only happen if some new code has introduced a bug
		return nil, errors.New("no proposal responses received - this might indicate a bug")
	}
	// all responses will be checked when the signed transaction is created.
	// for now, just set this so we check the first response's status
	proposalResp := responses[0]

	if invoke {
		if proposalResp != nil {
			if proposalResp.Response.Status >= shim.ERRORTHRESHOLD {
				return proposalResp, nil
			}
			// assemble a signed transaction (it's an Envelope message)
			env, err := protoutil.CreateSignedTx(prop, signer, responses...)
			if err != nil {
				return proposalResp, errors.WithMessage(err, "could not assemble transaction")
			}
			var dg *DeliverGroup
			var ctx context.Context
			if waitForEvent {
				var cancelFunc context.CancelFunc
				ctx, cancelFunc = context.WithTimeout(context.Background(), waitForEventTimeout)
				defer cancelFunc()

				dg = NewDeliverGroup(
					deliverClients,
					peerAddresses,
					signer,
					certificate,
					channelID,
					txid,
				)
				// connect to deliver service on all peers
				err := dg.Connect(ctx)
				if err != nil {
					return nil, err
				}
			}

			// send the envelope for ordering
			if err = bc.Send(env); err != nil {
				return proposalResp, errors.WithMessagef(err, "error sending transaction for %s", funcName)
			}

			if dg != nil && ctx != nil {
				// wait for event that contains the txid from all peers
				err = dg.Wait(ctx)
				if err != nil {
					return nil, err
				}
			}
		}
	}

	return proposalResp, nil
}
```



#### 总结一下，这个方法有以下几个步骤：

#### 1.通过invocation := &pb.ChaincodeInvocationSpec{ChaincodeSpec: spec}创建一个调用invoke示例

#### 2.通过protoutil.CreateChaincodeProposalWithTxIDAndTransient进行签名，生成一个链码提案prop

#### 3.通过protoutil.GetSignedProposal(prop, signer)将提案签名，获取一个签名过的提案signedProp

#### 4.通过processProposals(endorserClients, signedProp)，调用`processProposals`方法，传递背书客户端和签名的提案，处理提案并获取响应responses。

#### 5.通过protoutil.CreateSignedTx(prop, signer, responses...)创建一个签名的交易env，用来广播

#### 6.通过bc.Send(env)用广播客户端发送交易



#### 分析可知，调用的关键在第四步，processProposals方法



### 3.processProPosals方法源码

```go
func processProposals(endorserClients []pb.EndorserClient, signedProposal *pb.SignedProposal) ([]*pb.ProposalResponse, error) {
	responsesCh := make(chan *pb.ProposalResponse, len(endorserClients))
	errorCh := make(chan error, len(endorserClients))
	wg := sync.WaitGroup{}
	for _, endorser := range endorserClients {
		wg.Add(1)
		go func(endorser pb.EndorserClient) {
			defer wg.Done()
			proposalResp, err := endorser.ProcessProposal(context.Background(), signedProposal)
			if err != nil {
				errorCh <- err
				return
			}
			responsesCh <- proposalResp
		}(endorser)
	}
	wg.Wait()
	close(responsesCh)
	close(errorCh)
	for err := range errorCh {
		return nil, err
	}
	var responses []*pb.ProposalResponse
	for response := range responsesCh {
		responses = append(responses, response)
	}
	return responses, nil
}
```

#### 关键在endorser.ProcessProposal(context.Background(),signedProposal)一句



### 这个方法居然到了另一个包里fabric-protos-go！

`fabric-protos-go`是Hyperledger Fabric项目中的一个Go语言包，用于定义和生成与Fabric网络通信相关的协议缓冲区（Protocol Buffers）消息。

Protocol Buffers是一种语言无关、平台无关、可扩展的序列化机制，广泛用于不同系统之间的数据交换。Hyperledger Fabric使用Protocol Buffers来定义网络中的消息格式，以便在不同的组件之间进行通信。

`fabric-protos-go`包含了一系列的Protocol Buffers消息定义，这些定义描述了与Fabric网络中的各个组件进行交互所使用的消息结构和字段。这些组件包括区块链网络的成员（如节点、通道、链码等），消息传输协议（如gRPC和事件）以及与智能合约相关的操作（如事务提案、背书等）。

通过使用`fabric-protos-go`包，开发人员可以方便地在自己的应用程序中创建、序列化和反序列化Fabric网络消息，以与Fabric网络进行交互。这个包提供了一种方便的方式来处理与Fabric网络通信相关的复杂数据结构，而不需要手动解析和构建字节流。

总之，`fabric-protos-go`是一个关键的Go语言包，它提供了在Hyperledger Fabric网络中进行通信所需的消息定义和相关功能，使开发人员能够轻松地与Fabric网络进行交互和集成。

我怀疑这个包是所有sdk的底层，也是peer指令最后的底层

### 那看看吧，下一篇

https://xiutuuu.cloud/2023/06/03/%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90fabric%E7%9A%84peer%E5%91%BD%E4%BB%A44/