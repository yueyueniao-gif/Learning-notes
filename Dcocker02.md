---

title: Dcocker02
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-21 21:22:34
tags: Docker
categories: Docker
---

### Docker-Compose

```
Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。
```

##### 下载Docker-Compose

```sh
#去git官网搜索docker-compose，下载1.24.1的版本Docker-Compose（这里不推荐这样下载，因为比较慢）
https://github.com/docker/compose/releases/download/1.24.1

#推荐下载方式，使用命令下载
#（这样执行完毕就直接安装到我们的usr/local/bin里面了，并且起了个名字叫docker-compose）
curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

```sh
#需要将docker-compose变成一个可执行文件
chmod 777 docker-compose
```

##### 使用Docker-Compose管理MySql和Tomcat容器

```
yml文件以key: value的方式来指定配置信息

多个配置信息以换行+缩进的方式来区分

在docker-compose.yml中不要使用tab键
```

```yml
version: '3.1'
services:
  MySql:
    restart: always   #代表只要docker启动，那么这个容器就跟着一起启动
    image: daocloud.io/library/mysql:5.7.4  #指定镜像的路径
    container_name: mysql #指定容器名称
    ports:
      - 3306:3306 #指定端口号映射
    environment:
      MYSQL_ROOT_PASSWORD: root #指定MySql的ROOT的用户登录密码
      TZ: Asiz/Shanghai  #指定时区
    volumes:
      - /opt/docker_mysql_tomcat/mysql_data:/var/lib/mysql  #映射数据卷
  tomcat:
    restart: always
    image: daocloud.io/library/tomcat:8.5.15-jre8
    container_name: tomcat
    ports:
      - 8080:8080
    environment:
      TZ: Asiz/Shanghai
    volumes:
      - /opt/docker_mysql_tomcat/tomcat_webapps:/usr/local/tomcat/webapps
      - /opt/docker_mysql_tomcat/tomcat_logs:/usr/local/tomcat/logs
```

##### 使用docker-compose命令管理容器

```
在使用docker-compose的命令时，默认会在当前目录下找docker-compose.yml文件
```

```sh
#1. 基于docker-compose.yml启动管理的容器
docker-compose up -d
```

```sh
#关闭并删除容器
docker-compose down
```

```sh
#开启或关闭已经存在的由docker-compose维护的容器
docker-compose start|stop|restart
```

```sh
#查看由docker-compose管理的容器
docker-compose ps
```

```sh
#查看日志
docker-compose logs -f
```

##### docker-compose配置Dockerfile使用

```
使用docker-compose.yml文件以及Dockerfile文件生成自定义镜像的同事启动当前镜像，并且由docker-compose去管理容器
```

```sh
#docker-compose.yml文件
versio: '3.1'
services: 
  sssm:
    restart: always
    build:        #构建自定义镜像
      context: ../  #指定dockerfile文件所在路径
      dockerfile: Dockerfile #指定Dockerfile文件名称
    image: ssm:1.0.1
    contaomer_name: ssm
    prots:
      - 8081:8080
    environment:
      TZ: Asia/Shanghai
```

```sh
#Dcokerfile文件
from daocloud.io/library/tomcat:8.5.15-jre8
copy sssm.war /user/local/tomcat/webapps
```

```sh
#可以直接启动基于docker-compos.yml以及Dockerfile文件构建的自定义镜像
docker-compose up -d
#如果不存在，会帮助我们构建，如果已经存在，就会直接运行
#重新构建
docker-compose build
##运行前，重新构建
docker-compos up -d --build
```

