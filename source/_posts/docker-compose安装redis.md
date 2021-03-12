---
title: docker-compose安装redis
categories: docker-compose
tags: docker-compose
date: 2021-03-12 18:55:21
---

编写yml文件

```yml
version: '3'
services:
  redis:
    image: redis:latest
    restart: always
    container_name: redis-server
    command: redis-server /etc/redis/redis.conf
    volumes:
      - /root/middleware/redis/data:/data
      - /root/middleware/redis/redis.conf:/etc/redis/redis.conf
    ports:
      - "6379:6379"
```

同级目录中创建配置文件 `reids.conf`

```
requirepass 123456
appendonly yes
```

启动镜像

```bash
docker-compose up -d
```