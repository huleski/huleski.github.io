---
title: CentOS7安装maven
categories: maven
tags: 
    - CentOS7
    - maven
date: 2019-02-26 15:03:32
---

下载maven-3.5.4, 可以在[官网](http://mirror.bit.edu.cn/apache/maven/maven-3/)中选择不同版本
```
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
```

解压
```
tar -zxvf apache-maven-3.5.4-bin.tar.gz
```

配置环境变量, 编辑文件:
```
vim /etc/profile
```

加入以下内容 ( 根据自己maven实际解压路径配置 )
```
MAVEN_HOME=/home/software/maven-3.5.4
PATH=$PATH:$MAVEN_HOME/bin
export MAVEN_HOME
```

使配置生效
```
source /etc/profile
```

测试是否安装成功
```
mvn -v
```