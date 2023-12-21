---
title: fabric区块链(八)—向网络添加钱包和用户
date: 2023/5/29 11:12
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/34.jpg
---

# 向使用Fabric-CA新建用户

在test-network目录中新建fabric-ca-client和fabric-ca-server目录

并修改/etc/environment

```sh
export FABRIC_CA_CLIENT_HOME=/home/githubworkspace/fabric/scripts/fabric-samples/test-network/fabric-ca-client
export FABRIC_CA_HOME=/home/githubworkspace/fabric/scripts/fabric-samples/test-network/fabric-ca-server
```



### 1.初始化fabric-ca服务器

```sh
fabric-ca-server init -b wenyongsheng:88888888
```

出现下面内容说明启动成功

2023/05/29 08:52:11 [INFO] The revocation key was successfully stored. The public key is at: /home/githubworkspace/fabric/scripts/fabric-samples/test-network/fabric-ca-server/IssuerRevocationPublicKey, private key is at: /home/githubworkspace/fabric/scripts/fabric-samples/test-network/fabric-ca-server/msp/keystore/IssuerRevocationPrivateKey
2023/05/29 08:52:11 [INFO] Home directory for default CA: /home/githubworkspace/fabric/scripts/fabric-samples/test-network/fabric-ca-server
2023/05/29 08:52:11 [INFO] Initialization was successful

### 2.启动服务器

```sh
fabric-ca-server start -b wenyongsheng:88888888
```

出现下面内容说明启动成功

2023/05/29 08:54:38 [INFO] Operation Server Listening on 127.0.0.1:9443
2023/05/29 08:54:38 [INFO] Listening on http://0.0.0.0:7054



### 3.启动客户端

```sh
fabric-ca-client enroll -u http://wenyongsheng:88888888@localhost:7054
```

启动成功后会在fabric-ca-client生成如下的文件树

.
├── fabric-ca-client-config.yaml
└── msp
    ├── cacerts
    │   └── localhost-7054.pem
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── keystore
    │   ├── 14cb5a48a3a56beecf40cead64e98882690ee02f682c1c897a8e83a3b5db104e_sk
    │   ├── 2de26801551d1bce9797118c86e2ec7f98600cd51ec07d145810e4368495c5cb_sk
    │   ├── 56bd3fd39a18c058135958e0515b1750b2e78b77d2efd0a6560f11d3565713a5_sk
    │   ├── 649bf858813b39c728c42afa49ac4e369104f6b42f62f7c161ce514599c3ced9_sk
    │   ├── b5f3f29fc2021dac426b862963cea9afa12bdac2185b8af6bc7f9797e457a8c8_sk
    │   ├── eab9ccce5dd6947cbc2c7468ef5b1f46c98b63baf478ac292c961bfad1da4c29_sk
    │   └── f4e1c91becdfcb869f63c205da41d6dea2385686ec09817fd282d5b04865c937_sk
    ├── signcerts
    │   └── cert.pem
    └── user



### 4.注册一个身份

```sh
fabric-ca-client register \
    --id.name admin2     \
    --id.affiliation org1.department1 \
    --id.attrs 'hf.Revoker=true,admin=true:ecert'
```

生成如下日志：

2023/05/29 10:35:27 [INFO] Configuration file location: /home/githubworkspace/fabric/scripts/fabric-samples/test-network/fabric-ca-client/fabric-ca-client-config.yaml
Password: tsenIurZcILr



### 5.在server中查看表

进入/home/githubworkspace/fabric/scripts/fabric-samples/test-network/fabric-ca-server目录,查看db文件

```sh
sqlite3 fabric-ca-server.db
```

```sqlite
.tables
```

可以看到所有的表：

affiliations               properties
certificates               revocation_authority_info
credentials                users

然后查看users表：

```sqlite
select * from users;
```

可以看到下面的数据，多出来了xiaohong，说明创建成功了

wenyongsheng|$2a$10$hvquUet.tpRawlVicZJfPOxlWDwR.k0WWwakHO0GW9Ck/X/8LKjRm|client||[{"name":"hf.AffiliationMgr","value":"1"},{"name":"hf.Registrar.Roles","value":"*"},{"name":"hf.Registrar.DelegateRoles","value":"*"},{"name":"hf.Revoker","value":"1"},{"name":"hf.IntermediateCA","value":"1"},{"name":"hf.GenCRL","value":"1"},{"name":"hf.Registrar.Attributes","value":"*"}]|3|-1|2|0

xiaohong|$2a$10$kZIVWnNbu9bxOQThn4/nEeMwBFz1ZRWsLgvtAYeW4URe8VceuDsYi|client|org1.department1|[{"name":"hf.Revoker","value":"true"},{"name":"admin","value":"true","ecert":true},{"name":"hf.EnrollmentID","value":"xiaohong","ecert":true},{"name":"hf.Type","value":"client","ecert":true},{"name":"hf.Affiliation","value":"org1.department1","ecert":true}]|0|-1|2|0





```
./fabric-ca-client register -d --id.name org1admin --id.secret org1adminpw -u https://example.com:7054 --mspdir ./org1-ca/msp --id.type admin --tls.certfiles ../tls/tls-ca-cert.pem
```



### 6.然后在创建一个peer类型的，并指定密码

```sh
fabric-ca-client register \
    --id.name xiaowang \
    --id.type peer \
    --id.affiliation org1.department1 \
    --id.secret 88888888
```



我的博客即将同步至腾讯云开发者社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=mqkr4pbb8inz
