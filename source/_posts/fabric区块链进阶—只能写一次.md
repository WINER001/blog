---
title: fabric进阶—第二次调用SDK失败EndorseException
date: 2023/6/8 17:17
tags: 
  - fabric区块链
  - 疑难杂症
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/41.png
---





## org.hyperledger.fabric.client.EndorseException: io.grpc.StatusRuntimeException: ABORTED: failed to endorse transaction, see attached details for more info



### 一，问题描述

我的调用链是这样的

http请求————》springboot api ————》fabric java sdk ————》mycontract智能合约createwine方法

今天遇到一个非常有意思的bug，我在用http请求调用springboot api（使用java sdk调用fabric区块链）的时候，

第一次可以调用成功，

```
["****** create wine successfully ******"]
```

但是第二次调用，就报错

```
2023-06-08 20:01:24 INFO  c.y.c.controller.ContractController - org.hyperledger.fabric.client.EndorseException: io.grpc.StatusRuntimeException: ABORTED: failed to endorse transaction, see attached details for more info
        at org.hyperledger.fabric.client.GatewayClient.endorse(GatewayClient.java:73)
        at org.hyperledger.fabric.client.ProposalImpl.endorse(ProposalImpl.java:74)
        at org.hyperledger.fabric.client.Proposal.endorse(Proposal.java:65)
        at org.hyperledger.fabric.client.ContractImpl.submitTransaction(ContractImpl.java:47)
        at com.yuchengji.cn.controller.ContractController.createWine(ContractController.java:216)
        at com.yuchengji.cn.controller.ContractController.queryWineByKey(ContractController.java:97)
        at jdk.internal.reflect.GeneratedMethodAccessor34.invoke(Unknown Source)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:566)
        at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205)
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:150)


Caused by: io.grpc.StatusRuntimeException: ABORTED: failed to endorse transaction, see attached details for more info
        at io.grpc.stub.ClientCalls.toStatusRuntimeException(ClientCalls.java:271)
        at io.grpc.stub.ClientCalls.getUnchecked(ClientCalls.java:252)
        at io.grpc.stub.ClientCalls.blockingUnaryCall(ClientCalls.java:165)
        at org.hyperledger.fabric.protos.gateway.GatewayGrpc$GatewayBlockingStub.endorse(GatewayGrpc.java:472)
        at org.hyperledger.fabric.client.GatewayClient.endorse(GatewayClient.java:71)

```

然后不管怎么用sdk调用create都会失败。



### 二，寻找线索

#### 1.peer命令调用CreateWine是可以成功的

![](./img/81.png)



#### 2.http请求进来的通过sdk调用 CreateWine 失败

```java
contract.submitTransaction("CreateWine", assetId,name,price,owner,birth,capacity);
```

![](./img/82.png)



#### 3.http请求通过sdk调用 GetAllWines 成功

```
byte[] result = contract.evaluateTransaction("GetAllWines");
```

![](./img/83.png)



#### 4.http请求通过sdk调用 ReadWine成功

```java
byte[] evaluateResult = contract.evaluateTransaction("ReadWine", "11111");
```

![](./img/84.png)



### 三，分析对比成功和失败日志

#### 1.http请求通过sdk调用 ReadWine成功

```java
//第一条日志记录了一个调用事务的评估操作。操作名称是ReadWine，id为11111。
--> Evaluate Transaction: ReadWine, id : 11111

//第二条日志显示了一个SSL握手过程的细节。它提供了使用的协议和密码套件的信息。在本例中，协议是TLSv1.3，密码套件是TLS_AES_128_GCM_SHA256。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.n.handler.ssl.SslHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] HANDSHAKEN: protocol:TLSv1.3 cipher suite:TLS_AES_128_GCM_SHA256

//第三条日志是关于设置的信息。它显示了一组设置参数的值。其中，ENABLE_PUSH设置为0，MAX_CONCURRENT_STREAMS设置为0，INITIAL_WINDOW_SIZE设置为1048576，MAX_HEADER_LIST_SIZE设置为8192。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] OUTBOUND SETTINGS: ack=false settings={ENABLE_PUSH=0, MAX_CONCURRENT_STREAMS=0, INITIAL_WINDOW_SIZE=1048576, MAX_HEADER_LIST_SIZE=8192}

//第四条日志是一个窗口更新操作，增加了窗口大小。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] OUTBOUND WINDOW_UPDATE: streamId=0 windowSizeIncrement=983041

//第五和第六条日志是有关设置的信息，确认了之前发送的设置参数。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] INBOUND SETTINGS: ack=false settings={MAX_FRAME_SIZE=16384}
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] OUTBOUND SETTINGS: ack=true   
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] INBOUND SETTINGS: ack=true
    
//第七条日志显示了一个HTTP/2的头部信息，它包含了一些重要的元数据，如请求的目标地址、路径、方法等。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] OUTBOUND HEADERS: streamId=3 headers=GrpcHttp2OutboundHeaders[:authority: peer0.org1.example.com, :path: /gateway.Gateway/Evaluate, :method: POST, :scheme: https, content-type: application/grpc, te: trailers, user-agent: grpc-java-netty/1.53.0, grpc-accept-encoding: gzip, grpc-timeout: 4989199u] streamDependency=0 weight=16 exclusive=false padding=0 endStream=false
    
//第八条日志是一个数据发送操作，其中包含了数据流的详细信息。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] OUTBOUND DATA: streamId=3 padding=0 endStream=true length=1175 bytes=00000004920a40613030326662313930616530336330626237333463393663356664393866633430306237316366643037393763663364366433653230323933...
    
//第九和第十条日志分别是窗口更新和Ping操作的信息。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] INBOUND WINDOW_UPDATE: streamId=0 windowSizeIncrement=1175
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] INBOUND PING: ack=false bytes=145258749040133895
    
//第十一条日志是一个接收到的HTTP/2头部信息，表示服务器对请求的响应。其中包含了响应状态码和内容类型等信息。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] OUTBOUND PING: ack=true bytes=145258749040133895
    
//第十二条日志是一个接收到的数据信息，表示服务器返回的数据内容。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] INBOUND HEADERS: streamId=3 headers=GrpcHttp2ResponseHeaders[:status: 200, content-type: application/grpc] padding=0 endStream=false
    
//
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] INBOUND DATA: streamId=3 padding=0 endStream=false length=115 bytes=000000006e0a6c08c8011a677b224944223a223131313131222c224e616d65223a22e4ba8ce99485e5a4b4222c2256616c7565223a22efbfa53230222c224f77...
    
//第十三和第十四条日志是Ping操作的信息。
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] OUTBOUND PING: ack=false bytes=1234
2023-06-08 20:32:55 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x5b7bcb08, L:/127.0.0.1:51138 - R:localhost/127.0.0.1:7051] INBOUND HEADERS: streamId=3 headers=GrpcHttp2ResponseHeaders[grpc-status: 0, grpc-message: ] padding=0 endStream=true
    
//最后一条日志是一个信息记录，显示了调用ContractController.java文件中的readWineById方法的位置，并输出了一段JSON格式的结果数据。
2023-06-08 20:32:55 INFO  c.y.c.controller.ContractController - ContractController.java(readWineById:271)
*** Result:{
  "ID": "11111",
  "Name": "二锅头",
  "Value": "￥20",
  "Owner": "路人甲",
  "Birth": "2000",
  "Capacity": "500ML"
}

```

##### 

#### 2.重启区块链后，第一次 http请求进来的通过sdk调用 CreateWine 成功

详细日志，把和之前成功调用一样的部分省略

```java
--> Submit Transaction: CreateWine, creates new wine with ID, Color, Size, Owner and AppraisedValue arguments
//建立连接，SSL握手过程，成功，省略
    
//注意，从这里开始不一样了，之前成功的调用是/gateway.Gateway/Evaluate  ，而这里是/gateway.Gateway/Endorse
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] OUTBOUND HEADERS: streamId=3 headers=GrpcHttp2OutboundHeaders[:authority: peer0.org1.example.com, :path: /gateway.Gateway/Endorse, :method: POST, :scheme: https, content-type: application/grpc, te: trailers, user-agent: grpc-java-netty/1.53.0, grpc-accept-encoding: gzip, grpc-timeout: 14990619u] streamDependency=0 weight=16 exclusive=false padding=0 endStream=false
//一样的省略掉

    
//这里客户端向服务端发了第二次POST请求/gateway.Gateway/Submit
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] OUTBOUND HEADERS: streamId=5 headers=GrpcHttp2OutboundHeaders[:authority: peer0.org1.example.com, :path: /gateway.Gateway/Submit, :method: POST, :scheme: https, content-type: application/grpc, te: trailers, user-agent: grpc-java-netty/1.53.0, grpc-accept-encoding: gzip, grpc-timeout: 4999822u] streamDependency=0 weight=16 exclusive=false padding=0 endStream=false
    
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] OUTBOUND DATA: streamId=5 padding=0 endStream=true length=4188 bytes=00000010570a40376634653630653739353962383130613831373930383538663935636661356264396238613538626233643034303131353535636261306238...
    
/****这一条是失败的调用没有的*******/  
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] INBOUND PING: ack=true bytes=1234
/***********/  
    
//这里一样的省略

//成功的调用这里显示status200
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] INBOUND HEADERS: streamId=5 headers=GrpcHttp2ResponseHeaders[:status: 200, content-type: application/grpc] padding=0 endStream=false
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] INBOUND DATA: streamId=5 padding=0 endStream=false length=5 bytes=0000000000
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] INBOUND HEADERS: streamId=5 headers=GrpcHttp2ResponseHeaders[grpc-status: 0, grpc-message: ] padding=0 endStream=true
    
//这里第三次POST请求/gateway.Gateway/CommitStatus
2023-06-08 21:40:09 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] OUTBOUND HEADERS: streamId=7 headers=GrpcHttp2OutboundHeaders[:authority: peer0.org1.example.com, :path: /gateway.Gateway/CommitStatus, :method: POST, :scheme: https, content-type: application/grpc, te: trailers, user-agent: grpc-java-netty/1.53.0, grpc-accept-encoding: gzip, grpc-timeout: 59999834u] streamDependency=0 weight=16 exclusive=false padding=0 endStream=false

2023-06-08 21:40:11 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0xac380a2d, L:/127.0.0.1:55556 - R:localhost/127.0.0.1:7051] INBOUND HEADERS: streamId=7 headers=GrpcHttp2ResponseHeaders[grpc-status: 0, grpc-message: ] padding=0 endStream=true
2023-06-08 21:40:11 INFO  c.y.c.controller.ContractController - ContractController.java(createWine:221)
****** create wine successfully ******

```



#### 3.失败日志 http请求进来的通过sdk调用 CreateWine 失败

详细日志，把和之前成功调用一样的部分省略

```java
--> Submit Transaction: CreateWine, creates new wine with ID, Color, Size, Owner and AppraisedValue arguments
//建立连接，SSL握手过程，成功，省略

//注意，从这里开始不一样了，之前成功的调用是/gateway.Gateway/Evaluate  ，而这里/gateway.Gateway/Endorse就失败了！
2023-06-08 20:01:24 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x431bd91f, L:/127.0.0.1:48088 - R:localhost/127.0.0.1:7051] OUTBOUND HEADERS: streamId=3 headers=GrpcHttp2OutboundHeaders[:authority: peer0.org1.example.com, :path: /gateway.Gateway/Endorse, :method: POST, :scheme: https, content-type: application/grpc, te: trailers, user-agent: grpc-java-netty/1.53.0, grpc-accept-encoding: gzip, grpc-timeout: 14988161u] streamDependency=0 weight=16 exclusive=false padding=0 endStream=false
//客户端给服务端请求详细内容，之后的连接信息和成功调用时也一样，省略
2023-06-08 20:01:24 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x431bd91f, L:/127.0.0.1:48088 - R:localhost/127.0.0.1:7051] OUTBOUND DATA: streamId=3 padding=0 endStream=true length=1225 bytes=00000004c40a40343831313232646561356566363064663437326630313062613366363161613837623062313231383930346234653061353738396464336138...

//这里是服务端给客户端的回复，failed to endorse transaction,失败了
2023-06-08 20:01:24 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x431bd91f, L:/127.0.0.1:48088 - R:localhost/127.0.0.1:7051] INBOUND HEADERS: streamId=3 headers=GrpcHttp2ResponseHeaders[:status: 200, content-type: application/grpc, grpc-status: 10, grpc-message: failed to endorse transaction, see attached details for more info, grpc-status-details-bin: CAoSQWZhaWxlZCB0byBlbmRvcnNlIHRyYW5zYWN0aW9uLCBzZWUgYXR0YWNoZWQgZGV0YWlscyBmb3IgbW9yZSBpbmZvGpYBCid0eXBlLmdvb2dsZWFwaXMuY29tL2dhdGV3YXkuRXJyb3JEZXRhaWwSawobcGVlcjAub3JnMS5leGFtcGxlLmNvbTo3MDUxEgdPcmcxTVNQGkNjaGFpbmNvZGUgcmVzcG9uc2UgNTAwLCB0aGlzIHdpbmUgYXNzZXQxNjg2MTA1NTIyMTYzIGFscmVhZHkgZXhpc3Rz] padding=0 endStream=true
2023-06-08 20:01:24 INFO  c.y.c.controller.ContractController - ContractController.java(queryWineByKey:111)
call fabric fail1...
2023-06-08 20:01:24 INFO  c.y.c.controller.ContractController - org.hyperledger.fabric.client.EndorseException: io.grpc.StatusRuntimeException: ABORTED: failed to endorse transaction, see attached details for more info
        at org.hyperledger.fabric.client.GatewayClient.endorse(GatewayClient.java:73)
        at org.hyperledger.fabric.client.ProposalImpl.endorse(ProposalImpl.java:74)
        at org.hyperledger.fabric.client.Proposal.endorse(Proposal.java:65)
        at org.hyperledger.fabric.client.ContractImpl.submitTransaction(ContractImpl.java:47)
        at com.yuchengji.cn.controller.ContractController.createWine(ContractController.java:216)

```

#### 

### 四，分析源码

#### 1.ProposalImpl.java

#### (at org.hyperledger.fabric.client.ContractImpl.submitTransaction(ContractImpl.java:47))

```java
    @Override
    public Transaction endorse(final UnaryOperator<CallOptions> options) throws EndorseException {
        sign();
        final EndorseRequest endorseRequest = EndorseRequest.newBuilder()
                .setTransactionId(proposedTransaction.getTransactionId())
                .setChannelId(channelName)
                .setProposedTransaction(proposedTransaction.getProposal())
                .addAllEndorsingOrganizations(proposedTransaction.getEndorsingOrganizationsList())
                .build();
        EndorseResponse endorseResponse = client.endorse(endorseRequest, options);

        PreparedTransaction preparedTransaction = PreparedTransaction.newBuilder()
                .setTransactionId(proposedTransaction.getTransactionId())
                .setEnvelope(endorseResponse.getPreparedTransaction())
                .build();
        return new TransactionImpl(client, signingIdentity, preparedTransaction);
    }
```

可以看到是调用client.endorse(endorseRequest,options);时报错，继续

#### 2.GatewayClient.java

```java
import org.hyperledger.fabric.protos.gateway.GatewayGrpc;

final class GatewayClient {
    private final GatewayGrpc.GatewayBlockingStub gatewayBlockingStub;
    private final DeliverGrpc.DeliverStub deliverAsyncStub;
    private final DefaultCallOptions defaultOptions;

    GatewayClient(final Channel channel, final DefaultCallOptions defaultOptions) {
        GatewayUtils.requireNonNullArgument(channel, "No connection details supplied");
        GatewayUtils.requireNonNullArgument(defaultOptions, "defaultOptions");

        this.gatewayBlockingStub = GatewayGrpc.newBlockingStub(channel);
        this.deliverAsyncStub = DeliverGrpc.newStub(channel);
        this.defaultOptions = defaultOptions;
    }
        
    public EndorseResponse endorse(final EndorseRequest request, final UnaryOperator<CallOptions> options) 
        throws EndorseException {
        GatewayGrpc.GatewayBlockingStub stub = defaultOptions.applyEndorse(gatewayBlockingStub, options);
        try {
            return stub.endorse(request);
        } catch (StatusRuntimeException e) {
            throw new EndorseException(request.getTransactionId(), e);
        }
    }
}
```



可以看到是在执行stub.endorse(request);这句报错了

那就得看看GatewayGrpc.GatewayBlockingStub.endorse()

而这个包org.hyperledger.fabric.protos.gateway.GatewayGrpc打开了以后是个gateway.proto文件

Proto是Protocol Buffers的简称，是一种用于定义数据结构和序列化数据的语言和平台无关的协议。Proto使用.proto文件作为定义数据结构的文件，可以根据.proto文件生成各种编程语言的代码，用于在不同的系统之间进行数据交换和通信。





2023-06-08 20:01:24 DEBUG i.g.n.s.i.g.n.NettyClientHandler - [id: 0x431bd91f, L:/127.0.0.1:48088 - R:localhost/127.0.0.1:7051] INBOUND HEADERS: streamId=3 headers=GrpcHttp2ResponseHeaders[:status: 200, content-type: application/grpc, grpc-status: 10, grpc-message: failed to endorse transaction, see attached details for more info, grpc-status-details-bin:



TODO
