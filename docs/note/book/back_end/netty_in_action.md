# *Netty in Action*

# Chapter1 Netty concepts and architecture

## 1.1 Networking in Java

1. BIO(Blocking I/O) example

```java
ServerSocket serverSocket = new ServerSocket(portNumber);
Socket clientSocket = serverSocket.accept();
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream)));
PrientWriter out = new PrientWriter(clientSocket.getOutputStream(),true);
String request, response;
while(request = in.readLine() != null){
    if("Done".equals(request)){
        break;
    }
    response = processRequest(request);
    out.println(reponse);
}
```

![image-20220130181110994](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-bio.png)

3. implications of such an approach.
   1. many threads could be dormant
   2.  each thread requires an allocation of stack memory 
   3. the overhead of context-switching will begin to be troublesome long

### 1.1.1 Java NIO

### 1.1.2 Selectors

![image-20220130182539454](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-nio.png)

1. java.nio.channels.Seletors is the linchpin of Java NIO implementation,it uses the event notification API to indicate which, among a set ofnon-blocking sockets, are ready for I/O
2. compared to BIO
   1. Many  connections  can  be  handled  with  fewer  threads,  and thus with far lessoverhead due to memory management and context-switchin
   2. Threads can be retargeted to other tasks when there is no I/O to handle

## 1.2 Intrudcing Netty

1. The direct use of low-level APIs exposes complexity  andintroduces  a  critical  dependency  on  skills  that  tend  to  be  in  short  supply. Hence,  a fundamental concept of object orientation: hide the complexity of underlying imple-mentations behind simpler abstractions

2.  Netty leaves you freeto focus on what really interests you—the unique value of your application

3. Netty feature summary

   ![image-20220201153347898](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-feature.png)

### 1.2.1 Who uses Netty?

### 1.2.2 Asynchronous and  event-driven

1. 简单解释所谓的`asynchronous/event-driven`: it can respond to events occurring at any time and in any order
2. the connection between asynchrony and scalability
   1. Non-blocking network calls free us from having to wait for the completion of anoperation. Fully asynchronous I/O  builds on this feature and carries it a stepfurther: an  asynchronous method returns immediately and notifies  the  userwhen it is complete, directly or at a later time
   2. Selectors allow us to monitor many connections for  events  with  many  fewer threads.

## 1.3 Netty's core components

### 1.3.1 Channels

1. a basic construct of Java NIO.It represents **an  open  connection  to  an  entity  such  as  a  hardware  device,  a  file,  anetwork  socket,  or a program  component  that  is  capable  of  performingone or more distinct I/O operations, for example reading or writing.**
2. For now, think of a Channel as a vehicle for incoming (inbound) and outgoing (out-bound) data.

### 1.3.2 Callbacks

1. A callback  is  simply  a  method,  a reference  to  which  has  been  provided  to  another method.  

2. this  enables  the  latter  to  call  the  former  at  an  appropriate  time

3. when a callback is traggered, the event can be handled by an implementation of interface *ChannelHandler*

4. ChannelHandler trlggered by a callback

   ```java
   public class ConnectHandler extends ChannelInboundHandlerAdpater{
       @Override
       public void channelActive(ChannelHandlerContext ctx) throws Exception{
           log.info("Client" + ctx.channel().remoteAddress() + "connected");
       }
   }
   ```

   when  a  new  connection  has  been  established  the ChannelHandler callback channelActive() will be called and will print a message.

### 1.3.3 Futures

1. A Future provides another way to notify an application when an operation has completed

2. acts as a placeholder for the result of an asynchronous operation

3. will complete at some point in the future and provide access to the result

4. Netty提供了自己得Future——*ChannelFuture*,相较于JDK原生的Future，*ChannelFuture* + *ChannelFutureListener*的回调机制消除了需要手动checking操作是否结束的弊端

5. Each of Netty's outbound I/O operations returns a *ChannelFuture*, and none of them block

6. Asynchronous connect

   ```java
   Channel channel = ...;
   ChannelFuture future = channel.connect(new InetSocketAddress("192.168.0.1",25));
   ```

   `connect()` will return directly without blocking and the call will complete in the background,**when this will happen is abstracted away from the code**

7. Callback In action

   ```java
   Channel channel = ...'
   ChannelFuture future = channel.connect(new InetSocketAddress("192.168.0.1",25));
   future.addListener(new ChannelFutureListener({
       @Override
       public void operationComplete(ChannelFuture future){
           if(future.isSuccess()){
               ByteBuf buffer = Unpooled.copiedBuffer("Hello",Charset.defaultCharset());
               ChannelFuture wf = future.channel().writeAndFlush(buffer);
               ...
           }else{
               Throable cause = future.cause();
               cause.printStackTrace();
           }
       }
   }));
   ```

   1. connect to a remote peer
   2. register a new ChannelFutureListener
   3. check the status when connection is eatablished(if-else)and do things

8. ***a ChannelFutureListener is a more elaborate version of a callback; callbacks and Futures are complementary mechanisms***

### 1.3.4 Events and handlers

![image-20220207115246926](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-event-handler-chain.png)

1. for now can think of each handler instance as a kind of callback to be executed in response to a specific event
2. Internally,ChannelHandlers use events and futures themselves,making them consumers of the same abstractions your applications will employ

### 1.3.5 Putting it all together

1. Futures,Callbacks,Handlers
   1. Netty's asynchronous programming model is built on the concepts of Futures and callbacks, ***with the dispatching of events to handler methods happening at a deeper level***
   2. these elements allows the logic of your application to ***evolve independently of any concerns with network operations***
   3. intercepting operations and transforming inbound or outbound data requires only that you provide callbacks or utilize the *Futures* thatare returned by operations 
2. Selectors,Events,Event Loops
   1. Netty abstracts the *Selector* away from the application by firing events, and so,  we can eliminate all the handwritten dispatch code
   2. **an *EventLoop* is assigned to each *Channel*  to handle all of the events**
      1. *EventLoop* itself is driven by **only one thread** that handles all of the I/O events for one *Channel* and does not change during the lifetime of the *EventLoop*

## 1.4 Summary

# Chapter2 Your first Netty application

## 2.1 Setting up the development environment

Maven,JDK,IDE is enough

### 2.1.1 Obtaining and installing the Java Development Kit

### 2.1.2 Downloading and installing an IDE

### 2.1.3 Downloading and Installing Apache Maven

### 2.1.4 Configuring the toolset

## 2.2 Netty client/server overview

![image-20220207144840526](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-echo-client-and-server.png)

## 2.3 Writing the Echo server

All Netty servers require the following:

1. At least one *ChannelHandler* 
   1. implements the server's processing of data received from the client--its business logic
2. *Bootstrapping*
   1. the startup code that configure the sever

### 2.3.1 ChannelHandlers and business logic

```java
@Slf4j
@ChannelHandler.Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    /**
     * Called for each incoming message
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf in = (ByteBuf) msg;
        log.info("Server received: " + in.toString(CharsetUtil.UTF_8));
        ctx.write(in);
    }
 	/**
     * Notifies the handler that the last call made to channelRead() was the last message in the current batch
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)
                .addListener(ChannelFutureListener.CLOSE);
    } 
     /**
     * Called if an exception is thrown during the read operation
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

some key points about *ChannelHandler*

1. *ChannelHandlers* are invoked for different types of events
2. Applications implement or extend *ChannelHandlers* to hook into the event lifecycle and provide custom application logic
3. *ChannelHandlers* help to **keep your business logic decoupled from networking code**

### 2.3.2 Bootstrapping the server

```java
public class EchoServer {
    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            log.info("Usage: " + EchoServer.class.getSimpleName() + "<port>");
        }
        int port = Integer.parseInt(args[0]);
        new EchoServer(port).start();
    }

    private void start() throws Exception {
        //1. create the EventLoopGroup
        final EchoServerHandler serverHandler = new EchoServerHandler();
        EventLoopGroup group = new NioEventLoopGroup();

        //2. create the ServerBootstrap
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap.group(group)
                //3. Specifies the use of an NIO transport Channel
                .channel(NioServerSocketChannel.class)
        //4. Sets the socket address using the specified port
                .localAddress(new InetSocketAddress(port))
        //5. Adds an EchoServerHandler to the Channel's ChannelPipeline
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        socketChannel.pipeline().addLast(serverHandler);
                    }
                });
        //6. Binds the server asynchronously;sync() waits for the bind to complete
        ChannelFuture future = serverBootstrap.bind().sync();
        //7. Gets the CloseFuture of the Channel and blocks the current thread until it's complete
        future.channel().closeFuture().sync();
        //8. Shuts down the EventLoopGroup,releasing all resources
        group.shutdownGracefully().sync();
    }
}
```

##2.4 Writing an Echo client

### 2.4.1 Implementing the client logic with ChannelHandlers

```java
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    /**
     * Called when a message is received from the server
     * @param channelHandlerContext
     * @param byteBuf
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
      log.info("Client received: " + byteBuf.toString(CharsetUtil.UTF_8));
    }

    /**
     * Called after the connection to the server is established
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!",CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```



### 2.4.2 Bootstrapping the client

```java
public class EchoClient {
    private final String host;
    private final int port;

    public EchoClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start() throws Exception{
        //Create Bootstrap
        EventLoopGroup group = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(group)
                .channel(NioSocketChannel.class)
                .remoteAddress(new InetSocketAddress(host,port))
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        socketChannel.pipeline().addLast(new EchoClientHandler());
                    }
                });

        ChannelFuture future = bootstrap.connect().sync();
        future.channel().closeFuture().sync();

        group.shutdownGracefully().sync();
    }

    public static void main(String[] args) throws Exception {
        if(args.length != 2){
            log.info("Usage: " + EchoClient.class.getSimpleName() + "<host><port>");
            return;
        }

        String host = args[0];
        int port = Integer.parseInt(args[1]);
        new EchoClient(host,port).start();
    }
}
```

## 2.5 Building and running the Echo server and client

### 2.5.1 Running the build

### 2.5.2 Running the Echo server and  client

## 2.6 Summary

## Chapter3 Netty components and design

