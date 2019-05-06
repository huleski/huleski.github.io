---
title: CentOS7安装Elasticsearch
categories: Elasticsearch
tags: Elasticsearch
date: 2019-05-06 11:37:41
---

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

**启动容器**

在启动容器之前, 容器挂载的配置文件目录下面得要有配置文件，不然es是起不来的，比较方便的办法是，先不挂载启动es，然后用docker cp命令，把配置文件复制到宿主机挂载目录，然后再进行修改：

```bash
docker run -d -p 9200:9200 -p 9300:9300 --name=elasticsearch elasticsearch:5.6-alpine

docker cp elasticsearch:/usr/share/elasticsearch/config /data/software/elasticsearch/config 

docker cp elasticsearch:/usr/share/elasticsearch/data /data/software/elasticsearch/data

docker stop elasticsearch

docker rm elasticsearch

如果需要更改配置，可以直接修改config目录下的  elasticsearch.yml 文件，然后启动es

docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-v /data/software/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /data/software/elasticsearch/config:/usr/share/elasticsearch/config \
-v /data/software/elasticsearch/data:/usr/share/elasticsearch/data \
--restart unless-stopped --name=elasticsearch elasticsearch:5.6-alpine
```

检查es是否安装成功: 访问 `http://es_serverIp:9200`

## 安装ik分词器

[下载地址](https://github.com/medcl/elasticsearch-analysis-ik/releases)

下载对应版本, 解压放入es安装目录中的plugin文件夹中, 重启es即可

## 安装Kibana

Kibana 和 elasticsearch 同属于 elastic 公司。 Kibana是一个开源分析和可视化平台，旨在与Elasticsearch协同工作。使用Kibana搜索，可以查看和与存储在 Elasticsearch 索引中的数据进行交互。您可以轻松地执行高级数据分析，并在各种图表，表格和地图中可视化您的数据。

**Windows**

从[官方下载地址](https://www.elastic.co/cn/downloads/past-releases)中下载与elasticsearch 版本对应 的Kibana, 解压即可使用:

- 下载并解压缩 Kibana。
- 在编辑器中打开 `config/kibana.yml`。
- 设置 `elasticsearch.url` 为您的Elasticsearch实例，如本地：elasticsearch.url: "http://localhost:9200"(与es装在同一机器可以不用设置)。
- 运行bin/kibana.bat。
- 浏览器输入 http：// localhost：5601。

**CentOS**

```bash
docker pull kibana:5.6.16
docker run --init -d --name kibana --restart unless-stopped --link elasticsearch -p 5601:5601 kibana:5.6.16
```

