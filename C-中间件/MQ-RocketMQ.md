# 分布式消息队列RocketMQ

## 第一章 RokcetMQ概述

### 一、MQ概述

**1.MQ简介**

- MQ，	Message Queue，是一种提供消息服务的中间件，是一套提供了消息生产、存储、消费全过程API的软件系统。

**2.MQ用途**

- 限流销峰：MQ可以将超量请求暂存其中，以便系统后期可以慢慢进行处理，从而避免了请求的丢失或系统被压垮。
- 异步解耦：上游系统对下游系统的调用若为同步调用，则会大大降低系统的吞吐量与并发度，而且系统耦合度太高。而异步调用则会解决这些问题。所以两层之间若要实现由同步到异步的转化，一般性做法就是，在这两层之间添加一个MQ层。
- 数据收集：分布式系统会产生海量数据流，如业务日志、监控数据、用户行为等。针对这些数据流进行实时或批量采集汇总，然后对这些数据流进行大数据分析，这是当前互联网平台的必备技术。通过MQ完成此类数据收集是最好的选择。

### 二、RocketMQ概述

**RocketMQ是一个统一的处理消息引擎，轻量级的数据处理平台。**

- 低延迟，在高压下，1毫秒内的响应延迟超过99.6%。
- 高可用，具有跟踪和审核功能
- 万亿级消息容量保证
- 自最新的4.1版本以来，使用新的开放式分布式消息传递和流媒体标准
- 批量传输，多功能集成，以提高吞吐量
- 如果空间足够，可以在不损失性能的情况下存盘

## 第二章 RocketMQ的安装与启动

### 一、基本概念

 #### 1 消息（Message）

- 消息是指，消息系统所传输的物理载体，生产和消费数据的最小单位，没条消息必须属于一个主题。

#### 2 主题（Topic）

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220226165702.png)

- Topic表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。
- 一个生产者可以同时发送多种Topic的消息；而一个消费者只对某种特定的Topic感兴趣，即只可以订阅和消费一种Topic消息。

#### 3 标签（Tag）

- 为消息设置的标签，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效的保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费可以根据Tag实现对不同子主题的不同消费，实现更好的扩展性。

#### 4 队列（Queue）

- 存储消息的物理实体。一个Topic可以包含多个Queue，每个Queue中存放的就是该Topic的消息。一个Topic的Queue也被称为一个Topic中消息的分区。
- ![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220228162008.png)

#### 5 消息标识（MessageId/key）

- RocketMQ中每一个消息用友唯一的MessageId，且可以携带具有业务标识的Key，以方便对消息的查阅。不过需要注意的是，MesssageId有两个：在生产者send()消息时会自动生成一个MessageI的(msgId),当消息到达Broker后，Broker也会自动生成一个MessageId(offsetMsgId)。msgId、offsetMsgId与key都成为消息标识。
  - msgId：由produce端生产，其生成规则为：
    - producerIp+进程pid+MessageClientIDSetter类的ClassLoader的hashCod当前时间+AutomicInteger自增计数器。
  - offsetMsgId：由broker端生成，其生成规则为：
    - breokerIp+物理分区的offset
  - key：由用户指定的业务相关的唯一标识。 

### 二、系统架构

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220228190641.png)

#### 1. Producer

- 消息生产者，负责生产消息。Producer通过MQ的负载均衡模式选择相应的Broker集群队列进行消息投递，投递的过程支持快速失败并且低延迟。
- RocketMQ中消息生产者都是以生产者组（Producer Group）的形式出现的。生产者组是同一类生产者的集合，这类Producer发送相同Topic类型的消息。

>注意
>
>1. 一个生产者组可以同时生产多个Topic的消息

####  2. Consumer

- 消息消费者，负责消费消息。一个消费者会从Broker服务器中获取到消息，并对消息进行相关业务处理。
- RocketMQ中的消息消费者都是以消费者组（Consumer Group）的形式出现的，消费者组是同一类消费者的集合，这类Consumer消费的是同一个Topic类型的消息。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。
- ![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220228224218.png)
- 消费者组中Consumer的数量应该小于等于订阅Topic的Queue数量。如果超出Queue数量，则多出的Consumer将不能消费消息。
- ![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220228224400.png)
- 不过一个Topic类型的消息可以被多个消费者组同时消费

>注意
>
>1. 消费者组只能消费一个Topic的消息，不能同时消费多个Topic
>2. 一个消费者组中的消费者必须订阅完全相同的Topic

#### 3. 生产者组和消费者组

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220228230110.png)

**ProducerGroup（生产者组）：**

- 一个生产者组，代表一群topic相同 的Producer。即一个生产者组是同一类Producer的组合。
- 如图，Producer_1、Producer_2、Producer_3 为一个ProducerGroup，因为他们都订阅了topicA。同理Producer_4、Producer_5、Producer_6为另一个ProducerGroup，他们订阅了topicB。
- 如果Producer是TransactionMQProducer，则发送的是事务消息。如果节点1发送完消息后，消息存储到broker的Half Message Queue中，还未存储到目标topic的queue中时，此时节点1崩溃，则可以通过同一Group下的节点2进行二阶段提交，或回溯。

- 使用时，一个节点下，一个topic会对应一个producer

**ConsumerGroup（消费者组）：**

- 一个消费者组，代表着一群topic相同，tag相同（即逻辑相同）的Consumer。通过一个消费者组，则可容易的进行负载均衡以及容错
- 使用时，一个节点下，一个topic加一个tag可以对应一个consumer。一个消费者组就是横向上多个节点的相同consumer为一个消费组。

>首先分析一下producer。习惯上我们不会创建多个订阅了相同topic的Producer实例，因为一个Producer实例发送消息时是通过ExecutorService线程池去异步执行的，不会阻塞完全够用，如果创建了多个相同topic的Producer则会影响性能。
>而Consumer则不同。消息会在一topic下会细分多个tag，需要针对tag需要针对不同的tag创建多个消费者实例。

####  4. Name Server

##### 功能介绍

- NameServer是一个Broker与Topic路由的注册中心，支持Broker的动态注册与发现。

- 在MetaQ v1.0与v2.0版本中，依赖的仍是Zookerrper。从MetaQ v3.0，即RocketMQ开始去掉了Zookerrper依赖，是用来自己的NameServer。

主要包括两个功能：

- Broker管理：接受Broker集群的注册信息并且保存下来作为路由信息的基本数据；提供心跳检测机制，检查Broker是否还存活。
- 路由信息管理：每个NameServer中都保存着Broker集群的整个路由信息和用于客户端查询的队列信息。Producer和Conumser通过NameServer可以获取整个Broker集群的路由信息，从而进行消息的投递和消费。

##### 路由注册

- NameServer通常也是一集群的方式部署，不过，NameServer是无状态的，即NameServer集群中的各个节点间是无差异的，各个节点间相互不进行信息通讯。那各节点中的数据是如何进行数据同步的呢？在Broker节点启动时，轮询所属NameServer列表，于每个NameServer节点建立长链接，发起注册请求。在NameServer内部维护着一个Broker列表，用来动态存储Broker的信息。

>注意，这是与其他像zk，Eureka，Nacos等注册中心不同的地方。
>
>优点：NameServer集群搭建简单。
>
>缺点：对于Broker，必须明确指出所有NameServer地址。否则未指出的将不会去注册。也正因为如此，NameServer并不能随便扩容。因为，若Broker不重新配置，新增的NameServer对于Broker来说是不可见的，其不会像这个NameServer进行注册。
>
>

- Broker节点为了证明自己是活着的，为了维护与NameServer间的长连接，会将最新的信息以心跳包的方式上报给NameServer，每30秒就发送一次心跳。心跳中包含BrokerId、Broker地址、Broker名称、Broker所属集群名称等等。NameServer在接收到心跳包后，会更新心跳时间戳，记录这个Broker的最新存活时间。

##### 路由剔除

- 由于Broker关机、宕机或网络抖动等原因，NameServer没有收到Broker的心跳，NameServer可能会将其从Broker列表中剔除。
- NameServer中有一个定时任务，每隔10秒就会扫描一次Broker表，查看每一个Broker的最新心跳时间戳距离党建时间是否超过120秒，如果超过，则会判断Broker失效，然后将其从Broker列表中剔除。

##### 路由发现

- RokcetMQ的路由发现采用的是Pull模型，当Topic路由信息出现变化时，NameServer不会主动推送给客户端，而是客户端定时拉取主体最新的路路由。默认客户端每30秒会拉取一次最新的路由。

##### 客户端NameServer选择策略

> 这里的客户端是指Producer和Consumer

- 客户端在配置时必须要写上NameServer集群的地址，那么客户端到底连接的是哪个NameServer节点呢？客户端首先会首先选一个随机数，然后再与NameServer节点数量取模，此时得到的就是所要连接的节点索引，然后就会进行连接。如果连接失败，则会采用round-robin策略，逐个尝试去连接其他节点。
- 首先采用随机策略进行选择，失败后采用的是轮询策略。 

#### 5.Broker 

##### 功能介绍

- Broker充当着消息中转角色，负责存储消息，转发消息。Broker在RocketMQ系统中负责接收并存储从生产者发送来的消息，同时为消费者拉取请求做准备。Broker同时也存储这消息相关的元数据，包括消费者消费进度偏移offset、主题、队列等。

>Kafka0.8版本之后，offset石村爱Broker中的，之前版本是存放在Zookeeper中的。

##### 模块构成

 ![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220301194243.png)

- Remoting Module：整个Broker的实体，负责处理来自clients端的请求。而这个Broker实体由以下模块构成。
- Client Manager：客户端管理器。负责接收、解析客户端（Producer / Consumer）请求，管理客户端。例如，维护COnsumer的Topic订阅消息。
- Stroe Service：存储服务。停工方便简单的API接口，处理消息存储到物理硬盘和消息查阅功能。
- HA Service：高可用服务，提供Master Broker和Slave Broker之间的数据同步功能。
- Index Service：索引服务。根据特定的MEssage key，投递到Broker的消息进行索引服务，同时也提供根据Message Key对消息进行快速查询功能。

##### 集群部署

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220301195356.png)

- 为了增强 Borker性能与吞吐量，Broker一般都是以集群形式出现的。各集群节点中可能存放着相同Topic的不同Queue。不过，这里有个问题，如果某Broker节点宕机，如何保证数据不丢失？其实解决方案是，将每个Broker集群节点进行横向扩展，即将Broker节点再建为个HA集群，解决单点问题。
- Broker节点集群是一个主从集群，即集群中具有Master与Slave两种角色。Master负责处理读写操作请求，而Save仅负责度操作请求。一个Master可以包含多个Slave，但一个Slave只能隶属于一个Master。Master与Slave的关系是通过指定相同BrokerName、不同的BrokerId来确定的。BrokerId为0表示Master，非0表示Slave。每个Broker与NameServer集群中的所有节点简历长连接，定时注册Topic信息到所有NameServer。

#### 6. 工作流程

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220302003754.png)

##### 具体流程

1. 启动NameServer，NameServer启动后开始监听端口，等待Broker，Producer，Consumer连接。
2. 启动Broker时，Broker会与所有的NameServer建立并保持长连接，然后每30秒向NameServer定时发送心跳包。
3. 发送消息前，可以先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，当然，在创建Topic时会将Topic与Broker的关系写入到NameSerer中。不过，这步是可选的，也可以发送消息时自动创建Topic。
4. Producer发送消息。启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic所在的Broker；然后从队列列表中轮询选择一个队列，与队列所在的Broker建立长连接，进行消息的发送。
5. Consumer跟Proucer类似，跟其中一台NameServer建立长连接，获取当前订阅的Topic存在哪些Broker上，然后直接跟Broker建立长连接，然后根据算法策略获取到其所要消费的Queue，开始消费其中的消息。Consumer在获取到路由信息后，同样也会每30秒从NameSerer更新一次路由信息。不过不同于Producer的是，Consumer还会向Broker发送心跳，以确保Broker的存活状态。

##### Topic的创建模式

手动创建Topic有两种模式：

- 集群模式：该模式下的Topic在该集群中，所有Broker中的QUeue数量是相同的。
- Broker模式：该模式下创建的Topic创建的Topic在该集群中，每个Broker中的Queue数量可以不同。

**自动创建Topic时，默认采用的是Broker模式，会为每个Broker默认创建4个Queue。**

**读/写队列：**

- 从物理上来讲，读/写队列是同一个队列。所以，不存在读/写队列数据同步问题。读/写队列是逻辑上进行区分的概念。一般情况下，读/写队列数量是相通的。
- 例如，创建Topic时设置的写队列数量为8，读队列数量为4，此时系统会创建8个Queue，分别是0-7。Producer会将消息写入到这8个队列，但Consummer只会消费0-3这4个队列中的消息，4-7中的消息是不会背消费到的。
- 再如，创建Topic时设置的写队列数量为4，读队列数量为8，此时系统会创建8个Queue，分别是0-7。Producer会将消息写入到0-3这4个队列，但Consummer会消费这8个队列中的消息，4-7中是没有消息的。此时假设Consumer Group中包含两个Consumer，Consumer1消费0-3，而Consumer2消费4-7。但实际情况是Consumer是没有消息可消费的。

### 三、单机安装与启动

#### 1.下载安装

- 打开官网

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220302224014.png)

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220302224228.png)

- 下载到本地

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220302224253.png)

- 上传到服务器 opt/tools

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220302224404.png)

- 解压到服务器 opt/apps

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220302224530.png)

#### 2.修改初始内存

**修改runserver.sh**

使用vim命令打开bin/runserver.sh。现在将这些值修改如下：

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303150656.png)

**修改runbroker.sh**

使用vim命令打开bin/runbroker.sh。现在将这些值修改如下：

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303150719.png)

#### 3.启动

**启动NameServer**

```shell
  # 启动命令
  > nohup sh bin/mqnamesrv &
  # 查看是否启动成功
  > tail -f ~/logs/rocketmqlogs/namesrv.log
  # 出现以下代表启动成功
  The Name Server boot success...
```

**启动broker**

```shell
  # 启动命令  
  > nohup sh bin/mqbroker -n localhost:9876 &
  # 查看是否启动成功
  > tail -f ~/logs/rocketmqlogs/broker.log 
  # 出现以下代表启动成功
  The broker[%s, 172.30.30.233:10911] boot success...
```

#### 4.发送/接收消息测试

```shell
 # 在发送/接收消息之前，我们需要告诉客户端名称服务器的位置。RocketMQ 提供了多种方式来实现这一点。为简单起见，我们使用环境变量     NAMESRV_ADDR
 > export NAMESRV_ADDR=localhost:9876
 # 发送消息
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 # 显示发送成功
 SendResult [sendStatus=SEND_OK, msgId= ...
 # 消费消息
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 # 显示消费成功
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

#### 5.关闭server

```shell
# 先关闭boker
> sh bin/mqshutdown broker
# 显示关闭成功
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

# 在关闭nameserver
> sh bin/mqshutdown namesrv
# 显示关闭成功
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

### 四、控制台的安装与启动

#### 1.下载

下载地址：https://github.com/apache/rocketmq-externals/releases

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303162244.png)

#### 2.修改配置

修改其src/main/resources中的application.properties配置文件。

- 原来的端口号为8080，修改为一个不常用的
- 指定RocketMQ的name server地址

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303162334.png)

#### 3.添加依赖

在解压目录rocketmq-console的pom.xml中添加如下JAXB依赖。

> `JAXB`，Java Architechture for Xml Binding，用于XML绑定的Java技术，是一个业界标准，是一
> 项可以根据XML Schema生成Java类的技术。

```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-impl</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-core</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
```

#### 4.打包

在rocketmq-console目录下运行`maven`的打包命令。

*保证自己电脑里面运行环境装有maven*

```shell
mvn clean package -Dmaven.test.skip=true
```

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303164335.png)

他会在`rocketmq-console/target/`下生成编译后的jar文件

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303164756.png)

#### 5.启动

java -jar命令启动

java -jar rocketmq-console-ng-1.0.0

![image-20220303165113017](C:\Users\niaoniao\AppData\Roaming\Typora\typora-user-images\image-20220303165113017.png)

#### 6.访问

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303165852.png)

### 五、集群搭建理论

**针对broker集群**

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303215840.png)

#### 1.数据复制与刷盘策略

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220303221023.png)

**复制策略：**
复制策略是Broker的Master与Slave间的数据同步方式。分为同步复制与异步复制：

- 同步复制：消息写入master后，master会等待slave同步数据成功后才向producer返回成功ACK
- 异步复制：消息写入master后，master立即向producer返回成功ACK，无需等待slave同步数据成功

> 异步复制策略会降低系统的写入延迟，RT变小，提高了系统的吞吐量

**刷盘策略:**

刷盘策略指的是broker中消息的落盘方式，即消息发送到broker内存后消息持久化到磁盘的方式。分为同步刷盘与异步刷盘:

- 同步刷盘：当消息持久化到broker的磁盘后才算是消息写入成功。
- 异步刷盘：当消息写入到broker的内存后即表示消息写入成功，无需等待消息持久化到磁盘。

>1）异步刷盘策略会降低系统的写入延迟，RT变小，提高了系统的吞吐量
>2）消息写入到Broker的内存，一般是写入到了PageCache
>3）对于异步刷盘策略，消息会写入到PageCache后立即返回成功ACK。但并不会立即做落盘操作，而是当PageCache到达一定量时会自动进行落盘。 

#### 2.Broker集群模式

根据Broker集群中各个节点间关系的不同，Broker集群可以分为以下几类：

- 单Master
  只有一个broker（其本质上就不能称为集群）。这种方式也只能是在测试时使用，生产环境下不能使用，因为存在单点问题。

- 多Master
  broker集群仅由多个master构成，不存在Slave。同一Topic的各个Queue会平均分布在各个master节点上。

  - 优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，即使机器
    宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步
    刷盘一条不丢），性能最高；
  - 缺 点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅（不可消费），消息实时性会受到影响

  > 以上优点的前提是，这些Master都配置了RAID磁盘阵列。如果没有配置，一旦出现某Master宕机，则会发生大量消息丢失的情况。

- 多Master多Slave模式-异步复制

  - broker集群由多个master构成，每个master又配置了多个slave（在配置了RAID磁盘阵列的情况下，一个master一般配置一个slave即可）。master与slave的关系是主备关系，即master负责处理消息的读写请求，而slave仅负责消息的备份与master宕机后的角色切换。
  - 异步复制即前面所讲的复制策略中的异步复制策略，即消息写入master成功后，master立即向producer返回成功ACK，无需等待slave同步数据成功。
  - 该模式的最大特点之一是，当master宕机后slave能够自动切换为master。不过由于slave从master的同步具有短暂的延迟（毫秒级），所以当master宕机后，这种异步复制方式可能会存在少量消息的丢失问题。

  >Slave从Master同步的延迟越短，其可能丢失的消息就越少，
  >
  >对于Master的RAID磁盘阵列；若使用的也是异步复制策略，同样也存在延迟问题，同样也可能会丢失消息。
  >
  >但RAID阵列的秘诀是微秒级的（因为是由硬盘支持的），所以其丢失的数据量会更少。 

- 多Master多Slave模式-同步双写

  - 该模式是多Master多Slave模式的同步复制实现。所谓同步双写，指的是消息写入master成功后，master会等待slave同步数据成功后才向producer返回成功ACK，即master与slave都要写入成功后才会返回成功ACK，也即双写。
  - 该模式与异步复制模式相比，优点是消息的安全性更高，不存在消息丢失的情况。但单个消息的RT(响应时间)略高，从而导致性能要略低（大约低10%）。
  - 该模式存在一个大的问题：对于目前的版本，Master宕机后，Slave不会自动切换到Master（致命问题）。

- 最佳实践

  - 一般会为Master配置RAID10磁盘阵列，然后再为其配置一个Slave。即利用了RAID10磁盘阵列的高
    效、安全性，又解决了可能会影响订阅的问题。-------多M多S+RAID10阵列

  >1. RAID磁盘阵列的效率要高于Master-Slave集群。因为RAID是硬件支持的。也正因为如此，所以RAID阵列的搭建成本较高。
  >2. 多Master+RAID阵列，与多Master多Slave集群的区别是什么？
  >
  >  - 多Master+RAID阵列，其仅仅可以保证数据不丢失，即不影响消息写入，但其可能会影响到
  >    消息的订阅。但其执行效率要远高于多Master多Slave集群
  >  - 多Master多Slave集群，其不仅可以保证数据不丢失，也不会影响消息写入。其运行效率要低
  >    于多Master+RAID阵列

### 六、磁盘阵列RAID

#### 1. RAID历史

- 1988 年美国加州大学伯克利分校的 D. A. Patterson 教授等首次在论文 “A Case of Redundant Array ofInexpensive Disks” 中提出了 RAID 概念 ，即廉价冗余磁盘阵列（ Redundant Array of InexpensiveDisks ）。

- 由于当时大容量磁盘比较昂贵， RAID 的基本思想是将多个容量较小、相对廉价的磁盘进行有机组合，从而以较低的成本获得与昂贵大容量磁盘相当的容量、性能、可靠性。随着磁盘成本和价格的不断降低， “廉价” 已经毫无意义。因此， RAID 咨询委员会（ RAID Advisory Board, RAB ）决定用“ 独立 ” 替代 “ 廉价 ” ，于时 RAID 变成了独立磁盘冗余阵列（ Redundant Array of IndependentDisks ）。但这仅仅是名称的变化，实质内容没有改变。

> 内存：32m/6.4G（IBM 10.1G）

#### 2. RAID等级

- RAID 这种设计思想很快被业界接纳， RAID 技术作为高性能、高可靠的存储技术，得到了非常广泛的应用。 RAID 主要利用镜像、数据条带和数据校验三种技术来获取高性能、可靠性、容错能力和扩展性，根据对这三种技术的使用策略和组合架构，可以把 RAID 分为不同的等级，以满足不同数据应用的需求。

- D. A. Patterson 等的论文中定义了 RAID0 ~ RAID6 原始 RAID 等级。随后存储厂商又不断推出 RAID7、 RAID10、RAID01 、 RAID50 、 RAID53 、 RAID100 等 RAID 等级，但这些并无统一的标准。目前业界与学术界公认的标准是 RAID0 ~ RAID6 ，而在实际应用领域中使用最多的 RAID 等级是 RAID0 、RAID1 、 RAID3 、 RAID5 、 RAID6 和 RAID10。

- RAID 每一个等级代表一种实现方法和技术，等级之间并无高低之分。在实际应用中，应当根据用户的数据应用特点，综合考虑可用性、性能和成本来选择合适的 RAID 等级，以及具体的实现方式。

#### 3.关键技术

- **镜像技术**
  镜像技术是一种冗余技术，为磁盘提供数据备份功能，防止磁盘发生故障而造成数据丢失。对于 RAID而言，采用镜像技术最典型地的用法就是，同时在磁盘阵列中产生两个完全相同的数据副本，并且分布在两个不同的磁盘上。镜像提供了完全的数据冗余能力，当一个数据副本失效不可用时，外部系统仍可正常访问另一副本，不会对应用系统运行和性能产生影响。而且，镜像不需要额外的计算和校验，故障修复非常快，直接复制即可。镜像技术可以从多个副本进行并发读取数据，提供更高的读 I/O 性能，但不能并行写数据，写多个副本通常会导致一定的 I/O 性能下降。

- 镜像技术提供了非常高的数据安全性，其代价也是非常昂贵的，需要至少双倍的存储空间。高成本限制了镜像的广泛应用，主要应用于至关重要的数据保护，这种场合下的数据丢失可能会造成非常巨大的损失。

- **数据条带技术**
  将一个数据拆开，并行写给多个磁盘中；数据条带化技术是一种自动将 I/O操作负载均衡到多个物理磁盘上的技术。更具体地说就是，将一块连续的数据分成很多小部分并把它们分别存储到不同磁盘上。这就能使多个进程可以并发访问数据的多个不同部分，从而获得最大程度上的 I/O 并行能力，极大地提升性能。

- **数据校验技术**
  数据校验技术是指， RAID 要在写入数据的同时进行校验计算，并将得到的校验数据存储在 RAID 成员磁盘中。校验数据可以集中保存在某个磁盘或分散存储在多个不同磁盘中。当其中一部分数据出错时，就可以对剩余数据和校验数据进行反校验计算重建丢失的数据。

- 数据校验技术相对 于镜像技术的优势在于节省大量开销，但由于每次数据读写都要进行大量的校验运算，对计算机的运算速度要求很高，且必须使用硬件 RAID 控制器。在数据重建恢复方面，检验技术比镜像技术复杂得多且慢得多。

#### 4.RAID分类

从实现角度看， RAID 主要分为软 RAID、硬 RAID 以及混合 RAID 三种。

- 软 RAID
  所有功能均有操作系统和 CPU 来完成，没有独立的 RAID 控制处理芯片和 I/O 处理芯片，效率自然最低。软件完成

- 硬 RAID
  配备了专门的 RAID 控制处理芯片和 I/O 处理芯片以及阵列缓冲，不占用 CPU 资源。效率很高，但成本也很高。硬件完成

- 混合 RAID
  具备 RAID 控制处理芯片，但没有专门的I/O 处理芯片，需要 CPU 和驱动程序来完成。性能和成本在软RAID 和硬 RAID 之间。软硬完成

#### 5.常见RAID等级详解

- **JBOD**

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220304205109.png)

- JBOD ，Just a Bunch of Disks，磁盘簇。表示一个没有控制软件提供协调控制的磁盘集合，这是RAID区别与 JBOD 的主要因素。 JBOD 将多个物理磁盘串联起来，提供一个巨大的逻辑磁盘·。

- JBOD 的数据存放机制是由第一块磁盘开始按顺序往后存储，当前磁盘存储空间用完后，再依次往后面的磁盘存储数据。 JBOD 存储性能完全等同于单块磁盘，而且也不提供数据安全保护。

> 多块磁盘组成的逻辑磁盘；
>
> 其只是简单提供一种扩展存储空间的机制，JBOD可用存储容量等于所有成员磁盘的存储空间之和

JBOD 常指磁盘柜，而不论其是否提供 RAID 功能。不过，JBOD并非官方术语，官方称为Spanning。

- **RAID0**

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220304205213.png)

- RAID0 是 一种简单的、无数据校验的数据条带化技术。实际上不是一种真正的 RAID ，因为它并不提供任何形式的冗余策略 。 RAID0 将所在磁盘条带化后组成大容量的存储空间，将数据分散存储在所有磁盘中，以独立访问方式实现多块磁盘的并读访问。

- 理论上讲，一个由 n 块磁盘组成的 RAID0 ，它的读写性能是单个磁盘性能的 n 倍，但由于总线带宽等多种因素的限制，实际的性能提升低于理论值。由于可以并发执行 I/O 操作，总线带宽得到充分利用。再加上不需要进行数据校验，RAID0 的性能在所有 RAID 等级中是最高的。

- RAID0 具有低成本、高读写性能、 100% 的高存储空间利用率等优点，但是它不提供数据冗余保护，一旦数据损坏，将无法恢复。

- 应用场景：对数据的顺序读写要求不高，对数据的安全性和可靠性要求不高，但对系统性能要求很高的
  场景。

>RAID0与JBOD相同点：
>
>1）存储容量：都是成员磁盘容量总和
>
>2）磁盘利用率，都是100%，即都没有做任何的数据冗余备份
>
>RAID0与JBOD不同点：
>
>JBOD：数据是顺序存放的，一个磁盘存满后才会开始存放到下一个磁盘
>
>RAID：各个磁盘中的数据写入是并行的，是通过数据条带技术写入的。其读写性能是JBOD的n倍

- **RAID1**

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220304205309.png)

- RAID1 就是一种镜像技术，它将数据完全一致地分别写到工作磁盘和镜像磁盘，它的磁盘空间利用率为 50% 。 RAID1 在数据写入时，响应时间会有所影响，但是读数据的时候没有影响。 RAID1 提供了最佳的数据保护，一旦工作磁盘发生故障，系统将自动切换到镜像磁盘，不会影响使用。

- RAID1是为了增强数据安全性使两块磁盘数据呈现完全镜像，从而达到安全性好、技术简单、管理方便。 RAID1 拥有完全容错的能力，但实现成本高。

- 应用场景：对顺序读写性能要求较高，或对数据安全性要求较高的场景。

**RAID10**

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220304205350.png)

- AID10是一个RAID1与RAID0的组合体，所以`它继承了RAID0的快速和RAID1的安全`。

- 简单来说就是，`先做条带，再做镜像`。发即将进来的数据先分散到不同的磁盘，再将磁盘中的数据做镜像。

**RAID01**

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220304205416.png)

- RAID01是一个RAID0与RAID1的组合体，所以它继承了RAID0的快速和RAID1的安全。

- 简单来说就是，先做镜像,再做条带。即将进来的数据先做镜像，再将镜像数据写入到与之前数据不同的磁盘，即再做条带。

> RAID10要比RAID01的容错率再高，所以生产环境下一般是不使用RAID01的。

### 七、集群搭建实践

####  1、集群架构

这里要搭建一个双主双从异步复制的Broker集群。为了方便，这里使用了两台主机来完成集群的搭建。这两台主机的功能与broker角色分配如下表。

| 序号 | 主机名/IP       | IP              | 功能                | BROKER角色        |
| ---- | --------------- | --------------- | ------------------- | ----------------- |
| 1    | 192.168.237.128 | 192.168.237.128 | NameServer + Broker | Master1 和Slave2  |
| 2    | 192.168.237.129 | 192.168.237.129 | NameServer + Broker | Master2 和 Slave1 |

#### 2、修改192.168.237.128配置文件

- 配置文件位置

  - 要修改的配置文件在rocketMQ解压目录的`conf/2m-2s-async`目录中

  ![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220304231812.png)

  ![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220304231828.png)

- **修改broker-a.properties**----------Master机

  - 将该配置文件内容修改为如下：

    ```shell
    # 指定整个broker集群的名称，或者说是RocketMQ集群的名称
    brokerClusterName=DefaultCluster
    # 指定master-slave集群的名称。一个RocketMQ集群可以包含多个master-slave集群
    brokerName=broker-a
    # master的brokerId为0
    brokerId=0
    # 指定删除消息存储过期文件的时间为凌晨4点
    deleteWhen=04
    # 指定未发生更新的消息存储文件的保留时长为48小时，48小时后过期，将会被删除
    fileReservedTime=48
    # 指定当前broker为异步复制master
    brokerRole=ASYNC_MASTER
    # 指定刷盘策略为异步刷盘
    flushDiskType=ASYNC_FLUSH
    # 指定Name Server的地址
    namesrvAddr=192.168.237.128:9876;192.168.237.129:9876
    ```

- **修改broker-b-s.properties**-------Salve机

  - 将该配置文件内容修改为如下：

    ```shell
    brokerClusterName=DefaultCluster
    # 指定这是另外一个master-slave集群
    brokerName=broker-b
    # slave的brokerId为非0
    brokerId=1
    deleteWhen=04
    fileReservedTime=48
    # 指定当前broker为slave
    brokerRole=SLAVE
    flushDiskType=ASYNC_FLUSH
    namesrvAddr=192.168.237.128:9876;192.168.237.129:9876
    # 指定Broker对外提供服务的端口，即Broker与producer与consumer通信的端口。默认10911。由于当前主机同时充当着master1与slave2，而前面的master1使用的是默认端口。这里需要将这两个端口加以区分，以区分出master1与slave2
    listenPort=11911
    # 指定消息存储相关的路径。默认路径为~/store目录。由于当前主机同时充当着master1与slave2，master1使用的是默认路径，这里就需要再指定一个不同路径
    storePathRootDir=~/store-s
    storePathCommitLog=~/store-s/commitlog
    storePathConsumeQueue=~/store-s/consumequeue
    storePathIndex=~/store-s/index
    storeCheckpoint=~/store-s/checkpoint
    abortFile=~/store-s/abort
    ```

- **其它配置**

  - 除了以上配置外，这些配置文件中还可以设置其它属性。

    ```shell
    #指定整个broker集群的名称，或者说是RocketMQ集群的名称
    brokerClusterName=rocket-MS
    #指定master-slave集群的名称。一个RocketMQ集群可以包含多个master-slave集群
    brokerName=broker-a
    #0 表示 Master，>0 表示 Slave
    brokerId=0
    #多个nameServer地址，分号分割
    namesrvAddr=nameserver1:9876;nameserver2:9876
    #默认为新建Topic所创建的队列数
    defaultTopicQueueNums=4
    #是否允许 Broker 自动创建Topic，建议生产环境中关闭
    autoCreateTopicEnable=true
    #是否允许 Broker 自动创建订阅组，建议生产环境中关闭
    autoCreateSubscriptionGroup=true
    #Broker对外提供服务的端口，即Broker与producer与consumer通信的端口
    listenPort=10911
    #HA高可用监听端口，即Master与Slave间通信的端口，默认值为listenPort+1
    haListenPort=10912
    #指定删除消息存储过期文件的时间为凌晨4点
    deleteWhen=04
    #指定未发生更新的消息存储文件的保留时长为48小时，48小时后过期，将会被删除
    fileReservedTime=48
    #指定commitLog目录中每个文件的大小，默认1G
    mapedFileSizeCommitLog=1073741824
    #指定ConsumeQueue的每个Topic的每个Queue文件中可以存放的消息数量，默认30w条
    mapedFileSizeConsumeQueue=300000
    #在清除过期文件时，如果该文件被其他线程所占用（引用数大于0，比如读取消息），此时会阻止此次删除任务，同时在第一次试图删除该文件时记录当前时间戳。该属性则表示从第一次拒绝删除后开始计时，该文件最多可以保留的时长。在此时间内若引用数仍不为0，则删除仍会被拒绝。不过时间到后，文件将被强制删除
    destroyMapedFileIntervalForcibly=120000
    #指定commitlog、consumequeue所在磁盘分区的最大使用率，超过该值，则需立即清除过期文
    件
    diskMaxUsedSpaceRatio=88
    #指定store目录的路径，默认在当前用户主目录中
    storePathRootDir=/usr/local/rocketmq-all-4.5.0/store
    #commitLog目录路径
    storePathCommitLog=/usr/local/rocketmq-all-4.5.0/store/commitlog
    #consumeueue目录路径
    storePathConsumeQueue=/usr/local/rocketmq-all-4.5.0/store/consumequeue
    #index目录路径
    storePathIndex=/usr/local/rocketmq-all-4.5.0/store/index
    #checkpoint文件路径
    storeCheckpoint=/usr/local/rocketmq-all-4.5.0/store/checkpoint
    #abort文件路径
    abortFile=/usr/local/rocketmq-all-4.5.0/store/abort
    #指定消息的最大大小
    maxMessageSize=65536
    #Broker的角色
    # - ASYNC_MASTER 异步复制Master
    # - SYNC_MASTER 同步双写Master
    # - SLAVE
    brokerRole=SYNC_MASTER
    #刷盘策略
    # - ASYNC_FLUSH 异步刷盘
    # - SYNC_FLUSH 同步刷盘
    flushDiskType=SYNC_FLUSH
    #发消息线程池数量
    sendMessageThreadPoolNums=128
    #拉消息线程池数量
    pullMessageThreadPoolNums=128
    #强制指定本机IP，需要根据每台机器进行修改。官方介绍可为空，系统默认自动识别，但多网卡时IP地址可能读取错误
    brokerIP1=192.168.237.129
    ```

    

#### 3、修改192.168.237.129配置文件

对于192.168.237.129主机，同样需要修改rocketMQ解压目录的conf目录的子目录`2m-2s-async`中的两个配置文件。

- **修改broker-b.properties**-----Master机

  - 将该配置文件内容修改为如下：

    ```apl
    brokerClusterName=DefaultCluster
    brokerName=broker-b
    brokerId=0
    deleteWhen=04
    fileReservedTime=48
    brokerRole=ASYNC_MASTER
    flushDiskType=ASYNC_FLUSH
    namesrvAddr=192.168.237.128:9876;192.168.237.129:9876
    ```

- **修改broker-a-s.properties**---------Salve机

  - 将该配置文件内容修改为如下：

    ```shell
    brokerClusterName=DefaultCluster
    brokerName=broker-a
    brokerId=1
    deleteWhen=04
    fileReservedTime=48
    brokerRole=SLAVE
    flushDiskType=ASYNC_FLUSH
    namesrvAddr=192.168.109.104:9876;192.168.109.105:9876
    listenPort=11911
    storePathRootDir=~/store-s
    storePathCommitLog=~/store-s/commitlog
    storePathConsumeQueue=~/store-s/consumequeue
    storePathIndex=~/store-s/index
    storeCheckpoint=~/store-s/checkpoint
    abortFile=~/store-s/abort
    ```

#### 4、启动服务器

**启动NameServer集群**

- 分别启动192.168.237.128与192.168.237.129两个主机中的NameServer。启动命令完全相同。

```shell
nohup sh bin/mqnamesrv & tail -f ~/logs/rocketmqlogs/namesrv.log
```

**启动两个Master**

- 分别启动192.168.237.128与192.168.237.129两个主机中的broker master。注意，它们指定所要加载的配置

```shell
nohup sh bin/mqbroker -c conf/2m-2s-async/broker-a.properties &
tail -f ~/logs/rocketmqlogs/broker.log

nohup sh bin/mqbroker -c conf/2m-2s-async/broker-b.properties &
tail -f ~/logs/rocketmqlogs/broker.log
```

**启动两个Slave**

- 分别启动192.168.237.128与192.168.237.129两个主机中的broker slave。注意，它们指定所要加载的配置文
  件是不同的。

```shell
nohup sh bin/mqbroker -c conf/2m-2s-async/broker-b-s.properties &
tail -f ~/logs/rocketmqlogs/broker.log

nohup sh bin/mqbroker -c conf/2m-2s-async/broker-a-s.properties &
tail -f ~/logs/rocketmqlogs/broker.log
```

- *两台机器s4/s5，两个小集群，s4中broker-a|broker-b-s，s5中broker-b|broker-a-s组成一个rocket集群*

### 八、mqadmin命令

在mq解压目录的bin目录下有一个mqadmin命令，该命令是一个运维指令，用于对mq的主题，集群，broker 等信息进行管理。其实在控制台都可以完成，这个mqadmin命令了解一下就可以了。

#### 1、修改bin/tools.sh
- 在运行mqadmin命令之前，先要修改mq解压目录下bin/tools.sh配置的JDK的ext目录位置。本机的ext目录在/usr/local/java/jdk1.8.0_161/jre/lib/ext。

- 使用vim命令打开tools.sh文件，并在JAVA_OPT配置的-Djava.ext.dirs这一行的后面添加ext的路径。

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220305001441.png)

```shell
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn256m -
XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m"
JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib:${JAVA_HOME}/jre/lib/ext:${JAVA_HOME}/lib/ext:/usr/local/java/jdk1.8.0_161/jre/lib/ext"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"
```

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220305001524.png)

#### 2、运行mqadmin

直接运行该命令，可以看到其可以添加的commands。通过这些commands可以完成很多的功能。

```shell
[root@s4 bin]# ./mqadmin
The most commonly used mqadmin commands are:
   updateTopic          Update or create topic
   deleteTopic          Delete topic from broker and NameServer.
   updateSubGroup       Update or create subscription group
   deleteSubGroup       Delete subscription group from broker.
   updateBrokerConfig   Update broker's config
   updateTopicPerm      Update topic perm
   topicRoute           Examine topic route info
   topicStatus          Examine topic Status info
   topicClusterList     get cluster info for topic
   brokerStatus         Fetch broker runtime status data
   queryMsgById         Query Message by Id
   queryMsgByKey        Query Message by Key
   queryMsgByUniqueKey  Query Message by Unique key
   queryMsgByOffset     Query Message by offset
   QueryMsgTraceById    query a message trace
   printMsg             Print Message Detail
   printMsgByQueue      Print Message Detail
   sendMsgStatus        send msg to broker.
   brokerConsumeStats   Fetch broker consume stats data
   producerConnection   Query producer's socket connection and client version
   consumerConnection   Query consumer's socket connection, client version and subscription
   consumerProgress     Query consumers's progress, speed
   consumerStatus       Query consumer's internal data structure
   cloneGroupOffset     clone offset from other group.
   clusterList          List all of clusters
   topicList            Fetch all topic list from name server
   updateKvConfig       Create or update KV config.
   deleteKvConfig       Delete KV config.
   wipeWritePerm        Wipe write perm of broker in all name server
   resetOffsetByTime    Reset consumer offset by timestamp(without client restart).
   updateOrderConf      Create or update or delete order conf
   cleanExpiredCQ       Clean expired ConsumeQueue on broker.
   cleanUnusedTopic     Clean unused topic on broker.
   startMonitoring      Start Monitoring
   statsAll             Topic and Consumer tps stats
   allocateMQ           Allocate MQ
   checkMsgSendRT       check message send response time
   clusterRT            List All clusters Message Send RT
   getNamesrvConfig     Get configs of name server.
   updateNamesrvConfig  Update configs of name server.
   getBrokerConfig      Get broker config by cluster or special broker!
   queryCq              Query cq command.
   sendMessage          Send a message
   consumeMessage       Consume message
   updateAclConfig      Update acl config yaml file in broker
   deleteAccessConfig   Delete Acl Config Account in broker
   clusterAclConfigVersion List all of acl config version information in cluster
   updateGlobalWhiteAddr Update global white address for acl Config File in broker
   getAccessConfigSubCommand List all of acl config information in cluster

See 'mqadmin help <command>' for more information on a specific command.
```

#### 3、该命令的官网详解

该命令在官网中有详细的用法解释。

https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md

![](https://gitee.com/yueyueniao-gif/blogimage/raw/master/img/20220305001802.png)

## 第三章 RocketMQ工作原理









## 第四章 RocketMQ应用