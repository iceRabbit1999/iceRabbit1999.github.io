# *Nginx*

# 概述

## 简介

1. Nginx：轻量级、高性能、基于Http、反向代理服务器、静态web服务器
2. 正向代理：![image-20211202212948903](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/nginx_正向代理.png)
   1. 隐藏、翻墙、提速、缓存、授权
3. 反向代理：![image-20211202213116054](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/nginx_反向代理.png)
   1. 保护隐藏、分布式路由、负载均衡、动静分离、数据缓存

## 安装

1. 安装gcc `yum -y install gcc gcc-c++`
2. 安装依赖库 `yum -y install pcre-devel openssl-devel`
3. 下载上传nginx
4. 解压nginx `tar -zxvf nginx-... .tar.gz`
5. 生成makefile `make`
6. 编译安装 `make && make install`
7. 使nginx命令随处可用 etc/profile加入`export NGINX_HOME=/usr/local/nginx
   export PATH=P A T H : PATH:PATH:NGINX_HOME/sbin
   `



## 命令

1. `nginx -v`
2. 测试配置文件命令
   1. `nginx -t`：测试配置文件是否正确(默认conf/nginx.conf)
   2. `nginx -T`: 测试并显示配置文件内容
   3. `nginx -tq`：测试过程中只显示错误信息
3. 停止命令
   1. `nginx -s stop`：强制停止，无论当前进程是否正在处理工作
   2. `nginx -s quit`：优雅停止
4. 重启
   1. `nginx -s reload`：在不重启nginx的前提下重新加载配置文件——平滑重启
5. 日志
   1. `nginx -s reopen`
6. 存放路径
   1. `nginx -p`：指定配置文件的存放路径
7. 启动
   1. `nginx -c`：不指定配置文件则默认conf/nginx.conf

# 核心配置

## 性能调优

###概念

1. 零拷贝
   1. 定义：从一个存储区域到另一个存储区域的copy任务没有CPU参与
   2. 目的：减少CPU消耗和内存带宽占用，减少用户上下文与CPU内核上下文间的切
2. 多路复用器 select|poll|epoll![image-20211203004052467](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/nginx_多路复用器.png)
   1. select,poll,epoll是常见的三种多路复用器算法
      1. selsect：轮询所有进程
      2. poll：同selsect但实现不同(就绪队列为链表)
      3. epoll：回调方式

### 并发处理

1. 多进程、多线程、异步非阻塞
2. Nginx进程分为
   1. worker
      1. 可以同时处理多个用户请求
      2. 采用epoll多路复用机制处理后端服务器
   2. master
      1. 可以产生多个worker
3. 异步非阻塞实现：
   1. 后端服务器返回结果
   2. 回调epoll多路复用器
   3. 相应的worker进程进行通知(worker会挂起当前正在处理的事务，拿IO返回结果去响应客户端请求)
   4. 响应完毕后继续执行挂起的事务

### 全局调优

1. worker_processes
   1. 默认值1，指定nginx工作进程数量，一般设置为**CPU内核数量，或内核数量的整数倍**，可以指定为auto
2. worker_cpu_affinity
   1. 将worker进程与具体的内核进行绑定
   2. 是通过二进制进行的。每个内核使用一个二进制位表示，0代表内核关闭，1代表内核开启
3. worker_rlimit_nofile
   1. 一个worker进程所能打开的最多文件数量
   2. 默认值与当前Linux系统可以打开的最大文件描述符数量相同

### events模块调优

1. worker_connectons
   1. 每一个worker进程可以并发处理的最大连接数，不能超过worker_rlimit_nofile
2. accpect_mutex
   1. 默认值on:，表示当一个新连接到达时，那些没有处于工作状态的worker将以串行方式来处理
   2. off:新连接到达时，所有worker都会被唤醒，但只有一个会获得连接，其他worker进入阻塞，即”惊群“现象
3. accept_mutex_delay
   1. 队首worker会尝试获取互斥锁的时间间隔，默认500ms
4. multi_accep
   1. on：实时的统计出各个worker当前正在处理的连接个数，会按照“缺编”最多的worker的“缺编”数量，一次性将这么多的新连接分配给该worker
   2. off：逐个拿出新连接按照负载均衡策略，将其分配给当前处理连接个数最少的
5. use 
   1. 设置worker与客户端连接的处理方式。Nginx会自动选择适合当前系统的最高效的方式
   2. select | poll | epoll | rtsig | kqueue | /dev/poll

### http模块调优

1. 非调优属性
   1. include mine.types：将当前目录(conf目录)中的mime.types文件包含进来
   2. default_type application/octet-stream：对于无扩展名的文件，默认其为application/octet-stream类型
   3. charset utf-8：设置请求与响应的字符编码
2. 调优属性
   1. sendfile
      1. on：设置为on则开启Linux系统的零拷贝机制，否则不启用零拷贝
   2. tcp_nopush
      1. on：：以单独的数据包形式发送Nginx的响应头信息，而真正的响应体数据会再以数据包的形式发送
      2. off：默认值，响应头信息包含在每一个响应体数据包中
   3. tcp_nodelay
      1. on：不设置数据发送缓存，即不推迟发送，适合于传输小数据，无需缓存。
      2. off：开启发送缓存。若传输的数据是图片等大数据量文件，则建议设置为off
   4. keepalive_timeout
      1. 设置客户端与Nginx间所建立的长连接的生命超时时间，时间到达，则连接将自动关闭。单位秒。
   5. keepalive_requests 
      1. 设置一个长连接最多可以发送的请求数。该值需要在真实环境下测试
   6. client_body_timeout
      1. 客户端获取Nginx响应的超时时限

## 请求定位

### 资源访问

1. 修改配置文件

   ```
   location /xxx/ooo{
   	root /opt/aaa
   	index myfile.txt
   }
   ```

   

2. 创建目录及文件

   1. /opt/aaa/xxx/ooo/myfile.txt

### 路由匹配优先级

1. 优先级规则从高到低：普通匹配 < 长路径匹配 < 正则匹配 < 短路匹配 < 精确匹配

2. 普通匹配

   1. 只要请求以/xxx开头就可以命中，返回400

   ```perl
   location /xxx{
   	return 400;
   }
   ```

3. 长路径匹配

   1. 请求既可以匹配长路径又可以匹配段路径，优先长路径，返回402

      ```perl
      location /xxx{
      	return 400;
      }
      location /xxx/ooo{
      	return 402;
      }
      ```

4. 正则匹配

   1. 正则匹配与普通匹配（长路径匹配也属于普通匹配）均可匹配上时，正则匹配的优先级高，返回401

   2. 区分大小写，

      ```perl
      location /xxx{
      	return 400;
      }
      location /xxx/ooo{
      	return 402;
      }
      location ~/xxx{ # ~表示正则表达式，默认区分大小写
      	return 401;
      }
      ```

   3. 不区分大小写，`~`后加上``*``

5. 短路匹配

   1. 以`^~`开头的匹配路径称为短路匹配，表示只要匹配上就不再匹配其他，即使是正则匹配，返回403

      ```perl
      lcoation /xxx{
      	return 400;
      }
      location ~/xxx{
      	return 401;
      }
      lcoation ^~/xxx/ooo{
      	return 403
      }
      ```

6. 精确匹配

   1. 以`=`开头的匹配，最高优先级，返回405

      ```perl
      location /xxx{
      	return 400;
      }
      location ~/xxx{
      	return 401;
      }
      lcoation ^~/xxx/ooo{
      	return 403;
      }
      lcoation =/xxx/ooo{
      	return 405;
      }
      ```

### 缓存配置



+ 可以对请求的response进行缓存，起到类似CDN的作用
+ 后台服务器挂掉的时候可以返回托底数据，实现服务降级
+ 缓存功能由两部分构成：全局定义和局部定义

1. http{}模块的缓存全局定义
   1. proxy_cache_path
      1. 用于指定Nginx缓存的存放路径及相关配置
   2. proxy_temp_path
      1. 指定Nginx缓存的临时存放目录
2. location{}模块的缓存局部定义
   1. proxy_cache 
      1. 指定用于存放缓存key内存区域名称,其值为http{}模块中proxy_cache_path中的keys_zone的值
   2. proxy_cache_key
      1. 指定Nginx生成的缓存的key的组成
   3. proxy_cache_bypass
      1. 指定是否越过缓存
   4. proxu_cache_methods
      1. 指定客户端请求的哪些提交方法将被缓存，默认为GET与HEAD，但不缓存POST
   5. proxy_no_cache
      1. 指定对本次请求是否不做缓存。只要有一个不为0，就不对该请求结果缓存
   6. proxy_cache_purge
      1. 指定是否清除缓存
   7. proxy_cache_lock
      1. 指定是否采用互斥方式回源
   8. proxy_cache_lock_timeout
      1. 指定再次生成回源互斥锁的时限
   9. proxy_cache_valid
      1. 对指定的 HTTP 状态码的响应数据进行缓存，并指定缓存时间。默认指定的状态码为200，301，302
   10. proxy_cache_use_stale
       1. 设置启用托底缓存的条件
       2. 一旦这里指定了相应的状态码，则前面proxy_cache_calid中指定的相应状态码所生成的缓存就变为了托底缓存
   11. expires
       1. 为请求的静态资源开启浏览器端的缓存
3. Nginx变量
   1. 自定义变量：`set $aaa "hello"` `set $bbb 0`
   2. 内置变量![image-20211203173429811](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/nginx_内置变量1.png)

![image-20211203173456767](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/image-20211203173456767.png)





## 日志管理和自动切割

### 日志管理范围

日志记录的请求范围是不同的。Nginx日志一般可以指定三个范围：http{}模块范围、server{}模块范围，与location{}模块范围

1. http{}模块范围

   1. 只要有请求通过http协议访问该Nginx，就会有日志信息写入

      ```perl
      http{
      	log_format main ''
      	access_log 'xxx'
      	error_log 'xxx'
      	open_log_file_cache 'xxx'
      }
      ```

2. server{}模块范围

   1. 只要有请求访问当前Sever，就会有日志信息写入

      ```perl
      sever{
      	listen 80;
      	server_name localhost;
      	access_log logs/host.access.log main;
      	error_log logs/host.error.log;
      }
      ```

3. location{}模块范围

   1. 只要有请求访问当前location，就会有日志信息写入

      ```perl
      location / {
      	root html;
      	index index.html index.htm
      	
      	access_log logs/host.access.log main;
      	error_log logs/host.error.log;
      }
      ```

### 日志管理指令

+ Nginx的日志分为两类：访问日志与错误日志
+ 默认路径与名称在nginx.conf中均是可以修改
+ 配置文件中不仅定义了日志文件的路径及名称，还定义了日志格式

1. log_format main ''
   1. 设置访问日志的格式
   2. 其后的main是为该格式所起的名称，可以任意，而其后面的内容则为具体格式，通过Nginx内置变量定义
      1. $remote_addr：获取访问者的IP地址
      2. $http_x_forwarded_for：获取客户端浏览器的IP
      3. $remote_user：获取访问者的用户名
      4. $time_local：获取请求访问的时间与时区
      5. $request：获取请求的相关信息，包含请求方式、请求的URI，及访问协议
      6. $status：后端服务器向其返回的状态码，例如200。
      7. $body_bytes_sent：后端服务器向客户端发送的响应体内容字节数
      8. $http_referer：获取当前请求是从哪个页面过来的。其值在这里显示为杠(-)。
      9. $http_user_agent：用户所使用的代理，一般为浏览器。
2. access_log
   1. `access_log logs/access.log main buffer=64k`
   2. 设置访问日志。上面的格式包含三个参数
      1. 第一个参数是日志的存放路径与日志文件
      2. 第二个参数是日志格式名
      3. 第三个参数是日志文件所使用的缓存
3. error_log
   1. 指定错误日志的路径与文件名
   2. 不能指定格式，因为其有默认格式
   3. 关于错误日志，一般使用默认
   4. 错误日志级别由低到高有：[debug | info | notice | warn | error | crit | alert | emerg]，级别越高记录的信息越少。
   5. 默认开启
4. open_log_file_cache
   1. 打开日志文件读缓存，将日志信息读取到缓存中，以加快对日志的访问
   2. 默认off

### 默认的/favicon.ico请求

+ 客户端对于服务端页面会自动提交一个/favicon.ico请求，若没有favicon.ico文件则会在日志文件中报出404

### 日志自动切割

1. 创建shell脚本文件
2. 授权
3. 向crontab中添加定时任务

## 静态代理

将所有的静态资源，例如，css、js、html、jpg等资源存放到Nginx服务器，而不存放在应用服务器Tomcat

### 扩展名拦截

### 目录名拦截

### 页面压缩

## 反向代理

### 反向代理tomcat服务器

### 反向代理属性设置

## 负载均衡

### 分类

### 实现

### 策略

### Nginx Plux四层负载均衡实现

## 动静分离

