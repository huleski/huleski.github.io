---
title: docker安装rabbitmq
categories: docker
tags: docker
date: 2020-10-12 16:00:07
---

下载镜像
-------

进入[docker hub官方镜像仓库](https://hub.docker.com/)

搜索rabbitMq，进入官方的镜像，可以看到以下几种类型的镜像；我们选择带有“mangement”的版本（包含web管理页面）

编写docker-compose文件 docker-compose.yml

```yml
version: '3'
services:
  rabbitmq:
    container_name: rabbitmq
    image: rabbitmq:management-alpine:3.8.9
    environment:
      - RABBITMQ_DEFAULT_USER=root 
      - RABBITMQ_DEFAULT_PASS=123456 
    ports:
      - 5672:5672
      - 15672:15672
    volumes:
      - /root/middleware/rabbitmq/data:/var/lib/rabbitmq
    privileged: true
    restart: always
```

启动容器(在`docker-compose.yml`文件同级目录执行)

```bash
docker-compose up -d
```

启动rabbitmq_management

```bash
docker exec -it rabbit rabbitmq-plugins enable rabbitmq_management
```

现在已经启动完成了, 浏览器打开web管理端：http://ip:15672

输入前面docker-compose里面设置的用户名密码登录即可

官方安装的默认是不带延迟队列插件的, 需要自己安装

docker构建延迟队列插件rabbitmq
------

前面已经下载好镜像了, 另外需要下载插件

官方插件下载地址：[github下载](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/), 选择对应版本, 这里我们同样选3.8.9版本下载, 放到服务器上

创建DockerFile文件(在插件同级目录, 插件名为`rabbitmq_delayed_message_exchange-3.8.9.ez`), 内容如下

```bash
FROM rabbitmq:3-management
COPY ["rabbitmq_delayed_message_exchange-3.8.9.ez" , "/plugins/"]
RUN rabbitmq-plugins enable rabbitmq_delayed_message_exchange rabbitmq_management
```

开始打包构建

```bash
docker build -t holeski/rabbitmq-delay:3.8.9 .
```

修改`docker-compose.yml`文件:

```yml
version: '3'
services:
  rabbitmq:
    container_name: rabbitmq
    image: holeski/rabbitmq-delay:3.8.9
    environment:
      - RABBITMQ_DEFAULT_USER=root
      - RABBITMQ_DEFAULT_PASS=123456
    ports:
      - 5672:5672
      - 15672:15672
    volumes:
      - /root/middleware/rabbitmq/data:/var/lib/rabbitmq
    privileged: true
    restart: always
```

现在可以一个命令启动就行了

```bash
docker-compose up -d
```

镜像已上传到dockerhub, 可以直接拉取使用

```bash
docker pull holeski/rabbitmq-delay:3.8.9
```