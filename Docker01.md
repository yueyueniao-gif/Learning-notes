---
title: Docker01
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-21 02:25:54
tags: Docker
categories: Docker
---

### Docker介绍
Docker是一个开源的应用容器引擎；是一个轻量级容器技术；

Docker支持将软件编译成一个镜像；然后在镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；

运行中的这个镜像称为容器，容器启动是非常快速的

### Docker基本操作

##### 安装Docker

```sh
#1.下载关于Docker的依赖环境
yum -y install yum-utils device-mapper-persistent-data lvm2

#2.设置一下下载Docker的镜像源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#3.安装Docker
yum makacache fast
yum -y install docker-ce

# 4.启动，并设置为开机自动启动，测试
#启动Docker服务
systemctl start docker
#设置开机自动启动
systemctl enable docker
#测试
docker run hello-world
```

ps :  如果最后一步安装报错如下内容，

```sh
Error:
 Problem: package docker-ce-3:19.03.8-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.10-3.2.el7.x86_64 is excluded
  - package containerd.io-1.2.13-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.2-3.3.el7.x86_64 is excluded
  - package containerd.io-1.2.2-3.el7.x86_64 is excluded
  - package containerd.io-1.2.4-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.5-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.6-3.3.el7.x86_64 is excluded
```

则需在第三步之前执行

```sh
yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

然后就可以正常安装了

##### Docker的中央仓库

```sh
1. Docker官方的中央仓库：镜像最全，但是下载速度慢，因为服务器在国外
   https://hub.docker.com/
2. 国内的镜像网站：网易蜂巢，daoCloud。。。
   http://hub.daocloud.io/(推荐使用)
3. 在公司内部会采取斯福德方式拉去镜像(添加配置)
   #需要在/etc/docker/daemon.json
    {
       "registry-mirrors":["https://registry.docker-cn.com"]
       "insecure-registries":["(公司内部的ip和端口号)ip:port"]
    }
    #重启两个服务
    systemctl daemon-reload
    systemctl restart docker
```

##### 镜像的操作

```sh
#1. 拉取镜像到本地
docker pull 镜像名称[:tag]
#举个例子
docker pull daocloud.io/library/tomcat:8.5.15-jre8
```

```sh
#2. 查看全部本地的镜像
docker images
```

```sh
#3. 删除本地镜像
docker rmi 镜像唯一标识
```

```sh
#4. 镜像的导入导出(不规范)
#将本地的镜像导出
docker save -o 导出的路径 镜像的id
#加载本地的镜像文件
docker load -i 镜像文件
#修改镜像名称
docker tag 镜像id 新镜像名称:版本
```

##### 容器的操作

```sh
#1. 运行容器
#简单操作
docker run 镜像的标识||镜像名称[:tag]
#常用的参数
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像的标识||镜像名称[:tag]
#-d:代表后台运行容器
#-p 宿主机端口:容器端口：为了银蛇当前Linux的端口和容器的端口
#--name 容器名称：指定容器名称
```

```sh
#2. 查看正在运行的容器
docker ps [-qa]
# -a :查看全部的容器，包括没有运行的
# -q :只查看容器的标识
```

```sh
#3. 查看容器的日志
docker logs -f 容器id
#-f:标识可以滚动查看日志的最后几行
```

```sh
#4. 进入到容器内部
docker exec -it 容器id bsah
```

```sh
#5. 删除容器（删除容器前，需要先停止容器）
#停止容器
docker stop 容器id
#停止所有容器
docker rm $(docker ps -qa)
#删除容器
docker rm 容器id
#删除所有容器
docker rm $(docker ps -qa)
```

```sh
#6. 启动容器
docker start 容器id
```

##### 数据卷

```
数据卷：将宿主的一个目录映射到容器的一个目录中。
可以再宿主机中操作目录中的内容，那么容器内部映射的文件呢，也会一起改变
```

```sh
#1. 创建数据卷
docker volume create 数据卷名称
#创建数据卷之后，会默认存放在一个目录下/var/lib/docker/volumes/数据卷名称/_data
```

![](https://s3.jpg.cm/2020/08/21/ukjuC.png)

```sh
#2. 查看数据卷的详细信息
docker volume inspect 数据卷名称
```

```sh
#3. 查看全部的数据卷
docker volume ls
```

```sh
#4. 删除数据卷
docker volume rm 数据卷名称
```

```sh
#5. 应用数据卷
#当你映射数据卷是，如果数据卷不存在。Docker会帮你自动创建
docker run -v 数据卷名称:容器内部的路径 镜像id
#直接指定一个路径作为数据卷的存放位置
docker run -v 路径:容器内部的路径 镜像id
```

##### Docker自定义镜像

```
中央仓库上的镜像，也是Docker的用户自己上传过去的
```

```sh
#1. 创建一个Dockerfile文件，并且制定自定义镜像信息
#Dockerfile文件中常用的内容
from: 制定当前自定义镜像依赖的环境
copy： 将相对路径下的内容复制到自定义镜像中
workdir: 声明镜像的默认工作目录
cmd: 需要执行的命令（在workdir执行的，cmd可以写多个，只以最后一个为准）
#举个例子，自定义一个tomcat镜像，并且将ssm.war部署到tomcat中
from daocloud.io/library/tomcat:8.5.15-jre8
copy ssm.war /usr/local/tomcat/webapps
```

```sh
#2. 将准备好的Dockerfile和相应的文件拖拽到Linux操作系统中，通过Docker的命令制作镜像
docker build -t 镜像名称:[tag] .
```
