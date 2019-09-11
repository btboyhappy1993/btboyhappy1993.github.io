---
layout: post
title:  "【整理】docker-compose安装操作"
date:   2019-09-11 23:19
categories: 整理 docker
permalink: /archivers/20190911-2
---

## docker离线安装
[下载地址 密码：pah1](https://pan.baidu.com/s/1YZLBocHpUmGQZdc5FTdgzw)
[官方地址](https://download.docker.com/linux/static/stable/)

 1. 安装docker-18.03.1-ce

```
#解压
[root@smsr ~]# tar xzvf docker-18.03.1-ce.tar 

#将解压出来的 docker 文件所有内容移动到 /usr/bin/ 目录下
[root@smsr ~]# cp docker/* /usr/bin/

#开启 docker 守护进程
[root@smsr ~]# dockerd &

#查看版本
[root@smsr ~]# docker --version
```

 2. docker注册为service

	将以下文件放入 `/usr/lib/systemd/system/docker.service` 中，就可使用`service docker restart/stop` 等操作来启停docker
 
```
#创建或编辑docker.service
[root@smsr ~]# cd /usr/lib/systemd/system
[root@smsr ~]# vi docker.service
```
进入vim界面后，按`Insert`键，进入编辑模式，把下面`docker.service`内容右键复制进去。
然后按`Esc`键退出编辑，输入`:wq!`，回车保存文件即可。

docker.service：
```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
```
## Docker-Compose
compose主要用于开发／测试场合,适合小规模应用的部署, 并不适合生产环境使用

 1. 安装
 	下载docker-compose-Linux-x86_64文件，[国内下载地址](https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-Linux-x86_64)
 

```
#复制到linux下 /usr/local/bin并改名为docker-compose
[root@smsr ~]#mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose

#赋权
[root@smsr ~]#chmod +x /usr/local/bin/docker-compose

#查看版本
[root@smsr ~]#docker-compose -v
```

 2. 命令
 **注意：docker-compose命令必须在docker-compose.yml文件同目录下执行**
 

```
#修改文件后需要重新build，本地创建镜像
[root@smsr ~]#docker-compose build

#创建并启动容器，第一次可以直接up，可以跳过build环节，
[root@smsr ~]#docker-compose up –d --build

#关闭并删除容器
[root@smsr ~]#docker-compose down

#开启服务
[root@smsr ~]#docker-compose start

#停止服务
[root@smsr ~]#docker-compose stop

#重启服务
[root@smsr ~]#docker-compose restart

#删除服务中的各个容器
[root@smsr ~]#docker-compose rm

#显示容器
[root@smsr ~]#docker-compose ps

#显示容器中的输出内容
[root@smsr ~]#docker-compose logs
```

 3. 导入镜像
 
```
[root@smsr ~]# docker load –i  镜像名.tar  
```

 4. mysql+redis+springboot+nginx
 
	导入相关镜像后，在目录下创建`docker-compose.yml`文件，之后的相关操作都依赖该文件

```
[root@smsr ~]# vi docker-compose.yml
```
进入vim界面后，按`Insert`键，进入编辑模式，把下面`docker-compose.yml`内容右键复制进去。
然后按Esc键退出编辑，输入`:wq!`，回车保存文件即可。

docker-compose.yml:

```
version : '3'

services:

  mysql:
    image: mysql:5.7
    container_name: mysql
    expose:
      - "3306"
    environment:
      MYSQL_DATABASE: test
      MYSQL_ROOT_PASSWORD: root
      MYSQL_ROOT_HOST: '%'
    restart: always
    
  redis:
    image: redis:3.0.3
    container_name: redis
    restart: always
    expose:
      - "6379"
     
  springboot:
    image: springboot
    restart: always
    container_name: springboot
    expose:
      - "8083"
    environment:
      - spring.profiles.active=test 
      - TZ=Asia/Shanghai
    depends_on:
      - mysql
      - redis
      
  nginx: 
    image: mynginx
    container_name: mynginx
    restart: always
    ports:
      - "80:80"
    depends_on:
      - springboot
```
**version:**		使用第几代语法来编写文件

**services:**	表示该文件管理启停的服务

**image:**		指定启动容器的镜像，镜像仓库/标签或者镜像id

**container_name:** 该服务自定义容器名称

**restart:**		服务重启策略，指定为always时，容器总是重新启动。no是默认的重启策略，在任何情			况下都不会重启容器。

**expose:**		暴露端口，但不映射到宿主机，只被连接的服务访问。端口为内部端口

**ports:**		暴露端口信息。使用宿主：容器 （HOST:CONTAINER）格式

**environment:**	添加环境变量，mysql中的环境变量为数据库名，数据库密码等。
springboot两个环境变量分别是指定工程运行环境和运行时区
可以在这里添加环境变量覆盖工程中的相应配置，如添加`cacheType=jvm`可以覆盖工程中的`cacheType=redis`配置

**depends_on:**	服务依赖，指定服务之间的依赖关系，表示需要先启动`depends_on`下面的服务后，再启动本服务。工程中可以使用依赖的服务名代替相关IP信息

工程配置文件：
```
spring.datasource.url=jdbc:mysql://mysql:3306/test?characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.redis.hostName=redis
spring.redis.port=6379    
spring.redis.pool.maxActive=1024    
spring.redis.pool.maxWait=1000    
spring.redis.pool.maxIdle=200
spring.redis.timeout=0
```
nginx配置文件：
```
upstream webapp{
 server springboot:8083 max_fails=3 fail_timeout=20s;  
} 
 server {
     listen       80 default_server;
     listen       [::]:80 default_server;
     server_name  _;
     location /{
     proxy_pass http://webapp/;
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection keep-alive;
     proxy_set_header Host $host;
     proxy_cache_bypass $http_upgrade;
     }
 }
```

**network_mode:** 当需要获取主机信息时如MAC等，可以选择主机模式，此时容器使用和主机同一网络，此时不可使用depends_on中的依赖服务名代替相关IP信息，因所有容器都处于主机网络，可用localhost代替相关IP信息。

```
#桥接模式
network_mode: "bridge"

#主机模式
network_mode: "host"

#无网络模式（稍后可自定义）
network_mode: "none"

#与指定服务同网络
network_mode: "service:[service name]"

#与指定容器同网络
network_mode: "container:[container name/id]"
```

**注意：后面可以使用up、down等指令启停全部服务。**

## 相关文章
docker-compose
> https://www.cnblogs.com/neptunemoon/p/6512121.html
> https://blog.csdn.net/pushiqiang/article/details/78682323
> https://blog.csdn.net/whusj/article/details/80714200?utm_source=blogxgwz0
> https://www.cnblogs.com/anech/p/6873828.html

docker-compose.yml文件详解
> https://blog.csdn.net/liguangxianbin/article/details/79492866