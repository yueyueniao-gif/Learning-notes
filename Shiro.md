---


title: Shiro
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-18 19:06:14
tags: Shiro
categories: 权限控制
---

### Shiro简介

- Apache Shiro 是一个Java 的安全（权限）框架。

- Shiro 可以非常容易的开发出足够好的应用，其不仅可以用在JavaSE环境，也可以用在JavaEE环 境。

- Shiro可以完成，认证，授权，加密，会话管理，Web集成，缓存等。

  

### Shiro功能

![](https://s3.jpg.cm/2020/08/18/uHAW8.png)

```
Authentication：身份认证、登录，验证用户是不是拥有相应的身份；

Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限，即判断用户能否
进行什么操作，如：验证某个用户是否拥有某个角色，或者细粒度的验证某个用户对某个资源是否
具有某个权限！

Session Manager：会话管理，即用户登录后就是第一次会话，在没有退出之前，它的所有信息都
在会话中；会话可以是普通的JavaSE环境，也可以是Web环境；

Cryptography：加密，保护数据的安全性，如密码加密存储到数据库中，而不是明文存储；

Web Support：Web支持，可以非常容易的集成到Web环境；

Caching：缓存，比如用户登录后，其用户信息，拥有的角色、权限不必每次去查，这样可以提高
效率

Concurrency：Shiro支持多线程应用的并发验证，即，如在一个线程中开启另一个线程，能把权限
自动的传播过去

Testing：提供测试支持；

Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了
```

### Shiro架构

1. 外部架构

![](https://s3.jpg.cm/2020/08/18/uHXHi.png)

```
subject： 应用代码直接交互的对象是Subject，也就是说Shiro的对外API核心就是Subject，
Subject代表了当前的用户，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是
Subject，如网络爬虫，机器人等，与Subject的所有交互都会委托给SecurityManager；Subject其
实是一个门面，SecurityManageer 才是实际的执行者

SecurityManager：安全管理器，即所有与安全有关的操作都会与SercurityManager交互，并且它
管理着所有的Subject，可以看出它是Shiro的核心，它负责与Shiro的其他组件进行交互，它相当于
SpringMVC的DispatcherServlet的角色

Realm：Shiro从Realm获取安全数据（如用户，角色，权限），就是说SecurityManager 要验证
用户身份，那么它需要从Realm 获取相应的用户进行比较，来确定用户的身份是否合法；也需要从
Realm得到用户相应的角色、权限，进行验证用户的操作是否能够进行，可以把Realm看成
DataSource；

```

2. 内部架构

   ![](https://s3.jpg.cm/2020/08/18/uHcgk.png)

```
Subject：任何可以与应用交互的 ‘用户’；

Security Manager：相当于SpringMVC中的DispatcherServlet；是Shiro的心脏，所有具体的交互
都通过Security Manager进行控制，它管理者所有的Subject，且负责进行认证，授权，会话，及
缓存的管理。

Authenticator：负责Subject认证，是一个扩展点，可以自定义实现；可以使用认证策略
（Authentication Strategy），即什么情况下算用户认证通过了；

Authorizer：授权器，即访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能
访问应用中的那些功能；

Realm：可以有一个或者多个的realm，可以认为是安全实体数据源，即用于获取安全实体的，可
以用JDBC实现，也可以是内存实现等等，由用户提供；所以一般在应用中都需要实现自己的realm

SessionManager：管理Session生命周期的组件，而Shiro并不仅仅可以用在Web环境，也可以用
在普通的JavaSE环境中

CacheManager：缓存控制器，来管理如用户，角色，权限等缓存的；因为这些数据基本上很少改
变，放到缓存中后可以提高访问的性能；

Cryptography：密码模块，Shiro 提高了一些常见的加密组件用于密码加密，解密等
```

### HelloWorld快速开始

- 查看官网文档：http://shiro.apache.org/tutorial.html 

- 官方的quickstart：https://github.com/apache/shiro/tree/master/samples/quickstart/

1. 创建一个maven父工程，用于学习Shiro，删掉不必要的东西 

2. 创建一个普通的Maven子工程：shiro-01-helloworld

3.  根据官方文档，我们来导入Shiro的依赖

   ```xml
   <dependencies>
       
      <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.4.1</version>
      </dependency>
       
   <!-- Shiro uses SLF4J for logging. We'll use the 'simple' binding
   in this example app. See http://www.slf4j.org for more info. -->
   
     <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.7.21</version>
        <scope>test</scope>
      </dependency>
   
      <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.21</version>
        <scope>test</scope>
        </dependency>
   
   </dependencies>
   ```

4. 编写Shiro配置 log4j.properties

   ```properties
   log4j.rootLogger=INFO, stdout
   
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m %n
   
   # General Apache libraries
   log4j.logger.org.apache=WARN
   
   # Spring
   log4j.logger.org.springframework=WARN
   
   # Default Shiro logging
   log4j.logger.org.apache.shiro=INFO
   
   # Disable verbose logging
   log4j.logger.org.apache.shiro.util.ThreadContext=WARN
   log4j.logger.org.apache.shiro.cache.ehcache.EhCache=WARN
   ```

   shiro.ini

   ```ini
   # ----------------------------------------------------------------------
   -------
   # 用户及其分配的角色
   
   [users]
   # 用户“root”，密码为“secret”，角色为“admin”
   root = secret, admin
   # 用户'guest'，密码为'guest'，角色为'guest'
   guest = guest, guest
   # 用户 'presidentskroob' 密码为 '12345' 角色为'president'
   presidentskroob = 12345, president
   # 用户“darkhelmet”，密码为“ludicrousspeed”，角色为“darklord”和'schwartz'
   darkhelmet = ludicrousspeed, darklord, schwartz
   # 用户 'lonestarr'  密码为 'vespa' and 角色为 'goodguy'和'schwartz'
   lonestarr = vespa, goodguy, schwartz
   
   # ----------------------------------------------------------------------
   -------
   # 具有指定权限的角色
   
   [roles]
   # “admin”角色拥有所有权限，由通配符“*”表示。
   admin = *
   #“schwartz”角色可以做任何事
   schwartz = lightsaber:*
   # “goodguy”角色被允许 “驾驶”(动作)winnebago(类型)牌照'eagle5'(实例特定id)
   goodguy = winnebago:drive:eagle5
   ```

5. 编写我们的QuickStrat

   ```java
   import org.apache.shiro.SecurityUtils;
   import org.apache.shiro.authc.*;
   import org.apache.shiro.config.IniSecurityManagerFactory;
   import org.apache.shiro.mgt.SecurityManager;
   import org.apache.shiro.session.Session;
   import org.apache.shiro.subject.Subject;
   import org.apache.shiro.util.Factory;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   /**
   * 类的描述
   * 简单的快速入门应用程序，展示了如何使用Shiro的API。
   */
   public class Quickstart {
      private static final transient Logger log =
   LoggerFactory.getLogger(Quickstart.class);
       
   public static void main(String[] args) {
       
      // 1.通过工厂模式创建SecurityManager的实例对象
      //使用类路径根目录下的shiro.ini文件
   Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
   SecurityManager securityManager = factory.getInstance();
   
   SecurityUtils.setSecurityManager(securityManager);
     //2.获取当前的Subject
   Subject currentUser = SecurityUtils.getSubject();
       
      //3. session的操作
   Session session = currentUser.getSession();//获得session
   session.setAttribute("someKey", "aValue");//设置Session的值！
   String value = (String) session.getAttribute("someKey");//从session中获取值
       
   if (value.equals("aValue")) {//判断session中是否存在这个值！
   log.info("Retrieved the correct value! [" + value + "]");
   }
      //4.用户认证功能
   // 测试当前的用户是否已经被认证，即是否已经登录！    
   if (!currentUser.isAuthenticated()) {
   //将用户名和密码封装为 UsernamePasswordToken ；
   UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
   token.setRememberMe(true);//记住我功能
   try {
   currentUser.login(token);//执行登录，可以登录成功的！
   } catch (UnknownAccountException uae) {
   //如果没有指定的用户，则UnknownAccountException异常
   log.info("There is no user with username of " +token.getPrincipal());
   } catch (IncorrectCredentialsException ice) {
       //密码不对的异常！
   log.info("Password for account " +
   token.getPrincipal() + " was incorrect!");
   } catch (LockedAccountException lae) {
       //用户被锁定的异常
   log.info("The account for username " +token.getPrincipal() + " is locked. " +
   "Please contact your administrator to unlockit.");
   }
   catch (AuthenticationException ae) {//认证异常，上面的异常都是它的子类}
   }
   //说出他们是谁：
   //打印他们的标识主体（在本例中为用户名）：
   log.info("User [" + currentUser.getPrincipal() + "] logged insuccessfully.");
       
      //5. 角色检查    
   //是否存在某一个角色
   if (currentUser.hasRole("schwartz")) {
   log.info("May the Schwartz be with you!");
   } else {
   log.info("Hello, mere mortal.");
   }
      //6权限检查，粗粒度   
   //测试用户是否具有某一个权限，行为
   if (currentUser.isPermitted("lightsaber:wield")) {
   log.info("You may use a lightsaber ring. Use itwisely.");
   } else {
   log.info("Sorry, lightsaber rings are for schwartz mastersonly.");
   }
      //7.权限检查，细粒度  
   ///测试用户是否具有某一个权限，行为，比上面更加的具体！
   if (currentUser.isPermitted("winnebago:drive:eagle5")) {
   log.info("You are permitted to 'drive' the winnebago withlicense plate (id) 'eagle5'. " +"Here are the keys - have fun!");
   } else {
   log.info("Sorry, you aren't allowed to drive the 'eagle5'winnebago!");
   }
      //8.注销操作
   ///执行注销操作！
   currentUser.logout();
     //11. 退出系统       
   System.exit(0);
   }
   }
   
   ```

6. 测试运行一下

7. 报错，则导入一下commons-logging依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/commons-logging/commons-logging -->
   <dependency>
   <groupId>commons-logging</groupId>
   <artifactId>commons-logging</artifactId>
   <version>1.2</version>
   </dependency>
   ```

8. 发现，执行完毕什么都没有，可能是maven依赖中的作用域问题，我们需要将scope作用域删掉， 默认是在test，然后重启，那么我们的quickstart就结束了，默认的日志消息！

   ```
   [main] INFO org.apache.shiro.session.mgt.AbstractValidatingSessionManager
   - Enabling session validation scheduler...
   [main] INFO Quickstart - Retrieved the correct value! [aValue]
   [main] INFO Quickstart - User [lonestarr] logged in successfully.
   [main] INFO Quickstart - May the Schwartz be with you!
   [main] INFO Quickstart - You may use a lightsaber ring. Use it wisely.
   [main] INFO Quickstart - You are permitted to 'drive' the winnebago with
   license plate (id) 'eagle5'. Here are the keys - have fun!
   ```

   

