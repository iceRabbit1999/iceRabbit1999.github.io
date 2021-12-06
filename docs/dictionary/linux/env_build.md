# *Environment building*

# docker

1. 卸载旧版本

   ```shell
   yum remove docker \
   docker-client \
   docker-client-latest \
   docker-common \
   docker-latest \
   docker-latest-logrotate \
   docker-logrotate \
   docker-engine
   ```

2. 安装yum工具包

   1. `yum install -y yum-utils`

3. 告诉Linux，Docker应该去哪里获取镜像安装

   ```shell
   sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo
   ```

4. 安装Docker

   1. `sudo yum install docker-ce docker-ce-cli containerd.io`

5. 测试是否安装成功

   1. `docker -v`

6. 启动docker查询已有镜像，并设置随Linux系统开机自动启动

   ```shell
   docker images
   systemctl start docker
   docker images
   systemctl enable docker
   docker images
   ```

7. 配置docker镜像加速
   1. [阿里镜像加速](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)



# mysql(基于docker)

1. `docker pull mysql:5.7`

2. 启动mysql

   ```shell
   sudo docker run -p 3306:3306 --name mysql \
   -v /mydata/mysql/log:/var/log/mysql \
   -v /mydata/mysql/data:/var/lib/mysql \
   -v /mydata/mysql/conf:/etc/mysql \
   -e MYSQL_ROOT_PASSWORD=root \
   -d mysql:5.7
   ```

3. 查看已启动的容器 

   1. `docker ps`

4. 修改配置

   ```shell
   # 切换⽂件路径
   cd /mydata/mysql/conf
   # 创建配置⽂件
   vi my.cnf
   # 将下列配置信息拷⻉到配置⽂件中
   [client]
   default-character-set=utf8
   [mysql]
   default-character-set=utf8
   [mysqld]
   init_connect='SET collation_connection = utf8_unicode_ci'
   init_connect='SET NAMES utf8'
   character-set-server=utf8
   collation-server=utf8_unicode_ci
   skip-character-set-client-handshake
   skip-name-resolve
   # 重启配置信息
   docker restart mysql
   ```

5. 进入容器查看配置

   ```shell
   # 进⼊容器
   docker exec -it mysql /bin/bash
   # 如果不知道在哪，可以搜索MySQL⽂件
   whereis mysql
   # mysql: /usr/bin/mysql /usr/lib/mysql /etc/mysql /usr/share/mysql
   # 查看配置⽂件
   cat /etc/mysql/my.cnf
   ```

6. 设置启动Docker时运行mysql

   1. `docker update mysql --restart=always`

# redis(基于docker)

1. 拉取redis镜像

   1. `docker pull redis`

2. 配置redis服务

   ```shell
   # 创建配置⽂件存储路径
   mkdir -p /mydata/redis/conf
   # 创建配置⽂件
   touch /mydata/redis/conf/redis.conf
   # 填写配置信息【数据持久化AOF】
   echo "appendonly yes" >> /mydata/redis/conf/redis.conf
   ```

3. 启动redis镜像

   ```shell
   docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data \
   -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
   -d redis redis-server /etc/redis/redis.conf
   ```

4. 测试redis

   1. `docker exec -it redis redis-cli`

5. 设置redis在docker启动时启动

   1. `docker update redis --restart=always`



# ElasticSearch

1. 拉取镜像

   1. `docker pull elasticsearch:7.6.0`

2. 创建和修改配置文件

   1. `mkdir -p /mydata/elasticsearch`

   2. `vi /mydata/elasticsearch/elasticsearch.yml` 

      ```yaml
      cluster.name: "docker-cluster"
      network.host: 0.0.0.0
      # 访问ID限定，0.0.0.0为不限制，生产环境请设置为固定IP
      transport.host: 0.0.0.0
      # elasticsearch节点名称
      node.name: node-1
      # elasticsearch节点信息
      cluster.initial_master_nodes: ["node-1"]
      # 下面的配置是关闭跨域验证（可以不开启）
      http.cors.enabled: true
      http.cors.allow-origin: "*"

3. 启动镜像

   1. ```shell
      docker run -di -p 9200:9200 -p 9300:9300 --name=elasticsearch -v /mydata/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml elasticsearch:7.6.0  -e"discovery.type=single-node"
      
      #单机
      docker run --name elasticsearch -p 9200:9200 -p 9300:9300  -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx128m" -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.6.0
      
      
      ```

4. 设置在docker启动时启动

   1. `docker update elasticsearch --restart=always`

   
   
5. 启动几秒后自动退出，使用`docker logs -f 容器ID`查看日志并解决[AccessDeniedException](https://www.cnblogs.com/leozhanggg/p/12390850.html)

# Kibana

1. 拉取镜像

   1. `docker pull kibana:7.6.0`

2. 创建和修改配置文件

   ```shell
   vi /mydata/kibana/kibana.yml
   
   #配置如下
   server.port: 5601
   server.host: "0.0.0.0"
   elasticsearch.hosts: ["http://elasticsearch的IP:9200"]
   # 操作界面语言设置为中文
   i18n.locale: "zh-CN"
   ```

3. 启动镜像

   ```shell
   docker run -di --name kibana -p 5601:5601 -v /mydata/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.6.0   
   
   ```

4. 设置随docker启动而启动

   1. `docker update kibana --restart=always`

   

# logstash

1. 拉取

   1. `docker pull logstash:7.6.0`

2. 创建和修改配置文件

   1. ```shell
      input {
          tcp {
              port => 5044
      		# 输入为json数据
              codec => json_lines
          }
      }
      filter {
      
      }
      output {
      	# 这个是logstash的控制台打印（进行安装调试的时候开启，稍后成功后去掉这个配置即可）
      	stdout {
      		codec => rubydebug
      	}
      	# elasticsearch配置
      	elasticsearch {
      		hosts => ["elasticsearch的IP:9200"]
      		# 索引名称，没有会自动创建
      		index => "logstash-%{[server_name]}-%{+YYYY.MM.dd}"
      	}
      }
      ```

3. 启动镜像

   1. ```shell
      docker run -d --name=logstash logstash:7.6.0
      docker cp logstash:/usr/share/logstash /mydata/
      mkdir /mydata/logstash/config/conf.d
      chmod -R 777 logstash
      vi /mydata/logstash/config/logstash.yml
      #具体内容
      http.host: "0.0.0.0"
      xpack.monitoring.elasticsearch.hosts: [ "http://172.23.45.53:9200" ]
      path.config: /usr/share/logstash/config/conf.d/*.conf
      path.logs: /usr/share/logstash/logs
      --------------------------------------------------------
      vi /mydata/logstash/config/conf.d/syslog.conf
      #具体内容
      input {
        file {
          #标签
          type => "systemlog-localhost"
          #采集点
          path => "/var/log/messages"
          #开始收集点
          start_position => "beginning"
          #扫描间隔时间，默认是1s，建议5s
          stat_interval => "5"
        }
      }
      
      output {
        elasticsearch {
          hosts => ["192.168.31.196:9200"]
          index => "logstash-system-localhost-%{+YYYY.MM.dd}"
       }
      }
      --------------------------------------------------------
      docker run -d \
        --name=logstash \
        --restart=always \
        -p 5044:5044 -\
        -v /mydata/logstash:/usr/share/logstash \
        -v /var/log/messages:/var/log/messages \
        logstash:7.6.0
      
      
      ```

4. 设置随docker启动时启动

   1. `docker update logstash --restart=always`

   

# prometheus

1. pull

   1. `docker pull prome/prometheus`

2. 准备配置文件

   1. prometheus.yml

   2. ```shell
      scrape_configs:
      # 可随意指定
      - job_name: 'spring'
        # 多久采集一次数据
        scrape_interval: 15s
        # 采集时的超时时间
        scrape_timeout: 10s
        # 采集的路径
        metrics_path: '/doomed/actuator/prometheus'
        # 采集服务的地址，设置成Springboot应用所在服务器的具体地址
        static_configs:
        - targets: ['localhost:9426']
      ```

3. 启动容器

   1. ```shell
      docker run -d -p 9090:9090 -v /mydata/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus 
      
      ```

4. 设置docker启动时启动

   1. `docker update 容器id--restart=always`



# grafana

1. pull

   1. docker pull grafana/grafana 

2. 运行实例

   1. ```bash
      docker run -d -p 3000:3000 grafana/grafana
      ```

   2. 访问http://localhost:3000 检查是否成功，初始管理员账号密码为`admin/admin`。

3. 设置docker启动时启动

   1. `docker update 容器id--restart=always`

# nginx

1. pull

   1. docker pull nginx：latest

2. ```shell
   docker run --name nginx-test -p 80:80 -d nginx
   # 创建mydata/nginx/conf mydata/nginx/www mydata/nginx/logs
   docker cp 容器ID:/etc/nginx/nginx.conf /mydata/nginx/conf
   ```

3. ```shell
   docker run -d --name nginx -p 80:80 -v /mydata/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /mydata/nginx/html:/Users/mengfanxiao/nginx/html -v /mydata/nginx/logs:/var/log/nginx -v /mydata/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf nginx
   ```

