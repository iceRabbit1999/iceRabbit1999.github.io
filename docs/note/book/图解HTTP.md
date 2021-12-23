*图解HTTP*

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

1. 发送端从应用层往下走，接收端往应用层往上走

