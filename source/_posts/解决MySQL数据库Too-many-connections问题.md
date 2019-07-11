---
title: 解决MySQL数据库Too many connections问题
categories: MySQL
tags: MySQL
date: 2019-07-09 12:06:05
---

方法一:
--------

因为这种方法是临时修改, 连上MySQL后重启后会失效

连上MySQL后执行:

```bash
mysql> set GLOBAL max_connections=500;
```

方法二:
------

```bash
# 修改mysql配置文件my.cnf
vi /etc/my.cnf

# 在[mysqld]段中添加或修改max_connections值
max_connections=500
# 重启
```

- 查看mysql的最大连接数：

```bash
mysql> show variables like '%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections |  500  |
+-----------------+-------+
1 row in set (0.00 sec)
```

- 查询所有连接到这个服务器上的MySQL连接

```bash
mysql> show processlist;
```

获取到MySQL数据连接列表后，每一条记录都会有一个进程ID号（在上表的第一列）。执行以下命令关闭一条连接:

```bash
# 其中1180421是进程列表里找到并且要杀掉的进程号
mysql> kill 1180421;
```

- 查看服务器响应的最大连接数

```bash
mysql> show global status like 'Max_used_connections';
```
