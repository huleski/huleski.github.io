---
title: centos7安装redis及开机自启动
categories: redis
tags: 
    - centos7
    - redis
date: 2019-02-25 10:29:06
---

安装
----

首先确认是否具有root权限，因为vim、设定redis开机启动需要root权限
```
su
```

新建软件安装目录和配置文件存放目录(已有可以不用新建)
```
mkdir -p /home/software         # 存放redis
mkdir -p /usr/local/redis       # 存放配置文件
```

下载redis, 可以在[官网](http://download.redis.io/releases)获取指定版本
```
cd /home/software   # 进入安装目录
wget http://download.redis.io/releases/redis-5.0.3.tar.gz   #下载
```

依次执行以下命令
```
tar xzf redis-5.0.3.tar.gz      # 解压缩
cd redis-5.0.3                  # 进入解压后的文件目录
yum -y install gcc-c++          # 新安装的centos系统没有C++编译器, 需要安装, 有的就跳过
make                            # 编译安装
```

目前已经安装完毕

配置
----

复制配置文件( 相当于备份 )
```
cp /home/software/redis-5.0.3/src/redis-server /usr/local/redis/
cp /home/software/redis-5.0.3/src/redis-cli /usr/local/redis/
cp /home/software/redis-5.0.3/redis.conf /usr/local/redis/
```

编辑配置文件 redis.conf
```
vim /usr/local/redis/redis.conf
```

daemonize 改为yes 后台运行
```
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes
```

把 `bind 127.0.0.1`注释掉, 放开ip限制 ( 可选 )
```
# bind 127.0.0.1
```

把`# requirepass foobared`注释放开并修改密码为123456( 可选 )
```
requirepass 123456
```

添加开机自启动服务文件
```
vim /etc/systemd/system/redis.service
```

加入以下内容, 在vim中一定要检查是否一致
```
[Unit]
Description=The redis-server Process Manager
After=syslog.target network.target

[Service]
Type=simple
PIDFile=/var/run/redis_6379.pid
ExecStart=/usr/local/redis/redis-server /usr/local/redis/redis.conf         
ExecReload=/bin/kill -USR2 $MAINPID
ExecStop=/bin/kill -SIGINT $MAINPID

[Install]
WantedBy=multi-user.target
```

设置开机自启动
```
systemctl daemon-reload 
systemctl start redis.service 
systemctl enable redis.service
```

测试redis, 启动redis客户端, 依次执行以下命令
```
/usr/local/redis/redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> set name Holeski
OK
127.0.0.1:6379> get name
"holeski"
```

redis安装配置成功