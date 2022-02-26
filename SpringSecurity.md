---
title: SpringSecurity
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-19 15:28:51
tags: SpringSecurity
categories: 权限控制
---

### 认识SpringSecurity

###### Spring Security 是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型，他可 以实现强大的Web安全控制，对于安全控制，我们仅需要引入 spring-boot-starter-security 模 块，进行少量的配置，即可实现强大的安全管理！ 

#### 记住几个类：

- WebSecurityConfigurerAdapter： 自定义Security策略 

- AuthenticationManagerBuilder：自定义认证策略 

- @EnableWebSecurity：开启WebSecurity模式 

#### Spring Security的两个主要目标是 “认证” 和 “授权”（访问控制）。

- “认证”（Authentication） 身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。 身份验证通常通过用户名和密码完成，有时与身份验证因素结合使用。

- “授权” （Authorization） 授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置， 几乎任何内容）的完全权限。 这个概念是通用的，而不是只在Spring Security 中存在。



### 实验环境搭建

1.  新建一个初始的springboot项目 web模块 ， thymeleaf模块

2.  导入静态资源

   ```
   index.html
      |views
         |level1
            1.html
            2.html
            3.html
         |level2
            1.html
            2.html
            3.html
         |level3
            1.html
            2.html
            3.html
         Login.html
   
   ```

3. controller跳转

   ```java
   package com.kuang.controller;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   @Controller
   public class RouterController {
       
      @RequestMapping({"/","/index"})
      public String index(){
      return "index";
      }
       
      @RequestMapping("/toLogin")
      public String toLogin(){
      return "views/login";
      }
       
      @RequestMapping("/level1/{id}")
      public String level1(@PathVariable("id") int id){
      return "views/level1/"+id;
      }
       
      @RequestMapping("/level2/{id}")
      public String level2(@PathVariable("id") int id){
      return "views/level2/"+id;
      }
       
      @RequestMapping("/level3/{id}")
      public String level3(@PathVariable("id") int id){
      return "views/level3/"+id;
      }
   }
   
   ```

4. 测试实验环境是否ok

### 认证和授权

##### 目前，我们的测试环境，是谁都可以访问的，我们使用 Spring Security 增加上认证和授权的功能

1. 引入 Spring Security 模块

   ```xml
   <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 编写Spring Security配置类，继承WebSecurityConfigurerAdapter

   ```java
   package com.kuang.config;
   import
   org.springframework.security.config.annotation.web.builders.HttpSecurity;
   import
   org.springframework.security.config.annotation.web.configuration.EnableWebSe
   curity;
   import
   org.springframework.security.config.annotation.web.configuration.WebSecurity
   ConfigurerAdapter;
   
   @EnableWebSecurity // 开启WebSecurity模式
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       //授权
      @Override
      protected void configure(HttpSecurity http) throws Exception {
      }
       
       //认证
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           
      }
   }
   ```

3. 定制请求的授权规则

   ```java
   @Override
   protected void configure(HttpSecurity http) throws Exception {
      // 定制请求的授权规则
      // 首页所有人可以访问
      http.authorizeRequests()
         .antMatchers("/").permitAll()
         .antMatchers("/level1/**").hasRole("vip1")
         .antMatchers("/level2/**").hasRole("vip2")
         .antMatchers("/level3/**").hasRole("vip3");
   }
   ```

4. 测试一下:发现除了首页都进不去了，因为我们目前没有登录的角色，因为请求需要登录的角色拥有 对应的权限才可以！

5. 在 configure() 方法中加入以下配置，开启自动配置的登录功能！

   ```java
   
   @Override
   protected void configure(HttpSecurity http) throws Exception {
      // 定制请求的授权规则
      // 首页所有人可以访问
      http.authorizeRequests()
         .antMatchers("/").permitAll()
         .antMatchers("/level1/**").hasRole("vip1")
         .antMatchers("/level2/**").hasRole("vip2")
         .antMatchers("/level3/**").hasRole("vip3");
       
       // 开启自动配置的登录功能
       // /login 请求来到登录页
       // /login?error 重定向到这里表示登录失败
       http.formLogin();
       }
   ```

6. 测试一下：发现，没有权限的时候，会跳转到SpringSecurity自带登录的页面！

7. 我们可以定义认证规则，重写 configure(AuthenticationManagerBuilder auth) 方法

   ```java
   //定义认证规则
   @Override
   protected void configure(AuthenticationManagerBuilder auth) throws Exception
   {
      //在内存中定义，也可以在jdbc中去拿....
      auth.inMemoryAuthentication()
      .withUser("kuangshen").password("123456").roles("vip2","vip3")
      .and()
      .withUser("root").password("123456").roles("vip1","vip2","vip3")
      .and()
      .withUser("guest").password("123456").roles("vip1","vip2");
   }
   ```

8. 、测试，我们可以使用这些账号登录进行测试！发现会报错！

9. 原因，我们要将前端传过来的密码进行某种方式加密，否则就无法登录，修改代码

   ```java
   //定义认证规则
   @Override
   protected void configure(AuthenticationManagerBuilder auth) throws Exception
   {
        //在内存中定义，也可以在jdbc中去拿....
        //Spring security 5.0中新增了多种加密方式，也改变了密码的格式。
        //要想我们的项目还能够正常登陆，需要修改一下configure中的代码。我们要将前端传过来的密码进行      某种方式加密
       //spring security 官方推荐的是使用bcrypt加密方式
       
     auth.inMemoryAuthentication().passwordEncoder(newBCryptPasswordEncoder())
       .withUser("kuangshen").password(new BCryptPasswordEncoder().encode("123456"))
       .roles("vip2","vip3")
       .and()
       .withUser("root").password(new BCryptPasswordEncoder().encode("123456"))
       .roles("vip1","vip2","vip3")
       .and()
       .withUser("guest").password(new BCryptPasswordEncoder().encode("123456"))
       .roles("vip1","vip2");
       
   }
   
   ```

10. 测试，发现，登录成功，并且每个角色只能访问自己认证下的规则！

### 权限控制和注销

1.  开启自动配置的注销的功能

   ```java
   //定制请求的授权规则
   @Override
   protected void configure(HttpSecurity http) throws Exception {
      //开启自动配置的注销的功能
      // /logout 注销请求
      http.logout();
   }
   ```

2. 我们在前端，增加一个注销的按钮， index.html 导航栏中

   ```html
   <a class="item" th:href="@{/logout}">
      <i class="address card icon"></i> 注销
   </a>
   ```

3. .我们可以去测试一下，登录成功后点击注销，发现注销完毕会跳转到登录页面！

   

4. . 如果注销404了，就是因为它默认防止csrf跨站请求伪造，因为会产生安全问题，我们可以将请求 改为post表单提交，或者在spring security中关闭csrf功能；我们试试：在 配置中增加

   ```java
   http.csrf().disable();//关闭csrf功能:跨站请求伪造,默认只能通过post方式提交logout
   请求
   http.logout().logoutSuccessUrl("/");
   ```

   

5.  但是，我们想让他注销成功后，依旧可以跳转到首页，该怎么处理呢？

   ```java
   // .logoutSuccessUrl("/"); 注销成功来到首页
   http.logout().logoutSuccessUrl("/");
   ```

6. 测试，注销完毕后，发现跳转到首页OK

### 记住我

1. 开启记住我功能

   ```java
   //定制请求的授权规则
   @Override
   protected void configure(HttpSecurity http) throws Exception {
      //。。。。。。。。。。。
      //记住我
      http.rememberMe();
   }
   ```

2.  我们再次启动项目测试一下，发现登录页多了一个记住我功能，我们登录之后关闭 浏览器，然后重 新打开浏览器访问，发现用户依旧存在！

   我们可以查看浏览器的cookie，我们点击注销的时候，可以发现，spring security 帮我们自动删除了这个 cookie

   结论：：登录成功后，将cookie发送给浏览器保存，以后登录带上这个cookie，只要通过检查就可以 免登录了。如果点击注销，则会删除这个cookie，具体的原理我们在JavaWeb阶段都讲过了，这里 就不在多说了！

### 定制登录页

1.  在刚才的登录页配置后面指定 loginpage

   ```java
   http.formLogin().loginPage("/toLogin");
   ```

2. 然后前端也需要指向我们自己定义的 login请求

   ```html
   <a class="item" th:href="@{/toLogin}">
      <i class="address card icon"></i> 登录
   </a>
   ```

3.  我们登录，需要将这些信息发送到哪里，我们也需要配置，login.html 配置提交请求及方式，方式 必须为post: 在 loginPage()源码中的注释上有写明：

   ```html
   <form th:action="@{/login}" method="post">
      <div class="field">
         <label>Username</label>
         <div class="ui left icon input">
            <input type="text" placeholder="Username" name="username">
            <i class="user icon"></i>
         </div>
      </div>
   <div class="field">
      <label>Password</label>
      <div class="ui left icon input">
         <input type="password" name="password">
         <i class="lock icon"></i>
      </div>
   </div>
   <input type="submit" class="ui blue submit button"/>
   </form>
   ```

4. 这个请求提交上来，我们还需要验证处理，怎么做呢？我们可以查看 formLogin() 方法的源 码！我们配置接收登录的用户名和密码的参数！

   ```java
   http.formLogin()
   .usernameParameter("username")
   .passwordParameter("password")
   .loginPage("/toLogin")
   .loginProcessingUrl("/login"); // 登陆表单提交请求
   
   ```

5. 在登录页增加记住我的多选框

   ```html
   <input type="checkbox" name="remember"> 记住我
   ```

6.  后端验证处理！

   ```java
   //定制记住我的参数！
   http.rememberMe().rememberMeParameter("remember");
   ```

### 连接真实的数据库

这里我用的是整合mybatis，具体的controller，service和mapper我就不写了

1. 书写我们的配置类需要继承UserDetailsService

   ```java
   @Component
   public class DetailsServiceImpl implements UserDetailsService {
       @Autowired
       private EmployeeMapper employeeMapper;
       //总目标：根据表单提交的用户名查询User对象，并装配角色，权限等信息
       @Override
       public UserDetails loadUserByUsername(
               //表单提交的用户名
               String username
   
       ) throws UsernameNotFoundException {
           //从数据库查询employee对象
           List<Employee> employee = employeeMapper.getDeptByusername(username);
           for (Employee emp: employee) {
               System.out.println(emp);
           }
   
           //2.给employee设置角色权限信息
           List<SimpleGrantedAuthority> authorities=new ArrayList<SimpleGrantedAuthority>();
           for (Employee role: employee) {
               authorities.add(new SimpleGrantedAuthority(role.getRole()));
           }
           //把employee对象和authorities封装到一起UserDetail中
           Employee employee1 = employee.get(0);
           String password = employee1.getPassword();
           return new User(username,password,authorities);
       }
   }
   ```

2. 装配userDetailsService对象

   ```java
    @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           //装配userDetailsService对象
         auth.userDetailsService(detailsServiceImpl).passwordEncoder(passwordEncoder);
       }
   ```

   