---
title: fabric区块链(九)—按区块查询
date: 2023/6/29 10:12
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/29.jpg
---

# 按区块查询

在Hyperledger Fabric中，要按区块进行查询，可以使用`GetBlockByNumber`函数来检索指定区块的详细信息。该函数允许您按区块号查询区块数据，并获取包括交易、哈希、前一个区块哈希等在内的相关信息。

以下是对`GetBlockByNumber`函数的详细解析，包括用法和一个示例：

**函数签名：**
```go
func (stub *ChaincodeStub) GetBlockByNumber(blockNumber uint64) (*common.Block, error)
```

**参数说明：**

- `blockNumber`：要查询的区块号。

**返回值：**

- `*common.Block`：表示查询到的区块数据。
- `error`：如果查询发生错误，则返回错误信息。

**示例用法：**

```go
// 导入所需的包
import (
    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
    "github.com/hyperledger/fabric/protos/common"
    "github.com/golang/protobuf/proto"
)

// 定义链码结构体
type MyChaincode struct {
}

// 实现链码的Invoke函数
func (cc *MyChaincode) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // 调用GetBlockByNumber函数查询区块
    blockNumber := uint64(1)
    block, err := stub.GetBlockByNumber(blockNumber)
    if err != nil {
        return shim.Error(err.Error())
    }

    // 序列化区块数据为字节流
    blockBytes, err := proto.Marshal(block)
    if err != nil {
        return shim.Error(err.Error())
    }

    // 返回查询结果
    return shim.Success(blockBytes)
}
```

在上述示例中，我们创建了一个名为`MyChaincode`的链码结构体，并实现了其`Invoke`函数。在`Invoke`函数中，我们调用了`GetBlockByNumber`函数来查询指定区块号为1的区块数据。

`GetBlockByNumber`函数返回一个`*common.Block`对象，表示查询到的区块数据。我们使用`proto.Marshal`函数将该对象序列化为字节流，以便在链码中返回。

最后，我们将查询结果作为成功的响应返回。

请注意，此示例是一个简化版本，并未包含完整的链码实现和必要的错误处理。在实际开发中，应该根据具体需求进行适当的修改和错误处理。
