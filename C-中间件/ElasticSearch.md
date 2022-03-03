---
title: ElasticSearch
author: Wang Yue Niao
top: false
toc: true
date: 2020-12-16 17:14:13
tags: ElasticSearch
categories: ElasticSearch
---

# ElasticSearch

## 概述

- Elaticsearch，简称为es，es是一个开源的**高扩展**的**分布式全文检索引擎**，
- 它可以近乎**实时的存储、检索数据；**本身扩展性很好，可以扩展到上百台服务器，处理PB级别（大数据时代）的数据。
- es也是使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Luncene的复杂性，从而让全文搜索变得简单。

## ES安装及head插件安装

### 安装

**直接解压就可以了**

![kI0yz.png](https://s3.jpg.cm/2020/12/17/kI0yz.png)

**目录结构**

```
bin       启动文件
config    配置文件
    log4j2               日志配置文件
    jvm.options jav      a虚拟机相关配置
    elasticsearch.yml    ealsticsearch的配置文件！默认9200端口！跨域！
lib       相关jar包
logs      日志！
modules   功能模块
plugins   插件！
```

**启动**

1. 打开bin目录

2. 双击elasticsearch.bat文件

   ![](https://s3.jpg.cm/2020/12/17/kI37u.png)

3. 访问9200

   ![kID9Q.png](https://s3.jpg.cm/2020/12/17/kID9Q.png)

### 安装可视化界面 head插件

   **也是直接解压就可以了**

1. 下载插件，在elasticsearch-head-master目录下执行cnpm install

2. 解决跨域问题，在elasticsearch.yml文件中配置如下

   ![kL5kO.png](https://s3.jpg.cm/2020/12/17/kL5kO.png)

3. 启动 npm install

### 安装Kibana

1. 解压即用

2. 解压后的目录

   ![kLABr.png](https://s3.jpg.cm/2020/12/17/kLABr.png)

3. 启动，双击bin目录下的kibana.bat文件 ，访问测试

   ![kLify.png](https://s3.jpg.cm/2020/12/17/kLify.png)

4. 汉化i18n.locale: "zh-CN"

   ```
   修改elasticsearch\kibana-7.6.1-windows-x86_64\config目录下的kibana.yml如下：
   
   i18n.locale: "zh-CN"
   ```

   

  ## ES核心概念

**elasticsearch（集群）中可以包含多个索引（数据库），每个索引中可以包含多个类型（表），每个类型下又包含多个文档（行），每个文档中又包含多个字段（列）**

| Relational DB      | Elasticsearch         |
| ------------------ | --------------------- |
| 数据库（database） | 索引（indices）       |
| 表（tables）       | types（慢慢会被弃用） |
| 行（rows）         | documents             |
| 子弹（columns）    | fields                |

**物理设计：**

elasticsearch在后台把每个**索引划分成多个分片**，每分分片可以在集群中的不同服务器间迁移

**逻辑设计：**

一个索引类型中，包含多个文档，比如说文档1，文档2.当我们索引一篇文档时，可以通过这样的一个顺序找到它：索引>类型>文档ID，通过这个组合我们就能所引导某个值的文档。注意：ID不必是整数，实际上他是个字符串。

###  索引（index）

在elasticsearch中，索引有两种含义：

- 一种是名词意义上的索引。我们通常所说的索引也大多指这一种，这时候的索引就是文档(Document)的容器，是具有某种相似特性文档的集合，所以，在设计elasticsearch时，通常将相似的文档存放在同一个索引中（当然，个别document可能多一个或少一个field），这样Elasticsearch对磁盘存储的利用率最高。例如，对于博客类系统，可以将文章（artical）存放为一个索引中，将作者（author）存放为一个索引。索引由一个名称（必须全部是小写，不能以'_', '-', 或 '+'开头）标识，当对其中的文档执行索引、搜索、更新和删除操作时，该名称用于引用索引。通常索引是由两部分构成：Mapping和Setting。Mapping 定义该索引包含的文档的数据结构的信息；Setting定义了该索引的数据分布信息。
- 另一种索引是指动词意义上的索引。保存一个文档到索引(名词)的过程，就类似于SQL语句中的 INSERT关键词，当然，如果文档已存在，那就相当于数据库的UPDATE。



### 类型（type）

就目前7.X版本的ES来说，再谈类型（type）的概念已经不是太有意义，因为类型的概念正在被逐渐淡化抛弃。

在5.X或者更早的版本中，类型的概念确实很常用，类型是索引内部的逻辑分区(category/partition)，然而其意义完全取决于用户需求。因此，一个索引内部可定义一个或多个类型(type)。一般来说，类型就是为那些拥有相同的域的文档做的预定义。例如，在一个博客系统类的索引中，可以定义一个用于存储作者信息的类型，一个存储博客文章信息的类型，以及一个存储评论数据的类型。类比传统的关系型数据库领域来说，类型相当于“表”。

然而，从6.X开始单个索引中只能有一个类型，7.X的版本中只存在一个默认的名为“_doc”的类型，官方也不建议继续使用这一概念，甚至8.X以后完全不支持。这是为什么呢？

许多资料指出ES中“索引”的概念和关系数据库的“库”是相似的，“类型”和“表”是对等的。 这其实是一个不正确的对比，掩盖了一些基础的特性。在关系型数据库里,"表"是相互独立的,一个“表”里的列和另外一个“表”的同名列没有关系，互不影响。但在类型里字段不是这样的。在一个ES索引里，所有不同类型的同名字段内部使用的是同一个lucene字段存储。也就是说，上面例子中，作者信息类型的author_name字段和博客文章信息类型的author_name字段是存储在一个字段里的，两个类型里的这一字段必须有一样的字段定义。这可能导致一些问题，例如我们希望在类型1中的phone_number存储为数值类型，而在类型2中的phone_number字段存储为字符类型，这是不可能做到的。另外，在同一个索引中，存储仅有小部分字段相同或者全部字段都不相同的文档，会导致数据稀疏，影响Lucene有效压缩数据的能力。

所以，在一个索引中，最好只拥有一个类型，如果有多种实体数据，那就分别存储在多个索引中。



### 文档（document）

在索引中存储的每一个目标数据称为文档，ES中的文档可以类比关系型数据库中的每一行记录来理解。ES是面向文档的搜索，文档是ES所有可搜索数据的最小单元。在ES中文档会被序列化成json格式进行保存，每个文档都会有一个Unique ID，这个ID可以有用户在创建文档的时候指定，在用户未指定时则由ES自己生成，这个ID中在查询结果中以_id这个元数据表示。除了_id外，文档中还包括一下这些元数据：

- _index：文档所属索引名称。
- _type：文档所属类型名。
- _id：文档的ID值。
- _version：文档的版本信息。ES通过使用version来保证对文档的变更能以正确的顺序执行，避免乱序造成的数据丢失。
- _seq_no：严格递增的顺序号，每个文档一个，Shard级别严格递增，保证后写入的Doc的_seq_no大于先写入的Doc的_seq_no。
- primary_term：primary_term也和_seq_no一样是一个整数，每当Primary Shard发生重新分配时，比如重启，Primary选举等，_primary_term会递增1
- found：查询的ID正确那么ture, 如果 Id 不正确，就查不到数据，found字段就是false。
- _source：文档的原始JSON数据，这才是我们储存在ES中的业务数据。



### 映射（mapping）

ES中中的映射（Mapping）用来定义一个文档，类似于关系型数据库中的表结构的概念，但却比表结构具有更加强大的功能，如下图所示，具体来说主要包含以下功能：

- 文档中哪些字段需要定义成全文索引字段。
- 文档中哪些字段定义为精确值，例如日期，数字、地理位置等。
- 文档中哪些字段需要被索引（能通过该字段的值查询文档）。
- 日期值的格式。
- 动态添加字段的规则定义等。

映射可以分为动态映射和静态映射：

静态映射：若在数据写入索引前，映射（mapping）已经创建，那么在写入数据时，ES会根据映射和写入数据的key进行对比，然后写入相应的映射中；

动态映射：如果mapping没有创建，elasticsearch将会根据写入的数据的key自行创建相应的mapping，并写入数据。



### 集群（cluster）

集群是一个或多个节点(服务器)的集合，它们共同保存你的整个数据，并提供跨所有节点的联合索引和搜索功能。一个集群由一个唯一的名称标识，默认这个唯一标识的名称是"elasticsearch"（可修改）。这个名称很重要，因为如果节点被设置为按其名称加入集群，那么节点只能是集群的一部分。

注意，在同一环境中用最好使用不同的集群名称，否则可能导致节点加入到错误的集群中。例如，你可以使用"logging-dev", "logging-test", "logging-prod"分别用于开发、测试和正式集群的名字。

ES集群实际上是一个分布式系统，这是ES的一大优势，它需要具备两个特性：

　　1）高可用性

　　　　a）服务可用性：允许有节点停止服务；

　　　　b）数据可用性：部分节点丢失，不会丢失数据；

　　2）可扩展性

　　　　随着请求量的不断提升，数据量的不断增长，系统可以将数据分布到其他节点，实现水平扩展；

ES集群有三种健康状态：

- green：所有主要分片和复制分片都可用
- yellow：所有主要分片可用，但不是所有复制分片都可用
- red：不是所有的主要分片都可用

我们可以通过查询语句对健康状态进行查询，关注和提升ES集群的健康状态对保证数据安全性、高效查询十分有必要。



### 节点（node）

节点是一个ElasticSearch的实例，其本质就是一个Java进程，负责进行数据存储，并且参与集群的索引和搜索功能；一台机器上可以运行多个ElasticSearch实例，但是建议在生产环境中一台机器上只运行一个ElasticSearch实例。

与集群一样，节点由一个名称标识，默认情况下，该名称是在启动时分配给节点的随机通用唯一标识符（UUID）。如果不希望使用默认值，则可以定义所需的任何节点名称。

一个集群可以有任意数量的节点。此外，如果在网络上当前没有运行任何节点，那么此时启动一个节点将默认形成一个单节点的名字叫"elasticsearch"的集群。一个节点可以通过配置集群名称来加入到一个特定的集群中。默认情况下，每个节点都被设置加入到一个名字叫"elasticsearch"的集群中，这就意味着如果你启动了很多个节点，并且假设它们彼此可以互相发现，那么它们将自动形成并加入到一个名为"elasticsearch"的集群中。

在一个ES的集群中包含着多个ES的节点，往往每个节点所扮演的角色也不尽相同，ES的节点类型主要包含以下几类：

- Master-eligible Node：每个节点启动后，默认是一个 Master-eligible 节点，Master-eligible的节点可参加选主流程，成为Master节点，通过配置项 node.master:falase 可以禁用节点的Master-eligible职责，禁止后当前节点就不会参加选主流程；
- Master Node：ES集群中虽然每个节点都保存了集群状态，但是只有Master节点才有修改集群状态的权限，集群状态包括：集群中节点信息、所有索引和其相关的Mapping和Setting信息、分片的路由信息。在集群启动时，第一个启动的Master-eligible节点会将自己选举为主节点；
- Data Node：保存数据的节点，负责保存分片数据，对数据扩展有重要作用；
- Coordinating Node：负责接受Client请求，将请求分发到合适的节点获取响应后，将结果最终汇集在一起，每个节点默认都有Coordinating节点的职责；
- Machine Learning Node：负责运行机器学习的Job，用来做异常检测；
- Ingest Node：数据预处理的节点，支持Pipeline管道设置，可以使用Ingest对数据进行过滤、转换等操作。

注意，每个ES节点可以承担多个职责。



### 分片（shard）

ES的shard（分片）机制可将一个索引内部的数据分布地存储于多个节点，它通过将一个索引切分为多个底层物理的Lucene索引完成索引数据的分割存储功能，这每一个物理的Lucene索引称为一个分片(shard)。每个分片其内部都是一个全功能且独立的索引，因此可由集群中的任何主机存储。创建索引时，用户可指定其分片的数量，默认数量为5个。

Shard有两种类型：primary和replica，即主分片及副本分片。主分片用于文档存储，每个新的索引会自动创建5个主分片，当然此数量可在索引创建之前通过配置自行定义，不过，一旦创建完成，其主分片的数量将不可更改。Replica分片是主分片的副本，用于冗余数据及提高搜索性能，为保证副本分片的有效性，主分片绝不会被创建在同一节点上。每个主分片默认配置了一个副本，但也可以配置多个，且其数量可动态更改。ES会根据需要自动增加或减少这些Replica shard的数量。

ES集群可由多个节点组成，各分片分布式地存储于这些节点上。ES可自动在节点间按需要移动分片，例如增加节点或节点故障时。简而言之，分片实现了集群的分布式存储，而副本实现了其分布式处理及冗余功能。

## IK分词器

### 安装

**直接将下载完的文件解压到plugins中，建个文件夹名字ik，把解压完的文件放进去**

![kP9lT.png](https://s3.jpg.cm/2020/12/17/kP9lT.png)

### 使用kibana测试

其中ik_smart为最少切分

![kPTEE.png](https://s3.jpg.cm/2020/12/17/kPTEE.png)

il_max_word为最细粒度划分！

![kPajQ.png](https://s3.jpg.cm/2020/12/17/kPajQ.png)

- **发现问题，自己需要的词被拆分开了怎么办？**

  - 这种自己需要的词，需要我们自己加载到我们的分词器的字典中

    ![kPZI2.png](https://s3.jpg.cm/2020/12/17/kPZI2.png)

## Rest风格说明

一种软件架构风格，而不是标准，知识提供了一组设计原则的约束条件。他主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

| method | url地址                                         | 描述                   |
| ------ | ----------------------------------------------- | ---------------------- |
| PUT    | localhost:9200/索引名称/类型名称/文档id         | 创建文档（指定文档id） |
| POST   | localhost:9200/索引名称/类型名称                | 创建文档（随机文档id） |
| POST   | localhost:9200/索引名称/类型名称/文档id/_ipdate | 修改文档               |
| DELETE | localhost:9200/索引名称/类型名称/文档id         | 删除文档               |
| GET    | localhost:9200/索引名称/类型名称/文档id         | 查询文档通过文档id     |
| POST   | localhost:9200/索引名称/类型名称/_search        | 查询所有数据           |

### 基础测试

1. 创建一个索引

   ```
   PUT /索引名/类型名（未来就不写了）/文档id
   {请求体}
   ```

   ![kRlUy.png](https://s3.jpg.cm/2020/12/17/kRlUy.png)

2. 字段类型

   - 字符串类型：text、keyword
   - 数值类型：long、integer、short、byte、double、float、scaled、float
   - 日期类型：date
   - 布尔值类型：boolean
   - 二进制类型：binary

3. 指定字段类型

   ![kR6or.png](https://s3.jpg.cm/2020/12/17/kR6or.png)

   

## 关于文档的基本操作（重点）

### 简单操作

1. 添加数据

   ```
   PUT /wang/usr/3
   {
     "name": "李四",
     "age": 30,
     "desc": "一顿操作猛如虎",
     "tags": ["靓女","渣男","旅游"]
   }
   ```

   ![kR12G.png](https://s3.jpg.cm/2020/12/17/kR12G.png)

2. 获取数据 GET

   ![kRtbX.png](https://s3.jpg.cm/2020/12/17/kRtbX.png)

3. 更新数据 POST _update，推荐使用这种更新方式

   ![kRSDp.png](https://s3.jpg.cm/2020/12/17/kRSDp.png)
   
4. 搜索

   ![kRsFT.png](https://s3.jpg.cm/2020/12/17/kRsFT.png)

### 复杂操作

   **查询的参数使用json**![kRk7Q.png](https://s3.jpg.cm/2020/12/17/kRk7Q.png)



```json
#输出结果不想要那么多,例如只想要name和desc两个字段
GET /wang/usr/_search
{
    "query": {
        "mathc" :{
            "name" :"鸟"
        }
    },
    "_source" :["name","desc"]
}
```

```json
#排序，通过age字段进行排序
GET /wang/usr/_search
{
    "query":{
        "mathc":{
            "name" :"鸟"
        }
    },
    "_source" :["name","desc"],
    "sort":[
        {
            "age":{
                "order":"desc"
            }
        }
    ]
}
```

```json
#分页
GET /wang/usr/_search
{
    "query":{
        "mathc":{
            "name" :"鸟"
        }
    },
    "_source" :["name","desc"],
    "sort":[
        {
            "age":{
                "order":"desc"
            }
        }
    ],
    "from": 0,   #表示从第几行开始
    "size": 20   #表示查询多少条数据
}
```

```json
#bool查询的使用

#must（and），所有条件都要符合
#should（or），符合其中一个条件即可
#must_not，   不满足
GET /wang/usr/_search
{
    "query":{
        bool:{
            "must":[
                {
                     "mathc" :{
                         "name" :"鸟"
                     }
                },
                {
                    "mathc" :{
                         "age" :23
                     }
                }
            ]
        }
    },
}
```

```json
#过滤，过滤出来年龄大于等与10小于等与20
#gt   大于
#gte  大于等与
#lt   小于
#lte  小于等与
GET /wang/usr/_search
{
  "query": {
      "bool":{
          "must_not":[
              {
                  "match":{
                  "age": 23
              }
              }
          ],
          "filter":{
              "range":{ #范围
                  "age":{
                      "gte": 10,
                      "lte": 20
                  }
              }
          }
      }
  }
}
```

```json
# 如果一个条件多个值，可以直接空格
GET /wang/usr/_search
{
    "query": {
        "mathc" :{
            "tags" :"直男 旅游"
        }
    },
}
```

**关于分词**

- term，直接查询精确的
- match，会使用分词器解析！（先分析文档，然后通过分析的文档进行查询！）

**两个类型 text和keyword**

- 会被拆分
- 不会被拆分

