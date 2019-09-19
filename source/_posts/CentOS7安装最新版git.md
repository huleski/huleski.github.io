---
title: CentOS7安装最新版git
categories: categories
tags: tag
date: 2019-09-19 17:48:14
---

一般新装的linux会自带低版本的git, 如果需要安装新版git, 需要下载源码安装

下载源码
-----------

可以从[github](https://github.com/git/git/releases) 或者 [镜像站](https://mirrors.edge.kernel.org/pub/software/scm/git/)下载源码

```bash
 wget https://github.com/git/git/archive/v2.22.0.tar.gz
```

安装
----

```bash
# 卸载旧版本git
yum remove -y git
# 依赖库安装
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum install  gcc perl-ExtUtils-MakeMaker
# 解压gz源码压缩包
tar -xzf git-2.22.0.tar.gz
# 进入解压目录
cd git-2.22.0
# 编译安装
make prefix=/usr/local/git all
make prefix=/usr/local/git install
# 添加到环境变量
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc
# 查看版本号
git --version
```

安装完成!
