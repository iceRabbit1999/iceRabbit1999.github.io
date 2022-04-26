# *Computer Networking: A Top-Down Approach*

# Chapter 1 Computer Networks and the Internet

## 1.1 What  Is the Internet?

1. First, we can describe the nuts and bolts of the Internet, that is the basic hardware and software components that make up the Internet
2. Second, we can describe the Internet in terms of a networking infrastructure that provides services to distributed applications

### 1.1.1 A Nuts-and-Bolts Description

some jargon

1. host
2. end system
3. communication link
4. packet switches
5. transmission rate
6. router
7. link-layer switches
8. route/path
9. ISP: Internet Service Provider
10. TCP/IP
11. Internet standards
12. RFC: requests for comment

网络的传输  <--> 高速路、十字路口的传输

### 1.1.2 A Service Description

1. Describe the Internet from an entirely different angle: Namely, as *an infrastructure that provides services to applications*
   1. How does one program running on one end system instruct the Internet to deliver data to another program running on another end system 用写信的例子类比socket interface

### 1.1.3 What Is a Protocol

![image-20220219195054714](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-protocol.png)

1. The exchange of messages and the actions taken when these  messages are sent and received are the key defining elements of a protocol
2. A protocol defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event

## 1.2 The Network Edge

1. End system: they sit at the edge of the Internet  *host = end system*

### 1.2.1 Access Networks

1. Access network —the network that physically connects an end system to the first router(edge router)
2. DSL: digital subscriber line
3. Cable Internet access
4. Fiber to the home

### 1.2.2 Physical Media

介绍了接入网络的几种形式，材料啥的，没啥兴趣读完了

## 1.3 The Network Core

Network core —the mash of packet switches and links that interconnects the Internet's end systems

### 1.3.1 Packet Switching

1. Messages: can contain anything designer wants
2. Packets: The source breaks long messages into smaller chunks of data known as packets
3. Packet switches
   1. router
   2. link-layer switches
4. Store-and-Forward Transmission: The packet switch must receive the entire packet before it can begin to transmit the first bit of the packet onto the outbound link
5. Packet loss —either the arriving packet or one of the already-queued packets will be dropped
6. Forwarding table: each router has a forwarding table that maps destination address to that router's outbound links
7. The end-to-end routing process is analogous to a car driver who does not use maps but instead prefers to ask for directions

### 1.3.2 Circuit Switching

1. Traditional telephone networks are examples of circuit-switched networks
2. 多路复用，也没啥兴趣

### 1.3.3 A Network of Networks

1. The access ISPSs themselves must be interconnected. This is done by creating a *network of networks*—understanding this is the key to understanding the Internet
2. ![image-20220222105053131](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-interconnection-of-isps.png)
   1. 内容很多，循序渐进讲了模式的演变，但是最后这一张图就够了

## 1.4 Delay, Loss, and Throughput in Packet-Switched Networks

1. Throughput: the amount of data per second that can be tranferred

### 1.4.1 Overview of Delay in Packet-Switched Networks

1. Total delay
   1. nodal processing delay
   2. queuing delay
   3. transmission delay
   4. propagation delay
   5. ![image-20220222110552023](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-nodal-delay-at-routera.png)
2. 一些delay的具体介绍，其实感觉稍微有点面向0基础了，跳过一部分

### 1.4.2 Queuing Delay and Packet Loss

1. ![image-20220222112021614](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-queuing-delay-la-r.png)

   1. La/R: traffic intensity

   L: packets consist of L bits

   a: in units of packet/sec

   La:  the average rate at which bits arrive at the queue

   R: transmission rate

   花了很多篇幅来讲这个图，但其实图片一目了然，当然因为这个原理不是我所关注的重点

   

2. A packet can arrive to find a full queue. With no place to store such a packet, a router will **drop** that packet, the packet will be **lost**

### 1.4.3 End-to-End Delay

让跑一个程序感受一下delay，讲了讲终端系统可能带来的各方面延迟

### 1.4.4 Throughput in Computer Networks

取决于bottleneck link

## 1.5 Protocol Layers and Their Service Models

### 1.5.1 Layered Architecture

1. 用坐飞机来类比引入分层模型，其实依然是模块化的思想，水平、垂直、分级的思想(From SICP)，还看到有比如黑盒的特征，各层级之间解耦合
2. drawback
   1. one layer may duplicate lower-layer functionality 
   2. functionality at one layer may need information that is present only in another layer

3. Application Layer
   1. where network applications and their application-layer protocols reside

4. Transport Layer
   1. transports application-layer messages between application endpoints

5. Network Layer
   1. responsible for moving network-layer packets known as *datagrams* from one host to another
   2. simply referred to as the IP layer, reflecting the fact that IP is the glue that binds the Internet together

6. Link Layer
   1. to move a packet from one node to the next node in the route
   2. we'll refer to the link-layer packets as *frames*

7. Physical Layer
   1. to move the individual bits within the frame from one node to the next

8. The OSI Model
   1. presentation layer: to provide services that allow communicating applications to interpret the meaning of data exchanged
   2. session layer: provides for delimiting and synchronization of data exchange, including the means to build a checkpointing and recovery scheme

9. OSI多出来的两层是否需要取决于application和developer，如果应用需要，那么开发者应当构建相关的服务

### 1.5.2 Encapsulation

1. The Internet architecture puts much of its complexity at the edges of the network
2. payload: the payload is typically a packet from the layer above
3. Message在经过每一层的时候都会经历一次封装

## 1.6 Networks under Attack

## 1.7 History of Computer Networking and the Internet

## 1.8 Summary

 想快点到具体每一层了，跳过

# Chapter 2 Application Layer

1. Network applications are the raisons d’être of a computer network

## 2.1 Principles of Network Applications

Confining application software to the end systems has facilitated the rapid development and deployment of a vast array of network applications

### 2.1.1 Network Application Architectures

cs架构和P2P架构

### 2.1.2 Processes Communicating

1. It is not actually programs but processes that communicate
2. Processes on two different end systems communicate with each other by exchanging messages across the computer network
3. ![image-20220224101451024](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-process-socket-network.png)
   1. A socket is the interface between the application layer and the transport layer within a host
4. IP + port –> address

### 2.1.3 Transport Services Available to Applications

1. Reliable Data Transfer
2. Throughput
3. Timing
4. Security

### 2.1.4 Transports Services Provided by the Internet

1. TCP Services
   1. Connection-oriented service
   2. Reliable data transfer service
2. UDP Services
   1. Connectionless
   2. Unreliable data transfer service

### 2.1.5 Application-Layer Protocols

1. An application-layer protocol defines: 
   1. The types of messages exchanged, for example, request messages and response messages
   2. The syntax of the various message types, such as the fields in the message and how the fields are delineated
   3. The semantics of the fields, that is, the meaning of the information in the fields
   4. Rules for determining when and how a process sends messages and responds to messages
2. To distinguish between network applications and application-layer protocols
   1. An application-layer protocol is only one piece of network application(but a very important one)

### 2.1.6 Network Application Covered in This Book

## 2.2 The Web and HTTP

### 2.2.1 Overview of HTTP

简单讲HTTP的cs流程，比较基础

### 2.2.2 Non-Persistent and Persistent Connections

介绍non-persistent和persistent连接的区别，实际上就是长连接(连接复用的概念)

### 2.2.3 HTTP Message Format

简单讲request和response的结构

### 2.2.4 User-Server Interaction: Cookies

Cookies: allow sites to keep track of users

Cookie technology has four components:

1. A cookie header line in the HTTP response
2. A cookie header line in the HTTP request message
3. A cookie file kept on the user's end system and managed by the user's browser
4. A back-end database at the Web site

![image-20220224201643920](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-cookkie.png)

### 2.2.5 Web Caching

1. Web cache == proxy server
2. Has its own disk storage and keeps copies of recently requested objects in this storage
3. A cache is both a server and a client at the same time
4. Two reasons to deployment web cache in the Internet:
   1. reduce the response time for a client request
   2. reduce traffic on an institution's access link to the Internet
5. Conditional GET: allows a cache to verify that its objects are up to date
   1. GET + If-Modified-Since header

## 2.3 Electronic Mail in the Internet

### 2.3.1 SMTP

mail server to mail server

### 2.3.2 Comparison with HTTP

1. HTTP is mainly a pull protocol, SMTP is primarily a push protocol
2. SMTP requires each message to be in 7-bit ASCII format
3. HTTP encapsulates each object in its own HTTP response message, SMTP places all of the message's objects into one message

### 2.3.3 Mail Message Formats

### 2.3.4 Mail Access Protocol

![image-20220307095333801](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-email-access-protocol.png)

简单介绍了POP3，IMAP

## 2.4 DNS—The Internet's Directory Service

Internet identifier:

1. hostname
2. IP address

### 2.4.1 Services Provided by DNS

1. DNS: domain name system
   1. A directory service that translates hostnames to IP address
   2. A distributed database implemented in a hierarchy of DNS servers
   3. An application-layer protocol(runs over UDP and use port 53) that allows hosts to query the distributed database
2. When a browser running on some user's host, requests the URL
   1. User machine runs the client side of the DNS application
   2. browser extracts the hostname, from the URL and passes the hostname to the client side of the DNS application
   3. DNS client sends a query containing the hostname to a DNS server
   4. DNS client eventually receives a reply, which includes the IP address for the hostname
   5. Browser receives the IP address from DNS, it can initiate a TCP connection to the HTTP server process located at port 80 at the IP address
3. The desired IP address is often cached in a nearby DNS server, which helps to reduce DNS network traffic as well as the average DNS delay
4. DNS provides a few other important services
   1. Host aliasing
   2. Mail server aliasing
   3. Load distribution
5. see more in RFC1034 and RFC1035

### 2.4.2 Overview of How DNS Works

![image-20220307184132274](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-dns-servers.png)

![image-20220307185602672](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-interaction-of-dns-server.png)

![image-20220307185843964](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-recursive-queries-in-dns.png)

还是看图把

### 2.4.3 DNS Records and Messages

(Name,Value,Type,TTL)

DNS报文，依然不是目前想要关注和了解的，跳过

## 2.5 Peer-to-Peer File Distribution

1. In P2P file distribution, each peer can redistribute any portion of the file it has received to any other peers, thereby assisting the server in the distribution process
2. ![image-20220308145016311](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/cn-distribution-time-for-p2p-and-cs.png)
3. BitTorrent的简单机制介绍，简单阅读了一下，作了解。竞争的激励机制还是挺有意思的

## 2.6 Video Streaming and Content Distribution Networks

We will see they are implemented using application-level protocols and servers that function in some ways like a cache

### 2.6.1 Internet Video

1. A video is a sequence of images, typically being displayed at a constant rate
2. In order to provide continuous playout, the network must provide an average throughput to the streaming application that is at least as large as the bit rate of the compressed video

### 2.6.2 HTTP Streaming and DASH

1. 客户端定时抓取从HTTP响应中返回的视频帧(在客户端的缓存中)，在ua上面播放
2. DASH: Dynamic Adaptive Streaming over HTTP
   1. The video is encoded into several different versions, with each version having a different bit rate
   2. The client dynamically requests chunks of video segments of a few seconds in length 
3. DASH allows the client to freely switch among quality levels

### 2.6.3 Content Distribution Networks

1. CDN: A CDN manages servers in multiple geographically distributed locations, stores copies of the videos in its servers, and attempts to direct each user request to a CDN location that will provide the best user experience
2. CDN typically adopt one of two different server placement philisophies
   1. Enter Deep: To enter deep into the access networks of Internet Service Providers, by deploying server clusters in access ISPs all over the world
   2. Bring Home: To bring the ISPs home by building large clusters at a smaller number of sites, typically place their clusters in Internet Exchange Points

### 2.6.4 Case studying

## 2.7 Socket Programming: Creating Network Applications

# Chapter 3 Transport Layer

extending the network layer's delivery service between two end systems to a delivery service between two application-layer processes running on the end systems

## 3.1 Introduction and Transport-Layer Services

1. A transport-layer protocol provides for *logical communication* between application processes running on different hosts
   1. logical communication: 相对于 physical communication来说，即不考虑物理设施上的细节
2. transport-layer protocol implemented in the end systems but not in network routers

### 3.1.1 Relationship Between Transport and Network Layers

举例讲transport layer和network layer的区别：

1. 传输层提供logical-communication between processes running on different hosts
2. network layer提供logic-communication between hsots

### 3.1.2 Overview of the Transport Layer in the Internet

概括讲TCP和UDP的基础和接下来的讲解思路

## 3.2 Multiplexing and Demultiplexing

这里对multiplexing和demultiplexing的定义有点不好理解：

extending the host-to-host delivery service provided by the network layer to a process-to-process delivery service for applications running on the hosts

从source发出去的包，在经过socket的时候先被封装了不同的字段，再被交给network layer，此为multiplexing；demultiplexing则反过来，在destination接收的包，拆封字段以后送到正确的socket

TCP和UDP的multiplexing和demultiplexing的区别

## 3.3 Connectionless Transport: UDP

Reasons for UDP

1. Finer application-level control over what data is sent, and when
2. No connection establishment
3. No connection state
4. Small packet header overhead

### 3.3.1 UDP Segment Structure

UDP包的结构

### 3.3.2 UDP Checksum

checksum is used to determine whether bits within the UDP segment have been altered

checksum的计算

在发送端source port + destination port + length 三个16位字段相加后取反码 –> checksum

在接受端：将4个16位字段都相加，如果没有错误发生应该得到结果`1111111111111111`

why checksum?

讲到一个end-end principle in system design

functions placed at the lower levels may be redundant or of little value when compared to the cost of providing them at the higher level

## 3.4 Principles of Reliable Data Transfer

想一步步讲Reliable Data Transfer Protocol建立过程，图不太好看懂，有点受不了

### 3.4.1 Building a Reliable Data Transfer Protocol

## 3.4.2 Pipelined Reliable Data Transfer Protocol

### 3.4.3 Go-Back-N(GBN)

## 3.5 Connection-Oriented Transport: TCP

the Internet's transport-layer, connection-oriented, reliable transport protocol

### 3.5.1 The TCP Connection

中间网元对所谓的TCP连接是无感知的

point-to-point one sender-one receiver

### 3.5.2 TCP Segment Structure

1. Sequence Numbers and Acknowledgment Numbers





# Chapter 4 The Network Layer: Data Plane

# Chapter 5  The Network Layer: Control Plane

# Chapter 6 The Link Layer and LANs

# Chapter 7 Wireless and Mobile Networks

# Chapter 8 Security in Computer Networks

# Chapter 9 Multimedia Networking



