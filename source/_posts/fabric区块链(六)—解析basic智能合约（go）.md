---
title: fabric区块链(六)—解析basic智能合约（go）
date: 2023/5/21 10:28
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/61.png
---

# 解析basic智能合约（go）：

![](./img/61.png)

### basic合约是我们之前在调用示例合约的时候调用的合约，之前分析过java语言编写的，再分析一下go语言编写的。

### fabric官方提供了源码，在fabric/scripts/fabric-samples/asset-transfer-basic/chaincode-go目录

### 先分析一下这个目录结构

chaincode-go

├── assetTransfer.go（应用程序的主要入口文件或资产转移的核心逻辑实现文件。）

├── chaincode （包含与区块链智能合约相关的文件。）

│  ├── mocks （包含一些用于测试目的的模拟文件或桩文件。）

│  │  ├── chaincodestub.go （用于模拟链码（chaincode）的存根（stub）文件。）

│  │  ├── statequeryiterator.go （用于模拟状态查询迭代器的文件。）

│  │  └── transaction.go （模拟交易的文件。）

│  ├── smartcontract.go （区块链智能合约的实现文件。）

│  └── smartcontract_test.go （用于测试区块链智能合约的测试文件。）

├── go.mod （Go 项目的模块文件，其中定义了项目的依赖关系和版本信息。）

├── go.sum （包含项目依赖项的安全校验和的文件，用于验证下载的包的完整性。）

└── vendor（包含项目依赖项的目录，通常在使用特定版本管理工具时会将依赖项放在此目录中。）





### 然后再逐一看看内部的代码

#### assertTransfer.go

```go
package main

import (
        "log"
        "github.com/hyperledger/fabric-contract-api-go/contractapi"
        "github.com/hyperledger/fabric-samples/asset-transfer-basic/chaincode-go/chaincode"
)

func main() {
        assetChaincode, err := contractapi.NewChaincode(&chaincode.SmartContract{})
        if err != nil {
                log.Panicf("Error creating asset-transfer-basic chaincode: %v", err)
        }

        if err := assetChaincode.Start(); err != nil {
                log.Panicf("Error starting asset-transfer-basic chaincode: %v", err)
        }
}
```

#### 然后逐行分析：

1. `package main`: 这是Go语言中的一个包声明，表明这个文件是一个可执行程序的入口文件。

2. `import (...)`: 这里导入了多个包，用于引入所需的依赖项。

   - `"log"`: 这是Go语言标准库中的一个包，用于记录日志信息。

   - `"github.com/hyperledger/fabric-contract-api-go/contractapi"`: 这是一个Hyperledger Fabric提供的用于编写智能合约的Go API的包。

   - `"github.com/hyperledger/fabric-samples/asset-transfer-basic/chaincode-go/chaincode"`: 这是一个与Hyperledger Fabric示例中的资产转移基础应用程序相关的自定义包。

3. `func main() { ... }`: 这是程序的入口函数，它是程序启动时第一个被执行的函数。

4. `assetChaincode, err := contractapi.NewChaincode(&chaincode.SmartContract{})`: 这行代码创建了一个基于`contractapi`包的新的链码（chaincode）实例，并将其赋值给变量`assetChaincode`。它使用`chaincode.SmartContract{}`作为智能合约的实现。

5. `if err != nil { ... }`: 这是一个错误处理的条件语句，用于检查链码实例的创建过程中是否发生了错误。如果发生错误，将会输出错误日志并终止程序运行。

6. `if err := assetChaincode.Start(); err != nil { ... }`: 这是另一个错误处理的条件语句，用于检查链码实例的启动过程中是否发生了错误。如果发生错误，将会输出错误日志并终止程序运行。

在总体上，这段代码的功能是创建一个基于Hyperledger Fabric的区块链应用程序，并启动该应用程序的链码实例。它使用了Hyperledger Fabric提供的链码API和自定义的智能合约实现。如果创建或启动过程中出现错误，程序将输出相应的错误日志并终止运行。





#### 再看一下smartcontract.go

```go
package chaincode

import (
        "encoding/json"
        "fmt"

        "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract provides functions for managing an Asset
type SmartContract struct {
        contractapi.Contract
}

// Asset describes basic details of what makes up a simple asset
// Insert struct field in alphabetic order => to achieve determinism across languages
// golang keeps the order when marshal to json but doesn't order automatically
type Asset struct {
        AppraisedValue int    `json:"AppraisedValue"`
        Color          string `json:"Color"`
        ID             string `json:"ID"`
        Owner          string `json:"Owner"`
        Size           int    `json:"Size"`
}

// InitLedger adds a base set of assets to the ledger
func (s *SmartContract) InitLedger(ctx contractapi.TransactionContextInterface) error {
        assets := []Asset{
                {ID: "asset1", Color: "blue", Size: 5, Owner: "Tomoko", AppraisedValue: 300},
                {ID: "asset2", Color: "red", Size: 5, Owner: "Brad", AppraisedValue: 400},
                {ID: "asset3", Color: "green", Size: 10, Owner: "Jin Soo", AppraisedValue: 500},
                {ID: "asset4", Color: "yellow", Size: 10, Owner: "Max", AppraisedValue: 600},
                {ID: "asset5", Color: "black", Size: 15, Owner: "Adriana", AppraisedValue: 700},
                {ID: "asset6", Color: "white", Size: 15, Owner: "Michel", AppraisedValue: 800},
        }

        for _, asset := range assets {
                assetJSON, err := json.Marshal(asset)
                if err != nil {
                        return err
                }

                err = ctx.GetStub().PutState(asset.ID, assetJSON)
                if err != nil {
                        return fmt.Errorf("failed to put to world state. %v", err)
                }
        }

        return nil
}

// CreateAsset issues a new asset to the world state with given details.
func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
        exists, err := s.AssetExists(ctx, id)
        if err != nil {
                return err
        }
        if exists {
                return fmt.Errorf("the asset %s already exists", id)
        }

        asset := Asset{
                ID:             id,
                Color:          color,
                Size:           size,
                Owner:          owner,
                AppraisedValue: appraisedValue,
        }
        assetJSON, err := json.Marshal(asset)
        if err != nil {
                return err
        }

        return ctx.GetStub().PutState(id, assetJSON)
}

// ReadAsset returns the asset stored in the world state with given id.
func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, id string) (*Asset, error) {
        assetJSON, err := ctx.GetStub().GetState(id)
        if err != nil {
                return nil, fmt.Errorf("failed to read from world state: %v", err)
        }
        if assetJSON == nil {
                return nil, fmt.Errorf("the asset %s does not exist", id)
        }

        var asset Asset
        err = json.Unmarshal(assetJSON, &asset)
        if err != nil {
                return nil, err
        }

        return &asset, nil
}

// UpdateAsset updates an existing asset in the world state with provided parameters.
func (s *SmartContract) UpdateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
        exists, err := s.AssetExists(ctx, id)
        if err != nil {
                return err
        }
        if !exists {
                return fmt.Errorf("the asset %s does not exist", id)
        }

        // overwriting original asset with new asset
        asset := Asset{
                ID:             id,
                Color:          color,
                Size:           size,
                Owner:          owner,
                AppraisedValue: appraisedValue,
        }
        assetJSON, err := json.Marshal(asset)
        if err != nil {
                return err
        }

        return ctx.GetStub().PutState(id, assetJSON)
}

// DeleteAsset deletes an given asset from the world state.
func (s *SmartContract) DeleteAsset(ctx contractapi.TransactionContextInterface, id string) error {
        exists, err := s.AssetExists(ctx, id)
        if err != nil {
                return err
        }
        if !exists {
                return fmt.Errorf("the asset %s does not exist", id)
        }

        return ctx.GetStub().DelState(id)
}

// AssetExists returns true when asset with given ID exists in world state
func (s *SmartContract) AssetExists(ctx contractapi.TransactionContextInterface, id string) (bool, error) {
        assetJSON, err := ctx.GetStub().GetState(id)
        if err != nil {
                return false, fmt.Errorf("failed to read from world state: %v", err)
        }

        return assetJSON != nil, nil
}

// TransferAsset updates the owner field of asset with given id in world state, and returns the old owner.
func (s *SmartContract) TransferAsset(ctx contractapi.TransactionContextInterface, id string, newOwner string) (string, error) {
        asset, err := s.ReadAsset(ctx, id)
        if err != nil {
                return "", err
        }

        oldOwner := asset.Owner
        asset.Owner = newOwner

        assetJSON, err := json.Marshal(asset)
        if err != nil {
                return "", err
        }

        err = ctx.GetStub().PutState(id, assetJSON)
        if err != nil {
                return "", err
        }

        return oldOwner, nil
}

// GetAllAssets returns all assets found in world state
func (s *SmartContract) GetAllAssets(ctx contractapi.TransactionContextInterface) ([]*Asset, error) {
        // range query with empty string for startKey and endKey does an
        // open-ended query of all assets in the chaincode namespace.
        resultsIterator, err := ctx.GetStub().GetStateByRange("", "")
        if err != nil {
                return nil, err
        }
        defer resultsIterator.Close()

        var assets []*Asset
        for resultsIterator.HasNext() {
                queryResponse, err := resultsIterator.Next()
                if err != nil {
                        return nil, err
                }

                var asset Asset
                err = json.Unmarshal(queryResponse.Value, &asset)
                if err != nil {
                        return nil, err
                }
                assets = append(assets, &asset)
        }

        return assets, nil
}

```

