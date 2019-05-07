---
title: Elasticsearch入门教程
categories: Elasticsearch
tags: Elasticsearch
date: 2019-04-30 17:49:33
---

## Elasticsearch简介

Elasticsearch是一个高度可扩展的、开源的、基于 Lucene 的全文搜索和分析引擎。它允许您快速，近实时地存储，搜索和分析大量数据，并支持多租户。

它使用Java开发并使用 Lucene 作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的 RESTful API 来隐藏 Lucene 的复杂性，从而让全文搜索变得简单。

## 基本概念

#### 集群

> 集群(cluster)是一组具有相同cluster.name的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一个节点也可以组成一个集群。
> 
> 集群由唯一名称标识，默认情况下为“elasticsearch”。此名称很重要，因为如果节点设置为按名称加入集群的话，则该节点只能是集群的一部分。
确保不同的环境中使用不同的集群名称，否则最终会导致节点加入错误的集群。

#### 节点(Node)

> 一个运行的 ES 实例就是一个节点，节点存储数据并参与集群的索引和搜索功能。
就像集群一样，节点由名称标识，默认情况下，该名称是在启动时分配给节点的随机通用唯一标识符（UUID）。如果不需要默认值，可以定义所需的任何节点名称。此名称对于管理目的非常重要，您可以在其中识别网络中哪些服务器与 Elasticsearch 集群中的哪些节点相对应。
> 
> 可以将节点配置为按集群名称加入特定集群。默认情况下，每个节点都设置为加入一个名为 cluster 的 elasticsearch 集群，这意味着如果您在网络上启动了许多节点并且假设它们可以相互发现 - 它们将自动形成并加入一个名为 elasticsearch 的集群。

#### 索引（名词）

> 一个索引类似于传统关系数据库中的一个数据库 ，是一个存储关系型文档的地方，是ES对逻辑数据的逻辑存储，索引的结构是为快速有效的全文检索做准备。


#### 索引（动词）

> 索引一个文档就是存储一个文档到一个索引（名词）中以便它可以被检索和查询到。这非常类似于 SQL 语句中的 INSERT 关键词，除了文档已存在时新文档会替换旧文档情况之外。

#### 倒排索引

> 倒排索引源于实际应用中需要根据属性的值来查找记录。这种索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引(inverted index)。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件(inverted file)。

#### 文档

> 存储在ES上的主要实体叫文档

#### 文档类型（在 6.0.0 及以上废弃）

> 在ES中，一个索引对象可以存储很多不同用途的对象。

#### 映射

> 存储有关字段的信息，每一个文档类型都有自己的映射。

#### 面向文档

> 在应用程序中对象很少只是一个简单的键和值的列表。通常，它们拥有更复杂的数据结构，可能包括日期、地理信息、其他对象或者数组等。
> 
> 也许有一天你想把这些对象存储在数据库中。使用关系型数据库的行和列存储，这相当于是把一个表现力丰富的对象挤压到一个非常大的电子表格中：你必须将这个对象扁平化来适应表结构--通常一个字段>对应一列--而且又不得不在每次查询时重新构造对象。

>Elasticsearch 是 面向文档 的，意味着它存储整个对象或 文档_。Elasticsearch 不仅存储文档，而且 _索引 每个文档的内容使之可以被检索。在 Elasticsearch 中，你 对文档进行索引、检索、排序和过滤--而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。

#### 分片(Shards)

> 索引可能存储大量可能超过单个节点的硬件限制的数据。例如，占用1TB磁盘空间的十亿个文档的单个索引可能不适合单个节点的磁盘，或者可能太慢而无法单独从单个节点提供搜索请求。
> 
> 为了解决这个问题，Elasticsearch 提供了将索引细分为多个称为分片的功能。创建索引时，只需定义所需的分片数即可。每个分片本身都是一个功能齐全且独立的“索引”，可以托管在集群中的任何节点上。

> 设置分片的目的及原因主要是：
> 
> 它允许您水平拆分/缩放内容量
它允许您跨分片（可能在多个节点上）分布和并行化操作，从而提高性能/吞吐量
分片的分布方式以及如何将其文档聚合回搜索请求的机制完全由 Elasticsearch 管理，对用户而言是透明的。
> 
> 在可能随时发生故障的网络/云环境中，分片非常有用，建议使用故障转移机制，以防分片/节点以某种方式脱机或因任何原因消失。为此，Elasticsearch 允许您将索引的分片的一个或多个副本制作成所谓的副本分片或简称副本。

#### 副本(Replicasedit)

> 副本，是对分片的复制。目的是为了当分片/节点发生故障时提供高可用性，它允许您扩展搜索量/吞吐量，因为可以在所有副本上并行执行搜索。
> 
> 总而言之，每个索引可以拆分为多个分片。索引也可以复制为零次（表示没有副本）或更多次。复制之后，每个索引将具有主分片(从原始分片复制而来的)和复制分片(主分片的副本)。

**副本是乘法，越多越浪费，但也越保险。分片是除法，分片越多，单分片数据就越少也越分散。**

一个对比图来类比传统关系型数据库：
```s
关系型数据库   -> Databases(库) -> Tables(表)  -> Rows(行)         -> Columns(列)。
Elasticsearch -> Indeces(索引) -> Types(类型) -> Documents(文档) -> Fields(属性)。
```

## 与Elasticsearch交互

目前与 elasticsearch 交互主要有两种方式：Client API 和 RESTful API。

Client API方式：

Elasticsearch 为以下语言提供了官方客户端 --Groovy、JavaScript、.NET、 PHP、 Perl、 Python 和 Ruby--还有很多社区提供的客户端和插件，所有这些都可以在 [Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html) 中找到。

RESTful API with JSON over HTTP：

所有其他语言可以使用 RESTful API 通过端口 9200 和 Elasticsearch 进行通信，你可以用你最喜爱的 web 客户端访问 Elasticsearch 。事实上，正如你所看到的，你甚至可以使用 curl 命令来和 Elasticsearch 交互。

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：
```bash
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

参数 | 描述
---- | ---
VERB | 适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE
PROTOCOL | http 或者 https（如果你在 Elasticsearch 前面有一个https 代理）
HOST | Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点
PORT | 运行 Elasticsearch HTTP 服务的端口号，默认是 9200
PATH | API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm
QUERY_STRING | 任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)
BODY | 一个 JSON 格式的请求体 (如果请求需要的话)

**常用命令**

```bash
curl 'localhost:9200/brand/_search?pretty=true'         # 查询索引数据
curl -XPUT 'localhost:9200/customer?pretty'             # 创建索引
curl -XDELETE 'localhost:9200/customer'                 # 删除索引
curl 'localhost:9200/_cat/health?v'                     # 检测集群是否健康
curl 'localhost:9200/_cat/nodes?v'                      # 获取集群节点
curl 'localhost:9200/_cat/indices?v'                    # 列出所有索引
curl -x GET 'localhost:9200/index/_mapping'             # 查询指定索引的映射
curl -X GET 'localhost:9200/_cluster/health?pretty'     # 查看分片状态
```

## Example

1. 创建第一个简单索引

创建一个 NBA 球队的索引

```bash
PUT nba
{
  "settings":{
    "number_of_shards": 3,   
    "number_of_replicas": 1 
  },
  "mappings":{
    "nba":{
      "properties":{
        "name_cn":{ 
          "type":"text"
        },
        "name_en":{
          "type":"text"
        },
        "gymnasium":{
          "type":"text"
        },
        "topStar":{
          "type":"text"
        },
        "championship":{
          "type":"integer"
        },
        "date":{
          "type":"date",
          "format":"yyyy-MM-dd HH:mm:ss|| yyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}
```

字段说明：

字段名称 | 字段说明
--- | ---
nba | 索引
number_of_shards | 分片数
number_of_replicas | 副本数
name_cn | 球队中文名
name_en | 球队英文名
gymnasium | 球馆名称
championship | 总冠军次数
topStar | 当家球星
date | 加入NBA年份

创建成功则返回信息:
```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "nba"
}
```

2. 新增索引数据

```json
PUT /nba/nba/1
{
  "name_en":"San Antonio Spurs SAS",
  "name_cn":"圣安东尼安马刺",
  "gymnasium":"AT&T中心球馆",
  "championship": 5,
  "topStar":"蒂姆·邓肯",
  "date":"1995-04-12"
}

PUT /nba/nba/2
{
  "name_en":"Los Angeles Lakers",
  "name_cn":"洛杉矶湖人",
  "gymnasium":"斯台普斯中心球馆",
  "championship": 16,
  "topStar":"科比·布莱恩特",
  "date":"1947-05-12"
}

PUT /nba/nba/3
{
  "name_en":"Golden State Warriors",
  "name_cn":"金州勇士队",
  "gymnasium":"甲骨文球馆",
  "championship": 6,
  "topStar":"斯蒂芬·库里",
  "date":"1949-06-13"
}

PUT /nba/nba/4
{
  "name_en":"Miami Heat",
  "name_cn":"迈阿密热火队",
  "gymnasium":"美国航空球场",
  "championship": 3,
  "topStar":"勒布朗·詹姆斯",
  "date":"1988-06-13"
}

PUT /nba/nba/5
{
  "name_en":"Cleveland Cavaliers",
  "name_cn":"克利夫兰骑士队",
  "gymnasium":"速贷球馆",
  "championship": 1,
  "topStar":"勒布朗·詹姆斯",
  "date":"1970-06-13"
}
```

3. 查询全部球队的信息

```json
POST /nba/nba/_search
{
    "query": {
        "match_all": {}
    }
}
```

响应结果:

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "nba",
        "_type": "nba",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name_en": "Los Angeles Lakers",
          "name_cn": "洛杉矶湖人",
          "gymnasium": "斯台普斯中心球馆",
          "championship": 16,
          "topStar": "科比·布莱恩特",
          "date": "1947-05-12"
        }
      },
      {
        "_index": "nba",
        "_type": "nba",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name_en": "San Antonio Spurs SAS",
          "name_cn": "圣安东尼安马刺",
          "gymnasium": "AT&T中心球馆",
          "championship": 5,
          "topStar": "蒂姆·邓肯",
          "date": "1995-04-12"
        }
      },
      {
        "_index": "nba",
        "_type": "nba",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name_en": "Golden State Warriors",
          "name_cn": "金州勇士队",
          "gymnasium": "甲骨文球馆",
          "championship": 6,
          "topStar": "斯蒂芬·库里",
          "date": "1949-06-13"
        }
        ···
      }
    ]
  }
}
```

响应的数据结果分为两部分

```json
{
----------------first part--------------------
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
---------------second part---------------------
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```
第一部分为：分片副本信息，第二部分 hits 包装的为查询的数据集。

4. 查询英文名称为："Golden State Warriors" 的球队信息

```json
POST /nba/nba/_search
{
   "query": {
        "match": {
            "name_en": "Golden State Warriors"
        }
    }
}
```

可得到的查询结果为：

```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.9646256,
    "hits": [
      {
        "_index": "nba",
        "_type": "nba",
        "_id": "3",
        "_score": 1.9646256,
        "_source": {
          "name_en": "Golden State Warriors",
          "name_cn": "金州勇士队",
          "gymnasium": "甲骨文球馆",
          "championship": 6,
          "topStar": "斯蒂芬·库里",
          "date": "1949-06-13"
        }
      }
    ]
  }
}
```

5. 过滤查询 Filter

我们让搜索变的复杂一些。我们想要找到当家球星是勒布朗·詹姆斯，但是我们只想得到总冠军多于1次的球队。我们的语句将做一些改变用来添加过滤器(filter),它允许我们有效的执行一个结构化搜索

```json
POST /nba/nba/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "championship": {
            "gt": 1
          }
        }
      },
      "must": {
        "match": {
          "topStar": "勒布朗·詹姆斯"
        }
      }
    }
  }
}
```

我们发现每次查询，查询结果里面都有一个 _score字段，一般Elasticsearch根据相关评分排序，相关评分是根据文档与语句的匹配度来得出， _score值越高说明匹配度越高。


**查询命令**

1. query_string语法

```bash
curl -XGET 'localhost:9200/product/spu/_search?pretty=true' -d '{
    "query" : {
        "query_string" : {"query" : "brandName:本田"}
    }
}'
```

2. 分页查询

```bash
curl -XGET 'localhost:9200/product/spu/_search?pretty=true' -d '{
    "from" : 1,
    "size" : 1,
    "query" : {
        "query_string" : {"query" : "brandName:本田"}
    }
}'
```

3. 增加version值

```bash
curl -XGET 'localhost:9200/product/spu/_search?pretty=true' -d '{
    "version" : true,
    "from" : 1,
    "size" : 1,
    "query" : {
        "query_string" : {"query" : "brandName:本田"}
    }
}'
```

4. 限制得分

```bash
curl -XGET 'localhost:9200/product/spu/_search?pretty=true' -d '{
    "version" : true,
    "min_score" : 2.4,
    "query" : {
        "query_string" : {"query" : "brandName:本田"}
    }
}'
```

5. 选择要返回的字段

```bash
curl -XGET 'localhost:9200/product/spu/_search?pretty=true' -d '{
    "fields" : ["brandName","spuName"],
    "version" : true,
    "min_score" : 2.4,
    "query" : {
        "query_string" : {"query" : "brandName:本田"}
    }
}'
```

6. 选择要返回的字段

```bash
curl -XGET 'localhost:9200/product/spu/_search?pretty=true' -d '{
    "fields" : ["brandName","spuName"],
    "version" : true,
    "min_score" : 2.4,
    "query" : {
        "query_string" : {"query" : "brandName:本田"}
    }
}'
```

## SpringBoot与Elasticsearch集成

1. pom文件中添加添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>

<!-- 解决es no jna warning-->
<dependency>
    <groupId>com.sun.jna</groupId>
    <artifactId>jna</artifactId>
    <version>3.0.9</version>
</dependency>
```

2. application.yml添加配置

```yml
spring:
    data:
        elasticsearch:
            cluster-nodes: 127.0.0.1:9300
```

**注意**：elasticsearch jar版本要与安装的服务版本相容