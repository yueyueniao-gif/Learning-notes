---
title: Swagger
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-17 23:56:29
tags: swagger
categories: swagger
---

### Swagger简介

- 号称世界上最流行的API框架
- Restful Api 文档在线自动生成器 => **API 文档 与API 定义同步更新**
- 直接运行，在线测试API
- 支持多种语言 （如：Java，PHP等）
- 官网：[传送门](https://swagger.io/)

### Swagger使用

#### 项目搭建

1. 新建一个SpringBoot项目

2. 添加maven依赖

   ```xml
   <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>2.9.2</version>
   </dependency>
   <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger-ui</artifactId>
      <version>2.9.2</version>
   </dependency>
   ```

   

3. 编写helloController，确保运行成功

4. 要使用Swagger，我们需要编写一个配置类SwaggerConfig来配置Swagger

   ```java
   @Configuration //配置类
   @EnableSwagger2// 开启Swagger2的自动配置
   public class SwaggerConfig {  
   }
   ```

   

5. 访问 swagger-ui

   ```java
   启动项目后在浏览器中输入http://ip:port/swagger-ui.html即可以
   访问到 swagger-ui 页面，在页面中可以可视化的进行操作项目中所有
   接口。
   ```

#### 配置Swagger

1. Swagger实例Bean是Docket，所以通过配置Docket实例来配置Swaggger。

2. 可以通过apiInfo()属性配置文档信息。

3. Docket 实例关联上 apiInfo()。

   ```java
   @Configuration
   @EnableSwagger2
   public class SwaggerConfig{
       
      //配置docket用来配置Swagger具体参数
      //Docker关联上apiInfo
      @Bean 
      public Docket docket() {
          
         return new Docket(DocumentationType.SWAGGER_2)
           .apiInfo(apiInfo())
             // 通过.select()方法，去配置扫描接口
           .select()
            // 1、指定扫描的包，这些包下有用，其他的没用
           .apis(RequestHandlerSelectors.basePackage("com.kuang.swagger.controller"))
            // 配置如何通过path过滤,即这里只扫描请求以/kuang开头的接口
            .paths(PathSelectors.ant("/kuang/**"))
            .build();
          //这里的可选值还有
          //any() //任何请求都扫描
          //none()// 任何请求都不扫描
          //.enable(false) //配置是否启用Swagger，如果是false，在浏览器将无法访问
      }
       //配置文档信息
       private ApiInfo apiInfo() {
           //作者信息
         Contact contact = new Contact("姓名", "http://xxx.xxx.com/联系人访问链接", "邮箱");
           
         return new ApiInfo(
              "Swagger学习", // 标题
              "学习演示如何配置Swagger", // 描述
              "v1.0", // 你的版本
              "http://yueyueniao-gif.gitee.io/yueyueniao-gif/",
              contact, // 联系人信息，就是上面配置的作者信息
              "Apach 2.0 许可", // 许可
              "http://www.apache.org/licenses/LICENSE-2.0", // 许可连接
              new ArrayList<>()// 扩展
           );
      }
   }
   ```
   
   
   
4. 重启项目，访问测试 http://localhost:8080/swagger-ui.html 看下效果；

   

   ##### 如何动态配置当项目处于test、dev环境时显示swagger，处于prod时不显示

   ```java
   @Bean
   public Docket docket(Environment environment) {
      // 设置要显示swagger的环境
      Profiles of = Profiles.of("dev", "test");
      // 判断当前是否处于该环境
      // 通过 enable() 接收此参数判断是否要显示
      boolean b = environment.acceptsProfiles(of);
      
      return new Docket(DocumentationType.SWAGGER_2)
         .apiInfo(apiInfo())
         .enable(b) //配置是否启用Swagger，如果是false，在浏览器将无法访问
         .build();
   }
   ```

   ##### 配置API分组

   ```java
   //如果没有配置分组，默认是default。通过groupName()方法即可配置分组：
   @Bean
   public Docket docket1(){
      return new Docket(DocumentationType.SWAGGER_2)
          .groupName("group1");
   }
   @Bean
   public Docket docket2(){
      return new Docket(DocumentationType.SWAGGER_2)
          .groupName("group2");
   }
   @Bean
   public Docket docket3(){
      return new Docket(DocumentationType.SWAGGER_2)
          .groupName("group3");
   }
   ```

#### 常用注解

- @NotIncludeSwagger

  在不需要生成接口文档的方法上面添加@NotIncludeSwagger 注解后，该方法将不会被 Swagger 进行生成在接口文档中

  ```java
      @NotIncludeSwagger 
      @RequestMapping("/test") 
      public String test(){
      return "";
  }
  
  ```

  

- @Api

  @Api 是类上注解。控制整个类生成接口信息的内容。

  - tags：类的名称。可以有多个值，多个值表示多个副本。
  - description:描述，已过时

  ```java
  @RestController
  @Api(tags = {"mydemo"},description = "描述")
    public class HelloController {}
  ```

  

- @ApiOperation

  @ApiOperation 写在方法上，对方法进行总体描述

  - value：接口描述
  - notes：提示信息

  ```java
  @ApiOperation(value="接口描述",notes = "接口提示信息")
   @GetMapping("/hello")
      public String hello(){
          return "hello";
      }
  ```

  

  

- @ApiParam

  @ApiParam 写在方法参数前面。用于对参数进行描述或说明是否为必添项等说明。

  - name：参数名称
  - value：参数描述
  - required：是否是必须

  ```java
  public People getPeople(Integer id
                          ,@ApiParam(value="姓名",required = true) String name
                          ,String address)
  ```

  

- @ApiModel

  @ApiModel是类上注解，主要应用 Model，也就是说这个注解一般都是写在实体类上。

  - value：名称
  - description：描述

  ```java
  @ApiModel(value = "用户实体",description = "描述")
  public class User {
  }
  ```

  

- @ApiModelProperty

  @ApiModelProperty 可以用在方法或属性上。用于当对象作为参数时定义这个字段的内容。

  - value：描述
  - name：重写属性名
  - required：是否是必须的
  - example：示例内容
  - hidden：是否隐藏

  ```java
  @ApiModel("用户实体")
  public class User {
      @ApiModelProperty(value = "姓名",name = "name",required = true,example = "张三")
      public String name;
    
  }
  ```

  

- @ApiIgnore

  @ApiIgnore 用于方法或类或参数上，表示这个方法或类被忽略。和之前讲解的自定义注解

  @NotIncludeSwagger 效果类似。只是这个注解是 Swagger 内置的注解，而@NotIncludeSwagger 是我们自定义的注解。



- @ApiImplicitParam

  @ApiImplicitParam 用在方法上，表示单独的请求参数，总体功能和@ApiParam 类似。

  - name：属性名
  - value：描述
  - required：是否是必须的
  - paramType：属性类型
  - dataType：数据类型

  ```java
  @PostMapping("/getPeople") 
  @ApiImplicitParam(name = "address"
                    ,value = "地址"
                    ,required = true
                    ,paramType ="query"
                    ,dataType = "string") 
  public People getPeople(Integer id
                          , @ApiParam(value="姓名",required = true) String name
                          , String address){ }
  
  ```

  