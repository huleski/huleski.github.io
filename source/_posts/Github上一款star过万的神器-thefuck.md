---
title: Github上一款star过万的神器-thefuck
categories: Github
tags: Github
date: 2019-03-25 19:41:43
---

简介
=======

> The Fuck is a magnificent app, inspired by a @liamosaur tweet, that corrects errors in previous console commands.

![](https://raw.githubusercontent.com/nvbn/thefuck/master/example.gif)

[项目地址](https://github.com/nvbn/thefuck), 它可以帮你纠正大部分的命令行输入错误 
- 想要返回上一级目录, 手速过快（你懂得）输成了`cd..`？ 。。。
- 想查看数据库的运行状态, 忘记单词拼写 (^_^;) `systecl status mysql`？ 。。。

以上这些都不是问题, **`fuck`** 一下就好了, 就像你遇到bug了, 大喊一声: **艹**(第四声)


安装
----
安装环境是CentOS7, `The Fuck`的环境要求:
- python (3.4+)
- pip
- python-dev

由于CentOS 7 默认安装了python2.7.5, 一些命令要用它比如yum, 因此我们需要安装python3并与python2共存

查看是否已经安装python, 使用命令: `python -V`. 使用命令: `which python` 查看python可执行文件的位置, 我的是在 `/usr/bin` 目录下, 切换到该目录下执行: `ll python*`命令查看

里面会有个python2和python软链接, python指向的是python2.7, 因为我们要装python3, 所以python要指向python3才行, 我们后面需要对它进行备份, 先**安装相关环境**:
```
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
```

备份:
```
mv python python.bak
```

**安装python3**

获取安装包:
```
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tar.xz
```

下载完成, 解压:
```
tar -xvJf  Python-3.6.8.tar.xz
```

切换进入
```
cd Python-3.6.8
```

编译安装
```
./configure prefix=/usr/local/python3
make && make install
```


安装完毕，/usr/local/目录下就会有python3了

因此我们可以添加软链到执行目录下/usr/bin
```
ln -s /usr/local/python3/bin/python3 /usr/bin/python
```

再去`/usr/bin`目录下, 可以看到已经创建了一个指向python3的软链接python,测试是否安装成功:
```
python -V
```
看看输出的是不是python3的版本, 再执行 `python2` -V` 看到的就是python2的版本

因为执行yum需要python2版本，所以我们还要修改yum的配置，执行：
```
vi /usr/bin/yum
```
把 `#! /usr/bin/python` 修改为#! `/usr/bin/python2`

同理 `vi /usr/libexec/urlgrabber-ext-down` 文件里面的 `#! /usr/bin/python` 也要修改为`#! /usr/bin/python2`

现在python3已经安装成功, 同时python2也存在

**安装pip并升级pip工具**

1. 方法一: 直接安装

```
yum install -y python-pip
pip install --upgrade pip setuptools
```

2. 方法二: python3安装完成后默认已经带有pip, 你可以用以下命令,创建软链接

```bash
ln -s /usr/local/python3/bin/pip /usr/bin/pip
```

**安装python-dev**

需要注意的是: 在centOS7中该模块叫 **python-devel**

```bash
yum install -y python-devel
```

**安装thefuck**

```
pip install thefuck
```

修改文件 `vim /usr/bin/thefuck`, 把第一行的 `#!/usr/bin/python2` 改成 `#!/usr/bin/python`

查看thefuck是否安装成功:

```
fuck -v
```

如果看到如下信息:
```
The Fuck 3.28 using Python 3.6.8 and Bash 4.2.46(2)-release
```

则安装成功
下面我们来试一下:
```
[root@localhost usr]# pwf
bash: pwf: 未找到命令...
[root@localhost usr]# fuck
​​​​​​​​​​pwd [enter/↑/↓/ctrl+c]
/usr
[root@localhost usr]# 
```

OK! 以后有问题`fuck`一下就好了ヽ(￣▽￣)ﾉ