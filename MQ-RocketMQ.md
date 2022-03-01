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

### 四、控制台的安装与启动

### 五、集群搭建理论

### 六、磁盘阵列RAID

### 七、集群搭建实践

### 八、mqadmin命令

## 第三章 RocketMQ工作原理









## 第四章 RocketMQ应用