# *Commend*



# Java

1. 后台启动运行且把日志写到指定输出文件 `nohup java -jar xx.jar >xxx.log &`

# MySQL

1. 创建数据库并指定编码utf8 `create database xxx character set utf8 collate utf8_general_ci`







# WSL

1. wsl没有systemctl指令![image-20211206112945640](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/wsl_service.png)
1. WSL开启ssh (Ubuntu):`sudo /etc/init.d/ssh start`
1. 利用WSL2作为测试环境在局域网访问
   1. [链接参考](https://blog.csdn.net/cf313995/article/details/108871531)

   1. `netsh interface portproxy add v4tov4 listenaddress=192.168.92.163 listenport=9426 connectaddress=172.30.135.225 connectport=9426`
      1. 192.168.92.163:本机ip，172.30.135.225：wsl2 ip

   1. ` netsh interface portproxy show all`
      1. 查看所有端口转发信息

   1. `netsh interface portproxy delete v4tov4 listenport=9426 protocol=tcp`
      1. 删除端口9426的端口转发





# 防火墙

1. 开放防火墙端口`iptables -I INPUT -s 10.153.97.0/24 -p tcp --dport 9006 -j ACCEPT` `iptables-save`



# 配置文件 

1. 使配置文件生效:`source /etc/profile`
