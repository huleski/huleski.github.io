---
title: Linux设置闹钟定时播放一首歌
categories: Linux
tags: Linux
date: 2019-10-08 11:03:47
---

命令行播放音乐
-------

现在听歌都是播放mp3文件, 一些Linux系统没有提供对mp3的支持, 需要安装一些软件, 这里我选audacious, 安装

```bash
yum install -y audacious
```

命令行输入: `audacious`就可以弹出一个音乐播放窗口, 然后在窗口中操作播放音乐

也可以命令行播放某个mp3文件

```bash
audacious -Hq /usr/local/src/inspire.mp3
```

将上面的播放命令写到`/usr/local/src/play.sh`脚本中, 后面会用到

cron介绍
------

linux内置的cron进程能帮我们实现这些需求，cron搭配shell脚本，非常复杂的指令也没有问题。

我们经常使用的是crontab命令是cron table的简写，它是cron的配置文件，也可以叫它作业列表，我们可以在以下文件夹内找到相关配置文件。

- /var/spool/cron/ 目录下存放的是每个用户包括root的crontab任务，每个任务以创建者的名字命名
- /etc/crontab 这个文件负责调度各种管理和维护任务。
- /etc/cron.d/ 这个目录用来存放任何要执行的crontab文件或脚本。
- 我们还可以把脚本放在/etc/cron.hourly、/etc/cron.daily、/etc/cron.weekly、/etc/cron.monthly目录中，让它每小时/天/星期、月执行一次。

crontab的常用的命令如下：

```s
crontab [-u username]　　　　//省略用户表表示操作当前用户的crontab
    -e      (编辑工作表)
    -l      (列出工作表里的命令)
    -r      (删除工作作)
```

我们用crontab -e进入当前用户的工作表编辑，是常见的vim界面。每行是一条命令。

crontab的命令构成为 时间+动作，其时间有分、时、日、月、周五种，操作符有:

```s
* 取值范围内的所有数字
/ 每过多少个数字
- 从X到Z
，散列数字
```

加入一条命令:

```bash
# 每个工作日中午12点执行一次播放音乐脚本
0 12 * * 1,2,3,4,5 /usr/local/src/play.sh
```

闹钟就设定好了
