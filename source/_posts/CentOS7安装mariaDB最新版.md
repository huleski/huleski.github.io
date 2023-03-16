---
title: CentOS7安装mariaDB最新版
categories: mariaDB
tags: 
    - mariaDB
    - CentOS7
date: 2019-02-24 16:26:41
---

安装Maria DB
-------------

来自[官网的包源](https://downloads.mariadb.org/mariadb/repositories/)

编辑新增文件:  `vim /etc/yum.repos.d/MariaDB.repo` 保存以下内容

```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64/
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

加入上述内容，然后:wq!保存退出，如果在后面安装的时候很慢可以换[镜像地址](http://mirrors.neusoft.edu.cn/mariadb//mariadb-10.3.15/yum/centos6-amd64/), 接着要注意重新生成yum源缓存包：
```
yum clean all
yum makecache
```


移除已安装的mariaDB/MySQL:
```
yum remove $(rpm -qa | grep -i mysql)
yum remove $(rpm -qa | grep -i mari)
```

安装
```
yum install -y MariaDB-server MariaDB-client
```

配置数据库
--------

运行以下命令：
```
systemctl start mariadb         # 启动mariaDB
systemctl enable mariadb        # 设置开机自启动
mysql_secure_installation       # 开始初始化数据库
```
首先是设置密码，会提示先输入密码，后面是一些其他配置
```
Enter current password for root (enter for none): <–初次运行直接回车

Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车

New password: <– 设置root用户的密码

Re-enter new password: <– 再输入一次你设置的密码

Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车

Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车,

Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车

Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车
```

初始化MariaDB完成，接下来测试登录
```
mysql> mysql -uroot -ppassword
```
登录成功，查看数据库版本：
```
mysql> select version();
```

配置MariaDB的字符集

编辑文件：`vi /etc/my.cnf` ，在[mysqld]标签下添加
```
init_connect='SET collation_connection = utf8mb4_unicode_ci' 
init_connect='SET NAMES utf8mb4' 
character-set-server=utf8mb4 
collation-server=utf8mb4_unicode_ci 
skip-character-set-client-handshake
```

编辑文件：vi /etc/my.cnf.d/client.cnf ，在[client]下添加 ( 如果没有可以不加 )
```
default-character-set=utf8mb4
```

编辑文件： vi /etc/my.cnf.d/mysql-clients.cnf ，在[mysql]下添加
```
default-character-set=utf8mb4
```

 全部配置完成，重启mariadb
```
systemctl restart mariadb
```

之后进入MariaDB查看字符集
```
mysql> show variables like "%character%";show variables like "%collation%";
```

显示为：
```
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_unicode_ci |
| collation_database   | utf8mb4_unicode_ci |
| collation_server     | utf8mb4_unicode_ci |
+----------------------+--------------------+
3 rows in set (0.00 sec)
```
字符集配置完成。

添加用户，设置权限

1. 创建用户命令

```
create user username@'%' identified by 'password';
```

2. 授予当前登录用户所有权限给该用户 

```
grant all privileges on *.* to username@'%';
```

3. 授予权限并且可以授权

```
grant all privileges on *.* to username@'%' with grant option;
```

4. 授予dbname只读权限

```bash
GRANT SELECT ON dbname.* TO 'username'@'%';
```

5. 授予table_name表的修改权限

```bash
GRANT UPDATE ON dbname.table_name TO 'username'@'%';
```

6. 刷新权限

```
flush privileges;
```

安装成功。

## 可选配置(修改完需要重启)

### 修改端口, 编辑 `vim /etc/my.cnf`,在[mysqld]下添加:

```text
[mysqld]
port=3506
```

### 开启SSL加固

在mysql中查询是否已开启ssl: `show variables like 'have%ssl%';`

以下操作都在`/home`目录下(可自定义)

1. 生成证书, 为后面生成服务器和客户端证书做准备

```bash
# 生成 CA 私钥
openssl genrsa 2048 > ca-key.pem
# 通过 CA 私钥生成数字证书
openssl req -new -x509 -nodes -days 3600 -key ca-key.pem -out ca.pem
```

2. 生成服务器证书

```bash
# 创建 MySQL 服务器 私钥和请求证书
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout server-key.pem -out server-req.pem
# 将生成的私钥转换为 RSA 私钥文件格式
openssl rsa -in server-key.pem -out server-key.pem
# 用CA 证书来生成一个服务器端的数字证书
openssl x509 -req -in server-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
```

3. 生成客户端证书

```bash
# 创建客户端的 RSA 私钥和数字证书
openssl req -newkey rsa:2048 -days 3600  -nodes -keyout client-key.pem -out client-req.pem
# 将生成的私钥转换为 RSA 私钥文件格式
openssl rsa -in client-key.pem -out client-key.pem
# 用CA 证书来生成一个客户端的数字证书
openssl x509 -req -in client-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
```

4. 修改mysql配置 `vim /etc/my.cnf`, 在`[mysqld]`中加入:

```bash
# 开启ssl
ssl-ca=/home/ca.pem
ssl-cert=/home/server-cert.pem
ssl-key=/home/server-key.pem
```

重启mysql, `systemctl restart mariadb`

5. 设置ssl访问

```bash
grant all privileges on *.* to '用户名'@'%' identified by '密码' require ssl;
```
此时navicat等客户端工具都连不上mysql了, 需要在连接时配置SSL, 把客户端证书和私钥都配置好就可以连上了

### 每日自动备份数据库

编写shell脚本保存 `vim /home/back.sh`

```bash
#!/bin/bash
#当前时间格式化输出
ctime=$(date '+%Y%m%d%H%M')
#使用当前时间的方式命名备份的sql文件
name=$ctime".sql"
#备份前先检查存放sql备份的目录里面的文件是否过期（3天），过期则删除掉
find /home/sql-back/ -type f -mtime +3 | xargs rm -rf
#开始将数据库转存为sql文件
mysqldump -uroot -pxxxxx mysql > $name
#将转存的sql文件移动到专门用于存放备份的目录中
mv ./$name /home/sql-back/
```

find和mv后面的路径请确保真实存在，或者是更换为实际应用场景需要的目录。添加执行权限 `chmod +x /home/back.sh`

配置定时任务 `vim /etc/crontab`, 在底部插入内容：

```bash
#我这里设置的是每天2点52分，以root用户执行备份脚本
#分  时  天  月   周   执行用户    执行的动作
52 2 * * * root cd /home/ && ./back.sh
```

插入内容后重启定时任务的服务: `systemctl restart crond`. 完成

### 开启Binlog, 在[mysqld]下添加:

```conf
# 这一行开启Binlog
log-bin=mysql-bin
# 设置格式
binlog-format=ROW 
# 配置serverID
server_id=1 

# 日志30天自动过期清除(最大只能设置99),触发条件是：binlog大小超过500M
expire_logs_days=30
max_binlog_size=500M
```

查看是否开启成功: `show variables like '%log_bin%';`

