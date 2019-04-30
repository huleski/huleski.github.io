---
title: Elasticsearch入门教程
categories: Elasticsearch
tags: Elasticsearch
date: 2019-04-30 17:49:33
---

## Elasticsearch简介

Elasticsearch是一个高度可扩展的、开源的、基于 Lucene 的全文搜索和分析引擎。它允许您快速，近实时地存储，搜索和分析大量数据，并支持多租户。

它使用Java开发并使用 Lucene 作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的 RESTful API 来隐藏 Lucene 的复杂性，从而让全文搜索变得简单。

### 版本选择

Elasticsearch 目前有三个常用的稳定的主版本：2.x，5.x，6.x, 目前最新是7.0.0

> 中间没有3.x和4.x是为了ELK（ElasticSearch, logstash, kibana）技术栈的版本统一，免的给用
户带来混乱。

版本选择可以从以下几个方面考虑：

```
版本问题
2.x 版本较老，无法体验新功能，且性能不如 5.x。
6.x 版本有点新，网上资料相对比较少（开发时间充足的可以研究）。

数据迁移
2.x 版本数据可以直接迁移到 5.x；
5.X 版本的数据可以直接迁移到 6.x； 但是 2.x 版本数据无法直接迁移到 6.x。

周边工具
2.x 版本周边工具版本比较混乱；Kibana 等工具的对应版本需要自己查，不好匹配。
5.x 之后 Kibana 等工具的主版本号进行了统一。

Sql 语法支持
2.x，5.x，6.x 都可以安装 Elasticsearch-sql 插件，使用熟悉的SQL语法查询 Elasticsearch。
6.3.0 以后内置支持 SQL 模块，这个 SQL 模块是属于 X-Pack 的一部分。
```

我选择目前最新的5.x版: `elasticsearch-5.6.16`

## 安装Elasticsearch

### 安装包安装

**注意: Elasticsearch5.0之后的版本至少需要Java 8**

下载对应版本安装包: [官方下载地址](https://www.elastic.co/cn/downloads/past-releases)

下载后解压即可使用, 执行以下命令启动:

**Linux**

```bash
./bin/elasticsearch
```

**Windows** 进入elasticsearch安装目录/bin

```s
双击 elasticsearch.bat
```

### Docker安装

CentOS版太大了, 我选择alpine版的

```bash
docker pull elasticsearch:5.6-alpine
```

启动容器

```bash
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" \
-v /data/software/elasticsearch/config:/usr/share/elasticsearch/config \
-v /data/software/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/software/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
--name=elasticsearch docker.io/elasticsearch:5.6-alpine
```
