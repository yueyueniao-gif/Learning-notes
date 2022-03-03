---
2
title: Nginx
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-20 22:16:07
tags: Nginx
categories: Nginx
---

### Nginx简介

Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。

###### nginx最高并发支持50000个



#### 正向代理和反向代理

###### 正向代理

在客户端（浏览器）配置代理服务器，通过代理服务器进行互联网访问

![](https://s3.jpg.cm/2020/08/21/ukebt.png)

###### 反向代理

反向代理，其实客户端对代理是无感知的，因为哭护短不需要做任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，吃屎反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真是服务器IP地址

![](https://s3.jpg.cm/2020/08/21/ukl4R.jpg)



#### 负载均衡

我们增加了服务器的数量，然后将请求分发到各个服务器上，将原先的请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡

![](https://s3.jpg.cm/2020/08/21/ukU7X.png)

#### 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力

![](https://s3.jpg.cm/2020/08/21/ukHP6.png)

### Nginx的安装

##### 安装Nginx

![CmHkS.png](https://s3.jpg.cm/2021/03/21/CmHkS.png)



```yml
version: '3.1'
services:
  nginx:
    restart: always
    image: daocloud.io/library/nginx:latest
    container_name: nginx
    ports:
      - 80:80
    volumes:
      - /opt/docker_nginx/conf.d/:/etc/nginx/conf.d
```

##### Nginx配置文件

首先进入nginx容器内部，然后进入容器内部的 /etc/nginx/

然后直接cat nginx.conf，就可以看到以下文件内容了

```json
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

#以上统称为全局快
#worker_processes它的数值越大，Nginx的并发能力越强
#error_log代表Nginx的错误日志存放位置
#pid代表Nginx的运行表示（不用关心）

events {
    worker_connections  1024;
}
#event块
#worker_connections它的数值越大，Nginx并发能力越强

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
}
#http块
#include代表引入一个外部的文件        -----> /mine.types中存放着大量的媒体类型
#include /etc/nginx/conf.d/*.conf; ------>引入了conf.d目录下的以.conf为结尾的配置文件
```

其中http块中的 #include /etc/nginx/conf.d/*.conf 引入的文件如下：

```json
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    #location
    #root：将接收到的请求根据/usr/share/nginx/html去查找静态资源
    #index: 默认去上述路径中找到index.html或者index.htm文件
}
#server块
# listen： 代表Nginx监听端口
#localhost：代表Nginx接收请求的ip
```



### Nginx的使用

##### 基于Nginx实现反向代理

```
准备一个目标服务器
启动一个tomcat服务器
编写nginx配置文件，通过nginx访问tomcat服务器
```

```json
server{
  listen 80;
  server_name localhost;
    #基于反向代理访问到tomcat服务器
  location / {
    proxy_pass http://39.102.66.143:8081/;
  }
}
```

###### 关于nginx的location路径映射

优先级关系：

(location=) > (location /xxx) > (location ^~) > (location ~,~*) > (location /起始路径) > (location /)

```json
#1. = 匹配
location = / {
   #精准匹配，主机名后面不能带任何的字符串
}
```

```json
#2. 通用匹配
location /xxx {
   #匹配所有以/xxx开头的路径
}
```

```json
#3. 正则匹配
location ~ /xxx {
   #匹配所有以/xxx开头的路径
}
```

```json
#4. 匹配开头路径
location ^~ /images/ {
   #匹配所有以/images开头的路径
}
```

```json
#5. 匹配结尾路径
location ~* \.(gif|jpg|png)$ {
   #匹配以gif或者jpg或者png为结尾的路径
}
```

##### 负载均衡

```
Nginx为我们默认提供了三种负载均衡的策略：
  1.轮询：
     将客户端发起的请求，平均分配给每一台服务器
  2.权重：
     会将客户端的请求，根据的权重值不同，分配不同的数量
  3.ip_hash：
     基于发起请求的客户端地址不同，他始终会将请求发送到指定的服务器上,这种方式可以解决session相关的问题
```

###### 轮询

想实现Nginx的轮询负载均衡机制，只需要在配置文件中添加一下内容

```json
upstream 自己起的名字{
    server 39.102.66.143:8081;
    server 39.102.66.143:8080;
    server ip:port;
    ...
}
server {
    listen       80;
    server_name  localhost;

    location / {
       proxy_pass http://自己起的名字/;
    }
}
```

###### 权重

想实现Nginx的权重负载均衡机制，只需要在轮询中增加weight权重比例

```json
upstream 自己起的名字{
    server 39.102.66.143:8081;
    server 39.102.66.143:8080;
    server ip:port weight=权重比例;
    server ip:port weight=权重比例;
    ...
}
server {
    listen       80;
    server_name  localhost;

    location / {
       proxy_pass http://自己起的名字/;
    }
}
```

###### ip_hash

只需要加一个ip_hash

```
upstream 自己起的名字{
    ip_hsah;
    server 39.102.66.143:8081;
    server 39.102.66.143:8080;
    server ip:port;
    ...
}
server {
    listen       80;
    server_name  localhost;

    location / {
       proxy_pass http://自己起的名字/;
    }
}
```

##### 动静分离

```
Nginx的并发能力公式
   worker_processes*worker_connections/4|2=Nginx最终的并发能力
   动态资源需要/4，静态资源需要/2
Nginx通过动静分离，来提升Nginx的并发能力，更快的给用户响应
```

###### 动态资源代理

```json
#配置如下
location / {
    proxy_pass 路径；
}
```

###### 静态资源路径

```json
#配置如下
location / {
    root 静态资源路径;
    index 默认访问路径下的什么资源;
    autoindex on；  #代表展示静态资源下的全部内容，以列表形式展开
}


#先修改docker，添加一个数据卷，映射到Nginx服务器的一个目录
version: '3.1'
services:
  nginx:
    restart: always
    image: daocloud.io/library/nginx:latest
    container_name: nginx
    ports:
      - 80:80
    volumes:
      - /opt/docker_nginx/conf.d/:/etc/nginx/conf.d
      - /opt/docker_nginx/img:/date/img     #新增的映射数据卷
      - /opt/docker_nginx/html:/data/html   #新增的映射数据卷
```

### Nginx的集群

```
单点故障，避免nginx的宕机，导致整个程序的崩溃

准备多台Nginx

准备keepalived，监听nginx的健康状况

准备haproxt，提供一个虚拟的路径，统一的去接受用户的请求
```



![](https://s3.jpg.cm/2020/08/24/uCyV2.png)

