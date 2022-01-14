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

[get code here](https://github.com/iceRabbit1999/netty-tango)



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

