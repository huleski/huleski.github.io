---
title: centos7安装Nginx及配置
categories: Nginx
tags: 
    - Nginx
    - centos7
date: 2019-02-25 14:41:56
---

Nginx是一款轻量级的网页服务器、反向代理服务器。相较于Apache、lighttpd具有占有内存少，稳定性高等优势。**它主要的用途是提供反向代理服务**。

安装所需环境
----

1. gcc 安装

安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
```
yum install gcc-c++
```

2. PCRE pcre-devel 安装

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
```
yum install -y pcre pcre-devel
```

3. zlib 安装

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
```
yum install -y zlib zlib-devel
```

4. OpenSSL 安装

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
```
yum install -y openssl openssl-devel
```

安装Nginx
-----

- 直接下载.tar.gz安装包，地址：https://nginx.org/en/download.html
- 使用wget命令下载（推荐）:
```
wget -c https://nginx.org/download/nginx-1.14.2.tar.gz
```

- 解压
```
tar -zxvf nginx-1.10.1.tar.gz
```

- 执行以下命令配置安装
```
cd nginx-1.14.2         # 进入Nginx目录
./configure             # 使用默认配置
make && make install    # 编辑安装

```

- 查找安装路径：
```
whereis nginx
```

安装完成, 以下是Nginx常用命令:
```
/usr/local/nginx/sbin/nginx –t                          # 测试配置文件是否正常
/usr/local/nginx/sbin/nginx                             # 启动Nginx
/usr/local/nginx/sbin/nginx -s stop                     # 停止Nginx
/usr/local/nginx/sbin/nginx -s reload                   # 重新加载配置文件
```

- 开机自启动

编辑文件 rc.local
```
vim /etc/rc.local
```

在最下面增加一行:
```
/usr/local/nginx/sbin/nginx
```

设置执行权限：
```
chmod 755 /etc/rc.local
```
Nginx安装完毕

配置Nginx
--------



