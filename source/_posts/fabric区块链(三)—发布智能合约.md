---
title: fabric区块链(三)—发布智能合约
date: 2023/5/12
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/42.jpg
---

# 在Fabric上发布智能合约：

# TODO

1. ## 编写智能合约代码

您需要使用支持的编程语言（如Go、Java、JavaScript等）编写智能合约代码。您可以使用Hyperledger Fabric提供的示例智能合约作为参考，了解智能合约的结构和编写方式。示例智能合约可在Hyperledger Fabric的官方文档中找到。

以下是针对 Hyperledger Fabric 的智能合约示例：

### Go

```go
package main

import (
 "fmt"
 "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type MyContract struct {
 contractapi.Contract
}

func (c *MyContract) Set(ctx contractapi.TransactionContextInterface, key string, value string) error {
 err := ctx.GetStub().PutState(key, []byte(value))
 if err != nil {
  return fmt.Errorf("failed to set key %s: %v", key, err)
 }
 return nil
}

func (c *MyContract) Get(ctx contractapi.TransactionContextInterface, key string) (string, error) {
 value, err := ctx.GetStub().GetState(key)
 if err != nil {
  return "", fmt.Errorf("failed to get key %s: %v", key, err)
 }
 if value == nil {
  return "", fmt.Errorf("key %s does not exist", key)
 }
 return string(value), nil
}

func main() {
 cc, err := contractapi.NewChaincode(&MyContract{})
 if err != nil {
  fmt.Printf("Error creating MyContract chaincode: %s", err.Error())
  return
 }
 if err := cc.Start(); err != nil {
  fmt.Printf("Error starting MyContract chaincode: %s", err.Error())
 }
}
```

### Java

```java
package org.example;

import org.hyperledger.fabric.contract.ContractInterface;
import org.hyperledger.fabric.contract.annotation.Contract;
import org.hyperledger.fabric.contract.annotation.Default;
import org.hyperledger.fabric.contract.annotation.Transaction;

@Contract(
        name = "MyContract",
        info = @Default(
                org.hyperledger.fabric.contract.annotation.Info(
                        title = "My Contract",
                        description = "My Smart Contract"
                )
        )
)
public final class MyContract implements ContractInterface {

    @Transaction()
    public void set(final Context ctx, final String key, final String value) {
        byte[] buffer = value.getBytes();
        ctx.getStub().putState(key, buffer);
    }

    @Transaction()
    public String get(final Context ctx, final String key) {
        byte[] buffer = ctx.getStub().getState(key);
        if (buffer == null || buffer.length == 0) {
            throw new RuntimeException("Key does not exist");
        }
        return new String(buffer);
    }
}
```

### JavaScript

```javascript
'use strict';

const { Contract } = require('fabric-contract-api');

class MyContract extends Contract {
    async set(ctx, key, value) {
        await ctx.stub.putState(key, Buffer.from(value));
    }

    async get(ctx, key) {
        const value = await ctx.stub.getState(key);
        if (!value || value.length === 0) {
            throw new Error(`Key ${key} does not exist`);
        }
        return value.toString();
    }
}

module.exports = MyContract;
```

这些示例仅仅是 Hyperledger Fabric 的智能合约的简单入门，实际的智能合约通常更加复杂。

2. ## 打包智能合约代码


在发布智能合约之前，需要将智能合约代码打包为一个可执行文件。要打包代码，需要使用`peer lifecycle chaincode package`命令。以下是一个示例命令：

```
peer lifecycle chaincode package mycc.tar.gz --path /path/to/chaincode --lang node --label mycc_1.0
```

这个命令将使用`/path/to/chaincode`目录中的Node.js链码代码创建一个名为`mycc_1.0`的标签，并将其打包为`mycc.tar.gz`文件。



3. ## 安装智能合约

要安装智能合约，您需要使用`peer lifecycle chaincode install`命令。以下是一个示例命令：

```
peer lifecycle chaincode install mycc.tar.gz
```

这个命令将`mycc.tar.gz`文件安装到Peer节点上。请注意，此命令将返回智能合约的包ID，需要将其用于后续步骤。



4. ## 审批智能合约

在将智能合约发布到通道之前，您需要审批智能合约定义。要审批智能合约，您需要使用`peer lifecycle chaincode approveformyorg`命令。以下是一个示例命令：

```
peer lifecycle chaincode approveformyorg --channelID mychannel --name mycc --version 1.0 --package-id mycc_1.0:3b3f3d55f08c7d8e00fcb6f10ef6da24d158d371fe123bf54291be7cf32d1703 --sequence 1 --tls true --cafile /path/to/orderer/ca.crt
```

这个命令将批准`mychannel`通道上的`mycc`链码的1.0版本。请注意，此命令需要指定用于连接到Orderer的TLS证书和根证书。

5. ## 将智能合约提交到通道

   

在审批智能合约之后，您需要将其提交到通道。要将智能合约提交到通道，您需要使用`peer lifecycle chaincode commit`命令。以下是一个示例命令：

```
peer lifecycle chaincode commit -o localhost:7050 --tls true --cafile /path/to/orderer/ca.crt --channelID mychannel --name mycc --version 1.0 --sequence 1 --init-required --tlsRootCertFiles /path/to/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:7051 --tlsRootCertFiles /path/to/peer0.org1.example.com/tls