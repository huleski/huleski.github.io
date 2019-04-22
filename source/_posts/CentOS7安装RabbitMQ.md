---
title: CentOS7安装RabbitMQ
categories: CentOS7
tags: 
    - CentOS7
    - RabbitMQ
date: 2019-04-22 09:26:36
---

安装ErLang
---------

rabbitmq依赖于erlang, 因此要**先安装erlang**

```bash
wget  http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
yum install erlang
```

安装RabbitMQ
------------

```bash
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.14/rabbitmq-server-3.7.14-1.el7.noarch.rpm
yum -y install rabbitmq-server-3.7.14-1.el7.noarch.rpm
```

启动RabbitMQ
-----------

```bash
systemctl start rabbitmq-server             # 启动RabbitMQ
systemctl enable rabbitmq-server            # 开机自启动RabbitMQ
systemctl status rabbitmq-server            # 查看状态
```

访问web控制台
------------

初始帐号和密码都为：guest, 但是只能本地登录

```bash
rabbitmq-plugins enable rabbitmq-management                 # 启动RabbitMQ Web管理控制台
chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/               # 将RabbitMQ文件的所有权提供给RabbitMQ用户

rabbitmqctl add_user admin 123456                           # 为RabbitMQ Web管理控制台创建管理用户
rabbitmqctl set_user_tags admin administrator               # 为admin设置管理员角色
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"       # 为admin设置默认vhost（“/”）配置、写、读全部权限
```

访问: `http://RabbitMQ_IP:15672/` 就能看到RabbitMQ管理页面

角色说明
----------

- 超级管理员(administrator)
  
    可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。
- 监控者(monitoring)
  
    可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)
- 策略制定者(policymaker)

    可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。
- 普通管理者(management)

    仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。
- 其他

    无法登陆管理控制台，通常就是普通的生产者和消费者。