---
title: fabric区块链(二)—调用示例合约
date: 2023/5/11
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/34.jpg

---



## 1.启动Hyperledger Fabric网络


使用以下命令进入解压后的Hyperledger Fabric目录：

```bash
cd fabric-samples/test-network
```



然后使用以下命令启动网络：

```bash
./network.sh up
```

这个命令将启动一个包含两个组织和四个Peer节点的测试网络。如果一切顺利，可以使用以下命令检查网络是否启动成功：

![](./img/54.png)

```bash
./network.sh status
```



如果所有组织和Peer节点都处于运行状态，就说明网络启动成功了。



此时，网络创建成功了，但是还没有创建channel

```bash
./network.sh createChannel -c mychannel
```

![](./img/55.png)

到这里channel也创建好了

## 2.部署和测试示例智能合约

如果通道创建成功，可以使用joinChannel.sh脚本将peer节点加入到该通道中。testnetwork的目录下，运行以下命令将所有peer节点加入到mychannel通道中。

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
```

deployCC：deployCC 是 network.sh 脚本中的一个子命令，用于部署链码。
-ccn basic：-ccn 参数指定链码的名称，这里的 basic 是链码的名称。
-ccp ../asset-transfer-basic/chaincode-javascript/：-ccp 参数指定链码的路径，这里指定的路径是 ../asset-transfer-basic/chaincode-javascript/，它是链码的位置。
-ccl javascript：-ccl 参数指定链码的语言，这里指定的语言是 javascript，表示链码是用 JavaScript 编写的。


这个命令还将部署‘assert-transfer-basic'链码，部署成功就可以在通道上执行交易和查询了。

我再执行这个命令的时候报错jq command not found...，这是一个用于处理JSON数据的命令行工具，用下面的命令安装

```bash
sudo apt-get update
sudo apt-get install jq
```

![](./img/56.png)



#### 先把peer所在路径加到环境变量中



```bash
vim ~/.bashrc

export PATH=$PATH:$GOROOT/bin:$GOPATH/bin:/home/githubworkspace/fabric/scripts/fabric-samples/bin
```



#### 再把fabric config文件加到环境变量中



```bash
export FABRIC_CFG_PATH=/home/githubworkspace/fabric/scripts/fabric-samples/config
```

在/home/githubworkspace/fabric/scripts/fabric-samples/asset-transfer-basic/chaincode-go目录下执行

```sh
GO111MODULE=on go mod vendor
```

会在当前目录下生成vendor文件夹

#### 然后给Org1权限

```sh
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```



#### 接着调用示例合约

```sh
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
```


peer chaincode invoke: 这是用于调用链码的命令。

-o localhost:7050: 指定排序节点（orderer）的地址和端口号。在本例中，排序节点位于本地主机（localhost）的7050端口。

--ordererTLSHostnameOverride orderer.example.com: 该参数用于指定用于TLS连接的排序节点的主机名。在本例中，排序节点的主机名为orderer.example.com。

--tls: 指示使用TLS进行安全连接。

--cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem: 指定TLS连接时使用的根证书的路径。该证书用于验证排序节点的身份。

-C mychannel: 指定要在哪个通道上调用链码。在本例中，通道名称为mychannel。

-n basic: 指定要调用的链码的名称。在本例中，链码名称为basic。

--peerAddresses localhost:7051: 指定对等节点（peer）的地址和端口号。在本例中，第一个对等节点位于本地主机（localhost）的7051端口。

--tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt: 指定用于TLS连接的根证书的路径。该证书用于验证第一个对等节点的身份。

--peerAddresses localhost:9051: 指定第二个对等节点的地址和端口号。在本例中，第二个对等节点位于本地主机（localhost）的9051端口。

--tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt: 指定用于TLS连接的第二个对等节点的根证书的路径。该证书用于验证第二个对等节点的身份。

-c '{"function":"InitLedger","Args":[]}': 指定要调用的链码函数和参数。在本例中，调用的函数为InitLedger，不带任何参数。这里使用 JSON 格式来传递函数和参数。
![](./img/57.png)

#### 最后查询

```
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

![](./img/58.png)

