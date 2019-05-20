---
title: Linux系统下几个有趣的命令
categories: Linux
tags: Linux
date: 2019-05-20 19:51:12
---

作为一名程序员，在别人的眼里往往是充满科技感、神秘感的，而在我们自己的眼里却往往是觉得无聊、枯燥的。其实，在程序的世界里同样会充满着各种的彩蛋，这些彩蛋往往都是一些大神留下来的，我们未曾发现，只是我们缺少发现程序之美而已。今天我们就来介绍几个有趣的Linux命令, 来体验一波程序彩蛋之美。

文章参考: [Linux系统下好玩有趣的命令，你又用过几个？](https://www.jianshu.com/p/08e9094f61ce)

由于原文都是在Ubuntu系统下安装使用的, 而我自己是在CentOS7系统下操作, 偶尔有些不同, 我也只选择了其中几个很有趣的试了试

## sl （Steam Locomotive）

安装, 这个是最简单的, 经常在你想要查看当前目录的时候错误的输入了`sl`, 现在就回出现一辆小火车开过的动画...

```bash
yum -y install sl
```

运行

```bash
sl
```

## oneko

撸猫指令, oneko会生成一只图像猫, 在屏幕上乱跑

```bash
yum -y install oneko
```

运行

```bash
oneko
```

## cmatrix

该指令会在屏幕上下一场字符雨

```bash
## 下载压缩包
wget https://jaist.dl.sourceforge.net/project/cmatrix/cmatrix/1.2a/cmatrix-1.2a.tar.gz
## 解压
tar xvf cmatrix-1.2a.tar.gz
## 进入安装目录
cd cmatrix-1.2a
## 安装依赖
yum install ncurses-devel
## 编译源码并安装, 需要有gcc,gcc-c++, 如果没有就yum安装
./configure && make && make install
## 安装完毕, 运行
cmatrix
```

启动成功后, 有看到字符雨, 按`q`退出

## ASCIIquarium

彩蛋：把你的linux终端变成一个海洋世界，各种生物在不断呈现，有鱼、有水、有草…, 好鬼酷哦(wzr...)

```bash
## 安装依赖工具
yum -y install ncurses-devel perl-CPAN libyaml-devel perl-CGI perl-Curses perl-ExtUtils-MakeMaker
## 安装依赖文件
wget http://search.cpan.org/CPAN/authors/id/K/KB/KBAUCOM/Term-Animation-2.4.tar.gz
tar -zxvf Term-Animation-2.4.tar.gz
cd Term-Animation-2.4/
perl Makefile.PL && make
make install
## 安装ASCIIquarium
wget http://www.robobunny.com/projects/asciiquarium/asciiquarium.tar.gz
tar -zxvf asciiquarium.tar.gz
cd asciiquarium_1.1/
cp asciiquarium /usr/local/bin/
chmod 755 /usr/local/bin/asciiquarium
## 运行
asciiquarium
```

同样的运行成功后按`q`退出