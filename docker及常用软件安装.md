### Docker安装以及常用软件安装

#### docker安装

```
01 卸载之前的docker
	sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
02 安装必要的依赖
	sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
    
03 设置docker仓库  [设置阿里云镜像仓库可以先自行百度，后面课程也会有自己的docker hub讲解]	
	sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
      
    [访问这个地址，使用自己的阿里云账号登录，查看菜单栏左下角，发现有一个镜像加速器:https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors]

04 安装docker
	sudo yum install -y docker-ce docker-ce-cli containerd.io
	
05 启动docker
	sudo systemctl start docker
	
06 测试docker安装是否成功
	sudo docker run hello-world
	
07 安装 docker compose
	yum install -y epel-release
	yum install -y python3-pip
	pip3 install --upgrade pip
	pip3 install docker-compose
```

#### mysql 安装

```
docker run --name mysql --restart=always \
    -v ~/mysql/conf/:/etc/mysql/conf.d \
    -v ~/mysql/data:/var/lib/mysql \
    -v ~/mysql/log:/var/log/mysql \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD="root" \
    -e TZ=Asia/Shanghai \
    -d mysql:5.7 --lower-case-table-names=1
```

#### nginx 安装

```
docker run --name tmp-container -d nginx
docker cp tmp-container:/etc/nginx/nginx.conf ~/nginx/nginx.conf
docker rm -f tmp-container

docker run --name nginx --restart=always \
    -v ~/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v ~/nginx/html:/usr/share/nginx/html \
    -v ~/nginx/logs:/var/log/nginx \
	-v ~/nginx/conf.d:/etc/nginx/conf.d \
	-d nginx
```

nginx 集群安装

```
https://www.cnblogs.com/jinjiangongzuoshi/p/9313438.html
```



#### redis 安装(docker中安装的redis不带配置文件需自行下载)

```
docker run --name redis --restart=always --privileged=true -p 6379:6379 \
 -v ~/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf \
 -v ~/redis/data:/data \
 -d redis redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

#### redis 集群安装

```bash
# 端口号从7001到7006开始循环
for port in $(seq 7001 7006);  \
do \
# 创建node-port的文件夹
mkdir -p /mydata/redis/node-${port}/conf
# 给文件夹创建redis的配置类
touch /mydata/redis/node-${port}/conf/redis.conf
# 给redis.conf文件中写入内容，从EOF开始，到EOF结束。
cat << EOF > /mydata/redis/node-${port}/conf/redis.conf
# 定义端口号
port ${port}
# 开启集群模式
cluster-enabled yes
# 节点信息在nodes.conf保存着
cluster-config-file nodes.conf
# 每个节点的超时时间，超过多长时间连接不上就认为是断线了。超过该时间（毫秒），集群自动进行主从切换
cluster-node-timeout 5000
# 每个节点在集群中的ip
cluster-announce-ip 192.168.1.119
# 在集群的端口号
cluster-announce-port ${port}
# 跟其他redis进行总线交互用的端口
cluster-announce-bus-port 1${port}
appendonly yes
EOF
#  暴露端口号 端口号 集群交互暴露端口号  集群交互端口号      容器名字
docker run -p ${port}:${port} -p 1${port}:1${port} --name redis-${port}  \
# 挂载data目录 redis.conf 目录
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
# -d以后台方式 镜像版本 redis-server启动redis 加载自定义的配置文件
-d redis:5.0.7 redis-server /etc/redis/redis.conf; \
done

2) 创建集群节点
	# 进入redis-7001内部
    docker exec -it redis-7001 bash
    # 用redis-cli命令行执行 创建集群 一个副本
    redis-cli  --cluster create 192.168.1.119:7001 192.168.1.119:7002 192.168.1.119:7003 192.168.1.119:7004 192.168.1.119:7005 			192.168.1.119:7006 --cluster-replicas 1

3) 测试集群连接
	redis-cli -c  -h host -p port


```



#### tomcat 安装

```
docker run --name tmp-container -d tomcat
docker cp  tmp-container:/usr/local/tomcat/conf  ~/tomcat/conf/  
docker cp  tmp-container:/usr/local/tomcat/webapps  ~/tomcat/webapps/
docker rm -f tmp-container

docker run -d --name tomcat -p 8080:8080 \
-v ~/tomcat/conf/:/usr/local/tomcat/conf/ \
  -v ~/tomcat/webapps/:/usr/local/tomcat/webapps/ \
  -v ~/tomcat/logs/:/usr/local/tomcat/logs/ tomcat
```

#### RabbitMQ安装

```
docker run -d \
-v ~/rabbitmq/data:/var/lib/rabbitmq \
-p 5672:5672 -p 15672:15672 --name rabbitmq --restart=always \
--hostname myRabbit rabbitmq:3-management

# 启动rabbitmq_management, rabbitmq 为容器的名称，使用id也可以
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_management
# ip+端口号登录，用户名和密码默认都是guest
```

#### RabbitMQ集群安装

```
1) 安装rabbitmq 指定cookie
docker run -d --hostname rabbitmq01  --name rabbitmq01 \
-v /mydata/rabbitmq/rabbitmq01:/var/lib/rabbitmq \
-p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='dalianpai' \
rabbitmq:management
2) 安装从节点
docker run -d --hostname rabbitmq02  --name rabbitmq02 \ 
-v /mydata/rabbitmq/rabbitmq02:/var/lib/rabbitmq \
-p 15674:15672 -p 5674:5672 \
-e RABBITMQ_ERLANG_COOKIE='dalianpai' \
--link rabbitmq01:rabbitmq01  rabbitmq:management

docker run -d --hostname rabbitmq03  --name rabbitmq03 \
-v /mydata/rabbitmq/rabbitmq03:/var/lib/rabbitmq \
-p 15675:15672 -p 5675:5672 \ 
-e RABBITMQ_ERLANG_COOKIE='dalianpai' \
--link rabbitmq01:rabbitmq01 --link rabbitmq02:rabbitmq02  rabbitmq:management
3) 节点加入集群
	1. 进入第一个节点
	   docker exec -it rabbitmq01 /bin/bash
	   rabbitmqctl stop_app
	   rabbitmqctl reset
	   rabbitmqctl start_app
	   
	2. 进入第二个节点
	   docker exec -it rabbitmq02 /bin/bash
	   rabbitmqctl stop_app
	   rabbitmqctl reset
	   rabbitmqctl join_cluster --ram rabbit@rabbitmq01
	   rabbitmqctl start_app
	3. 进入第三个节点    
	   docker exec -it rabbitmq02 /bin/bash
	   rabbitmqctl stop_app
	   rabbitmqctl reset
	   rabbitmqctl join_cluster --ram rabbit@rabbitmq01
	   rabbitmqctl start_app
4) 实现rabbitmq 镜像集群
	 docker exec -it rabbitmq01 /bin/bash
	 1) 设置策略
	 rabbitmqctl set_policy -p / ha "^" '{"ha-mode":"all","ha-sync-mode":"automatic"}' 
	  [vhost /  策略名 ha  ^ 所有队列]
	 2) 查看策略
     rabbitmqctl list_policies -p / 
```



#### mysql 主从搭建

```
1. 安装master
docker run -p 3307:3306 --name mysql-master --restart=always \
-v /mydata/mysql/master/log:/var/log/mysql \
-v /mydata/mysql/master/data:/var/lib/mysql \
-v /mydata/mysql/master/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD="root" \
-e TZ=Asia/Shanghai \
-d mysql:5.7 --lower-case-table-names=1

2.修改配置文件
vim /mydata/mysql/master/conf/my.cnf

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

3.安装slaver
docker run -p 3308:3306 --name mysql-slaver-01 --restart=always \
-v /mydata/mysql/slaver/log:/var/log/mysql \
-v /mydata/mysql/slaver/data:/var/lib/mysql \
-v /mydata/mysql/slaver/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD="root" \
-e TZ=Asia/Shanghai \
-d mysql:5.7 --lower-case-table-names=1

4.修改配置文件
vim /mydata/mysql/slaver/conf/my.cnf

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

5.添加master主从复制
vim /mydata/mysql/master/conf/my.cnf
server_id=1
log-bin=mysql-bin
read-only=0
binlog-do-db=gmall_ums

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

6.添加slaver主从复制
vim /mydata/mysql/slaver/conf/my.cnf
server_id=2
log-bin=mysql-bin
read-only=1
binlog-do-db=gmall_ums

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

7.分别重启 master slaver 容器

8.进入主容器
	1) 进行授权 创建同步用户
	docker exec -it mysql-master /bin/bash
	grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
	GRANT FILE,SELECT,INSERT,UPDATE,REPLICATION SLAVE  ON *.* to 'backup'@'%' identified by '123456';
	flush privileges;
	2) 查看binlog
		show master status;

9.进入从容器
	1) 进行授权
	grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
	flush privileges;
	2) 设置主库连接
	change master to master_host='192.168.116.129',master_user='backup',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=0,master_port=3307;
	3) 启动同步
	start slave;
	



```

#### ES安装

```
1. 镜像下载
	docker pull elasticsearch:7.4.2（相当于mysql，存储和检索数据）
　　 docker pull kibana:7.4.2（相当于navicat，可视化检索数据）
　　 
2. 创建实例
	mkdir -p /mydata/elasticsearch/config
	mkdir -p /mydata/elasticsearch/data
	chmod 777 -R /mydata/elasticsearch
	echo "http.host: 0.0.0.0" > /mydata/elasticsearch/config/elasticsearch.yml
3. docker启动
	docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
    -e  "discovery.type=single-node" \
    -e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
    -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
    -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
    -v  /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
    -d elasticsearch:7.4.2
4. 命令详解
	-e指定参数，single-node指定es单节点运行，ES_JAVA_OPTS指定es初始内存和最大内存，如果不指定会占用全部内存导致整个虚拟机卡死

	-v挂载，，

	-d，后台启动es用es7.4.2镜像
```



#### ES集群安装

```
1.前置工作
	sysctl -w vm.max_map_count=262144
	echo vm.max_map_count=262144 >> /etc/sysctl.conf
	sysctl -p
2.创建docker网络
  	docker network create  --driver bridge --subnet=172.19.1.0/16 --gateway=172.19.0.1 mynet
  	查看网络信息
  	docker network inspect mynet
  	以后使用
  		

for port in $(seq 1 3); \
do \
mkdir -p /mydata/elasticsearch/master-${port}/config
mkdir -p /mydata/elasticsearch/master-${port}/data
chmod -R 777 /mydata/elasticsearch/master-${port}
cat << EOF > /mydata/elasticsearch/master-${port}/config/elasticsearch.yml
cluster.name: my-es  #集群的名称，同一个集群该值必须设置成相同的
node.name: es-master-${port}  #该节点的名字
node.master: true  #该节点有机会成为master节点
node.data: false #该节点可以存储数据
network.host: 0.0.0.0
http.host: 0.0.0.0   #所有http均可访问
http.port: 920${port}
transport.tcp.port: 930${port}
discovery.zen.ping_timeout: 10s #设置集群中自动发现其他节点时ping连接的超时时间
discovery.seed_hosts: ["172.19.1.21:9301","172.19.1.22:9302","172.19.1.23:9303"]
cluster.initial_master_nodes: ["172.19.1.21"] #新集群初始时的候选主节点，es7的新增配置
EOF
docker run --name elasticsearch-node-${port} \
-p 920${port}:920${port} -p 930${port}:930${port} \
--network=mynet --ip 172.19.1.2${port} \
-e ES_JAVA_OPTS="-Xms300m -Xmx300m"  \
-v /mydata/elasticsearch/master-${port}/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml  \
-v /mydata/elasticsearch/master-${port}/data:/usr/share/elasticsearch/data  \
-v /mydata/elasticsearch/master-${port}/plugins:/usr/share/elasticsearch/plugins  \
-d elasticsearch:7.4.2
done




for port in $(seq 4 6); \
do \
mkdir -p /mydata/elasticsearch/master-${port}/config
mkdir -p /mydata/elasticsearch/master-${port}/data
chmod -R 777 /mydata/elasticsearch/master-${port}
cat << EOF > /mydata/elasticsearch/master-${port}/config/elasticsearch.yml
cluster.name: my-es  #集群的名称，同一个集群该值必须设置成相同的
node.name: es-node-${port}  #该节点的名字
node.master: false  #该节点有机会成为master节点
node.data: true #该节点可以存储数据
network.host: 0.0.0.0
http.host: 0.0.0.0   #所有http均可访问
http.port: 920${port}
transport.tcp.port: 930${port}
discovery.zen.ping_timeout: 10s #设置集群中自动发现其他节点时ping连接的超时时间
discovery.seed_hosts: ["172.19.1.21:9301","172.19.1.22:9302","172.19.1.23:9303"]
cluster.initial_master_nodes: ["172.19.1.21"] #新集群初始时的候选主节点，es7的新增配置
EOF
docker run --name elasticsearch-node-${port} \
-p 920${port}:920${port} -p 930${port}:930${port} \
--network=mynet --ip 172.19.1.2${port} \
-e ES_JAVA_OPTS="-Xms300m -Xmx300m"  \
-v /mydata/elasticsearch/master-${port}/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml  \
-v /mydata/elasticsearch/master-${port}/data:/usr/share/elasticsearch/data  \
-v /mydata/elasticsearch/master-${port}/plugins:/usr/share/elasticsearch/plugins  \
-d elasticsearch:7.4.2
done
```



#### mysql 权限说明

```
对mysql用户等的操作
A-对新用户增删改

1.创建用户:
# 指定ip：192.118.1.1的alex用户登录
create user 'alex'@'192.118.1.1' identified by '123';
# 指定ip：192.118.1.开头的alex用户登录
create user 'alex'@'192.118.1.%' identified by '123';
# 指定任何ip的alex用户登录
create user 'alex'@'%' identified by '123';

2.删除用户
drop user '用户名'@'IP地址';

3.修改用户
rename user '用户名'@'IP地址' to '新用户名'@'IP地址';

4.修改密码
set password for '用户名'@'IP地址'=Password('新密码');


B-对当前的用户授权管理

#查看权限
show grants for '用户'@'IP地址'

#授权 alex用户仅对db1.t1文件有查询、插入和更新的操作
grant select ,insert,update on db1.t1 to "alex"@'%';

# 表示有所有的权限，除了grant这个命令，这个命令是root才有的。alex用户对db1下的t1文件有任意操作
grant all privileges  on db1.t1 to "alex"@'%';
#mjj用户对db1数据库中的文件执行任何操作
grant all privileges  on db1.* to "alex"@'%';
#mjj用户对所有数据库中文件有任何操作
grant all privileges  on *.*  to "alex"@'%';
 
#取消权限
 
# 取消alex用户对db1的t1文件的任意操作
revoke all on db1.t1 from 'alex'@"%";  

# 取消来自远程服务器的alex用户对数据库db1的所有表的所有权限

revoke all on db1.* from 'alex'@"%";  

取消来自远程服务器的alex用户所有数据库的所有的表的权限
revoke all privileges on *.* from 'alex'@'%';
```

#### docker 安装 torna

```
  
docker run --name torna --restart=always \
  -p 7700:7700 \
  -e JAVA_OPTS="-Xms256m -Xmx256m" \
  --link=mysql-master \
  -v /opt/torna/config:/torna/config \
  -d tanghc2020/torna:latest
```

#### docker 安裝jenkins

```
mkdir -p /var/jenkins_mount
chmod 777 /var/jenkins_mount
docker run -d -p 10240:8080 -p 10241:50000 \
  -v /var/jenkins_mount:/var/jenkins_home \
  --restart=always \
  -v /etc/localtime:/etc/localtime \
  -v /var/run/docker.sock:/var/run/docker.sock  \
  -v $(which docker):/usr/bin/docker  \
  -v $(which docker-compose):/usr/local/bin/docker-compose  --name myjenkins jenkins/jenkins:2.345
  
更改镜像源
 sed -i 's/https:\/\/updates.jenkins.io\/update-center.json/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins\/updates\/update-center.json/g'  hudson.model.UpdateCenter.xml
 
初次跳过插件安装、插件仓库可用初始化完后
 sed -i 's#https://updates.jenkins.io/download#https://mirrors.tuna.tsinghua.edu.cn/jenkins#g' updates/default.json
 sed -i 's#http://www.google.com#https://www.baidu.com#g' updates/default.json
 
重启jenkins 
 docker restart myjenkins
```

#### 	
