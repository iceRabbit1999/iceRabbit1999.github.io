# *Netty in Action*

# Part1 Netty concepts and architecture

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

### 1.3 Netty's core components

### 1.3.1 Channels

### 1.3.2 Callbacks

### 1.3.3 Futures

