# CSCF

1. Call Session Control Function
   1. P-CSCF:Proxy-CSCF
      1. 是IMS中与用户的第一个连接点
      2. 提供代理(Proxy)功能，接受业务请求并转发它们
      3. 也可提供用户代理（UA）功能，即在异常情况下中断和独立产生SIP会话
   2. S-CSCF(Serving CSCF)
      1. 在IMS核心网中处于核心的控制地位
      2. 负责对UE的注册鉴权和会话控制
      3. 执行针对主叫端及被叫端IMS用户的基本会话路由功能
      4. 进行到AS的增值业务路由触发及业务控制交互
   3. I-CSCF(Interrogating CSCF)
      1. 类似IMS的关口节点
      2. 提供本域用户服务节点分配、路由查询以及IMS域间拓朴隐藏功能
2. P/S/I-CSCF在物理实体上完全可以是合一的
3. 实际组网时，其划分和部署需综合考虑对IMS业务接入方式、CSCF的容量、能力及用户业务量需求等因素.

# HSS

1. HSS:The Home Subscribe Server
2. 位于IMS核心网络架构的最顶层
3. HSS是归属网络中保存IMS用户的签约信息
   1. 包括基本标识、路由信息以及业务签约信息等集中综合数据库
      1. –IMS用户标识（包括公共及私有标识）、号码和地址信息
      2. –IMS用户安全上下文：用户网络接入认证的密钥信息
      3. –IMS用户的路由信息：HSS支持用户的注册，并且存储用户的位置信息
      4. –IMS用户的业务签约信息：包括其他AS的增值业务数据

# AS

1. AS(Application Server)
   1. SIP AS
   2. OSA AS
      1. 通过OSA Service Capability Servers而不是直接与IMS网元交互
   3. IM-SSF
      1. 通过OSA Service Capability Servers而不是直接与IMS网元交互
2. 为IMS用户提供增值业务
3. 可以位于用户归属网，也可以由第三方提供
4. AS通过与HSS的接口 获得用户业务相关的数据和用户状态信息
5. S-CSCF与AS间的ISC接口用于AS进行相应的业务控制

# SLF

1. SLF(Subscription Locator Function )：在运营商内设置多个HSS的情况下，I-CSCF在登记注册及事务建立过程中通过SLF获得用户签约数据所在的HSS域名，可与HSS合设

# MGCF

1. Media Gateway Control Function
2. 实现IMS核心控制面与PSTN或PLMN CS的交互，支持ISUP/BICC与SIP的协议交互及呼叫互通
3. 完成PSTN或CS TDM承载与IMS域用户面RTP的实时转换

# IM-MGW

1. IMS-Media Gateway Function
2. 完成IMS与PSTN及CS域用户面宽窄带承载互通及必要的Codec编解码变换

# BGCF

1. Breakout Gateway Control Function
2. 根据互通规则配置或被叫分析，为IMS到PSTN/CS的呼叫选择MGCF，从而实现MGCF路由的自动获取

# MRFC

1. Multimedia Resource Function Control
2. 控制MRFP上的媒体资源，解析来自其他S-CSCF及AS的SIP资源控制命令，转换为对MRFP的对应控制命令并产生相应计费信息

# MRFP

1. Multmedia Resource Function Processor
2. 作为网络公共资源，在MRFC控制下提供资源服务
   1. 包括媒体流混合（多方会议）、多媒体信息播放（放音、流媒体）、媒体内容解析处理（码变换、语音识别等）

# DNS、ENUM SERVER

1. DNS(Domain Name System)服务器负责URL地址到IP地址的解析
2. ENUM(E.164 Number URI Mapping)服务器负责电话号码到URL的转换，一般需IMS运营商新建。

# DHCP SERVER

1. 在标准DHCP(Dynamic Host Configuration Protocol )服务功能的基础上，增加在动态分配IP地址过程中向IMS终端指定P-CSCF的URL地址的处理。