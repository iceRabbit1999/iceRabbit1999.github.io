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



# Chapter 3 Transport Layer

# Chapter 4 The Network Layer: Data Plane

# Chapter 5  The Network Layer: Control Plane

# Chapter 6 The Link Layer and LANs

# Chapter 7 Wireless and Mobile Networks

# Chapter 8 Security in Computer Networks

# Chapter 9 Multimedia Networking



