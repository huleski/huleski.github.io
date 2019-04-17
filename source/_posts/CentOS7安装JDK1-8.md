---
title: CentOS7安装JDK1.8
categories: CentOS7
tags: CentOS7
date: 2019-04-17 10:07:07
---

查看已安装的jdk并卸载
----------
查询jdk ( 新装的centos会默认安装openjre )

```bash
whereis java
which java （java执行路径）
echo $JAVA_HOME
echo $PATH
```

确定JDK的版本：

```bash
rpm -qa | grep jdk
```


可能的结果是：

```bash
java-1.7.0-openjdk-1.7.0.191-2.6.15.5.el7.x86_64
java-1.7.0-openjdk-headless-1.7.0.191-2.6.15.5.el7.x86_64
```

卸载jdk

```bash
yum -y remove java-1.7.0-openjdk-1.7.0.191-2.6.15.5.el7.x86_64  
yum -y remove java-1.7.0-openjdk-headless-1.7.0.191-2.6.15.5.el7.x86_64
```

下载安装
-------

[下载地址](http://www.oracle.com/technetwork/java/javase/downloads/index.htm)

将下载好的文件`jdk-8u191-linux-x64.tar.gz`放在 `/data` 目录下, 解压:

```bash
tar -zxvf jdk-8u191-linux-x64.tar.gz
```

解压成功后，则可以看到jdk1.8.0_191文件夹

配置环境变量
----------

打开配置文件

```bash
vim /etc/profile
```

在配置文件末尾添加一下内容

```bash
JAVA_HOME=/data/jdk1.8.0_191
JRE_HOME=/data/jdk1.8.0_191/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

保存退出, 是配置文件立即生效

```bash
source /etc/profie
```

查看java是否安装成功

```bash
java -version
```
