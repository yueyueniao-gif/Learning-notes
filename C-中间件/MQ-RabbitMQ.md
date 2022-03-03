---
title: RabbitMQ
author: Wang Yue Niao
top: false
toc: true
date: 2020-12-22 14:48:58
tags: RabbitMQ
categories: RabbitMQ
---

# RabbitMQ

## MQ引言

#### 什么是MQ

MQ（Message Quene）：翻译为消息队列，通过典型的生产者和消费者模型，生产者不断向消息队列中生产消息，消费者不断从队列中获取消息。因为消息的生产和消费都是异步的，而且只关心消息的发送和接收，没有业务逻辑的侵入，轻松实现系统间解耦。别名为消息中间件。

#### MQ有哪些

```markdown
# 1.ActiveMQ
  ActiveMQ是Apache，最流行的，能力强劲的开源消息总线。它是一个完全支持JMS规范的消息中间件。丰富的API，多种集群架构模式让ActiveMQ在业界成为老牌的消息中间件，在中小型企业颇受欢迎。 
# 2.Kafka
  kafka是LinkedIn开源的分布式发布-订阅消息系统，目前归属于Apache顶级项目。kafka主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的二亩地就是用于日志收集和传输。0.8版本开始支持复制，不支持事务，对消息的重复、丢失、错误没有严格要求，适合产生大量数据的互联网服务的数据收集业务。
# 3.RoketMQ
  RoketMQ是阿里开源的消息中间件，它是存Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。RoketMQ思路起源于kafaka，但并不是kafaka的一个Copy，他对消息的可靠传输及事务性做了优化，目前在阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流处理、binglog分发等场景。
# 4.RabbitMQ
RabbitMQ是使用Erlang语言开发的开源消息队列系统，基于AMQP协议来实现。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性，安全。AMQP协议更多用在企业系统内对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次。
```

> RabbitMQ比Kafka可靠，Kfka更适合IO高吞吐的处理，一般应用在大数据日志处理货对实时性（少量延迟），可靠性（少量丢数据）要求稍低的场景使用，比如ELK日志收集。 

  ## 安装

### 1. 更新基本系统

安装任何软件包之前，建议使用以下命令更新软件包和存储库

```
yum -y update
```

### 2. 安装Erlang

由于RabbitMQ是基于**Erlang（面向高并发的语言）**语言开发，所以在安装RabbitMQ之前，需要先安装Erlang。在本教程中我们将安装最新版本的Erlang到服务器中。 Erlang在默认的YUM存储库中不可用，因此您将需要安装EPEL存储库。 运行以下命令相同。

```
yum -y install epel-release

yum -y update
```

安装`Erlang`

```
yum -y install erlang socat
```

您现在可以使用以下命令检查Erlang版本。

```
erl -version
```

您将得到如下输出：

```
[root@liptan-pc ~]# erl -version
Erlang (ASYNC_THREADS,HIPE) (BEAM) emulator version 5.10.4
```

### 2. 安装RabbitMQ

RabbitMQ为预编译并可以直接安装的企业Linux系统提供RPM软件包。 唯一需要的依赖是将Erlang安装到系统中。 我们已经安装了Erlang，我们可以进一步下载RabbitMQ。 通过运行下载Erlang RPM软件包。

#### 2.1 下载RabbitMQ

下载RabbitMQ

```
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm
```

如果你没有安装wget ，可以运行yum -y install wget 。 您可以随时找到最新版本的RabbitMQ下载页面的链接。

#### 2.2 安装RabbitMQ

通过运行导入GPG密钥：

```
rpm –import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
```

运行RPM安装RPM包：

```
rpm -Uvh rabbitmq-server-3.6.10-1.el7.noarch.rpm
```

RabbitMQ现已经安装在系统上。

#### 2.3 修改配置文件

```markdown
# 默认安装完成后的配置文件模板在:  /usr/share/doc/rabbitmq-server-3.6.10/rabbitmq.config.example目录中。需要将配置文件复制到/etc/rabbitmq/目录中，并修改名称为rabbitmq.config
  cp /usr/share/doc/rabbitmq-server-3.6.10/rabbitmq.config.example /etc/rabbitmq/rabbitmq.config

# 修改配置文件,如下图
  vim /etc/rabbitmq/rabbitmq.config
```

![kBPJ2.png](https://s3.jpg.cm/2020/12/22/kBPJ2.png)

#### 2.4 启动

```markdown
# 运行
systemctl start rabbitmq-server
# 开机自启
systemctl enable rabbitmq-server
# 检查状态
systemctl status rabbitmq-server

# 执行如下命令，启动rabbitmq中的插件管理
 rabbitmq-plugins enable rabbitmq_management
```

要访问RabbitMQ的管理面板，请使用您最喜爱的Web浏览器并打开以下URL。

```markdown
# 再次之前需要关闭防火墙
# 暂时关闭防火墙
systemctl stop firewalld
# 永久关闭防火墙
systemctl disable firewalld

http://Your_Server_IP:15672/
# 第一次访问需要登录，默认的账号密码为：guest/guest
```

## 配置

#### RabbitMQ管理命令行

```markdown
# 服务启动相关
systemctl start|restart|stop|status rabbitmq-server

# 管理命令行  用来在不使用web管理界面情况下用命令操作RabbitMQ
rabbitmqctl help 可以查看更多命令

# 插件管理命令行
rabbitmq-plugins enable|list|disable
```

#### web管理界面介绍

## RabbitMQ的普通java程序

#### AMQP协议的回顾

![k0wOH.png](https://s3.jpg.cm/2020/12/23/k0wOH.png)

#### 引入依赖

```xml
<dependency>
   <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.7.2</version>
</dependency>
```

#### 第一种模型（直连）

##### 生产者

```java
public static void main(String[] args)throws Exception{
    //创建连接mq的连接工厂对象
    ConnectionFactory connectionFactory=new ConnectionFactory();
    //设置连接rabbitmq的主机
    connectionFactory.setHost(192.168.1.106);
    //设置端口号
    connectionFactory.setPorts(5672);
    //设置连接哪个虚拟主机
    connectionFactory.setVirtualHost("/ems");
    //设置访问的用户名和密码
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");
    
    //获取连接对象
    Connection connection=connectionFactory.newConnection();
    //获取连接通道
    Channel channel=connection.createChannel();
    //通道绑定对应消息队列
    //参数1：队列名称 如果队列不存在自动创建
    //参数2：用来定义队列特性是否要持久化   true持久化队列  false不持久化队列
    //参数3：exclusive 是否独占队列      true独占队列    false不独占队列
    //参数4：是否在消费完成后自动删除队列   true自动删除    false不自动删除
    //参数5：额外附加参数
    channel.queueDeclare("hello",false,false,false,null);
    
    //发布消息
    //参数1：交换机名称
    //参数2：队列名称
    //参数3：传递消息额外设置
    //参数4：消息的具体内容
    channel.basicPublish("","hells",null,"hello rabbitmq".getBytes())
        
    channel.close();
    connrction.close();
} 
```

##### 消费者

```java
public static void main(String[] args)throws Exception{
    ConnectionFactory connectionFactory=new ConnectionFactory();
    connectionFactory.setHost(192.168.1.106);
    connectionFactory.setPorts(5672);
    connectionFactory.setVirtualHost("/ems");
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");
    
    //获取连接对象
    Connection connection=connectionFactory.newConnection();
    //获取连接中通道
    Channel channel=connection.createChannel();
    //通道绑定对象
    channel.queueDeclare("hello",false,false,false,null);
    //消费消息
    //参数1：消费哪个队列的消息  队列名称
    //参数2：开始消息的自动确认机制
    //参数3：消费时的回调接口
    channel.basicConsume("hello",true,new DefaultConsumer(channel){
        @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("new String(body)="+new String(body));
        }
    });
    channel.close();
    connrction.close();
} 
```

#### 第二种模型（work quene）

- 消费者平均分配生产者生产的消息

##### 生产者

```java
public static void main(String[] args)throws Exception{
    ConnectionFactory connectionFactory=new ConnectionFactory();
    connectionFactory.setHost(192.168.1.106);
    connectionFactory.setPorts(5672);
    connectionFactory.setVirtualHost("/ems");
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");
    
    //获取连接对象
    Connection connection=connectionFactory.newConnection();
    //获取连接中通道
    Channel channel=connection.createChannel();
    //通道绑定对应消息队列
    channel.queueDeclare("hello",true,false,false,null);
    
    //发布消息
    for(0=int i=0;i<10;i++){
         channel.basicPublish("","hells",null,(i+"hello rabbitmq").getBytes())
    }  
    channel.close();
    connrction.close();
} 
```

##### 消费者1

```java
public static void main(String[] args)throws Exception{
    ConnectionFactory connectionFactory=new ConnectionFactory();
    connectionFactory.setHost(192.168.1.106);
    connectionFactory.setPorts(5672);
    connectionFactory.setVirtualHost("/ems");
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");
    
    //获取连接对象
    Connection connection=connectionFactory.newConnection();
    //获取连接中通道
    Channel channel=connection.createChannel();
    //通道绑定对象
    channel.queueDeclare("hello",true,false,false,null);
    //消费消息
    channel.basicConsume("hello",true,new DefaultConsumer(channel){
        @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("消费者1号"+new String(body));
        }
    });
    channel.close();
    connrction.close();
} 
```

##### 消费者2

```java
public static void main(String[] args)throws Exception{
    ConnectionFactory connectionFactory=new ConnectionFactory();
    connectionFactory.setHost(192.168.1.106);
    connectionFactory.setPorts(5672);
    connectionFactory.setVirtualHost("/ems");
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");
    
    //获取连接对象
    Connection connection=connectionFactory.newConnection();
    //获取连接中通道
    Channel channel=connection.createChannel();
    //通道绑定对象
    channel.queueDeclare("hello",true,false,false,null);
    //消费消息
    channel.basicConsume("hello",true,new DefaultConsumer(channel){
        @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("消费者2号"+new String(body));
        }
    });
    channel.close();
    connrction.close();
} 
```

##### 手动确认，实现能者多劳

```java
 //参数1：确认队列中哪个具体消息；参数2是否开启多个消息同时确定
channel.basicAck(envelope.getDeliveryTag(),false);
```

```java
public static void main(String[] args)throws Exception{
    ConnectionFactory connectionFactory=new ConnectionFactory();
    connectionFactory.setHost(192.168.1.106);
    connectionFactory.setPorts(5672);
    connectionFactory.setVirtualHost("/ems");
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");
    
    //获取连接对象
    Connection connection=connectionFactory.newConnection();
    //获取连接中通道
    Channel channel=connection.createChannel();
    //通道绑定对象
    channel.queueDeclare("hello",true,false,false,null);
    //消费消息
    channel.basicConsume("hello",fasle,new DefaultConsumer(channel){
        @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("消费者2号"+new String(body));
        }
    });
    channel.close();
    connrction.close();
} 
```

#### 第三种模型（fanout）

- 可以有多个消费者。
- 每个消费者都有自己的queue。
- 每个队列都要绑定到Exchange（交换机）.
- 生产者发送的消息，只能发送到交换机，交换机来决定要发给哪个队列，生产者无法决定。
- 交换机把消息发送给绑定过的所有队列。
- 队列的消费者都能拿到消息。实现一条消息被多个消费者消费。

##### 生产者

```java

public static void main(String[] args)throws Exception{
    ConnectionFactory connectionFactory=new ConnectionFactory();
    connectionFactory.setHost(192.168.1.106);
    connectionFactory.setPorts(5672);
    connectionFactory.setVirtualHost("/ems");
    connectionFactory.setUsername("ems");
    connectionFactory.setPassword("ems");
    
    //获取连接对象
    Connection connection=connectionFactory.newConnection();
    //获取连接中通道
    Channel channel=connection.createChannel();
    
    //声明交换机   参数1：交换机名称  参数2：交换机类型  fanout广播类型
    channel.exchangeDeclare("logs","fanout"); //广播  一条消息多个消费者同时消费
    //发布消息
    channel.basicPublish("logs","",null,"fanout type message".getBytes());
    
    channel.close();
    connrction.close();
} 
```

##### 消费者1

```java
//绑定交换机
channel.exchangeDeclare("logs","fanout");
//创建临时队列
String queue=channel.queueDeclare(),getQueue();
//将临时队列绑定exchange
channel.queueBind(queue,"logs","");
//处理消息
channel.basicConsume(queue,true,new DefaultConsumei(channel){
     @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("消费者1号"+new String(body));
        }
});
```

##### 消费者2

```java
//绑定交换机
channel.exchangeDeclare("logs","fanout");
//创建临时队列
String queue=channel.queueDeclare().getQueue();
//将临时队列绑定exchange
channel.queueBind(queue,"logs","");
//处理消息
channel.basicConsume(queue,true,new DefaultConsumei(channel){
     @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("消费者2号"+new String(body));
        }
});
```

#### 第四种模型（Routing）

##### 1. Routing之订阅模型-Direct（直连）

> 在Fanout模式下，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

在Direct模型下：

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）。
- 消息的发送方在向Exchange发送消息时，也必须指定消息的Routingkey。
- Exhcange不再把消息交给每一个绑定的队列，而是根据消息的Routing key进行判断，只有队列的Routingkey与消息的Routingkey完全一致，才会收到消息。

###### 生产者

```java
//声明交换机   参数1：交换机名称  参数2：交换机类型  基于指令的Routing key转发
channel.exchangeDeclare("logs_direct","direct");
String key="info";
//发布消息
channel.basicPublish("logs_direct",key,null,("指定的route key"+key+"消息").getBytes());
```

###### 消费者

```java
//绑定交换机
channel.exchangeDeclare("logs_direct","direct");
//创建临时队列
String queue=channel.queueDeclare().getQueue();
//将临时队列绑定exchange,可以绑定多个
channel.queueBind(queue,"logs","info");
channel.queueBind(queue,"logs","error");
//处理消息
channel.basicConsume(queue,true,new DefaultConsumei(channel){
     @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("消费者"+new String(body));
        }
});
```

##### 2.Routing之订阅模式-Topic

> Topic类型的Exchange与Direct相比都是可以根据outingKey把消息路由道不同队列。只不过Topic类型Exchange可以让队列绑定RoutingKey的时候使用通配符！这种模型RoutingKey一般由一个或多个单词组成，多个单词之间以"."分割，例如item.insert。

```
# 通配符
   * (star)匹配不多不少恰好一个词
   # (hash)匹配一个或多个词
# 如：
   audit.#   匹配audit.irs.corporate或者aduit.irs等
   audit.*   只匹配audit.irs
```

###### 开发者

```java
//声明交换机   参数1：交换机名称  参数2：交换机类型  topic使用动态路由（通配符方式）
channel.exchangeDeclare("topics","topic");
String key="user.save"; //动态路由key
//发布消息
channel.basicPublish("logs_direct",key,null,("这是路由中的动态订阅模型"+key+"消息").getBytes());
```

###### 消费者

```java
//绑定交换机
channel.exchangeDeclare("topics","topic");
//创建临时队列
String queue=channel.queueDeclare().getQueue();
//将临时队列绑定exchange,可以绑定多个
channel.queueBind(queue,"logs","user.*");
//处理消息
channel.basicConsume(queue,true,new DefaultConsumei(channel){
     @override  //最后一个参数：消息队列中取出的消息
        public void handleDelivery(String consumerTag,
                                   Envelope envelope,
                                   AMQP.BasicProperties peoterties,
                                   byte[] body)throws Exception{
            System.out.println("消费者"+new String(body));
        }
});
```

## SpringBoot整合RabbitMQ

#### 搭建初始环境

**1. 引入依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**2. 配置配置文件**

```yml
spring:
  application:
    name: springboot_rabbitmq
  rabbitmq:
    host: 192.168.1.106
    port: 5672
    username: ems
    password: ems
    virtual-host: /ems
```

#### 第一种hello word模型

##### 生产者

```java
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    void contextLoads() {
        rabbitTemplate.convertAndSend("hello","hello world");
    }
```

##### 消费者

```java
@Component  //默认持久化  非独占  不自动删除
@RabbitListener(queuesToDeclare = @Queue("hello"))
public class HelloCustomer {
    @RabbitHandler
    public void receivel(String message){
        System.out.println("message=:"+message);
    }
}
```

#### 第二种work模型使用

##### 生产者

```java
    @Test
    public void testwork(){
        for (int i=0;i<10;i++){
            rabbitTemplate.convertAndSend("work",(i+"work模型"));
        }
    }
```

##### 消费者

```java
@Component
public class WorkCustomer {
    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void recrvice1(String message){
        System.out.println("message1=:"+message);
    }
    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void recrvice2(String message){
        System.out.println("message2=:"+message);
    }
}
```

#### Fanout广播模型

##### 生产者

```java
public void testFanout(){
        rabbitTemplate.convertAndSend("logs","","这是日志广播");
    }
```

##### 消费者

```java
@Component
public class FanoutCustomer {
    @RabbitListener(bindings =@QueueBinding(
            value = @Queue,//创建临时队列
            exchange = @Exchange(name="logs",type = "fanout")  //绑定交换机类型
    ))
    public void reveivel(String message){
        System.out.println("message1="+message);
    }
    @RabbitListener(bindings =@QueueBinding(
            value = @Queue,
            exchange = @Exchange(name="logs",type = "fanout")
    ))
    public void reveive2(String message){
        System.out.println("message2="+message);
    }
}
```

#### Route路由模式

##### 1.Direct

###### 生产者

```java
 public void testdirect(){
        rabbitTemplate.convertAndSend("directs","error","error的日志信息");
    }
```

###### 消费者

```java
@Component
public class DirectCustomer {
    @RabbitListener(bindings =@QueueBinding(
            value = @Queue,//创建临时队列
            exchange = @Exchange(name="directs",type = "direct"),  //绑定交换机类型
            key = {"info","error","debug"}
    ))
    public void reveivel(String message){
        System.out.println("message1="+message);
    }
    @RabbitListener(bindings =@QueueBinding(
            value = @Queue,//创建临时队列
            exchange = @Exchange(name="directs",type = "direct"),  //绑定交换机类型
            key = {"error"}
    ))
    public void reveive2(String message){
        System.out.println("message2="+message);
    }
}
```

##### 2. Topic

###### 生产者

```java
 @Test
    public void testTopic(){
        rabbitTemplate.convertAndSend("topics","user.save","user.save的日志信息");
    }
```

###### 消费者

```java
@Component
public class TopicCustomer {
    @RabbitListener(bindings =@QueueBinding(
            value = @Queue,//创建临时队列
            exchange = @Exchange(name="topics",type = "topic"),  //绑定交换机类型
            key = {"user.save","user.*"}
    ))
    public void reveivel(String message){
        System.out.println("message1="+message);
    }
    @RabbitListener(bindings =@QueueBinding(
            value = @Queue,//创建临时队列
            exchange = @Exchange(name="topics",type = "topic"),  //绑定交换机类型
            key = {"user.#"}
    ))
    public void reveive2(String message){
        System.out.println("message2="+message);
    }
}
```

## MQ的应用场景

#### 异步处理

> 场景说明：用户注册后，需要发注册邮件和注册短信，传统做法有两种 1.串行方式    2.并行方式

- **串行方式**：将注册信息写入数据库后，发送注册邮件，在发送注册短信，以上三个任务全部完成才返回给客户端。这有一个问题就是，邮件，短信，并不是必须的，他只是一个通知，而这种做法这客户端等待没有必要等待的东西。
- **并行方式**：将注册信息写入数据苦后，发送送邮件的同时，发送短信，以上三个任务返回后，返回给客户端，并行的方式提高处理的时间。
- **消息队列**：假设三个业务节点分别使用50ms，串行方式使用时间150ms，并行时间100ms。虽然并行已经提高处理时间，但是，前面说过，邮件和短信对我正常使用网站没有任何影响，刻画段没有必要等着其发送完成才显示注册成功，应该是写入数据库就返回。**消息队列**：引入消息队列后，把发送邮件，短信不是必须的业务逻辑异步处理。

#### 应用解耦

> 场景：双11是购物狂欢节，用户下单后，订单系统需要通知库存系统，传统的做法就是订单系统调用库存系统的接口。

这种做法有一个缺点：当库存系统出现故障时，订单就会失败。订单系统和库存系统高耦合。

**引入消息队列**

- 订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功。
- 库存系统：订阅下单的消息，获取下单消息，进行库操作。就算库存系统出现故障，消息队列也能保证消息的可靠传递，不会导致消息丢失。

#### 流量削峰

> 场景：秒杀活动，一般会因为流量过大，导致应用挂掉，为了解决这个问题，一般在应用前加入消息队列。

**作用：**

- 1. 可以控制活动人数，超过此一定阈值的订单直接丢弃。
- 2. 可以缓解短时间的高流量压垮应用。

**用户的请求，服务器收到之后，首先写入消息队列，假如消息队列长度超过最大值，则直接抛弃用户请求或跳转到错误页面**

**秒杀业务根据消息队列中的请求，在做后续处理**



## RbbitMQ的集群

### 集群架构

#### 普通集群（副本集群）

##### 架构图

![kjzdr.png](https://s3.jpg.cm/2020/12/24/kjzdr.png)

##### 集群搭建

```markdown
# 0.集群规划
   node1: 192.168.1.104  mq1  master 主节点
   node2: 192.168.1.105  mq2  repl1 副本节点
   node3: 192.168.1.106  mq3  repl2 副本节点
   
# 1.克隆三台机器主机名和ip映射
   vim /etc/host加入:
      192.168.1.104  mq1
      192.168.1.105  mq2
      192.168.1.106  mq3
   node1: vim /etc/hostname 加入: mq1
   node2: vim /etc/hostname 加入: mq2
   node3: vim /etc/hostname 加入: mq3
   
# 2.三个机器安装rabbitmq，并且同步cookie文件，在node1上执行
   scp /var/lib/rabbitmq/.erlang.cookie root@mq2:/var/lib/rabbitmq/
   scp /var/lib/rabbitmq/.erlang.cookie root@mq3:/var/lib/rabbitmq/
   
# 3.查看cookie是否一致
   node1: cat /var/lib/rabbitmq/.erlang.cookie
   node2: cat /var/lib/rabbitmq/.erlang.cookie
   node3: cat /var/lib/rabbitmq/.erlang.cookie
   
# 4.后台启动rabbitmq所有节点执行如下命令，启动成功访问管理界面
   rabbitmq-server -detached
   
# 5. 在node2和node3执行加入集群命令
   1.关闭      rabbitmqctl stop_app
   2.加入集群   rabbitmqctl join_cluster rabbit@mq1    
   3.启动服务   rabbitmqctl start_app
   
# 6.查看集群状态，任意节点执行:
   rabbitmqctl cluster_status
```

#### 镜像集群

##### 架构图

![ky3l2.png](https://s3.jpg.cm/2020/12/24/ky3l2.png)

##### 配置集群架构

```markdown
# 0.策略说明
   rabbitmqctel set_policy [-p <vhost>] [--priority <priority>] [--apply-to <apply-to>]<name> <pattern>    <definitition> 
   -p Vhost: 可选参数，针对指定vhost下的queue进行设置
   Name:        policy的名称
   Pattern:     queue的匹配模式（正则表达式）
   Definition:  镜像定义，包括三个部分ha-mode,ha-params,ha-sync-mode
                ha-mode:指明镜像队列的模式，有效值为all/exactly/nodes
                       all:表示集群中所有节点上进行镜像
                       exactly:表示在指定个数的节点上进行镜像，节点个数由ha-params指定
                       nodes:表示在指定的节点上进行镜像，节点名称通过ha-params指定
                ha-params:ha-mode模式需要用到的参数
                ha-sync-mode:进行队列中消息同步方法，有效值为automatic和manual
                priority:可选参数，policay的优先级
# 1.查看当前策略
   rabbitmqctl list_policies
# 2.添加策略
   rabbitmqctl set_policy ha-all '^hello' '{"ha-mode":"all","ha-sync-mode":"automatic"}'
   说明：策略正则表达式为"^"表示所有匹配所有队列名称 ^hello:匹配hello开头队列
# 3.删除策略
   rabbitmqctl clear_policy ha-all
```



