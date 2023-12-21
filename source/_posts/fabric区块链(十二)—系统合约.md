---
title: fabric区块链(十二)—fabric系统合约
date: 2023/7/27 11:12
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/42.jpg
---

# fabric中使用系统合约通过区块号查询，以及查询区块高度



### 一，快速了解系统合约（干货）



#### 1.fabric自1.0版本开始，将链码分为系统链码和普通链码两种。普通链码（智能合约）用于实现业务逻辑，而系统链码则是用于系统管理，例如lscc，qscc等。



#### 2.系统链码在peer服务启动时随peer节点注册，同peer节点一起运行



#### 3.1.0版本时，有5个系统链码：

#### lscc：链码声明周期管理

#### qscc：区块/交易查询

#### cscc：通道配置管理

#### vscc：交易背书

#### escc：交易验证。



### 二，用法

#### 1.cscc

##### （1）JoinChain方法：使一个peer加入通道

```
$ peer channel join -b ch1.block
```

##### （2）GetConfigBlock方法：用于获取给定通道的当前的配置区块

```
$ peer chaincode query -C "mychannel" -n cscc -c '{"Args":["GetConfigBlock", "mychannel"]}'
```

```
peer channel fetch -o orderer0:7050 config -c mychannel
```

##### （3）GetChannels方法：用于获取peer目前所加入的通道

```
$ peer chaincode query -C "" -n cscc -c '{"Args":["GetChannels"]}'
```

```
$ peer channel list
```

##### （4）GetConfigTree：用于获取config结构

##### （5）SimulateConfigTreeUpdate：模拟执行config结构更新

​		如果要从一个通道添加或移除组织，必须获取config tree来进行修改，并在调用SimulateConfigTree方法时，必须获取CSCC的背书



#### 2.QSCC

##### （1）GetChainInfo

##### （2）GetBlockByNumber：通过区块号查询

```
$ peer chaincode query -C mychannel -n qscc -c '{"Args":["GetBlockByNumber", "mychannel", "3"]}'
```

##### （3）GetBlockByHash

##### （4）GetTransactionByID

##### （5）GetBlockByTxID



#### 3.LSCC

##### （1）install：用于存储chaincode程序到peer的文件系统

```
$ peer chaincode install -n mycc -v 1.0 -p 
```

##### （2）deploy：在给定通道上实例化合约，前两个参数必须，其他可选

```
$ peer chaincode instantiate -o orderer0:7050 -C mychannel -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR ('Org1MSP', 'Org2MSP')"
```

##### （3）upgrade：用于升级合约

```
$ peer chaincode upgrade -o orderer0:7050 -C mychannel -n mycc -v 2.0 -c '{"Args":["reinit"]}' -P "OR ('Org1MSP', 'Org2MSP')"
```

##### （4）getid：用于获取合约的id

```
$ peer chaincode query -C mychannel -n lscc -c '{"Args":["getid","mychannel","mycc"]}'
```

##### （5）getdepspec：用于获取安装在peer上的合约的chaincode deployment spec

```
$ peer chaincode query -C mychannel -n lscc -c '{"Args":["getdepspec", "mychannel", "mychannel"]}'
```

##### （6）getccdata：用于获取合约的数据

```
$ peer chaincode query -C mychannel -n lscc -c '{"Args":["getccdata","mychannel","mycc"]}'
```

##### （7）getchaincodes：用于获取部署在通道上的合约的列表

```
$ peer chaincode query -C mychannel -n lscc -c '{"Args":["getchaincodes"]}'
```

##### （8）getinstalledchaincodes：用于获取在peer上安装的合约的列表

```
$ peer chaincode query -C "" -n lscc -c '{"Args":["getinstalledchaincodes"]}'
```



#### 4.ESCC

ESCC 被背书节点(core/endorser/endorser.go)调用。背书节点在执行交易之后，将它的前面放在transaction response message中。其中，transaction response message也包括交集执行的结果，如交易状态、合约事件和read/write set等。一个调用功能可以包含5-7个参数，即Header、ChaincodeProposalPayload、ChaincodeID、Response、simulation result、events、payload visibility。



#### 5.VSCC

VSCC 被记账节点(core/committer/txvalidator/validator.go)调用,来根据合约的背书策略验证每个交易的签名集合。



参考文章：https://blog.csdn.net/zengqiang1/article/details/103974326
