---
layout: post
title:  "【整理】Jenkins自动部署maven工程"
date:   2019-09-12 00:05:00
categories: 整理
permalink: /archivers/20190912
---

我的Jenkins是安装在本地虚拟机的linux系统上的，首先介绍系统以及各种软件安装步骤：

## 1. 安装VMWare虚拟机软件

## 2. 安装centos7系统
推荐安装centos7系统，对最新的docker等软件支持的比较好，安装步骤见[教程1](https://www.jb51.net/article/148000.htm)
注意需要配置root账户密码。

- **配置网络、IP**
因为需要安装Jenkins，需要外部访问，需要配置固定IP，前面**教程1**的网络、IP设置可以略过，参考[教程2](https://blog.csdn.net/java_zyq/article/details/78280904)，这样可以在主机上访问虚拟机的IP。

- **配置防火墙**
步骤见[教程3](https://www.cnblogs.com/dongling/p/6239006.html)，确保常用端口可以访问，在本机使用`telnet`命令可以测试端口是否可以连接。

## 3. 安装jdk、tomcat、docker等

 - **安装jdk**
 在[oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载JDK8的linux版本，选择tar.gz后缀的。
 
解压压缩包：

```
tar -zxvf jdk-8u60-linux-x64.tar.gz
```

配置环境变量：

```
vi /etc/profile

#插入以下内容，目录根据实际情况来
JAVA_HOME=/usr/java/jdk1.8.0_60
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH

#使环境变量生效
source /etc/profile

#查看jdk是否配置成功
java -version
```

 - **安装tomcat**
 在[tomcat官网](http://tomcat.apache.org/download-80.cgi)可以下载tar.gz压缩包，注意下载核心版本就足够了。
 
 解压压缩包到相应目录：
 
```
tar -zxvf apache-tomcat-8.5.34.tar.gz
```

编辑配置文件`conf/server.xml`，配置端口、线程池：

```xml
<Connector port="9090" protocol="HTTP/1.1" connectionTimeout="20000"
redirectPort="8443" maxThreads="300"/>
```

编辑`bin/catalina.sh`，配置jvm：增加

```
JAVA_OPTS="-Xms2048m -Xmx4096m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m"
```

**启动tomcat命令**：
进入tomcat下`bin`目录，执行`./startup.sh`

**停止tomcat命令**：

```
#查看tomcat服务PID（XXXX）
ps -ef | grep tomcat

#停止tomcat服务
./shutdown.sh
```

 - **安装maven**
 下载maven，[下载网址](http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz)。

 在linux目录下解压：

```
tar -zxvf apache-maven-3.5.4-bin.tar.gz
```

配置环境变量：

```
vi /etc/profile

#插入以下内容，目录根据实际情况来
export MAVEN_HOME=/var/local/apache-maven-3.5.4
export MAVEN_HOME
export PATH=$PATH:$MAVEN_HOME/bin

#使环境变量生效
source /etc/profile

#查看maven是否配置成功
mvn -version
```

 - **安装docker**

```
#安装一些必要的系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2

#添加软件源信息：
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#更新 yum 缓存
yum makecache fast

#安装 Docker-ce
yum -y install docker-ce

#Docker 后台服务
systemctl start docker

#查看版本
docker version
```

配置镜像加速地址（网易）：

```
vi  /etc/docker/daemon.json
```

文件中修改成这个：

```json
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

 - **安装git**

```
yum -y install git
```

 - **安装svn**

```
yum install -y subversion
```

## 4. 安装Jenkins及配置

 - **安装Jenkins**
 下载最新稳定版本的Jenkins war包，[点击下载](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)。
 运行war包：
 
```
nohup java -jar jenkins.war --httpPort=8088 --prefix=/jenkins &
```

注意：
httpPort：端口号
prefix：URL后缀名
访问URL: http://ip:8088/jenkins

 - **安装插件**
 设置插件下载地址：（由于默认的下载地址是`https`可能访问不了，改成`http`就行了）
 选择“**系统管理-插件管理-高级-升级站点**”，设置为： `http://updates.jenkins.io/update-center.json` 选择“**可选插件**”搜索安装插件`Subversion Plug-in`、`Git plugin`、`Deploy to container Plugin`、`Maven Integration plugin`、`Publish Over FTP`、`Publish Over SSH`
 
 - **设置相关配置**
 选择“**系统管理-全局工具配置**”，设置各种软件的linux地址

## 5. 构建maven项目，并自动打成jar/war包

 1. **构建maven工程任务**
 
 ![在这里插入图片描述](https://img-blog.csdn.net/20180930173147231?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 2. **添加描述**

 ![在这里插入图片描述](https://img-blog.csdn.net/20180930173226487?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 3. **增加git/svn的代码管理地址**

![在这里插入图片描述](https://img-blog.csdn.net/20180930173240730?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![在这里插入图片描述](https://img-blog.csdn.net/20180930173257529?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 4. **构建命令**

![在这里插入图片描述](https://img-blog.csdn.net/20180930173421733?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


```
clean package -Dmaven.test.skip=true
```

 5. **构建后操作**

![在这里插入图片描述](https://img-blog.csdn.net/20180930173444356?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![在这里插入图片描述](https://img-blog.csdn.net/20180930173453584?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 6. 将jar包构建docker镜像
执行shell脚本：
![在这里插入图片描述](https://img-blog.csdn.net/20180930173506272?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


```
cp /root/.jenkins/workspace/demo/target/demo-0.0.1-SNAPSHOT.jar /root/demo
cd /root/demo
docker build -t demo .
```

## 7. 将war包部署到tomcat
配置tomcat：

在`conf/tomcat-users.xml`中的`<tomcat-users>`节点中增加：

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="tomcatUser" password="123456" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```

Jenkins任务配置：

![在这里插入图片描述](https://img-blog.csdn.net/20180930173536551?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2J0Ym95aGFwcHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**注意**：这里使用tomcat管理员权限远程部署war包有IP访问限制，一般只能将`Tomcat URL`写成`127.0.0.1`，若需要修改成其他IP则需要修改tomcat文件夹中`webapps/manager/META-INF/context.xml`，修改内容如下：

```xml
<Valve className="org.apache.catalina.valves.RemoteAddrValve" 
    allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|\d+\.\d+\.\d+\.\d+" />
```
