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
    image: rabbitmq:management-alpine
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