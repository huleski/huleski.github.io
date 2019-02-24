---
title: CentOS7安装mariaDB最新版
categories: mariaDB
tags: 
    - mariaDB
    - CentOS7
date: 2019-02-24 16:26:41
---
安装Maria DB
===========

获取MariaDB包源
-------------

来自[官网的包源](https://downloads.mariadb.org/mariadb/repositories/)

编辑新增文件:  vim /etc/yum.repos.d/MariaDB.repo 保存以下内容

```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64/
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

移除已安装的mariaDB/MySQL:
----------------
```
yum remove $(rpm -qa | grep -i mysql)
yum remove $(rpm -qa | grep -i mari)
```

安装
---
```
yum install -y MariaDB-server MariaDB-client
```

配置数据库
=======

运行以下命令：
```
systemctl start mariadb         #启动mariaDB
systemctl enable mariadb        #设置开机自启动
mysql_secure_installation       #开始初始化数据库
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
----------------

编辑文件：vi /etc/my.cnf ，在[mysqld]标签下添加
```
init_connect='SET collation_connection = utf8_unicode_ci' 
init_connect='SET NAMES utf8' 
character-set-server=utf8 
collation-server=utf8_unicode_ci 
skip-character-set-client-handshake
```

编辑文件：vi /etc/my.cnf.d/client.cnf ，在[client]中添加
```
default-character-set=utf8
```

编辑文件： vi /etc/my.cnf.d/mysql-clients.cnf ，在[mysql]中添加
```
default-character-set=utf8
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
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_unicode_ci |
| collation_database   | utf8_unicode_ci |
| collation_server     | utf8_unicode_ci |
+----------------------+-----------------+
3 rows in set (0.00 sec)
```
字符集配置完成。

添加用户，设置权限
----------------

1. 创建用户命令

```
mysql> create user username@localhost identified by 'password';
```

2. 直接创建用户并授权的命令

```
mysql> grant all on *.* to username@localhost indentified by 'password';
```

3. 授予外网登陆权限 

```
mysql> grant all privileges on *.* to username@'%' identified by 'password';
```

4. 授予权限并且可以授权

```
mysql> grant all privileges on *.* to username@'hostname' identified by 'password' with grant option;
```

5. 刷新权限

```
mysql> flush privileges;
```


简单的用户和权限配置基本就这样了。