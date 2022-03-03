---
title: Zookeeper
author: Wang Yue Niao
top: false
toc: true
date: 2020-12-15 10:09:17
tags: zookeeper
categories: zookeper
---

# Zookeeper

## 简介

### 概述

- Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的Apache项目
- Zookeeper从设计模式的角度来理解，是一个基于观察者模式设计的分布式的服务管理框架，它**负责存储和管理大家都关心的数据**，然后**接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责同志已经在Zookeeper上注册的那些观察者做出相应的反应**

### 特点

- Zookeeper：一个领导者，多个跟随着组成的集群。
- 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。
- 全局数据一致：每个Server保存一份相同的数据副本。Client无论连接到哪个Server，数据都是一致的。
- 更新请求顺序进行，来自同一个Client的更新请求按照其发送顺序依次执行。
- 数据更新原子性，一次数据更新要么成功，要么失败。
- 实时性，在一定时间范围内，Client能读到最新数据。

### 数据结构

- Zookeeper数据模型的结构与Liunx文件系统很类似，整体上可以看做是一颗树，每个节点称作一个ZNode。每一个ZNode默认能够存储1MB的数据，每个XNode都可以通过其路径唯一标识。
- [![Fabk4.png](https://s3.jpg.cm/2020/12/15/Fabk4.png)](https://imagelol.com/image/Fabk4)



## 单机版安装

### 安装前准备

1. 安装jdk
2. 拷贝Zookeeper安装包到Liunx系统下
3. 解压到指定目录

### 修改配置文件

1. 将zookeeper-3.4.10/conf下的zoo_sample.cfg修改为zoo.cfg

   ```lua
   [yueyueniao@localhost local]$ mv zoo_sample.cfg zoo.cfg
   ```

2. 打开zoo.cfg文件，修改dataDir路径

   ```lua
   [yueyueniao@localhost local]$ vim zoo.cfg
   
   修改内容如下
   dataDir=/opt/zookeeper-3.4.10/zkData
   ```

3. 在/opt/zookeeper-3.4/10/这个目录上创建zkData文件夹

   ```lua
   [yueyueniao@localhost local]$ mkdir zkData
   ```

### 操作Zookeeper

1. 启动zookeeper

   ```lua
   [yueyueniao@localhost local]$ bin/zkServer.sh start
   ```

2. 查看进程是否启动

   ```lua
   [yueyueniao@localhost local]$ jps
   4020 Jps
   4001 QuorumPeerMain
   ```

3. 查看状态

   ```lua
   [yueyueniao@localhost local]$ bin/zkServer.sh status
   ```

4. 启动客户端

   ```lua
   [yueyueniao@localhost local]$ bin/zkCli.sh
   ```

5. 退出客户端

   ```
   [yueyueniao@localhost local]$ quit
   ```

6. 停止Zookeeper

   ```lua
   [yueyueniao@localhost local]$ bin/zkServer.sh stop
   ```

### 配置参数解读

1. tickTime =2000：通信心跳数，Zookeeper服务器与客户端心跳时间，单位毫秒
   - Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。
   - 它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)
2. initLimit =10：LF初始通信时限
   - 集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。
3. syncLimit =5：LF同步通信时限
   - 集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。
4. dataDir：数据文件目录+数据持久化路径
   - 主要用于保存Zookeeper中的数据。
5. clientPort =2181：客户端连接端口
   - 监听客户端连接的端口。

## 集群版安装

- 真实的集群是需要部署在不同的服务器上的，但是我们在测试时启动多个虚拟机消耗内存太大，所以我们通常会搭建伪集群，也就是把所有的服务器搭建在一台虚拟机上，用端口进行区分。

### 安装前准备

1. 安装JDK

2. Zookeeper压缩包上传到服务器

3. 将Zookeeper解压到/opt/zkcluster，重命名为zookeeper01。复制两份分别命名为zookeeper02和zookeeper03。

   /opt/zookeeper-cluster/zookeeper01

   /opt/zookeeper-cluster/zookeeper02

   /opt/zookeeper-cluster/zookeeper03

4. 在三个Zookeeper目录下创建data目录，并分别修改zookeeper/conf目录下的zoo_sample.cfg重命名为zoo.cfg。

5. 修改zoo.cfg中dataDir=/opt/zookeeper-cluster/zookeeper01/data

6. 修改zoo.cfg中的clientPort分别为2181 2182 2183

   例如：修改/opt/zookeeper-cluster/zookeeper011/conf/zoo/cfg

   ```
   clientport=2181
   dataDir=/usr/local/zookeeper-cluster/zookeeper01/data
   ```

### 配置集群

1. 在每个zookeeper的data目录下创建一个myid文件，内容分别是1,2,3.这个文件就是记录每个服务器的ID。

2. 在每个zookeeper的zoo.cfg配置客户端访问端口（clientPort）和集群服务器IP列表。

   ```
   server.1=192.168.36.134:2881:3881
   server.2=192.168.36.134:2882:3882
   server.3=192.168.36.134:2883:3883
   server.服务器ID=服务器IP地址：服务器之间通信端口：服务器之间投票选举端口
   ```

3. 依次启动三个zk实例，其中有一个leadr和两个follower

## zookeeper基本使用

### 命令行使用

通过zkClient进入zookeeper客户端命令行，输入help查看zookeeper客户端指令

![FDvQ4.png](https://s3.jpg.cm/2020/12/16/FDvQ4.png)

- 如上图列出了zookeeper所有客户端命令行，下面主要讲解常见的几个命令行

  - 使用ls命令来查看当前znode中包含的内容

    ```
    ls path [watch]
    ```

  - 查看当前节点数据并能看到更新次数等数据

    ```
    ls2 path [watch]
    ```

  - 创建节点-s含有序列-e临时

    ```
    create
    ```

  - 获得节点的值

    ``` 
    get path [watch]
    ```

  - 设置节点的值

    ```
    set
    ```

  - 查看节点状态

    ```
    stat
    ```

  - 删除节点

    ```
    delete
    ```

  - 递归删除节点

    ```
    rmr
    ```

### api使用

- maven依赖

  ```xml
  <dependency>
     <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.4.13</version>
  </dependency>
  ```

- api

  ```java
  /* 
  创建zookeeper连接，connectString表示连接的zookeeper服务的地址，sessionTimeOut指定会话的超时时间，Watcher配置监听
  */
      Zookeeper zk =new Zookeeper(String connectString,int sessionTimeout,Watcher watcher)
          
  /*
  创建一个给定目录节点path，并给它设置数据，CreateMode标识有四种形式的目录节点，分别是
      PERSISTENT:持久化目录节点，这个目录节点存储的数据不会丢失;
      PERSISTENT_SEQUENTIAL:顺序自动编号的目录节点，这种目录节点会根据当前已经存在的节点数自动加1,然后返回给客户端已经     成功创建的目录节点名;
      EPHEMERAL:临时目录节点，一旦创建这个节点的客户端与服务器端口也就是session超时，这种节点会被动删除;
      EPHEMERAL_SEQUENTIAL:临时自动编号节点
  */
      String create(String path,byte[] data,List acl,CreateMode createMode)
          
  /*
  判断某个path是否存在，并设置是否监控这个目录节点，这里的watcher是在创建Zookeeper实例时指定的watcher,exists方法还有一个重载方法，可以指定特定的watcher
  */
          Stat exists(String path,boolean watch)
          
  /*
  重载方法，这里给某个目录节点设置特定的watcher，Watcher在Zookeeper是一个核心功能，Watcher可以监控目录节点的数据变化以及子目录的变化，一旦这些状态发生变化，服务器就会通知所有设置在这个目录节点上的Wathcer，从而每个客户端都很快知道它所关注的目录节点的状态发生变化，而做出相应的反应
  */
          Stat exists(String path,Watcher watch)
  /*
  删除path对应的目录节点，version为-1可以匹配任何版本，也就删除了这个目录节点所有数据
  */
          void delete(String path,int version)
  /*
  获取指定path下的所有子目录节点，同样getChildren方法也有一个重载方法可以设置特定的watcher监控子节点的状态。
  */
          List getChildren(String path,nollean watcher)
  /*
  给path设置数据，可以指定这个数据的版本号，如果version为-1则可以匹配任何版本
  */   
          Stat setData(String path,byte[] data,int version)
  /*
  获取这个path对应的目录节点存储的数据，数据的版本等信息可以通过stat来指定，同时还可以设置是否监控这个目录节点数据的状态
  */        
          byte[] getData(String path,boolean watcher,Stat stat)
  ```
  
- 例子

  ```java
  @Test
  public void test()throws Exception{
      //1.创建连接
      Zookeeper zooKeeper=new Zookeeper("192.168.36.134",2000,new Watcher(){
          public void process(WathcedEvent){
              System.out.println("触发了"watchedEvent.getType()+"的事件");
          }
      });
      //1.创建父节点
  String path=zooKeeper.create(
      "/itheima",
      "itheimaValue".getBytes(),
      ZooDefs.Ids.OPEN_ACL_UNSAFE,
      CreateMode.PERSISTENT);
      
      //3.创建子节点
      String childrenpath=zooKeeper.create( 
      "/itheima/children",
      "childrenValue".getBytes(),
      ZooDefs.Ids.OPEN_ACL_UNSAFE,
      CreateMode.PERSISTENT);
      
      //4.获取节点中的值（父节点和子节点）
      byte[] data=zooKeeper.getData("/itheima",false,null)
      System.out.println(new String(data));
      
      List<String> children=zooKeeper.getChildren("/itheima/children",false)
      for(String child:children){
         System.out.println(new String(child));
      }
      //5.修改节点中的值
      Stat stat=zooKeeper.setData("/itheima","itheimaUodate".getBytes(),-1);
      System.out.println(new String(stat));
          
      //6.判断某个节点是否存在
      Stat exists=zooKeeper.exists("/itheima/children",false);
      System.out.println(new String(exists));
          
      //7.删除节点
      zooKeeper.delete("/itheima/children",-1);
  }
  ```

  