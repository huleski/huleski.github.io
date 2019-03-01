---
title: linux常用命令
categories: linux
tags: linux
date: 2019-02-27 15:44:18
---

日志文件
-------

日志文件|说明
-------|----
/var/log/message  | 系统启动后的信息和错误日志，是Red Hat Linux中最常用的日志之一
/var/log/secure	  | 与安全相关的日志信息
/var/log/maillog  |	与邮件相关的日志信息
/var/log/cron     |	与定时任务相关的日志信息
/var/log/spooler  |	与UUCP和news设备相关的日志信息
/var/log/boot.log |	守护进程启动和停止相关的日志消息

系统
----
命令|说明
----|----
uname -a                    |	查看内核/操作系统/CPU信息
cat /etc/issue              |	登陆信息显示数据
cat /etc/redhat-release     |	查看操作系统版本
cat /proc/cpuinfo           |	查看CPU信息
hostname                    |	查看计算机名
lspci -tv                   |	列出所有PCI设备
lsusb -tv                   |	列出所有USB设备
lsmod                       |	列出加载的内核模块
env                         |	查看环境变量

资源
----
命令|说明
----|----
free -m                         |	查看内存使用量和交换区使用量
df -h                           |	查看各分区使用情况
du -sh <目录名>                  |  查看指定目录的大小
grep MemTotal /proc/meminfo     |	查看内存总量
grep MemFree /proc/meminfo      |	查看空闲内存量
uptime                          |	查看系统运行时间、用户数、负载
cat /proc/loadavg               |	查看系统负载

磁盘和分区
----
命令|说明
----|----
fdisk -l                |   # 查看所有分区	 
swapon -s               |   # 查看所有交换分区	 
fdisk /dev/vdb          |   # 对磁盘/dev/edb进行分区
mount /dev/vdb1 /data   |   # 将磁盘分区/dev/vdb1挂载到/data下
echo /dev/vdb1 /data ext4 defaults 0 0 >> /etc/fstab | 启动时自动挂载分区

网络
----
命令|说明
----|----
ifconfig	        |   查看所有网络接口的属性
iptables -L	        |   查看防火墙设置
route -n	        |   查看路由表
netstat -lntp	    |   查看所有监听端口
netstat -antp	    |   查看所有已经建立的连接
netstat -s	        |   查看网络统计信息

进程：
----
命令|说明
----|----
ps -ef  |	查看所有进程
top	    |   实时显示进程状态

用户：
----
命令|说明
----|----
w	                        |   查看活动用户
id <用户名>                 |   查看指定用户信息
last	                    |   查看用户登录日志
cut -d: -f1 /etc/passwd	    |   查看系统所有用户
cut -d: -f1 /etc/group	    |   查看系统所有组
crontab -l  	            |   查看当前用户的计划任务

服务：
----
命令|说明
----|----
chkconfig --list |	列出所有系统服务

程序：
----
命令|说明
----|----
rpm -qa	                |    查看所有安装的软件包
yum install <程序名>    |   yum安装
yum search <程序名>     |   yum搜索安装包
yum list               |   搜索yum已安装程序
tar xzvf 
