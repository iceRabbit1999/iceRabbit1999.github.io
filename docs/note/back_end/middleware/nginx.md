#  概述

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



# 命令

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

