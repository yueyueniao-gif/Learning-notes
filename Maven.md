---
title: Maven
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-18 01:52:05
tags: maven
categories: maven
---

### Maven的下载安装参考[传送门](https://www.cnblogs.com/xdp-gacl/p/3498271.html)

### Maven工程

```
POM工程
POM 工程是逻辑工程。用在父级工程或聚合工程中。用来做 jar 包的版本控制。

JAR工程
将会打包成 jar 用作 jar 包使用。即常见的本地工程     -Java Project

WAR工程
将会打包成 war，发布在服务器上的工程。如网站或服务。即常见的网络工程 - Dynamic Web Project。
war工程默认没有 WEB-INF 目录及 web.xml 配置文件，IDE 通常会显示工程 错误，提供完整工程结构可以解决。
```

##### 创建maven工程师的目录

```
Group Id 公司域名倒写 
Artifact Id 项目名 
Version 版本名 
Packaging 项目类型 
Jar java 项目 
War : web 项目 
POM: 父项目.如果项目有子项目,项目必须是 pom 
Name : 自定义名称,内容任意 
Description: 描述.详细描述. 
Parent Project: 如果当前项目有父项目时填写
```

##### maven项目结构

```
src/main/java 这个目录下储存 java 源代码 
src/main/resources 储存主要的资源文件。比如spring的xml配置文件和log4j的properties 文件。 src/test/java 储存测试用的类，比如 JUNIT 的测试一般就放在这个目录下面 
src/test/resources 储存测试用的资源文件 
src 包含了项目所有的源代码和资源文件，以及其他项目相关的文件。 
target 编译后内容放置的文件夹
pom.xml 是 Maven 的基础配置文件。配置项目和项目之间关系，包括配置依赖关系等 等
```

##### 工程关系

###### 依赖

即 A 工程开发或运行过程中需要 B 工程提供支持，则代表 A 工程依赖 B 工程。在这种 

情况下，需要在 pom.xml 文件中增加下属配置定义依赖关系

```
<dependencies> 
    <!-- 定义一个具体的依赖 -->
    <dependency> 
        <!-- 依赖的工程所在组名 -->
        <groupId>groupName</groupId> 
        <!-- 依赖的工程名 --> 
        <artifactId>artifactName</artifactId>
        <!-- 依赖的工程版本 --> 
        <version>versionNo</version> 
        <!-- 依赖的工程有效范围，其可选值有： 
        compile - 编译中有效 
        runtime - 运行中有效 
        system - 全部中有效[默认] 
        provided - 当前工程中有效. 
        test - 只在测试有效 -->
        <scope>system</scope> 
    </dependency> 
</dependencies>
```

###### 继承

如果 A 工程继承 B 工程，则代表 A 工程默认依赖 B 工程依赖的所有资源，且可以应用 

B 工程中定义的所有资源信息。被继承的工程（B 工程）只能是 POM 工程。

###### 聚合

当我们开发的工程拥有 2 个以上模块的时候，每个模块都是一个独立的功能集合。比如 某大学系统中拥有搜索平台，学习平台，考试平台等。开发的时候每个平台都可以独立编译， 测试，运行。这个时候我们就需要一个聚合工程。 在创建聚合工程的过程中，总的工程必须是一个 POM 工程（Maven Project），各子模 块可以是任意类型模块（Maven Module）。所有聚合工程和聚合模块必须处于同一个组 （groupId）中，且聚合工程可以嵌套

### Maven常用命令

```
1 install 本地安装， 包含编译，打包，安装到本地仓库 编译 - javac 打包 - jar， 将 java 代码打包为 jar 文件 安装到本地仓库 - 将打包的 jar 文件，保存到本地仓库目录中。 

2 clean 清除已编译信息。 删除工程中的 target 目录。 

3 compile 只编译。 javac 命令 

4 deploy 部署。 常见于结合私服使用的命令。 相当于是 install+上传 jar 到私服。 包含编译，打包，安装到本地仓库，上传到私服仓库。 

5 package 打包。 包含编译，打包两个功能。
```

### 项目架构设计

- 传统项目设计方式

![](https://s3.jpg.cm/2020/08/18/ubnZr.png)

- maven项目设计方式

![](https://s3.jpg.cm/2020/08/18/ub7X5.png)

#### 创建工程

###### 先创建Parent工程

1. 创建完父工程，将父工程的src删掉就可以了，因为父工程的作用是管理jar包  不参与任何逻辑算法。

2. pom.xml依赖

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.wangyueniao</groupId>
       <artifactId>aaa</artifactId>
       <!--父工程打包方式为pom-->
       <packaging>pom</packaging>
       <version>1.0-SNAPSHOT</version>
   
       <!--modules 模块标签
         1.  所有的父级工程  作用是管理jar包  不参与任何逻辑算法。所以可以干掉 SRC
         2.  列举了父工程中的所有子模块,子工程可以自动继承父级工程中的jar包  
         3.  每一个子模块都代表了不同的包（package）
       -->
       
       <!--表示该父工程下的子工程-->
       <modules>
           <module>bbb</module>
           <module>ccc</module>
           <module>ddd</module>
           <module>eee</module>
       </modules>
       
       <properties>
   		<!-- 声明属性，对Spring的版本进行统一管理 -->
   		<wangyueniao.spring.version>4.3.20.RELEASE</wangyueniao.spring.version>
   	</properties>
       
   1. 被<denpencyManagement>所控制的jar包，不会被子工程所继承，如果子工程有需要，可以重写父工程中的jar包，并且不需要携带版本号
   2. 如果不省略版本号，则jar包会从maven中进行下载，并不再是从父工程中重写的jar包
   <dependencyManagement>
       <dependencies>
           <!-- 在父级工程中，添加一些jar 包。 mysql的驱动包 -->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.38</version>
           </dependency>
           
          <dependency>
   				<groupId>org.springframework</groupId>
   				<artifactId>spring-orm</artifactId>
   				<version>${wangyueniao.spring.version}</version>
   	   </dependency>
           
           <dependency>
   				<groupId>org.springframework</groupId>
   				<artifactId>spring-webmvc</artifactId>
   				<version>${wangyueniao.spring.version}</version>
   		</dependency>
   		
           <!--
               spring
           -->
           
           <!--
               mybatis
           -->
           
           <!--
               shiro
           -->
       </dependencies>
       </dependencyManagement>
   </project>
   
   ```

   ###### 

###### 创建子工程，以bbb为例

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <!--表示该工程的父工程-->
    <parent>
        <artifactId>aaa</artifactId>
        <groupId>com.wangyueniao</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>bbb</artifactId>

    <!--
      1.  子工程如果需要，重写父级工程的jar包即可，但是不能 携带版本号
      2.将父级工程中的jar包 直接复制过来，但是不可携带版本号。
    -->

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        
          <dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-orm</artifactId>
	   </dependency>
    </dependencies>


</project>
```

