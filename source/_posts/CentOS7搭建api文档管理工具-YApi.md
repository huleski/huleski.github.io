---
title: CentOS7搭建api文档管理工具--YApi
categories: YApi
tags: YApi
date: 2019-07-29 20:35:59
---

[YApi](https://github.com/YMFE/yapi)简介
--------

> YApi 是高效、易用、功能强大的 api 管理平台，旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API，YApi 还为用户提供了优秀的交互体验，开发人员只需利用平台提供的接口数据写入工具以及简单的点击操作就可以实现接口的管理。

国内几大互联网公司都在用的本地api文档管理工具

环境要求:

- nodejs（7.6+)
- mongodb（2.6+）
- git

安装nodejs
---------

下载nodejs并解压:

```bash
wget http://nodejs.org/dist/v10.16.0/node-v10.16.0-linux-x64.tar.gz
tar -zxf node-v10.16.0-linux-x64.tar.gz
```

编辑配置文件 `vim /etc/profile` 在文件最后添加:

```bash
### nodejs environment
export NODE_HOME=/data/cordova/node-v10.16.0-linux-x64
export PATH=$NODE_HOME/bin:$PATH  
```

使配置立即生效: 

```bash
source /etc/profile
```

检查node版本命令: `node -v`

检查npm 版本命令: `npm -v`

安装mongodb
----------

新建一个文件 `/etc/yum.repos.d/mongodb-org-4.0.repo` 并加入以下内容:

```bash
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

```bash
## 开始安装
sudo yum install -y mongodb-org
## 启动mongo
systemctl start mongod
## 查看版本
mongo --version
```

修改mongo的配置文件能够远程访问:

```bash
vim /etc/mongod.conf
```

```conf
net:
  port: 27017
  # 将下面127.0.0.1换成 0.0.0.0就可以远程访问mongodb了
  bindIp: 127.0.0.1
```

全局安装yapi-cli并启动安装程序
-------------

```bash
npm install -g yapi-cli --registry https://registry.npm.taobao.org
# 启动api安装程序
yapi server
```

浏览器访问: http://ip:9090, 选择好配置, 点击部署

如果中途报错: `generated/aesprim-browser.js: Permission denied`则执行以下命令:

```bash
npm config set user 0 
npm config set unsafe-perm true
```

安装成功会提示: `切换到部署目录输入：node vendors/server/app.js`, 按照提示去部署目录启动, 再去浏览器访问部署地址就完成了。可是一关闭shell终端yapi会停止运行

所以我们需要使用pm2来保证进程永远都活着

```bash
# 安装pm2
npm install pm2 -g
# pm2启动项目
pm2 start vendors/server/app.js --name YApi
# 显示所有进程状态
pm2 list
# 停止指定的进程
pm2 stop 0
# 杀死指定的进程
pm2 delete 0

# pm2开机自启动项目 (在此之前先启动项目)
pm2 save   # 保存当前状态
pm2 startup   # 开启自启动

# 禁用开机自启动
pm2 unstartup
```
