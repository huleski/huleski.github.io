---
title: Springboot集成ELK
categories: Springboot
tags:
    - Springboot
    - ELK
date: 2021-12-16 15:10:40
---

## ELK简介

ELK是由 Elasticsearch、Logstash和Kibana 三部分组件组成。

Elasticsearch 是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

Logstash 是一个完全开源的工具，它可以对你的日志进行收集、分析，并将其存储供以后使用。
一般工作方式为c/s架构，client端安装在需要收集日志的主机上（本例中为集成在Springboot项目中），server端负责将收到的各节点日志进行过滤、修改等操作在一并发往elasticsearch上去。

kibana 是一个开源和免费的工具，它可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

在本示例中, 工作流畅是: Springboot集成Logstash收集日志, 再把日志发送到ElasticSearch中, 最后用户通过kibana连接ElasticSearch查看日志数据

## ELK安装

直接使用docker-compose安装简单粗暴, 不整那些花里胡哨的操作,

先新建目录: `mkdir -p /data/elk/{elasticsearch/data,kibana,logstash,}`

保存配置文件: docker-compose.yml (软件版本要统一, 当前最新版为 7.16.1) :

```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.16.1  #镜像
    container_name: elk_elasticsearch  #定义容器名称
    restart: always  #开机启动，失败也会一直重启
    environment:
      - "cluster.name=elasticsearch" #设置集群名称为elasticsearch
      - "discovery.type=single-node" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx1024m" #设置使用jvm内存大小
    volumes:
      - /data/elk/elasticsearch/plugins:/usr/share/elasticsearch/plugins #插件文件挂载
      - /data/elk/elasticsearch/data:/usr/share/elasticsearch/data #数据文件挂载
    ports:
      - 9200:9200
  kibana:
    image: kibana:7.16.1
    container_name: elk_kibana
    restart: always
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200 #设置访问elasticsearch的地址
    volumes:
      - /data/elk/kibana/config:/opt/kibana/config #配置文件挂载
    ports:
      - 5601:5601
  logstash:
    image: logstash:7.16.1
    container_name: elk_logstash
    restart: always
    volumes:
      - /data/elk/logstash/logstash-springboot.conf:/usr/share/logstash/pipeline/logstash.conf #挂载logstash的配置文件
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    ports:
      - 4560:4560
```

```shell
# 授权目录
cd /data/elk
chmod 777 elasticsearch/data

# 新建logstash/logstash-springboot.conf文件，新增以下内容
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => "es:9200"
    index => "springboot-logstash-%{+YYYY.MM.dd}"
  }
}

# 安装，运行ELK
docker-compose up -d
# 查看容器运行状态
docker ps
```

等一会容器都启动成功了就打开浏览器访问Kibana: `http://localhost:5601`, 正常情况就会出现欢迎界面

### 汉化kibana
编辑配置文件 `/data/elk/kibana/config/kibana.yml`, 新增: `i18n.locale: zh-CN`

改完配置后重启kibana: `docker restart elk_kibana `

## Springboot整合Logstash

在`pom.xml`新增依赖

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.0.1</version>
</dependency>
```

修改Springboot项目中日志配置文件`logback.xml`, 在相应位置增加配置:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false" scan="true" scanPeriod="1 seconds">
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:4560</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="LOGSTASH" />
    </root>
</configuration>
```

启动Springboot项目后再查看kibana就可以看到日志了