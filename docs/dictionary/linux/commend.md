# *Commend*

# WSL

1. wsl没有systemctl指令![image-20211206112945640](https://gitee.com/iceRabbit1999/forimage/raw/master/blog/wsl_service.png)
1. WSL开启ssh (Ubuntu):`sudo /etc/init.d/ssh start`
1. 利用WSL2作为测试环境在局域网访问
   1. `netsh interface portproxy add v4tov4 listenaddress=192.168.92.163 listenport=9426 connectaddress=172.30.135.225 connectport=9426`
      1. 192.168.92.163:本机ip，172.30.135.225：wsl2 ip

   1. ` netsh interface portproxy show all`
      1. 查看所有端口转发信息

   1. `netsh interface portproxy delete v4tov4 listenport=9426 protocol=tcp`
      1. 删除端口9426的端口转发


# 配置文件 

1. 使配置文件生效:`source /etc/profile`

