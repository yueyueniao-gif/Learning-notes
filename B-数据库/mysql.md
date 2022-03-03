---
title: mysql
author: Wang Yue Niao
top: false
toc: true
date: 2020-11-14 16:44:33
tags: mysql
categories: mysql
---

# mysql高级

![YpicW.png](https://s3.jpg.cm/2020/11/16/YpicW.png)



## 索引优化分析

### 索引简介

### 性能分析

#### MySql Query Optimizer

![](https://s3.jpg.cm/2020/11/15/YkAzS.png)

####  MySql常见瓶颈

- CPU:CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候 
- IO：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候
- 服务器硬件的性能瓶颈：yop,free,iostat和vmstat来查看系统的性能状态。

#### Explain

##### 是什么

- 使用EXPLAIN关键字可以**模拟优化器执行SQL查询语句**，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈。

##### 能干什么

- 表的读取顺序
- 数据读取操作的操作类型
- 那些索引可以被使用
- 那些索引被实际使用
- 表之间的引用
- 每张表有多少行被优化器查询

##### 怎么玩

- Explain+SQL语句

- 执行计划包含的信息

  [![YkUc2.png](https://s3.jpg.cm/2020/11/15/YkUc2.png)](https://imagelol.com/image/YkUc2)

- id

  - select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序。
  - 三种情况
    - id相同，执行顺序由上至下
    - id不同，如果是子查询，id的 序号会递增，id值越大优先级越高，越先被执行
    - id相同不同，同时存在

- select_type

  - [![YkJEU.png](https://s3.jpg.cm/2020/11/15/YkJEU.png)](https://imagelol.com/image/YkJEU)
  - SIMPLE:简单的select查询，查询中不包含子查询或者UNION
  - PRIMARY：查询中若包含任何复杂的子查询，最外层查询则被标记为PRIMARY
  - SUNQUERY:在SELECT或WHERE列表中包含了子查询
  - DERIVED：在FROM列表中包含的子查询被标记为DERIVED（衍生）MySQL会递归执行这些子查询，把结果放在临时表里 
  - UNION：若第二个SELECT出现在UNION之后， 则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
  - UNION RESULT：从UNION表获取结果的SELECT

- table

  - 显示这一行的数据是关于哪张表的

- type

  - [![YkthO.png](https://n1.i5h5.com/2020/11/15/YkthO.png)](https://imagelol.com/image/YkthO)
  - 显示查询使用了何种类型，**从最好到最差依次是 system>const>eq_ref>range>index>ALL**
  - system：表只有一行记录（等与系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计。
  - const：表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。
  - eq_ref：唯一性索引扫描，对于每个索引键，表汇总只有一条记录与之匹配。常见于主键或唯一索引扫描
  - ref：非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，他可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。
  - range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。
  - index：Full Index Scan，Index与ALL区别为index类型只遍历索引树。这通常比ALL块，因为索引文件通常比数据文件小。
  - ALL：Full Table Scan，将遍历权标以找到匹配的行。

- possible_keys:

  - 显示可能引用在这张表中的索引，一个或多个（查询涉及到的字段上若存在索引，则该索引将被列出，**单不一定被查询实际使用**）

- key: 

  - 实际使用的索引。如果为NULL，则表示没有使用索引（**查询中若使用了覆盖索引，则该索引仅出现在key列表中**）

- key_len：

  - 表示索引总使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好
  - key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的

- ref:

  - 显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。

- rows:

  - 根据表统计信息以及索引选用的情况，大致估算出找到所需的记录所需要读取的行数

- Extra: 包含不适合在其他列中显示，但十分重要的额外信息

  - Using filesort：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序成为“文件排序”
  - Using tempopary：使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by。
  - USING index：表示相应的select操作中使用了覆盖索引，避免访问了表的数据行，效率不错。
    - 如果同时出现using where，表明索引被用来执行索引键值的查找。
    - 如果没有同时出现using where 表明索引用来读取数据而而非执行查找动作，
  - Using where：标明使用了where过滤。
  - using join buffer：使用了连接缓存。
  - impossible where：where子句的值总是false，不能用来获取任何元组
  - select tables optimized away：在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段在进行计算，查询执行计划生成的阶段即完成优化。
  - distinct：优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作。 

### 索引优化

##### 索引分析

- 左连接加右表索引（左连接左边全都有）
- 右连接加左表索引（右链接右边全都有）
- 索引最好设置在需要经常查询的字段中

- join的优化
  - 尽可能减少join语句中的的循环总次数：“ 永远用小结果集驱动大的结果集 ”
  - 优先优化内层循环
  - 保证Join语句中被驱动表上Join条件字段已经被索引

##### 索引实现（应该避免）

- 全值匹配我最爱

- 最佳左前缀法则：如果索引了多列，要遵守左前缀法则。指的是查询**从索引最左前列开始**并且**不跳过索引中的列。**

- 不在索引列上做任何操作（计算、函数（自动or手动）类型转换），会导致索引失效而转向全表扫描

  - 例如：

    ![](https://n1.i5h5.com/2020/11/15/YpurU.png)

- 存储引擎不能使用索引中范围条件右边的列（范围之后全失效）

- 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *

- mysql在使用不等于（！=或者<>）的时候无法使用索引会导致全表扫描

- is null，is not null也无法使用索引

- like以通配符开头（‘ %abc... ’）mysql索引失效会变成全表的扫描（%like加左边）

  - 问题：解决like' %字符串%时索引不被使用的方法 '
    - 解决方案，覆盖索引（查询列要被所建的索引覆盖）

- 字符串不加单引号索引失效

- 少用or，用它来连接时会索引失效

- 口诀

  - 全值匹配我最爱，最左前缀要遵守

  - 带头大哥不能死，中间兄弟不能断（永远要符合最佳左前缀原则）

  - 索引列上不计算，范围之后全失效

  - LIKE百分写最右，覆盖索引不写*

  - 不等控制还有or，索引失效要少用

  - VAR引号不可丢，SQL高级也不难

    ![](https://s3.jpg.cm/2020/11/15/Yp1CO.png)


##### 一般建议

- 对于单键索引，尽量选择针对当前query过滤性更好的索引
- 在选择组合索引的时候，当前Query过滤性最好的字段在索引字段顺序中，位置越靠前越好。
- 在选择组合索引的时候，尽量选择可以能够包含当前query的where子句中农更多字段的索引
- 尽可能通过分析统计信息和调整query的写法来达到选择和是索引的目的

## 查询截取分析

### 查询优化

##### 永远小表驱动大表类似嵌套循环Nested Loop

![Ypxv2.png](https://s3.jpg.cm/2020/11/16/Ypxv2.png)

![YpDiH.png](https://s3.jpg.cm/2020/11/16/YpDiH.png)

##### order by关键字优化

- ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序

  - MySQL支持两种方式排序，FileSort和Index，Index效率高，它指MySQL扫描索引本身完成排序，FileSort方式效率低。
  - ORDER BY满足两种情况，会使用Index方式排序
    - ORDER BY语句使用索引最左前列
    - 使用Where子句与Order BY子句条件组合满足索引最左前列

- 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀

- 如果不在索引列上，filesort有两种算法：mysql就要启动双路排序和单路排序

  - 双路排序
    - MySQL4.1之前是使用双路排序，字面意思就是两次扫描磁盘，最终得到数据，读取行指针和order by列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取。
    - 从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。
  - 单路排序
    - 从磁盘读取查询需要的所有列，按照order by列在buffer对他们进行排序，然后你扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是他会使用更多的空间，因为他把每一行都保存在内存中了。
  - 由于单路是后出的，总体而言好过双路，但是单路有问题。
    - 在sort)buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出，所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排完再取sort_buffer容量大小，再排....从而多次IO。

- 优化策略

  - 增大sort_buffer_size参数的设置

  - 增大max_length_for_data参数的设置
  - order by时select *是一个大忌，只query需要的字段，这点非常重要

- ![YpZsf.png](https://s3.jpg.cm/2020/11/16/YpZsf.png)

##### GROUP BY关键字优化

- group by实质是先排序后进行分组，遵照索引建的最佳左前缀
- 当无法使用索引列，增大max_length_for_sort_data参数的设置+增大sort_buffer_size参数的设置
- where高于having，能写在where限定的条件就不要去having限定了

### 慢查询日志

#### 是什么

- mysql的慢查询日志是mysql提供的一种日志记录，他用来记录在mysql中相应时间超过阈值的语句，具体指运行时间超过long_query_time值的sql，则会被记录到慢查询日志中。
- 具体运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query)time的默认值为10，意思是运行10秒以上的语句。
- 由他来查看哪些SQL超出了我们的最大忍耐是时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前的explain进行全面分析。

#### 怎么玩

##### 说明

- 默认情况下，MySQL数据库没有开启慢查询日志，需要我们手动来设置这个参数
- 当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会多少带来一定性能影响。慢查询日志支持将日志记录写入文件

##### 查看是否开启以及如何开启

- 默认：SHOW VARIABLES LIKE '%slow_query_log%';
- 开启：set global slow_query_log=1;
- ![YvPJw.png](https://s3.jpg.cm/2020/11/16/YvPJw.png)
- 参数 slow_query_Log_file，他指定慢查询日志文件的存放路径，系统默认会给一个缺省的文件host_name-slow.log（如果没有指定参数slow_query_log_file的话）

##### 开启了慢查询日志后，什么样的SQL才会记录到慢查询日志里面呢？

- 这个是由参数long_query_time控制，默认情况下long_query_time的值为10秒。
- 命令：SHOW VARIABLES LIKE 'long_query_time%';
- 如果运行时间正好等与long_query_time,并不会被记录下来。

##### case

- 查看当前多少秒算慢：SHOW VARIABLES LIKE 'long_query_time%';
- 设置慢的阈值时间： set global long_query_time=3;
- 为什么设置后看不出变化？
  - 需要重新连接或新开一个会话才能看到修改值：SHOW VARIABLES LIKE 'long_query_time%';
  - 或者使用：show global variables like 'long_query_time';查看
- 查询当前系统有多少条慢查询记录：show global status like '%Slow_queries%';
- mysql永久生效配置
  - slow_query_log=1;
  - slow_query_log_file=路径
  - long_query_time=3;

##### 日志分析工具mysqldumpslow

![Yv7iy.png](https://s3.jpg.cm/2020/11/16/Yv7iy.png)

**工作常用参考**

![Yvdlr.png](https://s3.jpg.cm/2020/11/16/Yvdlr.png)

### Show Profile

#### 是什么

是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量

#### 官网

http://dev.mysql.com/doc/refman/5.5/en/show-profile.html

默认情况下，参数处于关闭状态，并保存最近15次运行结果。

#### 分析步骤

##### 是否支持

- 看看当前mysql版本是否支持
- Show variables like 'profiling';
- [![Y4uxk.md.png](https://s3.jpg.cm/2020/11/16/Y4uxk.md.png)](https://imagelol.com/image/Y4uxk)

##### 开启功能

- 默认是关闭，使用前需要开启
- set profiling=on;
- [![Y4H6e.md.png](https://s3.jpg.cm/2020/11/16/Y4H6e.md.png)](https://imagelol.com/image/Y4H6e)

##### 开启SQL

- select * from emp group by id%10 limit 150000;
- select * from emp group by id%20 order by 5;

##### 查看结果

- show profiles;

##### 诊断SQL

- show profile cpu,block io for query +(show profiles查询出来的SQL的前面的id)

##### 日常开发需要注意的结论

[![Y4tjr.md.png](https://s3.jpg.cm/2020/11/16/Y4tjr.md.png)](https://imagelol.com/image/Y4tjr)

### 全局查询日志

- 配置启用

  - 在mysql的my.cnf中，设置如下

    ```json
    #开启
    general_log=1
    #记录日志文件的路径
    general_log_file=/path/logfile
    #输出格式
    log_output=FILE
    ```

- 编码启用

  - set global gener_log=1;
  - set global log_output='TABLE'；
  - 此后你所编写的sql语句，将会记录到mysql库里的general_log表，可以用下面的命令查看
  - select * from mysql.general_log;

- 永远不要再生产环境开启这个工程

## mysql锁机制

### 概述

#### 锁的分类

##### 从对数据操作的类型（读\写）分

- 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。
- 写锁（排他锁）：当前操作没有完成前，他会阻断其他写锁和读锁。

##### 从对数据操作的细粒度分

- 行锁

- 表锁

### 三锁

**开销、加锁速度、死锁、力度、并发性能只能就具体应用的特点来说哪种锁更合适**

#### 表锁（偏读）

##### 特点

偏向MyISAM存储引擎，开销小，加锁块；无死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。

##### 加读锁

- 手动增加表锁
  - lock table 表名字 read，表名字2 read,其他；
- 查看表上加过的锁
  - show open tables;
- [![Y4FUz.png](https://s3.jpg.cm/2020/11/16/Y4FUz.png)](https://imagelol.com/image/Y4FUz)
- [![Y4kou.png](https://s3.jpg.cm/2020/11/16/Y4kou.png)](https://imagelol.com/image/Y4kou)
- [![Y4mDG.png](https://s3.jpg.cm/2020/11/16/Y4mDG.png)](https://imagelol.com/image/Y4mDG)

##### 加写锁

- 手动增加表锁
  - lock table 表名字 write，表名字2 write,其他；
- 查看表上加过的锁
  - show open tables;
- [![Y4v64.png](https://s3.jpg.cm/2020/11/16/Y4v64.png)](https://imagelol.com/image/Y4v64)
- [![Y4wFX.png](https://s3.jpg.cm/2020/11/16/Y4wFX.png)](https://imagelol.com/image/Y4wFX)

##### 小总结

[![Y4ojD.png](https://s3.jpg.cm/2020/11/16/Y4ojD.png)](https://imagelol.com/image/Y4ojD)

**简而言之，就是读锁会阻塞写，但是不会阻塞读。而写锁则会把读和写都阻塞**

**此外，Myisam的读写锁调度室写优先，这也是myisam不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞**

#### 行锁（偏写）

##### 特点

- 偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
- InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）；二是采用了行级锁。

##### **1.事务（Transaction）及其ACID属性**

  事务是由一组SQL语句组成的逻辑处理单元，事务具有4属性，通常称为事务的ACID属性。

- 原性性（Actomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
- 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以操持完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。
- 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。
- 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

##### **2.并发事务带来的问题**

  相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持可以支持更多的用户。但并发事务处理也会带来一些问题，主要包括以下几种情况。

- **更新丢失**（Lost Update）：当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题——最后的更新覆盖了其他事务所做的更新。例如，两个编辑人员制作了同一文档的电子副本。每个编辑人员独立地更改其副本，然后保存更改后的副本，这样就覆盖了原始文档。最后保存其更改保存其更改副本的编辑人员覆盖另一个编辑人员所做的修改。如果在一个编辑人员完成并提交事务之前，另一个编辑人员不能访问同一文件，则可避免此问题
- **脏读**（Dirty Reads）：一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读”。
- **不可重复读**（Non-Repeatable Reads）：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。
- **幻读**（Phantom Reads）：一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读”。

 

##### **3.事务隔离级别**

在并发事务处理带来的问题中，“更新丢失”通常应该是完全避免的。但防止更新丢失，并不能单靠数据库事务控制器来解决，需要应用程序对要更新的数据**加必要的锁**来解决，因此，防止更新丢失应该是应用的责任。

“脏读”、“不可重复读”和“幻读”，其实都是数据库读一致性问题，必须由数据库提供一定的事务隔离机制来解决。数据库实现事务隔离的方式，基本可以分为以下两种。

一种是在读取数据前，对其加锁，阻止其他事务对数据进行修改。

另一种是不用加任何锁，通过一定机制生成一个数据请求时间点的一致性数据快照（Snapshot），并用这个快照来提供一定级别（语句级或事务级）的一致性读取。从用户的角度，好像是数据库可以提供同一数据的多个版本，因此，这种技术叫做数据多版本并发控制（ＭultiVersion Concurrency Control，简称MVCC或MCC），也经常称为多版本数据库。

  数据库的事务隔离级别越严格，并发副作用越小，但付出的代价也就越大，因为事务隔离实质上就是使事务在一定程度上“串行化”进行，这显然与“并发”是矛盾的，同时，不同的应用对读一致性和事务隔离程度的要求也是不同的，比如许多应用对“不可重复读”和“幻读”并不敏感，可能更关心数据并发访问的能力。

  为了解决“隔离”与“并发”的矛盾，ISO/ANSI SQL92定义了４个事务隔离级别，每个级别的隔离程度不同，允许出现的副作用也不同，应用可以根据自己业务逻辑要求，通过选择不同的隔离级别来平衡＂隔离＂与＂并发＂的矛盾

##### **事务４种隔离级别比较**

| 隔离级别/读数据一致性及允许的并发副作用 | 读数据一致性                             | 脏读 | 不可重复读 | 幻读 |
| --------------------------------------- | ---------------------------------------- | ---- | ---------- | ---- |
| 未提交读（Read uncommitted）            | 最低级别，只能保证不读取物理上损坏的数据 | 是   | 是         | 是   |
| 已提交度（Read committed）              | 语句级                                   | 否   | 是         | 是   |
| 可重复读（Repeatable read）             | 事务级                                   | 否   | 否         | 是   |
| 可序列化（Serializable）                | 最高级别，事务级                         | 否   | 否         | 否   |

  最后要说明的是：各具体数据库并不一定完全实现了上述４个隔离级别，例如，Oracle只提供Read committed和Serializable两个标准级别，另外还自己定义的Read only隔离级别：SQL Server除支持上述ISO/ANSI SQL92定义的４个级别外，还支持一个叫做＂快照＂的隔离级别，但严格来说它是一个用MVCC实现的Serializable隔离级别。ＭySQL支持全部４个隔离级别，但在具体实现时，有一些特点，比如在一些隔离级下是采用MVCC一致性读，但某些情况又不是。

##### **InnoDB行锁实现方式**

  InnoDB行锁是通过索引上的索引项来实现的，这一点ＭySQL与Oracle不同，后者是通过在数据中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味者：只有通过索引条件检索数据，InnoDB才会使用行级锁，否则，InnoDB将使用表锁！

  在实际应用中，要特别注意InnoDB行锁的这一特性，不然的话，可能导致大量的锁冲突，从而影响并发性能。

##### 间隙锁（Next-Key锁）

  当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙(GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。

  举例来说，假如emp表中只有101条记录，其empid的值分别是1,2,...,100,101，下面的SQL：

```
SELECT * FROM emp WHERE empid > 100 FOR UPDATE
```

  是一个范围条件的检索，InnoDB不仅会对符合条件的empid值为101的记录加锁，也会对empid大于101（这些记录并不存在）的“间隙”加锁。

  InnoDB使用间隙锁的目的，一方面是为了防止幻读，以满足相关隔离级别的要求，对于上面的例子，要是不使用间隙锁，如果其他事务插入了empid大于100的任何记录，那么本事务如果再次执行上述语句，就会发生幻读；另一方面，是为了满足其恢复和复制的需要。有关其恢复和复制对机制的影响，以及不同隔离级别下InnoDB使用间隙锁的情况。

  很显然，在使用范围条件检索并锁定记录时，InnoDB这种加锁机制会阻塞符合条件范围内键值的并发插入，这往往会造成严重的锁等待。因此，在实际开发中，尤其是并发插入比较多的应用，我们要尽量优化业务逻辑，尽量使用相等条件来访问更新数据，避免使用范围条件。

##### 如何锁定一行

- 在SQL语句后面加上for update

#### **总结**

  对于**ＭyISAM**的表锁，主要有以下几点

  （１）共享读锁（S）之间是兼容的，但共享读锁（S）和排他写锁（X）之间，以及排他写锁之间（X）是互斥的，也就是说读和写是串行的。

  （２）在一定条件下，ＭyISAM允许查询和插入并发执行，我们可以利用这一点来解决应用中对同一表和插入的锁争用问题。

  （３）ＭyISAM默认的锁调度机制是写优先，这并不一定适合所有应用，用户可以通过设置LOW_PRIPORITY_UPDATES参数，或在INSERT、UPDATE、DELETE语句中指定LOW_PRIORITY选项来调节读写锁的争用。

  （４）由于表锁的锁定粒度大，读写之间又是串行的，因此，如果更新操作较多，ＭyISAM表可能会出现严重的锁等待，可以考虑采用InnoDB表来减少锁冲突。

 

  对于**InnoDB**表，主要有以下几点

  （１）InnoDB的行销是基于索引实现的，如果不通过索引访问数据，InnoDB会使用表锁。

  （２）InnoDB间隙锁机制，以及InnoDB使用间隙锁的原因。

  （３）在不同的隔离级别下，InnoDB的锁机制和一致性读策略不同。

  （４）ＭySQL的恢复和复制对InnoDB锁机制和一致性读策略也有较大影响。

  （５）锁冲突甚至死锁很难完全避免。

  在了解InnoDB的锁特性后，用户可以通过设计和SQL调整等措施减少锁冲突和死锁，包括：

- 尽量使用较低的隔离级别
- 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会。
- 选择合理的事务大小，小事务发生锁冲突的几率也更小。
- 给记录集显示加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁。
- 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大减少死锁的机会。
- 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响。
- 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁。
- 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能。

## 主从复制

### 复制基本原理

- slave会从master读取binlog（二进制文件）来进行数据同步

- 三步骤+原理图

  - [![YBLke.png](https://s3.jpg.cm/2020/11/16/YBLke.png)](https://imagelol.com/image/YBLke)

  - master将改变记录到二进制日志（binary log）。这些记录过程叫做二进制日志事件，binary log events;
  - slave将master的binary log events拷贝到它的中继日志（relay log）
  - slave重做中继日志的事件，将改变应用到自己数据库中。MySQL复制是异步的且串行化的。

### 复制的基本原则

- 每一个slave只有一个master
- 每个slave只能有一个唯一的服务器
- 每个master可以有多个slave

### 一主一从常见配置

- mysql版本一致且后台以服务运行

- 主从都配置在[mysqld]结点下，都是小写

- 主机修改my.ini配置文件

  - **[必须]**主服务器唯一ID    
    - server-id=1
  - **[必须]**启用二进制日志
    - log-bin=自己本地的路径/mysqlbin
  - **[可选]**启用错误日志
    - log-err=自己本地路径/mysqlerr
  - **[可选]**根目录
    - basedir="自己本地路径"
  - **[可选]**临时目录
    - tmpdir="自己本地路径"
  - **[可选]**数据目录
    - datadir="自己本地路径/Data/"
  - read-only=0
    - 代表主机，读写都可以
  - **[可选]**设置不要复制的数据库
    - Binlog-ignore-db=mysql
  - **[可选]**设置需要复制的数据库
    - binlog-do-db=需要复制的主数据库名字

- 从机修改my.cnf配置文件

  - **[必须]**从服务器唯一ID
  - **[可选]**启用二进制文件

- 因修改过配置文件，请主机+从机都重启后台mysql服务

- 主机从机都关闭防火墙

- 在Windows主机上简历账户并授权slave

  - GRANT REPLICATION SLAVE ON * . * TO 'zhangsan'@''从机数据库IP'IDENTIFIED BY '123456';
  - 刷新 flush privileges;
  - 查询master的状态
    - show master status;
    - 记录下File和Position的值
  - 执行完此步骤后不要再操作主服务器MYSQL，防止主服务器状态值变化

- 在Linux从机上配置需要复制的主机

  - CHANGE MASTER TO MASTER_HOST='主机IP'，MASTER_USER= 'zhangsan' ,master_password='123456',MASTER_LOG_FILE='File名字',MASTER_LOG_POS=Position数字

  - start slave;启动从服务器复制功能

  - show slave status\G

    - 下面两个参数都是Yes，则说明主从配置成功！

      Slave_IO_Running: Yes

      Slave_SQL_Running: Yes

- 主机新建库，新建表，insert记录，从机复制

- 如何停止从服务复制功能

  - stop slave;

