### Docker笔记

#### 1.Dockerfile

##### 1.1 FROM

​	指定基础镜像  比如FROM ubuntu:14.04

```
FROM ubuntu:14.04
```

##### 1.2 RUN

在镜像内部执行一些命令，比如安装软件，配置环境等，换行可以使用""

```
RUN groupadd -r mysql && useradd -r -g mysql mysql
```

##### 1.3 ENV

设置变量的值，ENV MYSQL_MAJOR 5.7，可以通过docker run --e key=value修改，后面可以直接使用${MYSQL_MAJOR}

```
ENV MYSQL_MAJOR 5.7
```

##### 1.4 LABEL

设置镜像标签

```
LABEL email="itcrazy2016@163.com"
LABEL name="itcrazy2016"
```

##### 1.5 VOLUME

指定挂载目录

```
VOLUME /var/lib/mysql
```

##### 1.6 COPY

将主机的文件复制到镜像内，如果目录不存在，会自动创建所需要的目录，注意只是复制，不会提取和解压

```
COPY docker-entrypoint.sh /usr/local/bin/
```

##### 1.7 ADD

将主机的文件复制到镜像内，和COPY类似，只是ADD会对压缩文件提取和解压

```
ADD application.yml /data/server/
```

##### 1.8 WORKDIR

指定镜像的工作目录，之后的命令都是基于此目录工作，若不存在则创建 类似于CD

```
WORKDIR /usr/local
WORKDIR tomcat
RUN touch test.txt
```

##### 1.9 CMD

容器启动的时候默认会执行的命令，若有多个CMD命令，则最后一个生效

```
CMD ["mysqld"]  推荐
或
CMD mysqld
```

##### 1.10 ENTRYPOINT

和CMD的使用类似

```
ENTRYPOINT ["docker-entrypoint.sh"]
```

##### 1.11  EXPOSE

指定镜像要暴露的端口，启动镜像时，可以使用-p将该端口映射给宿主机

```
EXPOSE 3306
```

##### 1.12  其它

```
MAINTAINER  指定镜像这信息
USER  用户
```



#### 2.Dockerfile编写

```
  FROM openjdk:8
  MAINTAINER my
  LABEL name="dockerfile-demo" version="1.0" author="my"
  COPY dockerfile-demo-0.0.1-SNAPSHOT.jar dockerfile-image.jar
  CMD ["java","-jar","dockerfile-image.jar"]
```

#### 3. 打包镜像

```
docker build -f Dockerfile -t test-docker-image .
```



#### 4.搭建自己的Docker Harbor

```
(1)访问github上的harbor项目
https://github.com/goharbor/harbor
(2)下载版本，比如1.7.1
https://github.com/goharbor/harbor/releases
(3)找一台安装了docker-compose[这个后面的课程会讲解]，上传并解压
tar -zxvf xxx.tar.gz
(4)进入到harbor目录
修改harbor.cfg文件，主要是ip地址的修改成当前机器的ip地址
同时也可以看到Harbor的密码，默认是Harbor12345
(5)安装harbor，需要一些时间
	sh install.sh
(6)浏览器访问，比如39.100.39.63，输入用户名和密码即可
```

#### 5.image常见操作

```
(1)查看本地image列表
		docker images
		docker image ls
(2)获取远端镜像
		docker pull
 
(3)删除镜像[注意此镜像如果正在使用，或者有关联的镜像，则需要先处理完]
    docker image rm imageid
    docker rmi -f imageid
    docker rmi -f $(docker image ls)   删除所有镜像
(4)运行镜像
	docker run image
(5)发布镜像
	docker push
```

#### 6.container常见操作

```
(1)根据镜像创建容器
    docker run -d --name -p 9090:8080 my-tomcat tomcat
(2)查看运行中的container
	docker ps
(3)查看所有的container[包含退出的]
	docker ps -a
(4)删除container
	docker rm containerid
	docker rm -f $(docker ps -a)  删除所有container
(5)进入到一个container中
	docker exec -it container bash
(6)根据container生成image
	docker
(7)查看某个container的日志
	docker logs container
(8)查看容器资源使用情况
	docker stats
(9)查看容器详情信息
	docker inspect container
(10)停止/启动容器
	docker stop/start container	
```

#### 7.docker 底层技术支持

```
Namespace：用来做隔离的，比如pid[进程]、net[网络]、mnt[挂载点]等
CGroups: Controller Groups用来做资源限制，比如内存和CPU等
Union file systems：用来做image和container分层
```

#### 8.docker 网络相关

##### 	8.1 linux中的网卡

###### 		8.1.1  查看网卡[网络接口]

```
01) ip link show
02) ls /sys/class/net
03) ip a
```

###### 	    8.1.2 ip a解读		

```
状态：UP/DOWN/UNKOWN等
link/ether：MAC地址
inet：绑定的IP地址
```

###### 		8.1.3 网卡配置文件

```
在Linux中网卡对应的其实就是文件，所以找到对应的网卡文件即可
比如：cat /etc/sysconfig/network-scripts/ifcfg-eth0
```

##### 	8.2 给网卡添加ip地址	

> 当然，这块可以直接修改ifcfg-*文件，但是我们通过命令添加试试

```
(1) 添加ip地址
	ip addr add 192.168.0.100/24 dev eth0
(2)删除IP地址
	ip addr delete 192.168.0.100/24 dev eth0
```

##### 	8.3 网卡启动、关闭

```
重启网卡 ：service network restart / systemctl restart network
启动/关闭某个网卡 ：ifup/ifdown eth0 / ip link set eth0 up/down
```

##### 	8.4 Network Namespace

> 在linux上，网络的隔离是通过network namespace来管理的，不同的network namespace是互相隔离的
> ip netns list：查看当前机器上的network namespace
> network namespace的管理	
>
> ```
> ip netns list    #查看
> ip netns add ns1   #添加
> ip netns delete ns1 #删除
> ```
>
> 

###### 			8.4.1 namespace实战

> (1)创建一个network namespace	

```
ip netns add ns1
```

> (2)查看该namespace下网卡的情况

```
ip netns exec ns1 ip a
```

> (3)启动ns1上的lo网卡

```
ip netns exec ns1 ifup lo
or
ip netns exec ns1 ip link set lo up
```

> 4)再次查看 可以发现state变成了UNKOWN

```
ip netns exec ns1 ip a
```

> (5)再次创建一个network namespace

```
ip netns add ns2
```

> (6)此时想让两个namespace网络连通起来

```
veth pair ：Virtual Ethernet Pair，是一个成对的端口，可以实现上述功能
```

> (7)创建一对link，也就是接下来要通过veth pair连接的link

```
ip link add veth-ns1 type veth peer name veth-ns2
```

> (8)查看link情况

```
ip link
```

> (9)将veth-ns1加入ns1中，将veth-ns2加入ns2中

```
ip link set veth-ns1 netns ns1
ip link set veth-ns2 netns ns2
```

> (10)查看宿主机和ns1，ns2的link情况

```
ip link
ip netns exec ns1 ip link
ip netns exec ns2 ip link
```

> (11)此时veth-ns1和veth-ns2还没有ip地址，显然通信还缺少点条件

```
ip netns exec ns1 ip addr add 192.168.0.11/24 dev veth-ns1
ip netns exec ns2 ip addr add 192.168.0.12/24 dev veth-ns2
```

> (12)再次查看，发现state是DOWN，并且还是没有IP地址

```
ip netns exec ns1 ip link
ip netns exec ns2 ip link
```

> (13)启动veth-ns1和veth-ns2

```
ip netns exec ns1 ip link set veth-ns1 up
ip netns exec ns2 ip link set veth-ns2 up
```

> (14)再次查看，发现state是UP，同时有IP

```
ip netns exec ns1 ip a
ip netns exec ns2 ip a
```

> (15)此时两个network namespace互相ping一下，发现是可以ping通的

```
ip netns exec ns1 ping 192.168.0.12
ip netns exec ns2 ping 192.168.0.11
```

##### 8.5 docker网络

###### 	8.5.1 docker network 常用命令

```
docker network ls   查看网络
docker network create tomcat-net  创建网络
docker network create --subnet=172.18.0.0/24 tomcat-net   创建网络
docker network connect tomcat-net tomcat01  容器连接网络
```

######    8.5.2  Host

> ​	(1)创建一个tomcat容器，并且指定网络为none

```
docker run -d --name my-tomcat-host --network host tomcat
```

> (2)查看ip地址

```
docker exec -it my-tomcat-host ip a
可以发现和centos是一样的
```

> (3)检查host网络

```
"Containers": {
"e1f00d47db344b6688e99c0f5b393e232309fbe1a4d9c3fc3e1ce7c107f3312d": {
"Name": "my-tomcat-host",
"EndpointID":
"f08456d9dca024cf6f911f8d32329ba2587ea89554c96b77c32698ace6998525",
"MacAddress": "",
"IPv4Address": "",
"IPv6Address": ""
}
}
```

###### 8.5.3 NONE

> (1)创建一个tomcat容器，并且指定网络为none

```
docker run -d --name my-tomcat-none --network none tomcat
```

> (2)查看ip地址
>

```
docker exec -it my-tomcat-none ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
```

> (3)检查none网络

```
"Containers": {
"bb3f0db4fa76a25b5377da9c3bbf087ac7ef0de0a3f9c37a4ae959983d33105c": {
"Name": "my-tomcat-none",
"EndpointID":
"26055c08c968f9d6d03d10b3b66dfea004c35f5d2bd4067a2306566973e92f9e",
"MacAddress": "",
"IPv4Address": "",
"IPv6Address": ""
}
}
```

#### 9 docker实战

##### 	9.1 搭建mysql集群

```
01 拉取pxc镜像
	docker pull percona/percona-xtradb-cluster:5.7.21
02 复制pxc镜像(实则重命名)
	docker tag percona/percona-xtradb-cluster:5.7.21 pxc
03 删除pxc原来的镜像
	docker rmi percona/percona-xtradb-cluster:5.7.21
04 创建一个单独的网段，给mysql数据库集群使用
	(1)docker network create --subnet=172.18.0.0/24 pxc-net
	(2)docket network inspect pxc-net  [查看详情]
	(3)docker network rm pxc-net    [删除]
05 创建和删除volume
	创建：docker volume create --name v1
	删除：docker volume rm v1
	查看详情：docker volume inspect v1
06 创建单个PXC容器demo
	[CLUSTER_NAME PXC集群名字]
 	[XTRABACKUP_PASSWORD数据库同步需要用到的密码]
 
    docker run -d -p 3301:3306
    -v v1:/var/lib/mysql
    -e MYSQL_ROOT_PASSWORD=jack123
    -e CLUSTER_NAME=PXC
    -e XTRABACKUP_PASSWORD=jack123
    --privileged --name=node1 --net=pxc-net --ip 172.18.0.2
    pxc
07 搭建PXC[MySQL]集群
	(1)准备3个数据卷
        docker volume create --name v1
        docker volume create --name v2
        docker volume create --name v3
	(2)运行三个PXC容器
        docker run -d -p 3301:3306 -v v1:/var/lib/mysql -e
        MYSQL_ROOT_PASSWORD=jack123 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=jack123 -
        -privileged --name=node1 --net=pxc-net --ip 172.18.0.2 pxc
	[CLUSTER_JOIN将该数据库加入到某个节点上组成集群]
        docker run -d -p 3302:3306 -v v2:/var/lib/mysql -e
        MYSQL_ROOT_PASSWORD=jack123 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=jack123 -
        e CLUSTER_JOIN=node1 --privileged --name=node2 --net=pxc-net --ip 172.18.0.3 pxc
        docker run -d -p 3303:3306 -v v3:/var/lib/mysql -e
        MYSQL_ROOT_PASSWORD=jack123 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=jack123 -
        e CLUSTER_JOIN=node1 --privileged --name=node3 --net=pxc-net --ip 172.18.0.4 pxc
(3)MySQL工具连接测试
        Jetbrains Datagrip
```



https://docs.docker.com/reference/





```
version: '3.7'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    volumes:
      - ./data:/data
    ports:
      - 2182:2181
       
  kafka9094:
    image: wurstmeister/kafka
    ports:
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 0
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.60:9092
      KAFKA_CREATE_TOPICS: "kafeidou:2:0"   #kafka启动后初始化一个有2个partition(分区)0个副本名叫kafeidou的topic 
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
    volumes:
      - ./kafka-logs:/kafka
    depends_on:
      - zookeeper
```

