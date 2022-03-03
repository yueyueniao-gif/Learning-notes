# SpringCloud 

![BHXXz.png](https://s3.jpg.cm/2021/02/03/BHXXz.png)

## 注册中心

#### 三个注册中心的异同点

![BkX2S.png](https://s3.jpg.cm/2021/02/04/BkX2S.png)

#### CAP

##### C：consistency（强一致性）

##### A：Availability（可用性）

##### P：Partition tolerance（分区错误性）

##### CAP理论关注粒度是数据，而不是整体系统设计





## 服务调用

### Ribbon
- Ribbon 是 Netflix开源的基于HTTP和TCP等协议负载均衡组件

- Ribbon 可以用来做客户端负载均衡，调用注册中心的服务

- Ribbon的使用需要代码里手动调用目标服务，请参考官方示例：https://github.com/Netflix/ribbon

### Feign
- Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端

- Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。

- Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务

- Feign支持的注解和用法请参考官方文档：https://github.com/OpenFeign/feign

- Feign本身不支持Spring MVC的注解，它有一套自己的注解

### OpenFeign
- OpenFeign是Spring Cloud 在Feign的基础上支持了Spring MVC的注解，如@RequesMapping等等。
- OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，
  并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。

##### 需要注意，@RequesMapping不能在类名上与@FeignClient同时使用



## 服务降级

### Hystrix

#### Hystrix重要概念

##### 服务降级

- 服务器忙，请稍后再试，不让客户端等待并立即一个友好提示，fallback
- 哪些情况会触发降级
  - 程序运行异常
  - 超时 
  - 服务熔断出发服务降级
  - 线程池/信号量打满也会导致服务降级

##### 服务熔断

- 类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示
- 就是保险丝
  - 服务降级->服务熔断->恢复调用链路

##### 服务限流

- 秒杀高并发操作，严禁一窝蜂过来的拥挤，大家排队，一秒钟N个，有序进行。

 

## 服务网关

- API 网关是一个服务器，是系统对外的唯一入口。API 网关封装了系统内部架构，为每个客户端提供定制的 API。所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有非业务功能。API 网关并不是微服务场景中必须的组件，如下图，不管有没有 API 网关，后端微服务都可以通过 API 很好地支持客户端的访问。
  - ![2yItE.png](https://s3.jpg.cm/2021/03/12/2yItE.png)

**路由（Route）**：路由是网关最基础的部分，路由信息由 ID、目标 URI、一组断言和一组过滤器组成。如果断言路由为真，则说明请求的 URI 和配置匹配。

**断言（Predicate）**：Java8 中的断言函数。Spring Cloud Gateway 中的断言函数输入类型是 Spring 5.0 框架中的 ServerWebExchange。Spring Cloud Gateway 中的断言函数允许开发者去定义匹配来自于 Http Request 中的任何信息，比如请求头和参数等。

**过滤器（Filter）**：指的是Spring框架中的GatewayFilter的实例，使用过滤器，可以在请求被路由的前或者之后对请求进行修改。

**路由配置规则**

```yml
spring:
  application:
    name: gateway-server # 应用名称
  cloud:
    gateway:
      # 路由规则
      routes:
        - id: payment_routh                     #路由的ID，没有固定的规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service       #匹配后提供服务的路由地址
          predicates:                         # 断言（判断条件）
            - Path=/payment/get/**              # 匹配对应 URL 的请求，将匹配到的请求追加在目标 URI 之后
            - After=2021-03-12T16:42:47.458+08:00[Asia/Shanghai] #这个时间之后这个请求才能生效
            - Cookie=username,zzyy    #Cookie Route Predicate需要两个参数，一个是Cookie name，一个是正则表达式。路由规则会通过获取对应的Cookie name值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行
            - Header=X-Request-Id, \d+  #请求头要有X-Request-Id属性并且值为正数的正则表达式
            - Query=username, \d  #要有参数名username并且值还要是整数才能路由
            # 剩下几种配置如下图
```

[![2yQFL.md.png](https://s3.jpg.cm/2021/03/12/2yQFL.md.png)](https://imagelol.com/image/2yQFL)

## 服务配置

### SpringCloud Config 基本使用

- 首先去码云或者github创建一个仓库，并新建一些配置文件，如下
  
- [![3GmgL.png](https://s3.jpg.cm/2021/03/13/3GmgL.png)](https://imagelol.com/image/3GmgL)
  
- 创建Maven工程config-server，添加依赖：

  - ```xml
       <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
    ```

- 配置application.yml

  ```yml
  server:
    port: 3344
  
  spring:
    application:
      name: cloud-config-center     #注册进Eureka服务器的微服务
    cloud:
      config:
        server:
          git:
            uri: https://gitee.com/yueyueniao-gif/springcloud-config.git    #GitHub上面的git仓库名字
            ####搜索目录
            search-paths:
              - speingcloud-config
        label: master
  
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:7001/eureka
  
  
  ```

  

- 创建启动类，并加上开启Config服务端注解@EnableConfigServer:

  - ```java
    @SpringBootApplication
    @EnableConfigServer
    public class MainAppConfigCenter3344 {
        public static void main(String[] args) {
            SpringApplication.run(MainAppConfigCenter3344.class,args);
        }
    }
    ```

- 启动项目，简单测试，访问：localhost:3344/master/config-dev.yml；访问方式有以下五种。

  - ```
    　        /{appication}/{profile}/[{label}]
      
      　　　　　　/{application}-{profile}.yml
      
      　　　　　　/{application}-{profile}.properties
      
      　　　　　　/{label}/{application}-{profile}.properties
      
      　　　　　　/{label}/{application}-{profile}.yml
      
    ```

  

## 服务总线

- 什么是总线
  - **在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个公用的消息主题，并让系统中的所有未付实例都连接上来。由于该主题中产生消息会被所有实例监听和消费，所以称它为消息总线**。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。
- 基本原理
  - ConfigClient实例都监听MQ中同一个topic（默认是springCloudBus）。当一个服务刷新数据的时候，他会把这个信息放入到Topic中，这样其他监听同一Topic的服务就能得到通知，然后去更新自身的配置。

	## Stream

- 什么是Spring Cloud Stream
  - 官方定义Spring Cloud Stream是一个构建消息驱动微服务的框架
  - 应用程序通过inputs或者outputs来与Spring Cloud Stream中binder对象交互。通过我们配置来binding（绑定），而Spring Cloud Stream的binder对象负责与消息中间件交互。所以，我们只需要搞清楚如何与Spring Cloud Stream交互就可以方便使用消息驱动的方式。
  - 通过Spring Integration来连接消息中间件以实现消息事件驱动。Spring Cloud Stream为一些供应商的消息中间件提供了个性化的自动配置实现，引用了发布、订阅、消费组、分区的三个核心概念。

