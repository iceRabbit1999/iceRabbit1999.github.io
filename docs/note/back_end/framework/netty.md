***netty***

[all code here](https://github.com/iceRabbit1999/netty-tango)

# 一 入门

## 1.1 概述

### 1.1.1 简介

1. Netty is *an asynchronous event-driven network application framework*for rapid development of maintainable high performance protocol servers & clients.Netty是一个异步事件驱动的网络应用框架。用于快速开发可维护的高性能协议服务器和客户端
2. Netty is a NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. It greatly simplifies and streamlines network programming such as TCP and UDP socket server.
3. 'Quick and easy' doesn't mean that a resulting application will suffer from a maintainability or a performance issue. Netty has been designed carefully with the experiences earned from the implementation of a lot of protocols such as FTP, SMTP, HTTP, and various binary and text-based legacy protocols. As a result, Netty has succeeded to find a way to achieve ease of development, performance, stability, and flexibility without a compromise.

### 1.1.2谁在使用netty

Dubbo、zk、RocketMQ、ElasticSearch、Spring5(对HTTP协议的实现)、GRpc、Spark等大型开源项目都在使用Netty作为底层通讯框架

### 1.1.3 netty核心概念

1. Channel
   1. 对Socket的封装，包含了一组API，大大简化了直接与Socket进行操作的复杂性
2. EventLoopGroup
   1. EventLoop池
   2. Netty为每个Channel分配了一个EventLoop，用于处理用户连接请求、对用户请求的处理等所有事件
   3. EventLoop本身只是一个线程驱动，在其生命周期内只会绑定一个线程，让该线程处理一个Channel的所有IO事件
   4. Channel与EventLoop的关系是n:1，而EventLoop与线程的关系是1:1
3. ServerBootStrap
   1. 用于配置整个Netty代码，将各个组件关联起来
   2. 服务端使用的是ServerBootStrap，而客户端使用的是则BootStrap
4. ChannelHandler、ChannelPipeline
   1. ChannelHandler是对Channel中数据的处理器，这些处理器可以是系统本身定义好的编解码器，也可以是用户自定义的
   2. 这些处理器会被统一添加到一个ChannelPipeline的对象中，然后按照添加的顺序对Channel中的数据进行依次处理
5. ChannelFuture
   1. Netty中定义了一个ChannelFuture对象作为这个异步操作的“代言人”，表示异步操作本身
   2. 如果想获取到该异步操作的返回值，可以通过该异步操作对象的addListener()方法为该异步操作添加监听器，为其注册回调：当结果出来后马上调用执行。
   3. Netty的异步编程模型都是建立在Future与回调概念之上的

### 1.1.4 netty执行流程

![image-20220113112538360](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-执行流程.png)



## 1.2 example

通过Netty实现HTTP请求的处理，即接收HTTP请求，返回HTTP响应



## 1.3 两个处理器的区别

1. SimpleChannelInboundHandler中的channelRead()方法**会**自动释放接收到的来自于对方的msg所占有的所有资源。
2. ChannelInboundHandlerAdapter中的channelRead()方法**不会**自动释放接收到的来自于对方的msg
3. 区别早就不同的运用场景
   1. 若对方没有向自己发送数据，则自定义处理器建议继承自ChannelInboundHandlerAdapter
      1. 若继承自SimpleChannelInboundHandler需要重写channelRead0()方法。而重写该方法的目的是对来自于对方的数据进行处理。因为对方根本就没有发送数据，所以也就没有必要重写channelRead0()方法
   2. 若对方向自己发送了数据，而自己又需要将该数据再发送给对方，则自定义处理器建议继承自ChannelInboundHandlerAdapter
      1. write()方法的执行是异步的，且SimpleChannelInboundHandler中的channelRead()方法会自动释放掉来自于对方的msg
      2. 若write()方法中正在处理msg，而此时SimpleChannelInboundHandler中的channelRead()方法执行完毕了，将msg给释放了。此时就会报错

# 二 TCP的粘包与拆包

## 2.1 拆包粘包简介

1. 拆包与粘包同时发生在数据的发送方与接收方两方
2. Frame：发送方通过网络每发送一批二进制数据包，那么这次所发送的数据包就称为一帧
3. in TCP:，TCP协议会将用户真正要发送的数据根据当前缓存的实际情况对其进行拆分或重组，变为用于网络传输的Frame
4. in netty:
   1. 将ByteBuf中的数据  拆分或重组为二进制的Fram
   2. 接收方则需要将接收到的Frame中的数据进行重组或拆分，重新恢复为发送方发送时的ByteBuf数据
5. 场景描述
   1. 发送方发送的ByteBuf较大，在传输之前会被TCP底层拆分为多个Frame进行发送，这个过程称为发送拆包；接收方在接收到需要将这些Frame进行合并，这个合并的过程称为接收方粘包
   2. 发送方发送的ByteBuf较小，无法形成一个Frame，此时TCP底层会将很多的这样的小的ByteBuf合并为一个Frame进行传输，这个合并的过程称为发送方的粘包；接收方在接收到这个Frame后需要进行拆包，拆分出多个原来的小的ByteBuf，这个拆分的过程称为接收方拆包
   3. 当一个Frame无法放入整数倍个ByteBuf时，最后一个ByteBuf会会发生拆包
      1. 这个ByteBuf中的一部分入入到了一个Frame中，另一部分被放入到了另一个Frame中。这个过程就是发送方拆包；但对于将这些ByteBuf放入到一个Frame的过程，就是发送方粘包
      2. 当接收方在接收到两个Frame后，对于第一个Frame的最后部分，与第二个Frame的最前部分会进行合并，这个合并的过程就是接收方粘包。但在将Frame中的各个ByteBuf拆分出来的过程，就是接收方拆包

## 2.2 发送方拆包

### 2.2.1 需求

1. 客户端
   1. 客户端作为发送方，向服务端发送两个大的ByteBuf数据包，这两个数据包会被拆分为若干个Frame进行发送。这个过程中会发生拆包与粘包
2. 服务端
   1. 服务端作为接收方，直接将接收到的Frame解码为String后进行显示，不对这些Frame进行粘包与拆包

### 2.2.2 example

## 2.3 发送方粘包

### 2.3.1 需求

1. 客户端
   1. 客户端作为发送方，向服务端发送100个小的ByteBuf数据包，这100个数据包会被合并为若干个Frame进行发送。这个过程中会发生粘包与拆包
2. 服务端
   1. 服务端作为接收方，直接将接收到的Frame解码为String后进行显示，不对这些Frame进行粘包与拆包

### 2.3.2 example

## 2.4 接收方的粘包拆包

1. 接收方的粘包拆包实际在做的工作是解码工作
   1. 解码基本思想：：发送方在发送数据中添加一个分隔标记，并告诉接收方该标记是什么

## 2.5 LineBasedFrameDecoder

如其名，按照行分隔符对数据进行拆包粘包，解码出ByteBuf

### 2.5.1 example

## 2.6 DelimiterBasedFrameDecoder

按照指定分隔符对数据进行拆包粘包，解码出ByteBuf

### 2.6.1 example

## 2.7 FixedLengthFrameDecoder

按照指定的长度对Frame中的数据进行拆粘包

## 2.8 LengthFieldBasedFrameDecoder

基于长度域的帧解码器，用于对LengthFieldPrepender编码器编码后的数据进行解码的

### 2.8.1 构造器参数

1. maxFrameLength：要解码的Frame的最大长度
2. lengthFieldOffset：长度域的偏移量
3. lengthFieldLength：长度域的长度
4. lengthAdjustment：要添加到长度域值中的补偿值，长度矫正值。
5. initialBytesToStrip：从解码帧中要剥去的前面字节

### 2.8.2 example

#3 三 netty高级应用

## 3.1 WebSocket长连接

### 3.1.1 WebSocket简介

WebSocket是HTML5中的协议，是构建在HTTP协议之上的一个网络通信协议，其以长连接的方式实现了客户端与服务端的全双工通信

### 3.1.2 需求

在页面上有两个左右并排的文本域，它们的中间有一个“发送”按钮。在左侧文本域中输入文本内容后，单击发送按钮，会显示到右侧文本域中。

### 3.1.3 example

### 3.1.6 WebSocket握手原理

![image-20220114142151245](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/websocket握手原理.png)

## 3.2 网络聊天webchat example

### 3.2.1 需求

实现一个网络群聊工具。参与聊天的客户端消息是通过服务端进行广播的

### 3.2.2 example

## 3.3 读写空闲检测

IdleStateHandler

## 3.4 心跳机制

1. 心跳：即在TCP长连接中, 客户端和服务器之间定期发送的一种特殊的数据包, 通知对方自己还“活着”,以确保TCP连接的有效性。

### 3.4.1 需求

Client端连接到Server端后，会循环执行一个定时任务： 随机等待几秒，然后ping一下Server端，即发送一个心跳。当Server端在等待了指定时间后没有读取到Client端发送的心跳，Server端会 主动断开连

### 3.4.5 客户端重连服务端

## 3.5 手写Tomcat

