---
title: fabric区块链(一)—搭建环境
date: 2023/5/10
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/13.jpg

---



## hyperledger Fabric是一款适用于企业级应用的区块链平台。在Ubuntu上搭建Hyperledger Fabric 2.4.9需要进行以下步骤：

1. ## 安装Docker和Docker Compose


在Ubuntu上安装Docker和Docker Compose可以使用以下命令：

```bash
# 安装Docker
sudo apt-get update
sudo apt-get install docker.io

# 安装Docker Compose
sudo apt-get install docker-compose
```



安装完成后，可以使用以下命令检查是否安装成功：

```bash
# 查看Docker版本
docker --version

# 查看Docker Compose版本
docker-compose --version
```

![](./img/51.png)



2. ## 安装Go语言


Hyperledger Fabric使用Go语言编写，因此需要安装Go语言环境。可以使用以下命令安装：

```bash
sudo apt-get install golang-go
```



安装完成后，可以使用以下命令检查是否安装成功：

```bash
go version
```

![](./img/52.png)

3. ## 安装Node.js和npm


Hyperledger Fabric的客户端应用使用Node.js开发，因此需要安装Node.js和npm。可以使用以下命令安装：

```bash
#安装Node.js和npm
sudo apt-get install nodejs
sudo apt-get install npm
```



安装完成后，可以使用以下命令检查是否安装成功：

```bash
# 查看Node.js版本
node -v


# 查看npm版本
npm -v
```

![](./img/53.pngls)

4. ## 下载Hyperledger Fabric


可以从Hyperledger Fabric的官方网站下载Hyperledger Fabric 2.4.9的二进制文件。下载地址为：https://hyperledger-fabric.readthedocs.io/en/release-2.4/install.html。

下载完成后，可以解压到任意目录。

5. ## 启动Hyperledger Fabric网络


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

6. ## 部署和测试链码

如果通道创建成功，可以使用joinChannel.sh脚本将peer节点加入到该通道中。testnetwork的目录下，运行以下命令将所有peer节点加入到mychannel通道中。

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
```

这个命令还将部署‘assert-transfer-basic'链码，部署成功就可以在通道上执行交易和查询了。

我再执行这个命令的时候报错jq command not found...，这是一个用于处理JSON数据的命令行工具，用下面的命令安装

```bash
sudo apt-get update
sudo apt-get install jq
```

![](./img/56.png)



在Hyperledger Fabric中，链码是一个智能合约，用于在区块链上执行业务逻辑。可以使用以下命令在测试网络上部署和测试一个示例链码：

```bash

# 安装链码
./network.sh deployCC

# 测试链码
./scripts/testCC.sh
```

