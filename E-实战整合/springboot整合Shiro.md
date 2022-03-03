---
title: springboot整合Shiro
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-19 22:03:16
tags: 
     - springboot
     - Shiro
categories: 权限控制
---

## SpringBoot集成Shiro

ps: 发现一篇好博客，学完shiro的单Realm可以去看一下多Realm的多种登录方式[传送门](https://hacpai.com/article/1582123725664)

### 准备工作

1. 搭建一个springboot项目，选中web模块即可

2. 导入maven依赖thymeleaf

   ```xml
   <!--thymeleaf模板-->
   <dependency>
   <groupId>org.thymeleaf</groupId>
   <artifactId>thymeleaf-spring5</artifactId>
   </dependency>
   
   <dependency>
   <groupId>org.thymeleaf.extras</groupId>
   <artifactId>thymeleaf-extras-java8time</artifactId>
   </dependency>
   
   ```

3. 编写一个页面index.html

   ```html
   <!DOCTYPE html>
   <html lang="en"xmlns:th="http://www.thymeleaf.org">
   <head>
   <meta charset="UTF-8">
   <title>Title</title>
   </head>
   <body>
   <h1>首页</h1>
   <p th:text="${msg}"></p>
   </body>
   </html>
   ```

4. 编写coontroller进行测试访问

   ```java
   package com.kuang.controller;
   import org.springframework.stereotype.Controller;
   import org.springframework.ui.Model;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @Controller
   public class MyController {
       
      @RequestMapping({"/","/index"})
      public String toIndex(Model model){
      model.addAttribute("msg","hello,Shiro");
      return "index";
      }
       
   }
   ```

### 整合shiro

##### 回顾核心API

1. Subject：用户主体
2. SecurityManager：安全管理器
3. Realm：Shiro 连接数

##### 步骤

1. 导入Shiro和spring整合的依赖

   ```xml
   <dependency>
   <groupId>org.apache.shiro</groupId>
   <artifactId>shiro-spring</artifactId>
   <version>1.4.1</version>
   </dependency>
   ```

2. 我们需要自定义一个 realm 的类,用来编写一些查询的方法，或者认证与授权的逻辑

   ```java
   package com.kuang.config;
   import org.apache.shiro.authc.AuthenticationException;
   import org.apache.shiro.authc.AuthenticationInfo;
   import org.apache.shiro.authc.AuthenticationToken;
   import org.apache.shiro.authz.AuthorizationInfo;
   import org.apache.shiro.realm.AuthorizingRealm;
   import org.apache.shiro.subject.PrincipalCollection;
   //自定义Realm
   public class UserRealm extends AuthorizingRealm {
   //执行授权逻辑
   @Override
   protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals){
      System.out.println("执行了=>授权逻辑PrincipalCollection");
      return null;
   }
   //执行认证逻辑
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
      System.out.println("执行了=>认证逻辑AuthenticationToken");
      }
   }
   ```

3. 编写Shiro配置类

   ```java
   package com.kuang.config;
   import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
   import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   //声明为配置类
   @Configuration
   public class ShiroConfig {
   //创建 ShiroFilterFactoryBean
   @Bean
   public ShiroFilterFactoryBean
   getShiroFilterFactoryBean(@Qualifier("securityManager")DefaultWebSecurityMan
   ager securityManager){
   ShiroFilterFactoryBean shiroFilterFactoryBean = new
   ShiroFilterFactoryBean();
       
   //设置安全管理器
   shiroFilterFactoryBean.setSecurityManager(securityManager);
   return shiroFilterFactoryBean;
       
   }
       
   //创建 DefaultWebSecurityManager
   @Bean(name = "securityManager")
   public DefaultWebSecurityManager
   getDefaultWebSecurityManager(@Qualifier("userRealm")UserRealm userRealm){
   DefaultWebSecurityManager securityManager = new
   DefaultWebSecurityManager();
   //关联Realm
   securityManager.setRealm(userRealm);
   return securityManager;
   }
       
   //创建 realm 对象
   @Bean
   public UserRealm userRealm(){
   return new UserRealm();
   }
   }
   ```

### 页面拦截实现

1.  编写两个页面、在templates目录下新建一个 user 目录 add.html 和 update.html

   ```html
   <body>
   <h1>add</h1>
   </body>
   ```

   ```html
   <body>
   <h1>update</h1>
   </body>
   ```

2. 编写跳转到页面的controller

   ```java
   @RequestMapping("/user/add")
   public String toAdd(){
   return "user/add";
   }
   
   @RequestMapping("/user/update")
   public String toUpdate(){
   return "user/update";
   }
   
   ```

3.  在index页面上，增加跳转链接

   ```html
   <a th:href="@{/user/add}">add</a> | <a th:href="@{/user/update}">update</a>
   ```

4. 测试页面跳转是否OK

5. 准备添加Shiro的内置过滤器

   ```java
   @Bean
   public ShiroFilterFactoryBean
   getShiroFilterFactoryBean(
       @Qualifier("securityManager"
                 )DefaultWebSecurityManager securityManager){
      ShiroFilterFactoryBean shiroFilterFactoryBean = new
      ShiroFilterFactoryBean();
      //设置安全管理器
      shiroFilterFactoryBean.setSecurityManager(securityManager);
      /*
      *添加Shiro内置过滤器，常用的有如下过滤器：
      *anon： 无需认证就可以访问
      *authc： 必须认证才可以访问
      *user： 如果使用了记住我功能就可以直接访问
      *perms: 拥有某个资源权限才可以访问
      *role： 拥有某个角色权限才可以访问
      */
      Map<String,String> filterMap = new LinkedHashMap<String, String>();
      filterMap.put("/user/add","authc");
      filterMap.put("/user/update","authc");
      shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
      return shiroFilterFactoryBean;
   }
   ```

6.  再起启动测试，访问链接进行测试！拦截OK！但是发现，点击后会跳转到一个Login.jsp页面，这 个不是我们想要的效果，我们需要自己定义一个Login页面！

7. 我们编写一个自己的Login页面

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
      <meta charset="UTF-8">
      <title>登录页面</title>
   </head>
   <body>
      <h1>登录页面</h1>
   <hr>
      <form action="">
         <p>
            用户名： <input type="text" name="username">
         </p>
      <p>
         密码： <input type="text" name="password">
      </p>
         <p>
            <input type="submit">
         </p>
      </form>
   </body>
   </html>
   ```

8.  编写跳转的controller

   ```java
   @RequestMapping("/toLogin")
   public String toLogin(){
      return "login";
   }
   ```

9.  在shiro中配置一下！ ShiroFilterFactoryBean() 方法下面

   ```java
   //修改到要跳转的login页面；
   shiroFilterFactoryBean.setLoginUrl("/toLogin");
   ```

10.  再次测试，成功的跳转到了我们指定的Login页面！

11. 优化一下代码，我们这里的拦截可以使用 通配符来操作

    ```java
    Map<String,String> filterMap = new LinkedHashMap<String, String>();
    //filterMap.put("/user/add","authc");
    //filterMap.put("/user/update","authc");
    filterMap.put("/user/*","authc");
    shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
    ```

12.  测试，完全OK！

### 登录认证操作

1. 编写一个登录的controller

   ```java
   //登录操作
   @RequestMapping("/login")
   public String login(String username,String password,Model model){
      //使用shiro，编写认证操作
      //1. 获取Subject
      Subject subject = SecurityUtils.getSubject();
      //2. 封装用户的数据
      UsernamePasswordToken token = new UsernamePasswordToken(username,password);
      //3. 执行登录的方法，只要没有异常就代表登录成功！
       try {
           subject.login(token); //登录成功！返回首页
           return "index";
       } catch (UnknownAccountException e) { //用户名不存在
           model.addAttribute("msg","用户名不存在");
           return "login";
       } catch (IncorrectCredentialsException e) { //密码错误
           model.addAttribute("msg","密码错误");
           return "login";
       }
   }
   ```

2. 在前端修改对应的信息输出或者请求！

3. 登录页面增加一个 msg 提示：

   ```html
   <p style="color:red;" th:text="${msg}"></p>
   ```

   给表单增加一个提交地址:

   ```html
   <form th:action="@{/login}">
     <p>用户名： <input type="text" name="username"></p>
     <p>密码： <input type="text" name="password"></p>
     <p> <input type="submit"> </p>
   </form>
   ```

4.  在 UserRealm 中编写用户认证的判断逻辑

   ```java
   //执行认证逻辑
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
      System.out.println("执行了=>认证逻辑AuthenticationToken");
      //假设数据库的用户名和密码
      String name = "root";
      String password = "123456";
      //1.判断用户名
      UsernamePasswordToken userToken = (UsernamePasswordToken)token;
      if (!userToken.getUsername().equals(name)){
      //用户名不存在
      return null; //shiro底层就会抛出 UnknownAccountException
     }
      //2. 验证密码,我们可以使用一个AuthenticationInfo实现类
      SimpleAuthenticationInfo
      // shiro会自动帮我们验证！重点是第二个参数就是要验证的密码！
      //第一个参数：传入的都是com.java.entity包下的User类的user对象。
      //第二个参数:  传入的是从数据库中获取到的password，然后再与token中的password进行对比，匹配上了就通过，匹配不上就报异常。
      //第三个参数，盐–用于加密密码对比。 若不需要，则可以设置为空 “ ”
   //第四个参数：当前realm的名字。
      return new SimpleAuthenticationInfo("", password, "");
   }
   ```
   
   

### 整合数据库

1.  导入Mybatis相关依赖

   ```xml
   <!-- 引入 myBatis，这是 MyBatis官方提供的适配 Spring Boot 的，而不是Spring
   Boot自己的-->
   <dependency>
   <groupId>org.mybatis.spring.boot</groupId>
   <artifactId>mybatis-spring-boot-starter</artifactId>
   <version>2.1.0</version>
   </dependency>
   <dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <scope>runtime</scope>
   </dependency>
   <!-- https://mvnrepository.com/artifact/log4j/log4j -->
   <dependency>
   <groupId>log4j</groupId>
   <artifactId>log4j</artifactId>
   <version>1.2.17</version>
   </dependency>
   <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
   <dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid</artifactId>
   <version>1.1.12</version>
   </dependency>
   
   ```

2.  编写配置文件-连接配置 application.yml

   ```yml
   spring:
     datasource:
     username: root
     password: 123456
     #?serverTimezone=UTC解决时区的报错
     url: jdbc:mysql://localhost:3306/mybatis?
   serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
     driver-class-name: com.mysql.jdbc.Driver
     type: com.alibaba.druid.pool.DruidDataSource
     
     #Spring Boot 默认是不注入这些属性值的，需要自己绑定
     #druid 数据源专有配置
     initialSize: 5
     minIdle: 5
     maxActive: 20
     maxWait: 60000
     timeBetweenEvictionRunsMillis: 60000
     minEvictableIdleTimeMillis: 300000
     validationQuery: SELECT 1 FROM DUAL
     testWhileIdle: true
     testOnBorrow: false
     testOnReturn: false
     poolPreparedStatements: true
     #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
     #如果允许时报错 java.lang.ClassNotFoundException:
     org.apache.log4j.Priority
     #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
     filters: stat,wall,log4j
     maxPoolPreparedStatementPerConnectionSize: 20
     useGlobalDataSourceStat: true
     connectionProperties:
   druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
   ```

3. .编写mybatis的配置 application.properties

   ```properties
   #别名配置
   mybatis.type-aliases-package=com.kuang.pojo
   mybatis.mapper-locations=classpath:mapper/*.xml
   ```

4. 编写实体类,引入Lombok

   ```xml
   <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.16.10</version>
   </dependency>
   ```

   ```java
   package com.kuang.pojo;
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class User {
   private int id;
   private String name;
   private String pwd;
   }
   ```

5. 编写Mapper接口

   ```java
   @Repository
   @Mapper
   public interface UserMapper {
   public User queryUserByName(String name);
   }
   
   ```

6.  编写Mapper配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
   PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
   "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.kuang.mapper.UserMapper">
   <select id="queryUserByName" parameterType="String"
   resultType="User">
   select * from user where name = #{name}
   </select>
   </mapper>
   ```

7. 编写UserService 层

   ```java
   public interface UserService {
   public User queryUserByName(String name);
   }
   ```

   ```java
   package com.kuang.service;
   import com.kuang.mapper.UserMapper;
   import com.kuang.pojo.User;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Service;
   @Service
   public class UserServiceImpl implements UserService {
   @Autowired
   UserMapper userMapper;
   @Override
   public User queryUserByName(String name) {
      return userMapper.queryUserByName(name);
   }
   }
   ```

8.  好了，一口气写了这些常规操作，可以去测试一下了，保证能够从数据库中查询出来

   ```java
   class Shiro02SpringbootApplicationTests {
   @Autowired
   UserServiceImpl userService;
   @Test
   void contextLoads() {
   User user = userService.queryUserByName("root");
   System.out.println(user);
   }
   }
   ```

9.  改造UserRealm，连接到数据库进行真实的操作！

   ```java
   //自定义Realm
   public class UserRealm extends AuthorizingRealm {
   @Autowired
   UserService userService;
   //执行授权逻辑
   @Override
   protected AuthorizationInfo
   doGetAuthorizationInfo(PrincipalCollection principals) {
   System.out.println("执行了=>授权逻辑PrincipalCollection");
   return null;
   }
   //执行认证逻辑
   @Override
   protected AuthenticationInfo
   doGetAuthenticationInfo(AuthenticationToken token) throws
   AuthenticationException {
   System.out.println("执行了=>认证逻辑AuthenticationToken");
   UsernamePasswordToken userToken = (UsernamePasswordToken)token;
   //真实连接数据库
   User user =
   userService.queryUserByName(userToken.getUsername());
   if (user==null){
   //用户名不存在
   return null; //shiro底层就会抛出 UnknownAccountException
   }
   return new SimpleAuthenticationInfo("", user.getPwd(), "");
   }
   }
   ```

10. 测试，现在查询都是从数据库查询的了！



### 用户授权操作

使用shiro的过滤器来拦截请求即可！

1.  在 ShiroFilterFactoryBean 中添加一个过滤器

   ```java
   //授权过滤器
   filterMap.put("/user/add","perms[user:add]"); //大家记得注意顺序！
   ```

2. 我们再次启动测试一下，访问add，发现以下错误 White;abel Errror Page

3.  注意：当我们实现权限拦截后，shiro会自动跳转到未授权的页面，但我们没有这个页面，所有401 了

4.  配置一个未授权的提示的页面，增加一个controller提示

   ```java
   @RequestMapping("/noauth")
   @ResponseBody
   public String noAuth(){
   return "未经授权不能访问此页面";
   }
   ```

5.  测试，现在没有授权，可以跳转到我们指定的位置了！

### Shiro授权

在UserRealm 中添加授权的逻辑，增加授权的字符串！

```java
//执行授权逻辑
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection
principals) {
System.out.println("执行了=>授权逻辑PrincipalCollection");
//给资源进行授权
SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
//添加资源的授权字符串
info.addStringPermission("user:add");
return info;
}
```

我们再次登录测试，发现登录的用户是可以进行访问add 页面了！授权成功！ 问题，我们现在完全是硬编码，无论是谁登录上来，都可以实现授权通过，但是真实的业务情况应该 是，每个用户拥有自己的一些权限，从而进行操作，所以说，权限，应该在用户的数据库中，正常的情 况下，应该数据库中是由一个权限表的，我们需要联表查询，但是这里为了大家操作理解方便一些，我 们直接在数据库表中增加一个字段来进行操作！

1. 修改实体类，增加一个字段

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class User {
   private int id;
   private String name;
   private String pwd;
   private String perms;
   }
   ```

2. 我们现在需要再自定义的授权认证中，获取登录的用户，从而实现动态认证授权操作！

- 在用户登录授权的时候，将用户放在 Principal 中，改造下之前的代码

```java
 return new SimpleAuthenticationInfo(user, user.getPwd(), "");
```

- 然后再授权的地方获得这个用户，从而获得它的权限

```java
//执行授权逻辑
@Override
protected AuthorizationInfo
doGetAuthorizationInfo(PrincipalCollection principals) {
System.out.println("执行了=>授权逻辑PrincipalCollection");
//给资源进行授权
SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
//添加资源的授权字符串
//info.addStringPermission("user:add");
Subject subject = SecurityUtils.getSubject(); //获得当前对象
User currentUser = (User) subject.getPrincipal(); //拿到User对象
info.addStringPermission(currentUser.getPerms()); //设置权限
return info;
}
```

3. 我们给数据库中的用户增加一些权限

   ![](https://s3.jpg.cm/2020/08/18/uH4sT.png)

4. 在过滤器中，将update请求进行权限拦截下

   ```java
   //授权过滤器
   filterMap.put("/user/add","perms[user:add]");
   filterMap.put("/user/update","perms[user:update]");
   ```

5. 我们启动项目，登录不同的账户，进行测试一下！

6. 测试完美通过OK！



### 整合Thymeleaf

根据权限展示不同的前端页面

1.  添加Maven的依赖；

   ```xmml
   <!--
   https://mvnrepository.com/artifact/com.github.theborakompanioni/thymeleaf
   -extras-shiro -->
   <dependency>
   <groupId>com.github.theborakompanioni</groupId>
   <artifactId>thymeleaf-extras-shiro</artifactId>
   <version>2.0.0</version>
   </dependency>
   
   ```

2.  配置一个shiro的Dialect ，在shiro的配置中增加一个Bean

   ```java
   //配置ShiroDialect：方言，用于 thymeleaf 和 shiro 标签配合使用
   @Bean
   public ShiroDialect getShiroDialect(){
   return new ShiroDialect();
   }
   ```

3. 修改前端的配置

   ```html
   <div shiro:hasPermission="user:add">
   <a th:href="@{/user/add}">add</a>
   </div>
   <div shiro:hasPermission="user:update">
   <a th:href="@{/user/update}">update</a>
   </div>
   ```

4.  我们在去测试一下，可以发现，现在首页什么都没有了，因为我们没有登录，我们可以尝试登录下 ，来判断这个Shiro的效果！登录后，可以看到不同的用户，有不同的效果，现在就已经接近完美 了~！还不是最完美

5.  为了完美，我们在用户登录后应该把信息放到Session中，我们完善下！在执行认证逻辑时候，加 入session

   ```java
   Subject subject = SecurityUtils.getSubject();
   subject.getSession().setAttribute("loginUser",user);
   ```

6. 前端从session中获取，然后用来判断是否显示登录

   ```html
   <p th:if="${session.loginUser==null}">
   <a th:href="@{/toLogin}">登录</a>
   </p>
   ```

7.  测试，效果完美

