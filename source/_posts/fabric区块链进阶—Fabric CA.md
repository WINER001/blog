---
title: fabric进阶—Fabric CA
date: 2023/5/28 17:17
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/41.png
---

# Hyperledger Fabric CA 是 Hyperledger Fabric 的证书颁发机构 (CA)。



## 一、功能：

> - 身份注册，或作为用户注册表连接到 LDAP
> - 颁发注册证书 (ECerts)
> - 证书更新和撤销



## 二、组成

#### 1.Hyperledger Fabric CA服务器

#### 2.Hyperledger Fabric CA客户端



## 三、安装

#### 1.在Ubuntu上安装GO 1.10+，这个之前已经安装过了

#### 2.在Ubuntu上安装libtool依赖项

```sh
sudo apt install libtool libltd1-dev
```

#### 3.install源码

```sh
go get -u github.com/hyperledger/fabric-ca/cmd/...
```

#### 4.启动CA服务器

##### 1.本地启动

```sh
fabric-ca-server start -b admin:adminpw
```

-b选项为引导程序管理员提供注册 ID 和密码；如果未使用“ldap.enabled”设置启用 LDAP，则这是必需的。

在本地目录中创建一个名为fabric-ca-server-config.yaml的默认配置文件，可以自定义。

##### 2.通过Docker启动

创建一个docker-compose.yml

```yaml
fabric-ca-server:
  image: hyperledger/fabric-ca:amd64-1.4.7
  container_name: fabric-ca-server
  ports:
    - "7054:7054"
  environment:
    - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
  volumes:
    - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
  command: sh -c 'fabric-ca-server start -b admin:adminpw'
```



在与docker-compose.yml文件相同的目录中打开终端并执行以下命令：

```sh
docker-compose up -d
```

3.创建自己的Fabric CA镜像

```sh
cd $GOPATH/src/github.com/hyperledger/fabric-ca
make docker
cd docker/server
docker-compose up -d
```



### 5.配置

配置有三种方式

##### 1.CLI flags

```sh
fabric-ca-client enroll --tls.client.certfile cert3.pem
```



##### 2.Environment variables

```sh
export FABRIC_CA_CLIENT_TLS_CLIENT_CERTFILE=cert2.pem
```



##### 3.Configuration files

```yaml
tls:
  # Enable TLS (default: false)
  enabled: false

  # TLS for the client's listenting port (default: false)
  certfiles:
  client:
    certfile: cert.pem
    keyfile:
```



```yaml
tls:
  enabled: true
  certfiles:
    - root.pem
  client:
    certfile: certs/cert.pem
    keyfile: /abs/path/key.pem
```



to be continues...

参考：https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html
