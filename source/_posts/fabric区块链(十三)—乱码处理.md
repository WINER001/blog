---
title: fabric区块链(十三)—fabric中GetBlockByNumber乱码
date: 2023/8/1 11:12
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/31.jpg
---

# fabric中GetBlockByNumber乱码



## 一，问题描述

 peer chaincode query -C mychannel -n qscc -c '{"Args":["GetBlockByNumber", "mychannel", "3"]}'



## 二，GetBlockByNumber源码

```go
func getBlockByNumber(vledger ledger.PeerLedger, number []byte) pb.Response {
	if number == nil {
		return shim.Error("Block number must not be nil.")
	}
	bnum, err := strconv.ParseUint(string(number), 10, 64)
	if err != nil {
		return shim.Error(fmt.Sprintf("Failed to parse block number with error %s", err))
	}
	block, err := vledger.GetBlockByNumber(bnum)
	if err != nil {
		return shim.Error(fmt.Sprintf("Failed to get block number %d, error %s", bnum, err))
	}
	// TODO: consider trim block content before returning
	//  Specifically, trim transaction 'data' out of the transaction array Payloads
	//  This will preserve the transaction Payload header,
	//  and client can do GetTransactionByID() if they want the full transaction details

	bytes, err := protoutil.Marshal(block)
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(bytes)
}
```

#### 发现返回的bytes数组是经过protoutil.Marshal加密的，所以获取后要进行解密



### 三，问题解决

```java
    public Block getBlockByNumber(Contract contract, String blockNumber) throws GatewayException, InvalidProtocolBufferException {
        
        FabricLogger.info(this,"\n--> Evaluate Transaction: GetBlockByNumber, block number : "+blockNumber);
        
        byte[] evaluateResult = contract.evaluateTransaction("GetBlockByNumber",Constant.ASOM_CHANNEL,blockNumber);
        
        //这里对byte数组进行解析，形成block对象
        Block block = Block.parseFrom(evaluateResult);

        FabricLogger.info(this,"block data:"+block.getData());
        FabricLogger.info(this,"block data hash:"+block.getHeader().getDataHash());
        
		//这里进行protoutil.unMarshal解密
        String result = JsonFormat.printer().print(block);
        FabricLogger.info(this,"result : "+result);

        return block;
        
    }
```

