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

要表示指定的URI，要使用涵盖全部必要信息的绝对URI、绝对URL以及相对URL

1. 绝对URI的格式

   ![image-20220126215344674](C:\Users\livingsnowman\AppData\Roaming\Typora\typora-user-images\image-20220126215344674.png)

   1. 协议方案名：也可使用data:或javascript:这类指定数据或脚本程序的方案名
   2. 登录信息：此项是可选项
   3. 服务器地址：使用绝对URI必须指定带访问的服务器地址
      1. 地址可以是类似hacker.jp这样DNS可解析的名称
      2. 也可以是192.168.1.1这类IPV4地址名
      3. 还可以是[0:0:0:0:0:0:1]这样用方括号括起来的IPV6地址名

# 第2章 简单的HTTP协议

##2.1 HTTP协议用于客户端和服务器端之间的通信

1. 在两台计算机之间使用HTTP协议通信时，在一条通信线路上必定有一端是客户端，另一端是服务端
2. 两台计算机的角色可能会互换，但从一条通信线路来说，C/S角色是确定的

## 2.2 通过请求和相应的交换达成通信

1. HTTP协议规定，通信一定是从客户端建立的，服务端没有收到请求之前不会发送响应

2. 请求报文示例

   ```http
   GET /index.htm HTTP/1.1
   Host: hackr.jp
   ```

   1. method后面的字符串/index.htm指明了请求访问的资源对象，也叫做**request-URI**
   2. 由请求方法、请求URI、协议版本、可选的请求首部字段和内容实体构成

3. 响应报文示例

   ```http
   HTTP/1.1 200 OK
   Date: Tue, 10 Jul 2012 ...
   Content-Length: 333
   Content-Type: text/html
   
   <html>
   ...
   ```

   

## 2.3 HTTP是不保存状态的协议

1. HTTP是一种sateless的协议。
   1. 在HTTP这个级别，协议对于发送过的请求或响应都不做持久化处理
2. 协议本身并不保留之前一切的请求或响应的报文信息
   1. 为了更快的处理大量事务，确保协议的可伸缩性
3. 随着Web技术发展，stateless的特性变的原来越棘手，于是有了cookie

## 2.4 请求URI定位资源

1. HTTP协议使用URI定位互联网上的资源
2. 客户端请求访问资源而发送请求时，URI需要将请求中的request-URI包含在内

## 2.5 告知服务器意图的HTTP方法

1. GET：获取资源
   1. 用来访问已被URI识别的资源。指定的资源经服务器解析后返回响应内容
2. POST：传输实体主体
   1. GET也可以传输实体的主体，POST的主要目的不是获取响应的主体内容
3. PUT：传输文件
   1. 像FTP协议的文件上传一样，要求在请求报文主体中包含文件内容，保存到指定URI位置
   2. HTTP/1.1的PUT自身不带验证机制，存在安全问题
4. HEAD：获得报文首部
   1. 和GET一样，只是不返回报文主体部分，用于确认URI的有效性及资源更新的日期时间等
5. DELETE：删除文件
   1. 与PUT相反的方法
   2. 在HTTP/1.1中同样不带安全验证机制
6. OPTIONS：询问支持的方法
   1. 查询针对request-URI指定的资源支持的方法
7. TRACE：追踪路径
   1. 让Web服务器将之前的请求通信环回给客户端的方法
   2. Max-Forwards逐跳递减，为0时停止传输
   3. 不常用且容易引发XST(Cross-Site Tracing)攻击
8. CONNECT：要求用隧道协议连接代理
   1. 要求在与代理服务器通信是建立隧道，实现用隧道协议进行TCP通信
      1. 主要使用SSL(Secure Scokets Layer)和TLS(Transport Layer Security)把通信内容加密后经过隧道传输

## 2.6 使用方法下达命令

方法的作用在于，可以指定请求的资源按期望产生某种行为

## 2.7 持久连接节省通信量

HTTP协议的初始版本中，每进行一次HTTP通信就要断开一次TCP连接

### 2.7.1 持久连接

HTTP Persistent Connections，也称为HTTP keep-alive或HTTP connection reuse

1. 持久连接的特点：只要任意一段没有明确提出断开连接，则保持TCP连接状态
2. 好处在于：减少了TCP连接的重复建立和断开造成的开销，减轻服务器的负载

### 2.7.2 管线化

持久连接使得多数请求以管线化(piplining)方式发送成为可能。即不用等待响应亦可直接发送下一个请求，能做到同时并发多个请求

## 2.8 使用Cookie的状态管理

1. Cookie技术通过在请求和响应报文中写入Cookie信息来控制客户端的状态
2. Cookie根据响应报文中的set-cookie字段信息通知客户端保存Cookie，当下次客户端再次往该服务器发送请求时，会自动在请求中加入Cookie值
3. 服务器接收到含Cookie值的请求报文后，会检查是从哪一个客户端发来的连接请求，对比服务器上的记录，得到之前的状态信息

# 第3章 HTTP报文内的HTTP信息

## 3.1 HTTP报文

1. 大致可分为报文首部和报文主体，通常不一定要有主体

## 3.2 请求报文及响应报文的结构

1. 请求行：包含用于请求的方法，request-URI，HTTP版本
2. 状态行：包含表明响应结果的状态码，原因缩语，HTTP版本
3. 首部字段：各类首部
   1. 通用首部、请求首部、响应首部、实体首部
4. 其他：Cookie等

## 3.3 编码提升传输速率

通过在传输时编码，能有效的处理大量的访问请求，但会消耗更多的CPU资源

### 3.3.1 报文主体和实体主体的差异

1. 报文 message：是HTTP通信的基本单位，由8位组字节流(octet sequence)组成，通过HTTP通信传输
2. 实体 entity：作为请求或响应的有效负载荷数据被传输，其内容由实体首部和实体主体组成
3. HTTP报文的主体用于传输请求或响应的实体主体
4. 通常，报文实体等于实体主体。只有当传输中进行编码操作时，实体主体的内容发生变化才出现差异

### 3.3.2 压缩传输的内容编码

内容编码指明应用在实体内容上的编码格式，并保持实体信息原样压缩。内容编码后的实体由客户端接收并负责解码

常用的内容编码：

1. gzip
2. compress
3. deflate
4. identity

### 3.3.3 分割发送的分块传输编码

分块传输编码 Chunked Transfer Coding：把实体主体分块的功能。能做到让浏览器逐步显示页面	

1. 将实体主体分成多个部分，每一块都会用十六进制标记块的大小，最后一块会用"0(CR+LF)"标记
2. 接收的客户端负责解码，恢复到编码前的实体主体

## 3.4 发送多种数据的多部分对象集合

发送的一份报文主体可包含多种类型实体

1. multipart/form-data：表单
2. multipart/byteranges
3. HTTP报文中使用多部分对象集合时需要加上Content-Type
4. 多部分对象集合的每个部分类型中都可以有首部字段

## 3.5 获取部分内容的范围请求

1. 为了实现可恢复机制(能从之前下载中断处恢复下载)
   1. 需要制定下载的实体范围——范围请求(Range Request)
   2. 使用Range字段来指定资源的byte范围
   3. 针对范围请求，会返回206(Partial Content),Content-Type会标明multipart/byteranges
   4. 若无法响应范围请求，会返回200

## 3.6 内容协商返回最合适的内容

1. 当浏览器的默认语言为英文/中文时，访问相同的URI的Web页面会显示对应的语言页面，这样的机制成为内容协商(Content Negotiation)
2. 内容协商机制是指客户端和服务器端就相应的资源内容进行交涉，然后提供给客户端最合适的资源。会以语言、字符集、编码方式等作为判断基准
3. Accept、Accept-Charset...

# 第4章 返回结果的HTTP状态码

## 4.1 状态码告知从服务器端返回的请求结果

1. 状态码的职责：当客户端向服务端发送请求时描述返回的结果

## 4.2 2XX 成功

1. 200 OK
2. 204 No Content 成功处理请求但返回报文中没有实体主体
3. 206 Partial Content 成功执行范围请求

## 4.3 3XX 重定向

1. 301 Moved Permanently 永久重定向，请求的资源已经被分配了新的URI
2. 302 Found 临时重定向，希望本次使用新的URI访问
3. 303 See Other 请求资源存在另一个URI，应该使用GET定向获取请求资源
4. 304 Not Modified

## 4.4 4XX 客户端错误

## 4.5 5XX 服务端错误

# 第5章 与HTTP协作的Web服务器

## 5.1 用单台虚拟主机实现多个域名





