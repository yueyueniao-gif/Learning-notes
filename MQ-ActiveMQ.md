---
title: ActiveMQ
author: Wang Yue Niao
top: false
toc: true
date: 2020-11-29 22:02:14
tags: MQ
categories: MQ
---

# ActiveMQ

## 下载安装

[官网](http://activemq.apache.org/)

![sJC6R.png](https://s3.jpg.cm/2020/11/29/sJC6R.png)

1. 上传到liunx的opt目录下![sJKEz.png](https://s3.jpg.cm/2020/11/29/sJKEz.png)

2. 然后使用tar -zxvf解压命令解压![sJhju.png](https://s3.jpg.cm/2020/11/29/sJhju.png)

3. 然后在根目录下创建一个文件夹myactivemq，将解压的activemq移动到根目录下面![sJ9t4.png](https://s3.jpg.cm/2020/11/29/sJ9t4.png)

4. 进入到/myactivemq/apache-activemq-5.16.0/bin目录下使用命令./activemq start启动activemq![sJT0X.png](https://s3.jpg.cm/2020/11/29/sJT0X.png)

5. 使用命令查看ps -ef|grep activemq![sJiID.png](https://s3.jpg.cm/2020/11/29/sJiID.png)

6. activemq默认后端端口是61616，可以使用命令查看进程号 lsof -i:61616![sJxUp.png](https://s3.jpg.cm/2020/11/29/sJxUp.png)

7. activemq默认前端端口号是8161，在windows本机上打开![sJZ6E.png](https://s3.jpg.cm/2020/11/30/sJZ6E.png)

8. 如果连接失败，检查防火墙和端口号是否打开，最后还是不行的话，

   需要去修改下“/opt/myActiveMQ/apache-activemq-5.16.0/conf/jetty.xml”这个文件里的配置修改如下

   ```xml
   <bean id="jettyPort" class="org.apache.activemq.web.WebConsolePort" init-method="start">
                <!-- the default port number for the web console -->
           <property name="host" value="0.0.0.0"/>
           <property name="port" value="8161"/>
       </bean>
   ```

   ![sJDo6.png](https://s3.jpg.cm/2020/11/29/sJDo6.png)

## 入门ActiveMQ

### 准备工作

```xml
   <!-- activemq所需要的jar包配置 -->
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-all</artifactId>
            <version>5.16.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.xbean</groupId>
            <artifactId>xbean-spring</artifactId>
            <version>3.16</version>
        </dependency>
```

### 消息生产者编码

1. ```java
   package com.atguigu.activemq;
   
   import org.apache.activemq.ActiveMQConnectionFactory;
   
   import javax.jms.*;
   
   /**
    * @ClassName:JmsProduce
    * @Description: TODO
    * @author: wang yue niao
    * @date 2020/11/29 21:01
    * @Version 1.0
    */
   public class JmsProduce {
       public static  final String ACTIVEMQ_ERL="tcp://192.168.36.134:61616";
       public static  final String QUEUE_NAME="queue01";
   
       public static void main(String[] args) throws JMSException {
           //1.创建连接工场,按照给定的url地址，采用默认用户名和密码(admin,admin)
           ActiveMQConnectionFactory am = new ActiveMQConnectionFactory(ACTIVEMQ_ERL);
           //通过连接工场,获得connection并启动访问
           Connection connection = am.createConnection();
           connection.start();
           //创建会话session
           //两个参数，第一个叫事务，第二个叫签收
           Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
           //创建目的地(具体是队列还是主题topic)
           Queue queue = session.createQueue(QUEUE_NAME);
           //创建消息的生产者
           MessageProducer messageProducer = session.createProducer(queue);
           //通过使用messageProducer产生三条小修发送到MQ的队列里面
           for (int i=1;i<=3;i++){
               //创建消息
               TextMessage textMessage = session.createTextMessage("msg---" + i);
               //通过消息生产者MessageProducer发送给mq
               messageProducer.send(textMessage);
           }
           //关闭资源
           messageProducer.close();
           session.close();
           connection.close();
           System.out.println("*****消息发发布到MQ完成");
       }
   }
   
   ```

2. 查看![stGFQ.png](https://s3.jpg.cm/2020/11/30/stGFQ.png)![stnLH.png](https://s3.jpg.cm/2020/11/30/stnLH.png)

3. 现在可以看到已经有三条消息发送过去了![stY7u.png](https://s3.jpg.cm/2020/11/30/stY7u.png)

### 消息消费者编码

**两种方式来创建消费者**

```java
package com.atguigu.activemq;

import lombok.SneakyThrows;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

/**
 * @ClassName:JmsConsumer
 * @Description: TODO
 * @author: wang yue niao
 * @date 2020/11/30 11:22
 * @Version 1.0
 */
public class JmsConsumer {
    public static  final String ACTIVEMQ_ERL="tcp://192.168.36.134:61616";
    public static  final String QUEUE_NAME="queue01";

    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("我是1号消费者");
        //1.创建连接工场,按照给定的url地址，采用默认用户名和密码(admin,admin)
        ActiveMQConnectionFactory am = new ActiveMQConnectionFactory(ACTIVEMQ_ERL);
        //通过连接工场,获得connection并启动访问
        Connection connection = am.createConnection();
        connection.start();
        //创建会话session
        //两个参数，第一个叫事务，第二个叫签收
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        //创建目的地(具体是队列还是主题topic)
        Queue queue = session.createQueue(QUEUE_NAME);
        //创建消费者
        MessageConsumer consumer = session.createConsumer(queue);
        /*
        同步阻塞方式(receive)
        订阅或接受者调用MessageConsumer的receive()方法来接受消息，receive方法在能够接受到消息之前(或超时之前)将一直阻塞
        while(true){
            TextMessage message =(TextMessage)consumer.receive();
            if(null!=message)
            {
                System.out.println("****消费者接收到消息"+message.getText());
            }else{
                break;
            }
        }
        consumer.close();
        session.close();
        connection.close();
        */
        //通过监听的方式来消费消息
        consumer.setMessageListener(new MessageListener() {
            @SneakyThrows
            @Override
            public void onMessage(Message message) {
                if(null!=message&&message instanceof TextMessage) {
                    try {
                        TextMessage textMessage = (TextMessage) message;
                        System.out.println("****消费者接收到消息" + textMessage.getText());
                    }catch (JMSException e){
                        e.printStackTrace();
                    }
                }
            }
        });
        System.in.read();
        consumer.close();
        session.close();
        connection.close();
    }
}

```

### JMS开发的基本步骤

![sSGWS.png](https://s3.jpg.cm/2020/11/30/sSGWS.png)

1. 创建一个connection factory
2. 通过connection factory来创建JMS connection
3. 启动JMS connection
4. 通过connection创建JMS session
5. 创建JMS destination
6. 创建JMS produce或者创建JMS message并设置destination
7. 创建JMS consumer或者是注册一个JMS message listener
8. 发送或者接受JMS message(s)
9. 关闭所有的JMS资源(connection,session,producer,consumer等)

### topic

1. 生产者将消息发布到topic中，每个消息可以有多个消费者，属于1:N的关系
2. 生产者和消费者之间时间上的相关性。订阅某一个主题的消费者只能消费自它订阅之后发布的消息
3. 生产者生产时，topic不保存消息它时无状态的不落地，假如无人订阅就去生产，那就是一条废消息，所以，一般先启动消费者在启动生产者

**生产者**

```java
package com.atguigu.activemq;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * @ClassName:JmsProduce_Topic
 * @Description: TODO
 * @author: wang yue niao
 * @date 2020/11/30     12:50
 * @Version 1.0
 */
public class JmsProduce_Topic {
    public static  final String ACTIVEMQ_ERL="tcp://192.168.36.134:61616";
    public static  final String TOPIC_NAME="topic01";

    public static void main(String[] args) throws JMSException {
        //1.创建连接工场,按照给定的url地址，采用默认用户名和密码(admin,admin)
        ActiveMQConnectionFactory am = new ActiveMQConnectionFactory(ACTIVEMQ_ERL);
        //通过连接工场,获得connection并启动访问
        Connection connection = am.createConnection();
        connection.start();
        //创建会话session
        //两个参数，第一个叫事务，第二个叫签收
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        //创建目的地(具体是队列还是主题topic)
        Topic  topic = session.createTopic(TOPIC_NAME);
        //创建消息的生产者
        MessageProducer producer = session.createProducer(topic);
        //通过使用messageProducer产生三条小修发送到MQ的队列里面
        for (int i=1;i<=6;i++){
            //创建消息
            TextMessage textMessage = session.createTextMessage("msg---" + i);
            //通过消息生产者MessageProducer发送给mq
            producer.send(textMessage);
        }
        //关闭资源
        producer.close();
        session.close();
        connection.close();
        System.out.println("*****消息发发布到MQ完成");
    }
}

```

**消费者**

```java
package com.atguigu.activemq;

import lombok.SneakyThrows;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

/**
 * @ClassName:JmsConsumer_Topic
 * @Description: TODO
 * @author: wang yue niao
 * @date 2020/11/30 14:44
 * @Version 1.0
 */
public class JmsConsumer_Topic {
    public static  final String ACTIVEMQ_ERL="tcp://192.168.36.134:61616";
    public static  final String TOPIC_NAME="topic01";

    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("我是1号消费者");
        //1.创建连接工场,按照给定的url地址，采用默认用户名和密码(admin,admin)
        ActiveMQConnectionFactory am = new ActiveMQConnectionFactory(ACTIVEMQ_ERL);
        //通过连接工场,获得connection并启动访问
        Connection connection = am.createConnection();
        connection.start();
        //创建会话session
        //两个参数，第一个叫事务，第二个叫签收
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        //创建目的地(具体是队列还是主题topic)
        Topic topic = session.createTopic(TOPIC_NAME);
        //创建消费者
        MessageConsumer consumer = session.createConsumer(topic);
        /*
        同步阻塞方式(receive)
        订阅或接受者调用MessageConsumer的receive()方法来接受消息，receive方法在能够接受到消息之前(或超时之前)将一直阻塞
        while(true){
            TextMessage message =(TextMessage)consumer.receive();
            if(null!=message)
            {
                System.out.println("****消费者接收到消息"+message.getText());
            }else{
                break;
            }
        }
        consumer.close();
        session.close();
        connection.close();
        */
        //通过监听的方式来消费消息
        consumer.setMessageListener(new MessageListener() {
            @SneakyThrows
            @Override
            public void onMessage(Message message) {
                if(null!=message&&message instanceof TextMessage) {
                    try {
                        TextMessage textMessage = (TextMessage) message;
                        System.out.println("****消费者接收到消息" + textMessage.getText());
                    }catch (JMSException e){
                        e.printStackTrace();
                    }
                }
            }
        });
        System.in.read();
        consumer.close();
        session.close();
        connection.close();

    }
}
```

### topic和队列的总结

| 比较项目   | Topic模式队列                                                | Queue模式队列                                                |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 工作模式   | “订阅-发布”模式，如果当前没有订阅者，消息将会被丢弃，如果有多个订阅者，那么这些订阅者都会受到消息 | “负载均衡”模式，如果当前没有消费者，消息也不会被丢弃；如果有多个消费者；那么一条消息也只会个发送给其中一个消费者，并且要求消费者ack消息 |
| 有无状态   | 无状态                                                       | Queue数据默认会在mq服务器上以文件形式保存                    |
| 传递完整性 | 如果没有订阅者，消息会被丢弃                                 | 消息不会被丢弃                                               |
| 处理效率   | 由于消息要按照订阅者的数量进行复制，所以处理性能会随着订阅者的增加而明显降低，并且还要结合不同消息协议自身的性能差异 | 由于一条消息只发送给一个消费者，所以就算消费者再多，性能也不会有明显的降低，当然不同消息协议的具体性能也是有差异的 |

## JMS

### 什么是JMS

Java Message Service（java消息服务是javaEE中的一个技术）

java消息服务是指两个应用程序之间进行异步通信的API，他为标准消息协议和消息服务提供了一组通用接口，包括创建，发送，读取消息等，用于支持JAVA应用程序开发。在JavaEE中，当两个应用程序使用JMS进行通信时，他们之间并不是直接相连的，而是通过一个公共的消息收发服务组件关联起来以达到解耦/异步削峰的效果。

### JMS的组成结构和特点

#### JMS provider

- 实现JMS接口和规范的消息中间件，也就是我们的MQ服务

#### JMS prodeucer

- 消息生产者，创建和发送JMS消息的客户端应用

#### JMS consumer

- 消息消费者，接收和处理JMS消息的客户端应用

#### JMS message

##### 消息头

- JMSDestination ：消息发送的目的地，主要指Queue和Topic
- JMSDdeliveryMode：持久和非持久模式
  - 一条持久性的消息：应该被传送“一次仅仅一次”，这就意味着如果JMS提供出现故障，该消息并不会丢失，他会在服务器恢复之后再次传递
  - 一条非持久的消息：最多会传送一次，这意味着服务器出现故障，该消息将永远丢失
- JMSExpiration：消息的过期时间
  - 可以设置消息在一定时间后过期，默认是永不过期
  - 消息过期时间，等与Destination的send方法中的timeToLive值加上发送时刻的GMT时间值
  - 如果timeToLive值等与零，则JMSExpiration被设为零，表示该消息永不过期
  - 如果发送后，在消息过期时间之后消息还没有被发送到目的地，则该消息被清除
- JMSPriority：消息优先级
  - 从0-9十个级别，0到4是普通消息，5-9是加急消息。
  - JMS不要求MQ严格按照这个十个优先级发送消息，但必须保证加急消息要先于普通消息到达。默认是4级
- JMSMessageID：唯一识别每个消息的标识由MQ产生

##### 消息体

- 封装具体的消息数据
- 5中消息体格式
  - TextMessage：普通字符串消息，包含一个String
  - MapMessage：一个Map类型消息，key为String类型，而值为java的基本类型
  - BytesMessage：二进制数组消息，包含一个byte[]
  - StreamMessage：java数据流消息，用标准流操作来顺序的填充和读取
  - ObjectMessage：对象消息，包含一个可序列化的java对象
- 发送和接收的消息体类型必须一致对应

##### 消息属性

- 如果需要除消息头字段以外的值，那么可以使用消息属性
- 识别/去重/重点标注等操作非常有用的方法
- 他们是属性名和属性值对的形式制定的。可以将属性是为消息头得扩展，属性制定一些消息头没有包括的附加信息，比如可以在属性里指定消息选择器。
- 消息属性就像可以分配给一条消息的附加消息头一样。它们允许开发者添加有关消息的不透明附加信息。它们还用于暴露消息选择器在消息过滤时使用的数据。
- ![srRTU.png](https://s3.jpg.cm/2020/11/30/srRTU.png)

### JMS的可靠性

#### PERSISTENT：持久性

##### 参数说明

- 非持久
  - messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT)
  - 非持久化：当服务器宕机，消息不存在

- 持久
  - messageProducer.setDeliveryMode(DeliveryMode.PERSISENT)
  - 持久化：当服务器宕机，消息依然存在

##### 持久的Queue

- 队列默认的是**持久化消息**传送模式，此模式保证这些消息只被传送一次和成功使用一次。

##### 持久的Topic

- 效果：

  - 运行生产者，发布消息，多个消费者可以正常收到
  -  关闭一个消费者，运行生产者，发布消息后再启动被关闭的消费者，可以收到离线后的消息；
  - 关闭所有消费者，运行生产者，发布消息后，关闭ActiveMQ再启动，启动所有消费者，都可以收到消息。

- 代码

  - ```java
    package com.atguigu.activemq;
    
    import org.apache.activemq.ActiveMQConnectionFactory;
    
    import javax.jms.*;
    
    /**
     * @ClassName:JmsProduce_Topic
     * @Description: TODO
     * @author: wang yue niao
     * @date 2020/11/30     12:50
     * @Version 1.0
     */
    public class JmsProduce_Topic {
        public static  final String ACTIVEMQ_ERL="tcp://192.168.36.134:61616";
        public static  final String TOPIC_NAME="topic01";
    
        public static void main(String[] args) throws JMSException {
            ActiveMQConnectionFactory am = new ActiveMQConnectionFactory(ACTIVEMQ_ERL);
            Connection connection = am.createConnection();
    
            Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
            Topic  topic = session.createTopic(TOPIC_NAME);
            MessageProducer producer = session.createProducer(topic);
            producer.setDeliveryMode(DeliveryMode.PERSISTENT);
            connection.start();
            for (int i=1;i<=6;i++){
                TextMessage textMessage = session.createTextMessage("msg---" + i);
                producer.send(textMessage);
            }
            producer.close();
            session.close();
            connection.close();
            System.out.println("*****消息发发布到MQ完成");
        }
    }
    
    ```

  - ```java
    package com.atguigu.activemq;
    
    import lombok.SneakyThrows;
    import org.apache.activemq.ActiveMQConnectionFactory;
    
    import javax.jms.*;
    import java.io.IOException;
    
    /**
     * @ClassName:JmsConsumer_Topic
     * @Description: TODO
     * @author: wang yue niao
     * @date 2020/11/30 14:44
     * @Version 1.0
     */
    public class JmsConsumer_Topic {
        public static  final String ACTIVEMQ_ERL="tcp://192.168.36.134:61616";
        public static  final String TOPIC_NAME="topic01";
    
        public static void main(String[] args) throws JMSException, IOException {
            System.out.println("我是zs消费者");
            ActiveMQConnectionFactory am = new ActiveMQConnectionFactory(ACTIVEMQ_ERL);
            Connection connection = am.createConnection();
            connection.setClientID("zs");
    
            Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
            Topic topic = session.createTopic(TOPIC_NAME);
            TopicSubscriber topicSubscriber = session.createDurableSubscriber(topic, "remark..");
            connection.start();
            Message message=topicSubscriber.receive();
            while (null!=message){
                TextMessage textMessage=(TextMessage)message;
                System.out.println("收到持久化topic："+textMessage.getText());
                message = topicSubscriber.receive(1000L);
            }
            session.close();
            connection.close();
        }
    }
    
    ```

#### 事务

- false

  - 只要执行send，就进入到队列中。
  - 关闭事务，那第二个签收参数的设置需要有效。

- true

  - 先执行send再执行commit，消息才被真正的提交到队列中。

    ```java
    session.commit();
    ```

    

  - 消息需要批量发送，需要缓冲区处理。

  - 如果一次提交十条消息，一条出错了，可以回滚

    ```java
    try{
        //ok
    }catch{
        session.rollback();
    }finally{
        if(null!session){
            session.close();
        }
    }
    ```

#### Acknowledge：签收

- 非事务
  - 自动签收（默认）Session.AUTO_ACKNOWLEDGE
  - 手动签收 Session.CLIENT_ACKNOWLEDGE
    - 客户端调用acknowledge方法手动签收   message.acknowledge
  - 允许重复消息的签收 Session.DUPS_OK_ACKNOWLEDGE
- 事务
  - 生产事务开启，只有commit后才能将全部消息变为已消费
  - 事务开启，则默认为自动签收，无论第二个签收参数写什么都是相当于自动签收
  - 如果开启事务，最后不session.commit的话会重复消费，而且消息不会被确认
- 签收和事务的关系
  - 在事务会话中，当一个事务被成功提交则消息被自动签收。如果事务回滚，则消息会被再次传送。
  - 非事务会话中，消息何时 被确认取决于创建会话时的应答模式是自动签收还是手动签收等等。

#### JMS点对点总结

- 点对对模型是基于队列的，生产者发消息到队列，消费者从队列接收消息，队列的存在使得消息的异步传输成为可能。和我们平时给朋友发送短信类似。
- 如果在Session关闭时所有部分消息已被收到但还没有被签收，那消费者下次连接到相同的队列时，这些消息还会被再次接收。
- 队列可以长久地保存消息知道消费者收到消息。消费者不需要因为担心消息会丢失而时刻和队列保持激活的连接状态，充分体现了**异步传输模式的优势**。

#### JMS发布订阅总结

- JMS Pub/Sub 模型定义了如何向一个内容节点发布和订阅消息，这些节点被称作topic。
- 主题可以被认为是消息的传输中介，发布者发布消息到主题，订阅者从主题订阅消息。
- 主题使得消息订阅者发布者保持回想独立，不需要解除即可保证消息的传送。

##### 非持久订阅

- 非持久订阅只有当客户端处于激活状态，也就是和MQ保持连接状态才能收到发送到某个主题的消息。

- 如果消费者处于离线状态，生产者发送的主题消息将会丢失作废，消费者永远不会收到

  **一句话：要先订阅注册才能接受到发布，只给订阅者发布消息**

##### 持久订阅

- 客户端首先向MQ注册一个自己的身份ID识别号，当这个客户端处于离线时，生产者会为这个ID保存所有发送到主 题的消息，当客户再次连接到MQ时会根据消费者的ID得到所有当自己离线时发送到主题的消息。
- 非持久订阅状态下，不能回复或重新派送一个未签收的消息。
- 持久订阅才能回复或重新派送一个未签收的消息。

##### 用哪个？

- 当所有消息必须被接受，则用持久订阅。当丢失消息能够被容忍，则用非持久订阅。

## SpringBoot整合ActiveMQ

### 队列生产者

#### 1.新建maven工程并设置包名类名

- 工程名  boot_mq_produce
- 包名  com.atguigu.boot.activemq

#### 2.POM.xml

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath/>
</parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
            <version>2.1.5.RELEASE</version>
        </dependency>
    </dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

#### 3.application.yml

```yml
server:
  port: 8080
spring:
  activemq:
    broker-url: tcp://192.169.36.134:61616 #自己的mq服务器地址
    user: admin
    password: admin
  jms:
    pub-sub-domain: false    #false=Queue    true=Topic



#自己定义队列名称
myqueue: boot-activemq-queue


```

#### 4.配置Bean

```java
@Component
@EnableJms
public class ConfingBean {
    @Value("${myqueue}")
    private String myQueue;
    @Bean
    public Queue queue(){
        return new ActiveMQQueue(myQueue);
    }
}
```

#### 5.Queue_Produce

```java
@Component
public class Queue_Produce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    @Autowired
    private Queue queue;

    public void prodeuceMsg(){
        jmsMessagingTemplate.convertAndSend(queue,"*******"+ UUID.randomUUID().toString().substring(0,6));
    }
}
```

#### 6.主启动类MainApp_Prodeuce

```java
@SpringBootApplication
public class MainApp_Prodeuce {
    public static void main(String[] args) {
        SpringApplication.run(MainApp_Prodeuce.class,args);
    }
}

```

#### 7.单元测试

```java
@SpringBootTest(classes = MainApp_Prodeuce.class)
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
public class TestActivemq {
    @Resource
    private Queue_Produce queue_produce;

    @Test
    public void testsend()throws  Exception{
        queue_produce.prodeuceMsg();
    }
}

```

### 队列消费者

#### 1.新建maven工程并设置报名类名

- 工程名  boot_mq_consumer
- 包名   com.atguigu.boot.activemq

#### 2.POM.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.wangyueniao</groupId>
    <artifactId>boot_mq_consumer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
            <version>2.1.5.RELEASE</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

#### 3.application.yml

```yml
server:
  port: 8081

spring:
  activemq:
    broker-url: tcp://192.168.36.134:61616 #自己的mq服务器地址
    user: admin
    password: admin
  jms:
    pub-sub-domain: false    #false=Queue    true=Topic



#自己定义队列名称
myqueue: boot-activemq-queue


```

#### 4.Queue_Consumer

```java
public class Queue_Consumer {
    @JmsListener(destination = "${myqueue}")
    public void receive(TextMessage textMessage)throws Exception{
        System.out.println("******消费者收到消息"+textMessage.getText());
    }
}

```

#### 5.主启动类

```java
@SpringBootApplication
public class MainApp_Consumer {
    public static void main(String[] args) {
        SpringApplication.run(MainApp_Consumer.class,args);
    }
}

```

### Topic生产者

#### 1.新建maven工程斌设置包名和类名

- 工程名  boot_mq_topic_produce
- 包名   com.atguigu.boot.activemq.topic.produce

#### 2.POM.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.wangyueniao</groupId>
    <artifactId>boot_mq_topic_produce</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
            <version>2.1.5.RELEASE</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>

```

#### 3.application.yml

```yml
server:
  port: 8080

spring:
  activemq:
    broker-url: tcp://192.168.36.134:61616    #自己服务器地址
    user: admin
    password: admin
  jms:
    pub-sub-domain: true   #值表示topic


#自己定义topic的名字
myTopic: boot-activemq-topic

```

#### 4.配置Bean

```java
@Component
@EnableJms
public class ConfigBean {
    @Value("myTopic")
    private String topicName;
    @Bean
    public Topic topic(){
        return new ActiveMQTopic(topicName);
    }
}

```

#### 5.Topic_Produce

```java
@Component
public class Topic_Produce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    @Autowired
    private Topic topic;

    public void produceTopic(){
        jmsMessagingTemplate.convertAndSend(topic,"主题消息：123456");
    }
}
```

### Topic消费者

#### 1.新建maven工程并设置包名类名

- 工程名   boot_mq_topic_consumer
- 包名   com.atguigu.boot.activemq.topic.consumer

#### 2.POM.xml

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
            <version>2.1.5.RELEASE</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```



#### 3.application.yml

```yml
server:
  port: 8082

spring:
  activemq:
    broker-url: tcp://192.168.36.134:61616    #自己服务器地址
    user: admin
    password: admin
  jms:
    pub-sub-domain: true   #值表示topic


#自己定义topic的名字
myTopic: boot-activemq-topic

```

#### 4.Topic_Consumer

```java
@Component
public class Topic_Consumer {
    @JmsListener(destination = "${myTopic}")
    public void receive(TextMessage text)throws JMSException{
        try{
            System.out.println("消费者收到订阅的主题："+text.getText());
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```

## ActiveMQ的传输协议

### 面试题

- 默认61616端口号如何修改

- 你生产上的链接协议如何配置？使用tcp吗？

### 官网

http://activemq.apache.org/configuring-version-5-transports.html

### 是什么

- 打开conf目录下的activemq.xml可以看到如下
  - ![EGug6.png](https://s3.jpg.cm/2020/12/02/EGug6.png)
  - 在上文给出的配置信息中，URL描述信息的头部都是采用协议名称：例如，
    - 描述amqp协议的监听端口时，采用的是URL描述格式为“amqp://......”
    - 描述Stomp协议的监听端口时，采用的是URL描述格式为“stomp://......”
    - 唯独在进行openwire协议描述时，URL头却采用的“tcp://......”，这是因为ActiveMQ中默认协议就是openwire。

### 有哪些

- Transmission Control Protocol(TCP)我是默认
  - 这是默认的Broker配置，TCP的Client监听端口61616
  - 在网络传输数据前，必须要序列化数据，消息是通过一个叫wire protocol的来序列化成字节流。默认情况下ActiveMQ把wire protocol叫做OpenWire，它的目的是促使网络上的效率和数据快速交互。
  - TCP连接的RUL形式如下：tcp://hostname:port?key=value&key=value,后面参数是可选
  - TCP传输的优点：
    - 传输可靠性强，稳定性强
    - 高效性：字节流方式传递，效率很高
    - 有效性、可用性：应用广泛，支持任何平台
- New I/O API Protocol(NIO)
  - NIO协议和TCP协议类似但NIO更侧重于底层的访问操作。它允许开发人员对同一资源有更多的client调用和服务端与更多负载。
  - 适合使用NIO协议的场景：
    - 可能有大量的client去连接到Broker，一般情况下，大量的Client去连接Broker是被操作系统线程限制的，因此，NIO的实现比TCP需要更少的线程去运行，所以建议使用NIO协议
    - 可能对于Broker有一个儿很吃顿的网络传输，NIO比TCP提供更好的性能。
  - NIO连接的URL形式：nio//hostname:port?key=value
- AMQP协议
- stomp协议
- Secure Sockerts Layer Protocol(SSL)
- mqtt协议
- ws协议
- 小总结

### NIO案例演示

```xml
<transportConnectors>
    <transportConnector name="nio" uri="nio://0.0.0.0:61618?trace=true"/>
</transportConnectors>


如果你不特别的指定ActiveMQ的网络监听端口，那么这些端口豆浆使用BIO网络IO模型。（OpenWire，STOMP，AMQP....）
所以为了首先提高单节点的网络吞吐性能，我们需要明确指定Active的网络IO模型，
如下图所示：URL格式头以"nio"，表示这个端口使用以TCP协议为基础的NIO网络IO模型。
```

![EGxh4.png](https://s3.jpg.cm/2020/12/02/EGxh4.png)

- 代码如下，其实就是将tcp改成nio，并且端口配置到nio的端口

```java
package com.atguigu.activemq;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * @ClassName:JmsProduce
 * @Description: TODO
 * @author: wang yue niao
 * @date 2020/11/29 21:01
 * @Version 1.0
 */
public class JmsProduce {
    public static  final String ACTIVEMQ_ERL="nio://192.168.36.134:61618";
    public static  final String QUEUE_NAME="transport";

    public static void main(String[] args) throws JMSException {
        //1.创建连接工场,按照给定的url地址，采用默认用户名和密码(admin,admin)
        ActiveMQConnectionFactory am = new ActiveMQConnectionFactory(ACTIVEMQ_ERL);
        //通过连接工场,获得connection并启动访问
        Connection connection = am.createConnection();
        connection.start();
        //创建会话session
        //两个参数，第一个叫事务，第二个叫签收
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        //创建目的地(具体是队列还是主题topic)
        Queue queue = session.createQueue(QUEUE_NAME);
        //创建消息的生产者
        MessageProducer producer = session.createProducer(queue);
        //通过使用messageProducer产生三条小修发送到MQ的队列里面
        for (int i=1;i<=4;i++){
            //创建消息
            TextMessage textMessage = session.createTextMessage("msg---" + i);
            //通过消息生产者MessageProducer发送给mq
            producer.send(textMessage);
        }
        //关闭资源
        producer.close();
        session.close();
        connection.close();
        System.out.println("*****消息发发布到MQ完成");
    }
}
```

### NIO案例演示增强

- 问题
  - URL格式以"nio"开头，表示端口使用以TCP协议为基础的NIO网络IO模型。但是这样的设置方式，只能使这个端口支持Openwire协议。
- 怎么既然这个端口支持NIO网络IO模型，又让它支持多个协议呢？
- 解决
  - 使用auto关键字
  - 使用"+"符号来为端口设置多种特性
  - 如果即需要某一个端口支持NIO网络IO模型，又需要它支持多个协议

## ActiveMQ的消息存储和持久化

- 为了避免意外的宕机后丢失信息，需要做到重启后可以回复消息队列，消息系统一般都会采用持久化机制。
- ActiveMQ的消息持久化机制有JDBC,AMQ,KahaDB,和LevelDB，无论使用哪种持久化方式，消息的存储逻辑都是一致的。
- 就是在发送者将消息发送出去后，消息中心首先将消息存储到本地数据文件，内存数据库或者远程数据库等再视图将消息发送给接收者，成功则将消息从存储中删除，失败则继续尝试发送
- 消息中心启动后，首先要检查指定的存储位置，如果有未发送成功的消息，则需要把消息发送出去。

### KahaDB消息存储（默认）

- 基于日志文件，从ActiveMQ5.4开始默认的持久化插件、
- 说明
  - KahaDB时目前默认的存储方式，可以用于任何场景，提高了性能和回复能力。
  - 消息存储使用一个十五日志和仅仅用一个索引文件来存储它所有地址
  - KahaDB是一个专门针对消息持久化的解决方案，它对典型的消息使用模式进行了优化
  - 数据被追加到data logs中。当不需要log文件中的数据的时候，log文件会被丢弃
  - [![ELDfp.md.png](https://s3.jpg.cm/2020/12/03/ELDfp.md.png)](https://imagelol.com/image/ELDfp)
- KahaDB的存储原理
  - KahaDB在消息保存目录中只有4类文件和一个lock，跟ActiveMQ的其它几种文件存储引擎相比这就非常简洁了
  - ![ELMZ6.png](https://s3.jpg.cm/2020/12/03/ELMZ6.png)
  - ![EPGpE.png](https://s3.jpg.cm/2020/12/03/EPGpE.png)
  - ![EPPdh.png](https://s3.jpg.cm/2020/12/03/EPPdh.png)
  - ![EPqKW.png](https://s3.jpg.cm/2020/12/03/EPqKW.png)
  - ![EPRrS.png](https://s3.jpg.cm/2020/12/03/EPRrS.png)
- [官网传送门]( http://activemq.apache.org/persistence.html)

### JDBC消息存储

- **1. 将数据库的驱动jar包和你想使用的数据库连接池的jar包复制到/myactivemq/apache-activemq-5.16.0/lib目录下，这里默认使用的是dbcp数据库连接池**

- **2. 修改activemq.xml文件如下**

![EWZ0W.png](https://s3.jpg.cm/2020/12/03/EWZ0W.png)

- **3. 数据库连接池-配置（放在</broker>之外和<import>里面，也就是他俩之间）**

![image-20201203162642153](C:\Users\LENOVO\AppData\Roaming\Typora\typora-user-images\image-20201203162642153.png)

- **4. 建一个名为activemq的数据库**

  - 三张表的说明

    - AACTIVEMQ_MSGS

      ![EnRxf.png](https://s3.jpg.cm/2020/12/03/EnRxf.png)

    - ACTIVEMQ_ACKS

      ![EnnEO.png](https://s3.jpg.cm/2020/12/03/EnnEO.png)

    - ACTIVEMQ_LOCK

      ![En5jw.png](https://s3.jpg.cm/2020/12/03/En5jw.png)

- 小总结

  - 如果是queue，在没有消费者消费的情况下会将消息保存到activemq_msgs表中，只要有任意一个消费者消费过了，在消费之后这些消息将会立即被删除。
  - 如果是topic，一般是先启动消费订阅然后再生产的情况下会将消息保存到activemq_acks。

- 开发中的坑

  - 记得需要使用到的相关jar文件放置到ActiveMQ安装路径下的lib目录下。mysql-jdbc驱动的jar包和对应的数据库连接池jar包。
  - 在jdbcPersistenceAdapte标签中设置了createTablesOnStartup属性为true时在第一次启动ActiveMQ时，ActiveMQ服务节点会自动创建所需要的数据表。启动完成后可以去掉这个属性，或者更改create TablesOnStartup属性为false。
  - "java.lang.l llegalStateException:BeanFactory not initialized or already closed"这是因为操作系统的机器名中有" _ "符号。更改机器名重启即可解决问题。

### JDBC Message store with ActiveMQ Journal

- 这种方式克服了JDBC Store的不足，JDBC每次消息过来，都需要去写库和读库。
- ActiveMQ Journal，使用高速缓存写入技术，大大提高了性能。
- 当消费者的消费速度能够及时跟上生产者消息的生产速度时，Journal文件能够大大减少需要写入到DB中的消息。
- 举个例子：生产者生产了1000条消息，这1000条消息会保存到journal文件，如果消费者的消费速度很快的情况下，在journal文件还没有同步到DB之前，消费者已经消费了90%的以上消息，那么这个时候只需要同步剩余的10%的消息到DB。
- 如果消费者的消费速度很慢，这个时候journal文件可以使消息以批量方式写到DB。
- 配置
  - ![EnBb2.png](https://s3.jpg.cm/2020/12/03/EnBb2.png)

### LevelDB消息存储（了解）

- 这种文件系统是从ActiveMQ5.8之后引进的，他和KahaDB非常相似，也是基于文件的本地数据库储存形式，但是他提供给比KahaDB更快的持久性

- 但它不使用自定义B-Tree实现来索引预写文件，而是使用基于LevalDB的索引

- 默认配置如下:

  - ```xml
    <persistenceAdapter>
           <levelDBdirectory="activemq-data"/>
    </persistenceAdapter>
    ```

## ActiveMQ多节点集群

### ZK+Replicated LevelDB Store

- 原理说明

  - 使用ZooKeeper集群注册所有的ActiveMQ Broker但只有其中的一个Broker可以提供服务它将被视为Mater，其他的Broker处于待机状态被视为Slave。
  - 如果Master因故障而不能提供服务ZooKeeper会从中选举一个Broker充当Master。
  - Slave连接Master并同步他们的存储状态，Slave不接受客户端连接。所有的存储操作都将被复制到连接至Master的Slaves。
  - 如果Master宕机得到了最新更新的Slave会成为Master。故障节点在恢复后会重新加入到集群中并连接Master进入Slave模式
  - 所有需要同步的消息操作都将等待存储状态被复制到其他法定节点的操作完成才能完成。
  - 所以，如果你配置了replicas=3，那么法定大小时（3/2）+1=2.Master将会存储并更新然后等待（2-1）=1个Slave存储和更新完成，才汇报success。
  - 有一个node要作为观察者存在。当一个新的Master被选中，你需要至少保障一个法定node在线以能够找到拥有最新状态的node。这个node才可以成为新的Master。

- 部署规划和步骤

  - 创建三台集群目录![En9Pk.png](https://s3.jpg.cm/2020/12/03/En9Pk.png)

  - 修改管理控制台端口

    - mq_node01全部默认不动

    - mq_node02

      [![EnTue.md.png](https://s3.jpg.cm/2020/12/03/EnTue.md.png)](https://imagelol.com/image/EnTue)

    - mq_node03

      ![Ena4y.png](https://s3.jpg.cm/2020/12/03/Ena4y.png)

  - hostname名字映射

    - ![EniMr.png](https://s3.jpg.cm/2020/12/03/EniMr.png)

  - AvtiveMQ集群配置

    - 三个节点的brokerName都要一样
    - ![EQe0h.png](https://s3.jpg.cm/2020/12/03/EQe0h.png)
    - ![image-20201203192544716](C:\Users\LENOVO\AppData\Roaming\Typora\typora-user-images\image-20201203192544716.png)
    - 修改持久化配置
    - ![image-20201203192940914](C:\Users\LENOVO\AppData\Roaming\Typora\typora-user-images\image-20201203192940914.png)

  - 修改各节点的消息端口

    - mq_node01默认不动
    - ![EQVbW.png](https://s3.jpg.cm/2020/12/03/EQVbW.png)

  - 按顺序启动3个ActiveMQ节点，到这步前提是zk集群已经成功启动

## 高级特性和大厂常考重点

### 引入消息队列之后该如何保证其高可用性

- zookeeper+replicated-leveldb-store

### 异步投递Async Sends

![EQbyU.png](https://s3.jpg.cm/2020/12/03/EQbyU.png)

![EQJ28.png](https://s3.jpg.cm/2020/12/03/EQJ28.png)

### 延迟投递和定时投递

### 分发策略

### ActiveMQ消费重试机制

### 死信队列

### 如何保证消息不被重复消费呢？幂等性问题你谈谈



