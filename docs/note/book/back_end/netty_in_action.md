# *Netty in Action*

# Part 1 Netty concepts and architecture

# Chapter1 Netty—asynchronous and event driven

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

# Chapter3 Netty components and design

## 3.1 Channel, EventLoop, and ChannelFuture

Netty's networking abstraction:

1. **Channel——Sockets**
2. **EventLoop——Control flow, multithreading, concurrency**
3. **ChannelFuture——Asynchronous notification**

### 3.1.1 Interface Channel

1. *Channel* greatly reduces the complexity of working directly with Sockets
2. have many predefined, specialized implementations

### 3.1.2 Interface EventLoop

1. Netty's core abstraction for handling events that occur during the lifetime of a connection

2. the relationships among *Channel*, *EventLoop*, *Thread*, *EventLoopGroup*

   ![image-20220208095446950](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-in-action-3.1.png)

   1. **An *EventLoopGroup* contains one or more *EventLoops***
   2. **An *EventLoop* is bound to a single *Thread* for its lifetime**
   3. **All I/O events processed by an *EventLoop* are handled on its dedicated *Thread***
   4. **A *Channel* is registered for its lifetime with a single *EventLoop***
   5. **A single *EventLoop* may be assigned to one or more *Channels***
   6. EventLoopGroup n↔1 EventLoop
   7. EventLoop 1↔1 Thread
   8. EventLoop 1↔1 Channel
   9. eliminate synchronous
      1. 对每一个给定的Channel来说，所有的I/O操作都由一个相同的线程执行

### 3.1.3 Interface ChannelFuture

1. Netty is asynchronous, so we need a way to determine its result at a later time
2. `addListener()` registers a *ChannelFutureListener* to be notified when an operation has completed
3. **Think of a *ChannelFuture* as a placeholder for the result of an operation that's to be executed in the future**

## 3.2 ChannelHandler and ChannelPipeline

### 3.2.1 Interface ChannelHandler

1. Serve as the container for all application logic that applies to handling inbound and outbound data
   1. *ChannelHandler* methods are triggered by network events

### 3.2.2 Interface ChannelPipeline

1. Provides a container for a chain of *ChannelHandlers* and defines an API for propagating the flow of inbound and outbound events along the chain

2. When a *Channel* is created, it is automatically assigned its own *ChannelPipeline*

3. *ChannelHandlers* are installed in the *ChannelPipeline* as follows:

   1. A *ChannelInitializer* implementation is registered with a *ServerBootstrap*
   2. When `ChannelInitializer.initChannel()` is called, the *ChannelInitializer* installs a custom set of *ChannelHandlers* in the pipeline
   3. The *ChannelInitializer* removes itself from the *ChannelPipeline*

4. for *ChannelHandler*,

   1. **you can think of it as a generic container for any code that process events/data coming and going through the *ChannelPipeline***
   2. The movement of an event through the pipeline is the work of the *ChannelHandlers* that have been installed during the initialization, or bootstrapping phase of the application
   3. *ChannelHandler* 是chain形式的，他的执行顺序取决于其被添加时的顺序，我们把其执行的这种有序排列称为*ChannelPipeline *( It is this ordered arrangement of *ChannelHandlers* that we refer to as the *ChannelPipeline* )

5. *ChannelPipeline* with inbound and outbound *ChannelHandlers*

   ![image-20220208110935820](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-pipeline-and-hendler.png)

   1. Netty ensures that data is passed only between handlers of the same directional type(inbound and outbound *ChannelHandler*)

6. `ChannelHandlerContext`

   1. When a *ChannelHandler* is added to a *ChannelPipeline*, it's assigned a *ChannelHandlerContext*, **which represents the binding between a *ChannelHandler* and the *ChannelPipeline***
   2. An event can be forwarded to the next handler in the current chain by using the `ChannelHandlerContext` that's supplied as an argument to each method

7. Two ways if sending messages in Netty

   1. Write directly to the *Channel*
      1. causes the message to start from the tail of the *ChannelPipeline*
   2. Write to a *ChannelHandlerContext* object associated with a *ChannelHandler*
      1. causes the message to start from the next handler in the *ChannelPipeline*

### 3.2.3 A closer look at ChannelHandlers

1. Netty provides a number of default handler implementations in the form of adapter class. These adapter classes(and their subclasses) forwarding events to the next handler in the chain automatically. So we can override the methods and events we want to specialize
2. Why adapters?
   1. they provide default implementations of all the methods defined in the corresponding interface so that  reduce the effort of writing custom *ChannelHandlers* to bare minimum

### 3.2.4 Encoders and decoders

1. network data is always a series of bytes

### 3.2.5 Abstract class SimpleChannelInboundHandler

receives a decoded message and applies business logic to the data, extend this class and override one or more method

## 3.3 Bootstrapping

Netty's bootstrap classes provide containers for the configuration of an application's network layer

1. Comparison of Bootstrap classes

   ![image-20220208141934154](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-comparison-of-bootstrap.png)

   1. *ServerBootstrap* binds to a port, *Bootstrap* connect a remote peer
   2. BootStrapping a client requires only a single *EventLoopGroup* ; A *ServerBootstrap* requires two(which can be the same instance) 

2. Server with two *EventLoopGroups*

   ![image-20220208142637964](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-server-with-two-eventloopgroup.png)

   1. The first set will contain a single *ServerChannel*  representing the server's own listening socket, bound to a local port
   2. The second set will contain all of the *Channels* that have been created to handle incoming client connections——one for each connection the server has accepted
   3. The *EventLoopGroup* associated with the *ServerChannel* assigns an *EventLoop* that is responsible for creating *Channels* for incoming connection requests
   4. Once connection has been accepted, the second *EventLoopGroup* assigns an *EventLoop* to its *Channel*

## 3.4 Summary

# Chapter4 Transports

1. Network transport, a concept that helps us to **abstract away the underlying mechanics of data transfer**
2. Netty layers a common API over all its transport implementations

## 4.1 Case study: transport migration

### 4.1.1 Using OIO and NIO without Netty

OIO: blocking, NIO: asynchronous

1. Blocking networking without Netty

   ```java
   public class PlainOioServer {
       public void serve(int port) throws Exception{
           //Bind the server to specified port
           final ServerSocket socket = new ServerSocket(port);
           for(;;){
               //Accept a connection
               final Socket clientSocket = socket.accept();
               log.info("Accepted connection from " + clientSocket);
   
               //Create a new thread to handle the connection
               new Thread(new Runnable() {
                   @SneakyThrows
                   @Override
                   public void run() {
                       //Write message to the connected client
                       OutputStream out = clientSocket.getOutputStream();
                       out.write("Hi!\r\n".getBytes(StandardCharsets.UTF_8));
                       out.flush();
                       //Close the connection
                       clientSocket.close();
                   }
               }).start();
           }
       }
   }
   ```

2. Asynchronous networking without Netty

   ```java
   public class PlainNioServer {
       public void serve(int port) throws IOException {
           ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
           serverSocketChannel.configureBlocking(false);
           ServerSocket ssocket = serverSocketChannel.socket();
           InetSocketAddress address = new InetSocketAddress(port);
           //Bind the server to the selected port
           ssocket.bind(address);
           //Open the selector for handling channels
           Selector selector = Selector.open();
           //Registers the ServerSocket with the Selector to accept connections
           serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
           final ByteBuffer msg = ByteBuffer.wrap("Hi!\r\n".getBytes());
   
           for (; ; ) {
               //Wait for new events to process; blocks until the next incoming event
               try {
                   selector.select();
               } catch (IOException e) {
                   e.printStackTrace();
                   break;
               }
           }
           //Obtains all SelectionKey instances tha received events
           Set<SelectionKey> readyKeys = selector.selectedKeys();
           Iterator<SelectionKey> iterator = readyKeys.iterator();
           while (iterator.hasNext()) {
               SelectionKey key = iterator.next();
               iterator.remove();
               //Check if the event is a new connection ready to be accepted
               if (key.isAcceptable()) {
                   ServerSocketChannel server = (ServerSocketChannel) key.channel();
                   SocketChannel client = server.accept();
                   client.configureBlocking(false);
                   //Accepts client and registers it with the selector
                   client.register(selector, SelectionKey.OP_WRITE | SelectionKey.OP_READ, msg.duplicate());
                   log.info("Accepted connection from " + client);
               }
               //Check if the socket is ready for writing data
               if (key.isWritable()) {
                   SocketChannel client = (SocketChannel) key.channel();
                   ByteBuffer buffer = (ByteBuffer) key.attachment();
                   while (buffer.hasRemaining()) {
                       //Write data to the connected client
                       if (client.write(buffer) == 0) {
                           break;
                       }
                   }
                   //Close the connection
                   client.close();
               }
           }
       }
   }
   ```

3. Blocking networking with Netty

   ```java
   public class NettyOioServer {
       public void serve(int port) throws Exception{
           ByteBuf buf = Unpooled.unreleasableBuffer(Unpooled.copiedBuffer("Hi!\r\n", StandardCharsets.UTF_8));
           EventLoopGroup group = new OioEventLoopGroup();
           //Create a ServerBootstrap
           ServerBootstrap b = new ServerBootstrap();
           b.group(group)
                   //Use OioEventLoopGroup to allow blocking mode
                   .channel(OioServerSocketChannel.class)
                   .localAddress(new InetSocketAddress(port))
                   //Specifies ChannelInitializer that will be called for each accepted connection
                   .childHandler(new ChannelInitializer<SocketChannel>() {
                       @Override
                       protected void initChannel(SocketChannel socketChannel) throws Exception {
                           //Add a ChannelInboundHandlerAdapter to intercept handler events
                           socketChannel.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                               @Override
                               public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                   ctx.writeAndFlush(buf.duplicate())
                                           .addListener(ChannelFutureListener.CLOSE);
                               }
                           });
                       }
                   });
           //Bind server to accept connections
           ChannelFuture f = b.bind().sync();
           f.channel().closeFuture().sync();
           //Release all resources
           group.shutdownGracefully().sync();
       }
   }
   ```

4. Asynchronous networking with Netty

   ```java
   public class NettyNioServer {
       public void serve(int port)throws Exception{
           ByteBuf buf = Unpooled.unreleasableBuffer(Unpooled.copiedBuffer("Hi!\r\n", StandardCharsets.UTF_8));
           //Use NioEventLoopGroup for non-blocking mode
           EventLoopGroup group = new NioEventLoopGroup();
           //Create ServerBootstrap
           ServerBootstrap b = new ServerBootstrap();
           b.group(group)
                   .channel(NioServerSocketChannel.class)
                   .localAddress(new InetSocketAddress(port))
                   //Specifies ChannelInitializer to be called for each accepted connection
                   .childHandler(new ChannelInitializer<SocketChannel>() {
                       @Override
                       protected void initChannel(SocketChannel socketChannel) throws Exception {
                           //Add ChannelInboundHandlerAdapter to receive events and process them
                           socketChannel.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                               //Write message to client and add ChannelFutureListener to close the connection once the message is written
                               @Override
                               public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                   ctx.writeAndFlush(buf.duplicate())
                                           .addListener(ChannelFutureListener.CLOSE);
                               }
                           });
                       }
                   });
           //Bind server to accept connections
           ChannelFuture f = b.bind().sync();
           f.channel().closeFuture().sync();
           //Release all resources
           group.shutdownGracefully().sync();
       }
   }
   ```

## 4.2 Transport API

1. At the heart of the transport API is interface *Channel*

2. The most important *Channel* method

   ![image-20220208161825432](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-channel-method.png)

3. Netty's *Channel* implementations are thread-safe

## 4.3 Included transports

![image-20220208162810213](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-provided-transports.png)

### 4.3.1 NIO——non-blocking I/O

1. The basic concept behind the selector is to serve as a registry where you request to be notified when the state of a *Channel* changes

2. Internal details of NIO are hidden by the user-level API common to all of Netty's transport implementations

   ![image-20220208163515497](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-selecting-and-processing-state-change.png)

### 4.3.2 Epoll—native non-blocking transport for linux

### 4.3.3 OIO—old blocking I/O

1. How Netty support OIO

   1. Netty makes use of the SO_TIMEOUT Socket flag, which specifies the maximum number of milliseconds to wait for an I/O operation to complete.
   2. If the operation fails to complete within the specified interval, a `SocketTimeoutException` is thrown
   3. Netty catches this exception and continues the processing loop. On the next EventLoop run, it will try again
   4. This is the only way an asynchronous framework like Netty can support OIO

2. OIO processing logic

   ![image-20220208165345692](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-oio-processing-logic.png)

### 4.3.4 Local transport for communication within a JVM

### 4.3.5 Embedded transpot

## 4.4 Transport use cases

## 4.5 Summary

# Chapter5 ByteBuf

*ByteBuf* : Netty's alternative to *ByteBuffer*

## 5.1 The ByteBuf API

Netty's API for data handling:

1. abstract class ByteBuf
2. interface ByteBufHolder

## 5.2 Class ByteBuf—Netty's data container

### 5.2.1 How it works

![image-20220209095129538](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-bytebuf-layout.png)

*ByteBuf* maintains two distinct indices: one for reading and one for writing

An array of bytes with distinct indices to control read and write access

### 5.2.2 ByteBuf usage patterns

1. HEAP BUFFERS

   1. stores the data in the heap space of JVM
   2. fast allocation and dealloction in situations where poo;ing isn't use
   3. well suited to cases where you have to handle legacy data
   4. this pattern is similar to uses of the JDK's ByteBuffer

   ```java
   ByteBuf heapBuf = ...;
   if(heapBuf.hasArray()){
       byte[] array = heapBuf.array();
       int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();
       int length = heapBuf.readableBytes();
       handleArray(array,offset,length);
   }
   ```

2. Direct Buffers

   1. The contents of direct buffers will reside outside of the normal garbage-collected heap

   ```java
   ByteBuf directBuf = ...;
   if(directBuf.hasArray()){
       int length = directBuf.readableBytes();
       byte[] array = new byte[length];
       directBuf.getBytes(directBuf.readerIndex(),array))
       handleArray(array,0,length);
   }
   ```

3. Composite Buffers

   1. Add and delete ByteBuf instances as needed

   ![image-20220209102113111](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-compositebuf.png)

   1. 例如，当只需要改写头时，可以共用相同的body，避免了不必要的资源消耗
   2. CompositeByteBuf may not allow access to a backing array, so accessing the data in a CompositeByteBuf resembles the direct buffer pattern

## 5.3 Byte-level operations

### 5.3.1 Random access Indexing

```java
//简单的遍历
ByteBuf buffer = ...;
for(int i=0;i<buffer.capcity() - 1; i++){
    byte b = buffer.getByte(i);
    log.info(b);
}
```

accessing the data using one of the methods that takes an index argument doesn't alter the value of either readerIndex or writerIndex

### 5.3.2 Sequential aceess indexing

how a *ByteBuf* is partitioned by its two indices into three areas

![image-20220209104036569](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-bytebuf-internal-segmentation.png)

### 5.3.3 Discardable bytes

after *discardedBytes()*

![image-20220209143351864](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netyy-bytebuf-after-discardedbytes.png)

this will cause meomry copying

### 5.3.4 Readable bytes

read all data

```java
ByteBuf b = ...;
while(b.isReadable()){
    log.info(b.readByte());
}
```

### 5.3.5 Writable bytes

write data

```java
//Fills the writable bytes of a buffer with random integers
ByteBuf b = ...;
while(b.writableBytes() >= 4){
    b.writeInt(random.nextInt());
}
```

### 5.3.6 Index management

`markReaderIndex(), markWriterIndex(), resetReaderIndex(), resetWriterIndex(), clear()`

### 5.3.7 Search operations

Using ByteBufProcessor to find \r

```java
ByteBuf b =...;
int index = b.forEachByte(ByteBufProcessor.FIND_CR);
```

### 5.3.8 Derived buffers

1. A derived buffer provides a view of a *ByteBuf* that represents its contents in a specialized way

2. If you modify its contents you are modifying the source instance as well

3. Slice a *ByteBuf*

   ```java
   Charset utf8 = Charset.forName("UTF-8");
   //Creating a ByteBuf, which holds bytes for given string
   ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!",utf8);
   //Creating a new slice of ByteBuf starting at index 0 and ending at index 4
   ByteBuf sliced = buf.slie(0,14);
   //Prient Netty in Action rocks!
   log.info(sliced.toString(utf8));
   //Update the byte at index 0
   buf.setByte(0,(byte) 'J');
   //Successd because the data is shared—modifications made to one will be visible in the other
   assert buf.getByte(0) == sliced.getByte(0);
   ```

   

4. Copying a *ByteBuf*

   ```java
   Charset utf8 = Charset.forName("UTF-8");
   ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!",utf8);
   ByteBuf copy = buf.copy(0,14);
   log.info(sliced.toString(utf8));
   buf.setByte(0,(byte) 'J');
   assert buf.getByte(0) != sliced.getByte(0);
   ```

5. Whenever possible, use `slice()` to avoid the cost of copying memory

### 5.3.9 Read/Write operations

1. Most frequently used `get(), set()` method (get/set operations that start at a given index and leave it unchanged)

   1. ![image-20220209151227915](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-get-operation.png)

   2. ![image-20220209151254470](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty0set-operation.png)

   3. `get()` and `set()` usage

      ```java
      Charset utf8 = Charset.forName("UTF-8");
      ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!", utf8);
      log.info((Byte)buf.getByte(0));
      int readerIndex = buf.readerIndex();
      int writerIndex = buf.writerIndex();
      buf.setByte(0, (Byte) 'B');
      log.info((Char) buf.getByte(0));
      //Succeed because these operations don't modify the indices
      assert readerIndex == buf.readerIndex();
      assert writerIndex == buf.writerIndex();
      ```

      

2. Most frequently used `write(), read()` method (write/read operations that start at a given index and adjust it by the number of bytes accessed)

   1. ![image-20220209151944089](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-read-operation.png)

      ![image-20220209152010547](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-read-operation-continued.png)

   2. ![image-20220209152408798](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-write-operation.png)

   3. `read()` and `write()` operations on the *ByteBuf*

      ```java
      Charset utf8 = Charset.forName("UTF-8");
      ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!", utf8);
      log.info((Byte)buf.getByte(0));
      int readerIndex = buf.readerIndex();
      int writerIndex = buf.writerIndex();
      buf.writeByte(0, (Byte) '?');
      log.info((Char) buf.getByte(0));
      
      assert readerIndex == buf.readerIndex();
      assert writerIndex != buf.writerIndex();
      ```

      

### 5.3.10 More opeartions

![image-20220209153112485](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-other-operation.png)

## 5.4 Interface ByteBufHolder

![image-20220209153423922](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-bytebufferholder-operation.png)

## 5.5 ByteBuf allocation

### 5.5.1 On-demand: Interface ByteBufAllocator

1. To reduce the overhead of the allocating and deallocating memory, Netty implements pooling with the interface *ByteBufAllocator*, which can be used to allocate instances of any of the *ByteBuf* varieties
2. Netty provides tow implementations of *ByteBufAllocator* : *PooledByteBufAllocator* and *UnpooledByteBufAllocator* 
3. Netty uses the *PooledByteBufAllocator* by default

### 5.5.2 Unpooled buffers

1. Netty provides a utility class called *Unpolled*, which provides a static helper methods to create unpooled *ByteBuf*  instances
2. ![image-20220209155720590](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-unpolled-method.png)

### 5.5.3 Class ByteBufUtil

1. *ByteBufUtil* provides static helper methods for manipulating a *ByteBuf*

## 5.6 Reference counting

1. Reference counting is a technique for optimizing memory use and performance by releasing the resources held by an object when it is no longer referenced by other objects
2. Netty introduced reference counting in version 4 for *ByteBuf* and *ByteBufHolder*

## 5.7 Summary

1. The use of distinct read and write indices to control data access
2. Different approaches to memory usage—backing arrays and direct buffers
3. The aggregate view of multiple ByteBufs using CompositeByteBuf
4. Data-access methods: searching, slicing, and copying
5. The read, write, get, and set APIs
6. ByteBufAllocator pooling and reference counting

# Chapter6 ChannelHandler and ChannelPipeline

## 6.1 The ChannelHandler family

### 6.1.1 The Channel lifecycle

![image-20220209163500747](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-channel-state-model.png)

As these state changes occur, corresponding events are generated. These are forwarded to *ChannelHandler* in the *ChannelPipeline*, which can then act on them

### 6.1.2 The ChannelHandler lifecycle

1. handlerAdded
2. handlerRemoved
3. exceptionCaught

### 6.1.3 Interface ChannelInboundHandler

![image-20220209164620229](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-ChannelInboundHandler-method.png)

When a *ChannelInboundHandler* implementation overrides `channelRead()` , it is responsible for explicitly releasing the memory associated with pooled *ByteBuf* instances. You can use `ReferenceCountUtil.release()` to do that. A simple alternative is to use *SimpleChannelInboundHandler*, it releases resources automatically

### 6.1.4 Interface ChannelOutboundHandler

1. A powerful capability of *ChannelOutboundHandler* is to defer an operation or event on demand, which allows for sophisticated approaches to request handling
2. Most of the methosd in *ChannelOutboundHandler* take a *ChannelPromise* argument to be notified when the operation completes. *ChannelPromise* is a subinterface of *ChannelFuture* that defines the writable methods

### 6.1.5 ChannelHadnler adapters

1. *ChannelInboundHandlerAdapter* and *ChannelOutboundHandlerAdapter* provide basic implementataions of *ChannelHandler* respectively

2. hierarchy

   ![image-20220210094442584](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-channelHandlerAdapter-hierarchy.png)

### 6.1.6 Resource management

1. It's important to adjust the reference count after you have finished using a *ByteBuf* 

2. Netty provides class *ResourceLeakDector*, which will sample 1% of your application's buffer allocations to check for memory leaks

3. Leak-detection levels

   ![image-20220210095830590](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-leak-detection-level.png)

## 6.2 Interface ChannelPipeline

1. Think of a *ChannelPipeline* as a chain of *ChannelHandler* instances that intercept the inbound and outbound events that flow through a *Channel*
2. Every new *Channel* that's created is assigned a new *ChannelPipeline*. This association is permanent
3. This is a fixed operation in Netty's component lifecycle and requeires no action on the part of the developer
4. *ChannelHandlerContext*
   1. A *ChannelHandlerContext* enables a *ChannelHandler* to interact with its *ChannelPipeline* and with other handlers
   2. A handler can notify the next *ChannelHandler* in the *ChannelPipeline* and even dynamically modify the *ChannelPipeline* it belons to
5. Netty always identifies the inbound entry to the *ChannelPipeline* as the beginning and the outbound entry as the end
6. A handler might implement both *ChannelInboundHandler* and *ChannelOutboundHandler*

### 6.2.1 Modifying a ChannelPipeline

1. A *ChannelHandler* can modify the layout of a *ChannelPipeline* in real time by adding, removing, or replacing other *ChannelHandler*. It's can remove itself from the *ChannelPipeline* as well
2. ![image-20220210104238958](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-modifying-channelPipeline.png)

### 6.2.2 Firing events

In summary:

1. A *ChannelPipeline* holds the *ChannelHandlers* associated with a *Channel*
2. A *ChannelPipeline* can modified dynamically by adding and removing *ChannelHandlers* as needed
3. *ChannelPipeline* has a rich API for invoking actions in response to inbound and outbound events

## 6.3 Interface ChannelHandlerContext

1. A *ChannelHandlerContext* repsents an association between a *ChannelHandler* and a *ChannelPipeline*
2. Is created whenever a *ChannelHandler* is added to a *ChannelPipeline* 
3. The primary function of a *ChaneelHandlerContext* is to manage the interaction  of its associated *ChannelHandler* with others in the same *ChannelPipeline*
4. The same methods called on a *ChannelHandlerContext* will start at the current associated *ChannelHandler* and propagate only to the next *ChannelHandler* in the pipeline that is capable of handling the event
5. When using the *ChannelHandlerContext* API
   1. The *ChannelHandlerContext* associated with a *ChannelHandler* never changes. so it's safe to cache a reference to it
   2. *ChannelHandlerContext* methods involve a shorter event flow than do the identically named methods available on other classes

### 6.3.1 Using ChannelHandlerContext

1. Relationships among *ChannelHandlerContext*, *Channel*, *ChannelPipeline*

   ![image-20220210112426935](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-relationship-among-channel-*.png)

2. `channel.write` and `pipeline.write`

   1. ![image-20220210113949446](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-event-propagate.png)
   2. 两个write方法的总体流程是一样的，但实现形式有不同，channel.write应该是红线

3. Why propagate an event starting at a specific point in the *ChannelPipleine*

   1. To reduce the overhead of passing the event through *ChannelHandlers* that are not interested in it
   2. To prevent processing of the event by handlers that would be intersted in the event

4. `channelHandlerContext.write`

   1. ![image-20220210115512090](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-channelHandlerContext-write-flow.png)

### 6.3.2 Advanced uses of ChannelHandler and ChannelHandlerContext

1. Caching a *ChannelHandlerContext*

   ```java
   public class WriteHandler extends ChannelHandlerAdapter{
       privite ChannelHandlerContext ctx;
       
       @Override
       public void handlerAdded(ChannelHandlerContext ctx){
           //Store reference to ChannelHandlerContext for later use
           this.ctx = ctx;
       }
       
       //Send message using previously stored ChannelHandlerContext
       public void send(String msg){
           ctx.writeAndFlush(msg);
       }
   }
   ```

   

2. A sharable *ChannelHandler*

   1. `@Sharable`: use it only if you're certaion that your *ChannelHandler* is thread-safe(doesn't hold any state)

3. Why share a *ChannelHandler*

   1. A common reason is to gather statistics across mutiple *Channels*

## 6.4 Exception handling

### 6.4.1 Handling inbound exceptioons

1. just override `public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception`
2. The exception will continue to flow in the inbound direction, so the *ChannelInboundHandler* that implements the method `exceptionCaught` usually olaced last in the *ChannelPipeline*. This ensures all inbound exceptions are alwayshandled, wherever in the *ChannelPipeline* they may occur
3. Summerarize
   1. The default implementation of `ChannelInboundHandler.exceptionCaught()` forwards the current exceptino to the next handler in the pipeline
   2. If an exception reaches the end of the pipline, it's logged ass unhandled
   3. To define custom handling,  you override exceptionCaught()

### 6.4.2 Handling outbound exceptions

1. Notification mechanisms:

   1. Every outbound operation returns a *ChannelFuture*. The *ChannelFutureListener* registered with a *ChannelFuture* are notified of success or error when the operation completes
   2. Almost all methods of *ChannelOutboundHandler* are paased an instance of *ChannelPromise* (a subclass of *ChannelFuture*, can also be assigned listeners for asychrounous notification)

2. Adding a *ChannelFutureListener* to a *ChannelFuture*

   ```java
   ChannelFuture future = channel.write(msg);
   future.addListener(new ChannelFutureListener() {
       @Override
       public void operationComplete(ChannelFuture f){
           if(!f.isSuccess()){
               f.cause(.printStackTrace());
               f.channel().close();
           }
       }
   })
   ```

   

3. Adding a *ChannelFutureListener* ti a *ChannelPromise* 

   ```java
   public class OutboundExceptionHandler extends ChannelOutboundHandlerAdapter {
       @Override
       public void write (ChannelHandlerContext ctx, Object msg, ChannelPromise promise){
           promise.addListener(new ChannelFutureListener() {
               @Override
               public void operationComplete(ChannelFuture f){
                   if(!f.isSuccess()){
                       f.cause().printStackTrace();
                       f.channel().close();
                   }
               }
           })
       }
   }
   ```

   

# Chapter7 EventLoop and threading model

1. Simply stated, a *threading model* specifies key aspects of thread management in the context of an OS, programming, language, framework, or a application
2. JUC—java.util.concurrent --> 《Java Concurrency in Pratice》by Brian Goetz, et al

## 7.1 Threading model overview

1. The basic thread pooling pattern can be described as:
   1. A *Thread* is selected from the pool's free list and assigned to run a submitted task
   2. When the task is complete, the *Thread* is returned to the list and becomes available for reuse 
   3. ![image-20220210145206209](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-executor-execution-logic.png)

## 7.2 Interface EventLoop

1. Exceuting tasks In an event loop

   ```java
   while(!terminated){
       //Block until there are events that are ready to run
       List<Runnable> readyEvents = blockUtilEventReady();
       //Loops over and runs all the events
       for(Runnable event : readyEvents){
           event.run();
       }
   }
   ```

2. Netty's *EventLoop* is part of a collaborative design that employs two fundamental APIs: concurrency and networking

   ![image-20220210150046972](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-eventloop-hierarchy.png)

   1. In this model, an *EventLoop* is powered by exactly one *Thread* that never changes and tasks can be submitted directly to *EventLoop* implementations for immediate or scheduled execution
   2. Multiple *EventLoop* may becreatred in order to optimize resource use
   3. A single *EventLoop* may be assigned to service multiple *Channels*

3. Events and tasks are executed in FIFO—first-in-first-out order

### 7.2.1 I/O and event handling in Netty 4

1. **In Netty 4 all I/O operations and events are handled by the *Thread* that has been assigned to the *EventLoop***, This differs from the model that was used in Netty 3

### 7.2.2 I/O operations in Netty 3

1. In short, it wasn't possible to gurantee that multiple threads wouldn't try to access an outbound event at the same time
2. Netty 4 resolves these problems by handling everything that occurs in a given *EventLoop* in the same thread

## 7.3 Task scheduling

### 7.3.1 JDK scheduling API

1. Scheduling a task with a ScheduledExecutorService

   ```?
   //Create a ScheduledExecutorService with a pool of 10 threads
   ScheduledExecutorService executor = Executors.newScheduledThreadPool(10);
   
   //Create a Runnable to schedul for later execution
   ScheduledFuture<?> future = executor.schedule(new Runnable(){
   	@Override
   	public void run(){
   		log.info("60 Seconds later");
   	}
   },60,TimUtil.SECONDS);
   ...
   executor.shutdown();
   ```

### 7.3.2 Scheduling tasks using EventLoop

1. Scheduling a task with *EventLoop*

   ```java
   Channel ch = ...;
   ScheduledFuture<?> future = ch.eventLoop().schedule(
   	new Runabble(){
           @Override
           public void run(){
               log.info("60 seconds later");
           }
       }, 60, TimeUtil.SECONDS);
   ```

   

2. Scheduling a recurring task with *EventLoop*

   ```java
   Channel ch = ...;
   ScheduledFuture<?> future = ch.eventLoop().scheduleAtFixedRate(
   	new Runabble(){
           @Override
           public void run(){
               log.info("60 seconds later");
           }
       }, 60, 60, TimeUtil.SECONDS);
   ```

3. Cancelling a task using *ScheduleFuture*

   ```java
   ScheduledFuture<?> future = ch.eventLoop().scheduleAtFixedRate(...);
   
   boolean mayInterruptIfRunning = false;
   future.cancel(mayInterruptIfRunning);
   ```

## 7.4 Implementation details

### 7.4.1 Thread management

1. The superior performance of Netty's threading model hinges on determining the identity of the currently executing *Thread*
2. that is, whether or not it is the one assigned to the current *Channel* and its *EventLoop* 
   1. If the calling *Thread* is that of the *EventLoop*, the code block in question is executed
   2. Otherwise, the *EventLoop* schedules a task for later execution and puts it in an internal queue
   3. When the *EventLoop* next processes its events, it will execute those in the queue
3. Each *EventLoop* has its own task queue, independent of that of any other *EventLoop* 
   1. ![image-20220210154613816](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-eventloop-excution-logic.png)
4. Never put a long-running task in the execution queue, if must, we advise the use of the dedicated *EventExecutor*

### 7.4.2 EventLoop/thread allocation

1. Asynchronous transports

   ![image-20220210155449336](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-eventloop-for-nio-transpot.png)

2. Blocking transports

   ![image-20220210155754057](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-eventloop-allocation-of-oio.png)



## 7.5 Summary



# Chapter 8 Bootstrapping

1. Simply stated, *Bootstrapping* an application is the process of configuring it to run—though the details of the process may not be simple as its definition, especially in network applications
2. Netty handles bootstrapping in a way that insulates your application, whether client or server, from the network layer

## 8.1 Bootstrap classes

1. Namely, a server devotes a parent channel to accepting connections from clients and creating child channels for conversing with them, whereas a client will most likely require only a single, non-parent channel for all network interactions

## 8.2 Bootstrapping clients and connectionsless protocols

### 8.2.1 Bootstrapping a client

1. ![image-20220210162239034](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-bootstrapping-process.png)

2. Bootstrapping a client

   ```java
   public class Bootstrapping {
       public static void main(String[] args) {
           EventLoopGroup group = new NioEventLoopGroup();
           Bootstrap bootstrap = new Bootstrap();
   
           bootstrap.group(group)
                   .channel(NioServerSocketChannel.class)
                   .handler(new SimpleChannelInboundHandler<ByteBuf>() {
                       @Override
                       protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
                           log.info("Received data");
                       }
                   });
   
           ChannelFuture future = bootstrap.connect(new InetSocketAddress("127.0.0.1", 80));
           future.addListener(new ChannelFutureListener() {
               @Override
               public void operationComplete(ChannelFuture channelFuture) throws Exception {
                   if(channelFuture.isSuccess()){
                       log.info("Connection established");
                   }else{
                       log.info("Connection attempt failed");
                       channelFuture.cause().printStackTrace();
                   }
               }
           });
       }
   }
   
   ```

### 8.2.2 Channel and EventLoopGroup compatibility

Must call the following methods to set up the required components

1. `group()`
2. `channel()` or `channelFactory()`
3. `handler()`

## 8.3 Bootstrapping servers

### 8.3.1 The ServerBootstrap class

### 8.3.2 Bootstrapping a server

1. ![image-20220210164655226](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-serverbootstrapping.png)

## 8.4 Bootstrapping clients from a Channel

![image-20220210165135300](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-eventloop-shared-between-channels.png)

```java
public class Bootstrapping2 {
    public static void main(String[] args) {
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        //Set the EventLoopGroups that provide EventLoops for processing Channel events
        serverBootstrap.group(new NioEventLoopGroup(), new NioEventLoopGroup())
                .channel(NioServerSocketChannel.class)
                .childHandler(new SimpleChannelInboundHandler<ByteBuf>() {
                    @Override
                    public void channelActive(ChannelHandlerContext ctx) throws Exception {
                        Bootstrap bootstrap = new Bootstrap();
                        bootstrap.channel(NioSocketChannel.class)
                                .handler(new SimpleChannelInboundHandler<ByteBuf>() {
                                    @Override
                                    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
                                        log.info("Received data");
                                    }
                                });

                        bootstrap.group(ctx.channel().eventLoop());
                        ChannelFuture future = bootstrap.connect(new InetSocketAddress("127.0.0.1", 80));
                    }
                    @Override
                    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
                        //Do something with the data
                    }
                });

        ChannelFuture future = serverBootstrap.bind(new InetSocketAddress(8080));
        future.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture channelFuture) throws Exception {
                if(channelFuture.isSuccess()){
                    log.info("Server bound");
                }else{
                    log.info("Bind attempt failed");
                    channelFuture.cause().printStackTrace();
                }
            }
        });
    }
}
```



## 8.5 Adding multiple ChannelHandlers during a bootstrap

Netty提供了类*ChannelInitializer* 来实现为一个*ChannelPipeline* 里面提供多个*ChannelHandler*(即 channelHandler chain)

1. Bootstrapping and using *ChannelInitializer*

   ```java
   ServerBootstrap bootstrap = new ServerBootstrap();
   bootstrap.group(new NioEventLoopGroup(),new NioEventLoopGroup())
       .channel(NioSeverSocketChannel.class)
       .childHandler(new ChannelInitializerImpl());
   ChannelFuture future = bootstrap.bind(new InetSocketAddress(8080));
   future.sync();
   
   final class ChannelInitializerImpl extends ChannelInitializer<Channel> {
       @Override
       protected void initiChannel(Channel ch) throws Exception {
           ChannelPipeline pipeline = ch.pipeline;
           pipeline.addLast(new HttpClientCodec());
           pipeline.addLast(new HttpObjectAggregator(Integer.MAX_VALUE));
       }
   }
   ```

   

## 8.6 Using Netty ChannelOptions and attributes

1. Using attributes

   ```java
   final AttributeKey<Integer> id = new AttributeKey<>("ID");
   Bootstrap bootstrap = new Bootstrap();
   bootstrap.group(new NioEventLoopGroup())
       .channel(NioSocketChannel.class)
       .handler(new SimpleChannelInboundHandler<ByteBuf>(){
           @Override
           public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
               Integer idValue = ctx.channel().attr(id).get();
               //Do something with the idValue
           }
           
           @Override
           protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
               log.info("Received data");
           }
       });
   
   bootstrap.option(ChannelOption.SO_KEEPLIVE,true)
       .option(ChannelOption.CONNECT_TIMEOUT_MILLS,5000);
   bootstrap.attr(id,123456);
   ChannelFuture future = bootstrap.connect(new InetSocketAddress("127.0.0.1",80));
   future.syncUniterruptibly();
   ```

## 8.7 Bootstrapping DatagramChannels

Don't call `connect()` but only `bind()`

## 8.8 Shutdown

`EventLoopGrouo.shutdownGracefully()`

## 8.9 Summary

# Chapter9 Unit testing

*EmbeddedChannel*, that Netty provides specifically to facilitate unit testing of *ChannelHandlers*

## 9.1 Overview of EmbeddedChannel

1. straightforward idea

   1. write inbound or outbound data into a n EmbeddedChannel 
   2. then check whether anything reached the end of the *ChannelPipeline*

2. EmbeddedChannel data flow

   ![image-20220211104958603](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-embeddedchannel-data-flow.png)

## 9.2 Testing ChannelHandlers with EmbeddedChannel

### 9.2.1 Testing inbound message

EmbeddedChannel eh = new EmbeddedChannel(要测试的handler)

`eh.writeInbound`, `eh.readInbound`, 看read的结果是否和预期一样

### 9.2.2 Testing outbound message

## 9.3 Testing exception handling

## 9.4 Summary

# Part 2 Codecs

# Chapter 10 The codec framework

## 10.1 What is a codec

encoder + decoder

## 10.2 Decoders

1. Netty's decoders implement *ChannelInboundHandler* 
2. you can chain together multiple decoders thanks to the design of *ChannelPipeline* 

### 10.2.1 Abstract class ByteToMessageDecoder

```java
public class ToIntegerDecoder extends ByteToMessageDecoder {
    @Override
    public void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        if(in.readableBytes() >= 4){
            out.add(in.readInt());
        }
    }
}
```



### 10.2.2 Abstract class ReplayingDecoder

```java
public class ToIntegerDecoder2 extends ReplayingDecoder<Void> {
    @Override
    public void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception{
        out.add(in.readInt());
    }
}
```

1. *ReplayingDecoder* is slightly slower than *ByteToMessageDecoder*

### 10.2.3 Abstract class MessageToMessageDecoder

```java
public class IntegerToStringDecoder extends MessageToMessageDecoder<Integer> {
    @Override
    public void decode(ChannelHandlerContext ctx, Integer msg, List<Object> out) throws Exception {
        out.add(String.valueOf(msg));
    }
}
```

### 10.2.4 Class TooLongFrameException

Is intended to be thrown by decoders if a frame exceeds a specified size limit 

```java
public class SafeByteToMessageDecoder extends ByteToMessageDecoder {
    private static final int MAX_FRAME_SIZE = 1024;
    @Override
    public void decode(ChannelHandelrContext ctx, ByteBuf in, List<Object> out) throws Exception {
        int readable = in.readableBytes();
        if(readable > MAX_FRAME_SIZE){
            in.skipBytes(readable);
            throw new TooLoongFrameException("Frame too big!");
        }
    }
}
```

## 10.3 Encoders

### 10.3.1 Abstract class MessageToByteEncoder

 ```java
 public class ShortToByteEncoder extends MessageToByteEncoder<Short> {
     @Override
     public void encode(ChannelHandlerContext ctx, Short msg, ByteBuf out) throws Exception {
         out.writeShort(msg);
     } 
 }
 ```

### 10.3.2 Abstract class MessageToMessageEncoder

```java
public class IntegerToStringEncoder extends MessageToMessageEncoder<Integer> {
    @Override
    public void encode(ChannelHandlerContext ctx, Integer msg, ByteBuf out) throws Exception {
        out.add(String.valueOf(msg));
    } 
}
```

## 10.4 Abstract codec classes

### 10.4.1 Abstract class ByteToMessageCodec

### 10.4.2 Abstract class MessageToMessageCodec

### 10.4.3 Class CombinedChannelDuplexHandler

This class acts as a container for a *ChannelInboundHandler* and a *ChannelOutboundHandler*

```java
public class CombineByteCharCodec extends CombinedChannelDuplexHandler<ByteToCharDecoder, charToByteEncoder> {
    public CombinedByteCharCodec(){
        super(new ByteToCharDecoder(), new CharToByteEncoder());
    }
}
```

# Chapter 11 Provided ChannelHandlers and codecs

## 11.1 Securing Netty applications with SSL/TLS

*SslHandler* data flow

![image-20220214112202669](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-sslhandler-data-flow.png)

暂时用不到，先略过

## 11.2 Building Netty HTTP/HTTPS applications

1. HTTP request component parts

   ![image-20220214112553800](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-http-component-ports.png)

2. HTTP response component parts

![image-20220214112635620](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-http-response-component-parts.png)

3. Adding support for HTTP

   ```java
   public class HttpPipelineInitializer extends ChannelInitializer<Channel> {
       private final boolean client;
   
       public HttpPipelineInitializer(boolean client){
           this.client = client;
       }
   
       @Override
       protected void initChannel(Channel channel) throws Exception {
           ChannelPipeline pipeline = channel.pipeline();
           
           if(client) {
               pipeline.addLast("decoder",new HttpResponseDecoder());
               pipeline.addLast("encoder",new HttpRequestEncoder());
           }else {
               pipeline.addLast("decoder",new HttpRequestDecoder());
               pipeline.addLast("encoder",new HttpResponseEncoder());
           }
       }
   }
   ```

### 11.2.2 HTTP message aggregation

Automatically aggregating HTTP message fragments

```java
public class HttpAggregatorInitializer extends ChannelInitializer<Channel> {
    private final boolean isClient;

    public HttpAggregatorInitializer(boolean isClient){
        this.isClient  = isClient;
    }

    @Override
    protected void initChannel(Channel channel) throws Exception {
        ChannelPipeline pipeline = channel.pipeline();
        if(isClient) {
            pipeline.addLast("codec",new HttpClientCodec());
        }else{
            pipeline.addLast("codec",new HttpServerCodec());
        }

        pipeline.addLast("aggregator", new HttpObjectAggregator(512 * 1024));
    }
}
```

### 11.2.3 HTTP compression

### 11.2.4 Using HTTPS

### 11.2.5 WebSocket

![image-20220214114719256](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-websocket-protocol.png)

## 11.3 Idle connections and timeouts

*IdleStateHandler*, *ReadTimeoutHandler*, *WriteTimeoutHandler*

Sending heartbeats

![image-20220214115555135](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-sending-heartbeat.png)



## 11.4 Decoding delimited and length-based protocols

SMTP, POP3, IMAP, Telnet

暂时用不到，略过

### 11.4.1 Delimited protocols

### 11.4.2 Length-based protocols

## 11.5 Writing big data

## 11.6 Serializing data

### 11.6.1 JDK serialization

### 11.6.2 Serialization with JBoss Marshalling

### 11.6.3 Serialization via Protocol Buffers

## 11.7 Summary

# Part 3 Network protocols

# Chapter 12 WebSocket

暂时跳过

# Chapter 13 Broadcasting events with UDP

## 13.1 UDP basics

1. UDP, conversely, resembles dropping a bunch of postcards in a mailbox. Your can't know the order in which they will arrive at their destination, or even if they all will arrive
2. UDP is a good fit applications that can handle or tolerate lost messages

## 13.2 UDP broadcast

1. Multicast
   1. Transmission to a defined group of hosts
2. Broadcast
   1. Transmission to all of the hosts on a network( or a subnet )

## 13.3 The UDP sample application

![image-20220214141852607](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-broadcast-system-overview.png)

## 13.4 The message POJO: LogEvent

```java
@Data
public class LogEvent {
    public static final byte SEPARATOR = ':';
    private InetSocketAddress source;
    private String logfile;
    private String msg;
    private long received;

    public LogEvent(){}
    
    public LogEvent(String logfile, String msg) {
        this.logfile = logfile;
        this.msg = msg;
    }

    public LogEvent(InetSocketAddress source, long received, String logfile, String msg) {
        this.source = source;
        this.received = received;
        this.logfile = logfile;
        this.msg = msg;
    }
}
```

## 13.5 Writing the broadcaster

1. Netty's *DatagramPacket* is a simple message container used by *DatagramChannel* implementations to communicate with remote peers

2. LogEventBroadcaster : ChannelPipeline and LogEvent flow

   1. ![image-20220214143143105](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-logEventBroadcaster-flow.png)

   2. ```java
      public class LogEventBroadcaster {
          private EventLoopGroup group;
          private Bootstrap bootstrap;
          private File file;
      
          public LogEventBroadcaster(InetSocketAddress address, File file) {
              group = new NioEventLoopGroup();
              bootstrap = new io.netty.bootstrap.Bootstrap();
              bootstrap.group(group)
                      .channel(NioDatagramChannel.class)
                      .option(ChannelOption.SO_BROADCAST, true)
                      .handler(new LogEventEncoder(address));
      
              this.file = file;
      
          }
      
          public void run() throws Exception {
              Channel ch = bootstrap.bind(0).sync().channel();
              long pointer = 0;
              for (; ; ) {
                  long len = file.length();
                  if(len < pointer){
                      //file was reset
                      pointer = len;
                  }else if(len > pointer){
                      //Content was added
                      RandomAccessFile raf = new RandomAccessFile(file,"r");
                      raf.seek(pointer);
                      String line;
                      while((line = raf.readLine()) != null){
                          ch.writeAndFlush(new LogEvent(null, -1, file.getAbsolutePath(),line));
                      }
      
                      pointer = raf.getFilePointer();
                      raf.close();
                  }
                  Thread.sleep(1000);
              }
          }
      
          public void stop(){
              group.shutdownGracefully();
          }
      
          public static void main(String[] args) throws Exception {
              LogEventBroadcaster broadcaster = new LogEventBroadcaster(new InetSocketAddress("255.255.255.255",Integer.parseInt(args[0])),new File(args[1]));
              broadcaster.run();
          }
      
      }
      ```

## 13.6 Writing the monitor

![image-20220214151321136](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/netty-logEventMonitor.png)

