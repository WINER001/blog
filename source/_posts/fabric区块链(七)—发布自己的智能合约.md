---
title: fabric区块链(七)—发布自己的智能合约（go）
date: 2023/5/23 13:17
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/34.jpg
---

# 发布自己的智能合约（go）

### 废话不多说，先贴出来自己的合约代码，我的这个合约是基于官方提供的basic合约模拟写的

## 1.合约代码

### 项目结构

mycontract
├── CreateContract.go（应用程序的主要入口文件或资产转移的核心逻辑实现文件。）
├── bo（包含与区块链智能合约相关的文件。）
│  	├── WineContract.go （区块链智能合约的实现文件。）
├── go.mod （Go 项目的模块文件，其中定义了项目的依赖关系和版本信息。）
├── go.sum （包含项目依赖项的安全校验和的文件，用于验证下载的包的完整性。）
└── vendor（包含项目依赖项的目录，通常在使用特定版本管理工具时会将依赖项放在此目录中。）



### CreateContract.go

```go
package main

import (
	"fmt"
	"log"
	"mycontract/bo"
	"github.com/hyperledger/fabric-contract-api-go/contractapi"

)

func main(){
	fmt.Println("create trade contract start ...")
	accountChainCode, err := contractapi.NewChaincode(&bo.WineContract{})
	if err != nil{
		log.Panicf("create trade contract fail : %v",err)
	}

	if err := accountChainCode.Start(); err != nil{
		log.Panicf("start trade contract fail : %v",err)
	}
	fmt.Println("create trade contract end ...")
}
```

### WineContract.go

```go
package bo



import(
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

//继承contractapi的Contract类（go语言里叫嵌入和结构体）
type WineContract struct{
	contractapi.Contract
}

//Wine
type Wine struct{
	ID string `json:"ID"`
	Name string `json:"Name"`
	Value string `json:"Value"`
	Owner string `json:"Owner"`
	Birth string `json:"Birth"`
	Capacity string `json:"Capacity"`
}

//初始化酒窖
func (s *WineContract) InitWineCellar(ctx contractapi.TransactionContextInterface) error{
	wineCellar := []Wine{
		{ID:"11111",Name: "二锅头", Value: "￥20", Owner: "路人甲", Birth: "2000",Capacity: "500ML"},
		{ID:"22222",Name: "拉菲", Value: "￥20000", Owner: "路人乙", Birth: "1882",Capacity: "500ML"},
		{ID:"33333",Name: "国窖1573", Value: "￥1080", Owner: "路人甲", Birth: "1998",Capacity: "500ML"},
		{ID:"44444",Name: "茅台", Value: "￥3080", Owner: "路人乙", Birth: "1949",Capacity: "400ML"},
	}

	for _, wine := range wineCellar {
		wineJSON, err := json.Marshal(wine)
		if err != nil{
			return err
		}

		err = ctx.GetStub().PutState(wine.ID,wineJSON)
		if err != nil{
			return fmt.Errorf("failed to put to world state. %v",err)
		}
	}

	return nil
}

//insert
func (s *WineContract) CreateWine(ctx contractapi.TransactionContextInterface,id string,name string,value string,owner string, birth string,capacity string)error{
	exists,err := s.WineExists(ctx, id)
	if err != nil{
		return err
	}
	if exists {
		return fmt.Errorf("this wine %s already exists",id)
	}

	wine := Wine{
		ID: id,
		Name: name,
		Value: value,
		Owner: owner,
		Birth: birth,
		Capacity: capacity,
	}

	wineJSON,err := json.Marshal(wine)
	if err != nil{
		return err
	}

	return ctx.GetStub().PutState(id,wineJSON)

}

//selete
func (s *WineContract) ReadWine(ctx contractapi.TransactionContextInterface,id string)(*Wine,error){
	wineJSON, err := ctx.GetStub().GetState(id)
	if err != nil{
		return nil, fmt.Errorf("failed to read from world state %v",err)
	}
	if wineJSON == nil{
		return nil, fmt.Errorf("wine %s does not exist",id)
	}

	var wine Wine
	err = json.Unmarshal(wineJSON,&wine)
	if err != nil{
		return nil, err
	}

	return &wine,nil
}

//update
func (s *WineContract)UpdateWine(ctx contractapi.TransactionContextInterface,id string,name string,value string,owner string, birth string,capacity string)error{
	exists, err := s.WineExists(ctx,id)
	if err != nil{
		return err
	}

	if !exists{
		return fmt.Errorf("the wine %s does not exist",id)
	}

	//overwrite
	wine := Wine{
		ID: id,
		Name: name,
		Value: value,
		Owner: owner,
		Birth: birth,
		Capacity: capacity,
	}
	wineJSON, err := json.Marshal(wine)
	if err != nil{
		return err
	}
	return ctx.GetStub().PutState(id,wineJSON)
}

func (s *WineContract) DeleteWine(ctx contractapi.TransactionContextInterface,id string)error{
	exists,err := s.WineExists(ctx,id)
	if err != nil{
		return err
	}

	if !exists{
		return fmt.Errorf("the wine %s does not exist",id)
	}

	return ctx.GetStub().DelState(id)
}

func(s *WineContract) WineExists(ctx contractapi.TransactionContextInterface,id string)(bool,error){
	wineJSON,err := ctx.GetStub().GetState(id)
	if err != nil{
		return false,fmt.Errorf("failed to read from world state : %v",err)
	}

	return wineJSON != nil,nil
}

//transfer,update owner
func(s *WineContract) TransferWine(ctx contractapi.TransactionContextInterface,id string,newOwner string)(string,error){
	wine,err := s.ReadWine(ctx,id)
	if err != nil{
		return "",err
	}
	oldOwner := wine.Owner
	wine.Owner = newOwner

	wineJSON,err := json.Marshal(wine)
	if err != nil{
		return "",err
	}

	err = ctx.GetStub().PutState(id,wineJSON)
	if err != nil{
		return "",err
	}

	return oldOwner,nil
}

func (s *WineContract) GetAllWines(ctx contractapi.TransactionContextInterface)([]*Wine,error){
	resultsIterator,err := ctx.GetStub().GetStateByRange("","")
	if err != nil{
		return nil,err
	}
	defer resultsIterator.Close()
	var wineCellar []*Wine
	for resultsIterator.HasNext(){
		queryResponse,err := resultsIterator.Next()
		if err != nil{
			return nil,err
		}

		var wine Wine
		err = json.Unmarshal(queryResponse.Value,&wine)
		if err != nil{
			return nil,err
		}
		wineCellar = append(wineCellar,&wine)
	}
	return wineCellar,nil
}

```



## 2.启动区块链网络

#### 1.先关停之前存在的网络

进入脚本目录

```sh
cd /home/githubworkspace/fabric/scripts/fabric-samples/test-network
```

关停网络

```sh
./network.sh down
```

#### 2.再重新开启网络

```sh
./network.sh up
```

#### 3.开通channel（winechannel）

```sh
./network.sh createChannel -c mychannel
```



## 3.发布智能合约



#### 1.进入/home/githubworkspace/fabric/scripts/fabric-samples/test-network目录

```sh
cd /home/githubworkspace/fabric/scripts/fabric-samples/test-network

```



#### 2.使用network.sh脚本发布智能合约

```sh
./network.sh deployCC -c winechannel -ccn mycontract -ccp /home/myfabric/mycontract -ccl go 
```

![](./img/67.png)

## 4.调用智能合约

#### 1.调用合约里的InitWineCellar方法初始化酒库

```sh
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C winechannel -n mycontract --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitWineCellar","Args":[]}'
```

![](./img/68.png)

#### 2.调用合约里的GetAllWines方法查询所有酒库里的酒

```sh
peer chaincode query -C winechannel -n mycontract -c '{"Args":["GetAllWines"]}'
```

![](./img/69.png)

#### 3.调用合约里的ReadWine方法根据酒号查询某一款酒

```sh
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C winechannel -n mycontract --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"ReadWine","Args":["11111"]}'
```

![](./img/70.png)

#### 4.调用合约里的TransferWine方法，将44444这款酒的所有人换成“小明”

```sh
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C winechannel -n mycontract --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferWine","Args":["44444","小明"]}'
```

![](./img/71.png)

#### 5.调用合约里的CreateWine方法，创建一款酒

```sh
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C winechannel -n mycontract --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"CreateWine","Args":["55555","汾酒","￥58","小明","1996","800ML"]}'
```

![](./img/72.png)

