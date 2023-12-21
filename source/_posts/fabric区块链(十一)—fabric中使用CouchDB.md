---
title: fabric区块链(十一)—fabric中使用CouchDB
date: 2023/7/10 11:12
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/29.jpg
---

# fabric中使用CouchDB

### 一，修改fabric-samples/config/core.yaml

将stateDatabase 从 goleveldb改成CouchDB

并配置账号密码



### 二，创建索引到合约的META-INF/statedb/couchdb/indexes文件夹下

```json
{
  "index":{
      "fields":["airline"] // Names of the fields to be queried
  },
  "ddoc":"index1Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index1",
  "type":"json"
}

{
  "index":{
      "fields":["pnr"] // Names of the fields to be queried
  },
  "ddoc":"index2Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index2",
  "type":"json"
}

{
  "index":{
      "fields":["flight"] // Names of the fields to be queried
  },
  "ddoc":"index3Doc", // (optional) Name of the design document in which the index will be created.
  "name":"index3",
  "type":"json"
}
```



### 三，编辑使用select语句的智能合约

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type MyContract struct {
	contractapi.Contract
}

type Data struct {
	ID   string `json:"id"`
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func (mc *MyContract) QueryDataByAge(ctx contractapi.TransactionContextInterface, age int) ([]*Data, error) {
	queryString := fmt.Sprintf(`{
		"selector": {
			"age": %d
		}
	}`, age)

	queryResults, err := ctx.GetStub().GetQueryResult(queryString)
	if err != nil {
		return nil, fmt.Errorf("failed to execute the query: %w", err)
	}
	defer queryResults.Close()

	var results []*Data
	for queryResults.HasNext() {
		queryResult, err := queryResults.Next()
		if err != nil {
			return nil, fmt.Errorf("failed to get the next query result: %w", err)
		}

		var data Data
		err = json.Unmarshal(queryResult.Value, &data)
		if err != nil {
			return nil, fmt.Errorf("failed to unmarshal data: %w", err)
		}

		results = append(results, &data)
	}

	return results, nil
}

// ... 其他合约方法

func main() {
	chaincode, err := contractapi.NewChaincode(&MyContract{})
	if err != nil {
		fmt.Printf("Error creating MyContract chaincode: %s", err.Error())
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting MyContract chaincode: %s", err.Error())
	}
}

```



### 四，在fabric-samples/test-network/目录下重启网络

```
./network.sh up createChannel -s couchdb
```



### 五，在通道上发布智能合约

```
./network.sh deployCC -c mychannel -ccn winecontract -ccp /home/myfabric/winecontract -ccl go
```

