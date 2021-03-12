---
title: docker-compose安装MariaDB
categories: docker-compose
tags: docker-compose
date: 2021-03-12 16:58:20
---

创建`docker-compose.yml`文件

```yml
version: '3'
services:
  mariadb:
    container_name: mariadb
    image: mariadb
    environment:
      # root 密码
      - MYSQL_ROOT_PASSWORD=root123
      # root 允许登录的host
      - MYSQL_ROOT_HOST=%
      # 时区
      - TIME_ZONE=Asia/Shanghai
    ports:
      - 3306:3306
    volumes:
      # 容器与宿主机时间同步
      - /etc/localtime:/etc/localtime
      # 数据库目录映射
      - /data/mariadb/data/:/var/lib/mysql
      # 数据库配置文件
      - /data/mariadb/config:/etc/mysql/conf.d
    privileged: true
    restart: always

```

在宿主机的配置目录`/data/mariadb/config`中创建自定义配置文件:

```bash
vim my.cnf
```

配置文件内容:

```cnf
[client]
port            = 3306
socket          = /var/run/mysqld/mysqld.sock
default-character-set = utf8mb4

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

# This was formally known as [safe_mysqld]. Both versions are currently parsed.
[mysqld_safe]
socket          = /var/run/mysqld/mysqld.sock
nice            = 0

[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

lower_case_table_names = 1
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc_messages_dir = /usr/share/mysql
lc_messages     = en_US
skip-external-locking

max_connections         = 1000
connect_timeout         = 5
wait_timeout            = 600
max_allowed_packet      = 16M
thread_cache_size       = 128
sort_buffer_size        = 4M
bulk_insert_buffer_size = 16M
tmp_table_size          = 32M
max_heap_table_size     = 32M

# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched. On error, make copy and try a repair.
myisam_recover_options = BACKUP
key_buffer_size         = 128M
#open-files-limit       = 2000
table_open_cache        = 400
myisam_sort_buffer_size = 512M
concurrent_insert       = 2
read_buffer_size        = 2M
read_rnd_buffer_size    = 1M

query_cache_limit               = 128K
query_cache_size                = 64M

slow_query_log_file     = /var/log/mysql/mariadb-slow.log
long_query_time = 10
#log_slow_rate_limit    = 1000
#log_slow_verbosity     = query_plan

#sync_binlog            = 1
expire_logs_days        = 10
max_binlog_size         = 100M

# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
default_storage_engine  = InnoDB
# you can't just change log file size, requires special procedure
#innodb_log_file_size   = 50M
innodb_buffer_pool_size = 256M
innodb_log_buffer_size  = 8M
innodb_file_per_table   = 1
innodb_open_files       = 400
innodb_io_capacity      = 400
innodb_flush_method     = O_DIRECT

[galera]

[mysqldump]
quick
quote-names
max_allowed_packet      = 16M

[mysql]
default-character-set = utf8mb4

[isamchk]
key_buffer              = 16M

!include /etc/mysql/mariadb.cnf
!includedir /etc/mysql/conf.d/
```

保存后启动镜像: `docker-compose up -d`

启动成功后可进入镜像查看myql运行情况
  ```bash
  # 进入镜像
  $ docker exec -it mariadb /bin/bash
  # 登录mariadb
  > mysql -uroot -pWTAherui@20#
  # 查看编码
  > show variables like '%character%'
  ```

创建数据库时指定编码

```bash
CREATE DATABASE  `user` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
