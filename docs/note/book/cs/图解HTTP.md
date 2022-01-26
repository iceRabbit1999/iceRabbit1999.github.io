#*图解HTTP*

# 第1章 了解Web及网络基础

## 1.1 使用HTTP协议访问web

1. HTTP通常被译为超文本传输协议，但更严谨的译名应该是**超文本转移协议**
2. Web是建立在HTTP协议上通信的

## 1.2 HTTP的诞生

### 1.2.1 为知识共享而规划Web

1. 最初设想的基本理念：**借助多文档之间相互关联形成的超文本**(HyperText)，连成可相互参阅的WWW(World Wide Web，万维网)
2. 现在已提出的3项WWW构建技术
   1. SGML(Standed Generalized Markup Language)作为页面的文本标记语言的HTML(HyperText Markup Language)
   2. 作为文档传递协议的HTTP
   3. 指定文档所在地址的URL(Uniform Resource Locator，统一资源定位符)

### 1.2.2 Web成长时代

1. HTML1.0
   1. 存在多出模糊不清的部分，被直接废弃
2. Apache0.2、HTML2.0
   1. Web技术的突飞猛进
3. 微软与网景
   1. 两家通信公司对HTML的不同扩展至今依旧另前端工程师感到棘手
4. Firefox
   1. 第二次浏览器大战

### 1.2.3 驻足不前的HTTP

1. HTTP/0.9
   1. 问世，并没有被作为标准建立
2. HTTP/1.0
   1. 被记载于RFC1945，该协议至今仍被广泛使用
3. HTTP/1.1
   1. 目前主流的HTTP版本，先被修订为RFC2616
4. HTTP/2.0

## 1.3 网络基础 TCP/IP

1. 通常使用的网络是在TCP/IP协议族的基础上运作的，HTTP属于它内部的一个子集

### 1.3.1 TCP/IP协议族

1. 计算机与网络设备要相互通信，双方就要基于相同的方法(规则 -> 协议)

2. FTP,IP,DNS,TCP,UDP,FDDI,HTTP,SNMP,ICMP...

   协议中存在各式各样的内容，把互联网相关联的协议集合起来总称为TCP/IP

### 1.3.2 TCP/IP的分层管理

1. 按层次分为(解耦)
   1. 应用层
      1. **决定了向用户提供应用服务时通信的活动**
      2. TCP/IP预存了各类通用的服务：FTP(File Transfer Protocol)、DNS(Domain Name System)
      3. **HTTP处于该层**
   2. 传输层
      1. 对上层应用层提供处于网络连接中的两台计算机之间的**数据传输**
      2. 有两个不同性质的协议
         1. TCP(Transmission Control Protocol)
         2. UDP(User Data Protocol)
   3. 网络层
      1. 用来处理在网络上流动的**数据包(**网络传输的最小数据单位)
      2. 规定了通过怎样的**路径**(传输路线)到达对方计算机，并把数据包传送给对方
   4. 数据链路层
      1. 用来处理连接网络的**硬件部分**
         1. 操作系统
         2. 硬件设备驱动
         3. NIC(Network Interface Card)
         4. 网络适配器
         5. 光纤
         6. ...

### 1.3.3 TCP/IP通信传输流

![image-20211222235935771](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/http_tcp_ip_通信传输流.png)

发送端从应用层往下走，接收端往应用层往上走。用HTTP来举例

1. 客户端在应用层(HTTP协议)发出想看某个Web界面的HTTP请求
2. 传输层TCP把从应用层接收到的HTTP报文分割，并在各个报文上打上标记序号及端口号后转发给网络层
3. 在网络层IP协议，增加作为通信目的地的MAC地址后转发给链路层

至此，发往网络的通信请求准备齐全

Encapsulate：发送端在层与层之间传输数据时，每经过一层必定会被打上一个该层所属的首部信息

![image-20220126150713603](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/http-encapsulate.png)

## 1.4 与HTTP关系密切的协议：IP、TCP和DNS

### 1.4.1 负责传输的IP协议

1. IP协议的作用是把各种数据包传送给对方，要保证确实传送到对方，需要满足两个重要条件：
   1. IP地址：指明了节点被分配到的地址
   2. MAC地址(Media Access Control Address)：指网卡所属的固定地址
   3. IP地址可以和MAC地址进行配对，IP地址可变换，MAC地址基本不会更改
2. IP间的通信依赖MAC地址，MAC地址使用ARP协议进行通信
   1. 通常情况下(除开局域网)经过多台计算机和网络设备中转才能连接到对方。中转时会利用下一站设别的MAC地址来搜索下一个中转目标，此时会采用ARP协议
   2. ARP(Address Resolution  Protocol)：一种用以解析地址的协议，根据通信方的IP地址就可以反查出对应的MAC地址
3. 路由选择(routing)
   1. 在到达通信目标前的中转过程中，那些计算机和路由器等网络折别只能获悉很粗略的传输路线
   2. 无论哪台计算机、哪台网络设备都无法全面掌握互联网中的细节

### 1.4.2 确保可靠性的TCP协议

1. 按层次分，TCP属于传输层，提供可靠的字节流服务
   1. 字节流服务(Byte Stream Service)：为了方便传输，将大块数据分割成以segment为单位的数据包进行管理
   2. 可靠传输服务：能够把数据准确可靠的传送给对方
2. 确保数据能到达目标
   1. 三次握手策略(three-way handshaking)：
      1. 首先发送一个带有SYN(synchronize)标志的数据给对方
      2. 接收端收到以后，回传一个带有SYN/ACK标志的数据包以示传达确认信息
      3. 发送端再回传一个带ACK(acknowledgement)标志的数据包，代表握手结束

## 1.5 负责域名解析的DNS服务

1. DNS(Domain Name System)：是和HTTP协议一样位于应用层的协议，它提供域名到IP地址之间的解析服务
2. 计算机既可以被赋予IP地址，也可以被赋予主机名和域名
   1. 用户通常使用主机或域名来访问计算机，但要让计算机理解名称，就显得较为困难。计算机显然更擅长处理一长串数字

![image-20220126152413169](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/http-dns.png)

## 1.6 各种协议与HTTP协议的关系

![image-20220126152537289](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/http-各协议与http关系.png)



## 1.7 URI和URL

### 1.7.1 统一资源标识符URI

Uniform Resource Identifier

RFC2396中的相关定义

1. Uniform

   1. > Uniformity provides several benefits: it allows different types of resource identifiers to be used in the same context, even when the mechanisms used to access those resources may differ; it allows uniform semantic interpretation of common syntactic  conventions across different types of resource identifiers; it  allows introduction of new types of resource identifiers  without interfering with the way that existing identifiers are  used; and, it allows the identifiers to be reused in many different contexts, thus permitting new applications or   protocols to leverage a pre-existing, large, and widely-used  set of resource identifiers.

2. Resource

   1. >A resource can be anything that has identity.  Familiar examples include an electronic document, an image, a service (e.g., "today's weather report for Los Angeles"), and a collection of other resources.  Not all resources are network "retrievable"; e.g., human beings, corporations, and bound books in a library can also be considered resources The resource is the conceptual mapping to an entity or set of entities, not necessarily the entity which corresponds to that  mapping at any particular instance in time.  Thus, a resource can remain constant even when its content---the entities to which it currently corresponds---changes over time, provided that the conceptual mapping is not changed in the process.

3. Identifer

   1. >An identifier is an object that can act as a reference to something that has identity.  In the case of URI, the object is a sequence of characters with a restricted syntax.

综上所述，URI就是由某个协议方案表示的资源的定位标识符，协议方案是指访问资源所使用的协议类型名称(HTTP、ftp、telnet、file...)

1. URI用字符串标识某一互联网资源，而URL表示资源的地点(互联网上所处的位置)，可见URL是URI的子集

### 1.7.2 URI格式

