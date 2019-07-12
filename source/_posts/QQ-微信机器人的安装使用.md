---
title: QQ/微信机器人的安装使用
categories: 机器人
tags: 机器人
date: 2019-05-12 16:39:47
---

前言
===

经常在微信/QQ上看到可以自动聊天的机器人, 感觉很有趣, 今天我们就可以来创建这样一个机器人. 

**前期准备**: 
> 在[图灵机器人网站](http://www.tuling123.com)上注册一个账号, 注册成功后在网站上创建一个机器人, 勾选qq/微信即可, 创建完成后会有一个apikey, 记住这个apikey, 后面会用到. 

**图灵机器人的作用**: 
> 在聊天机器人中, 当我们发送消息给机器人账号时, 机器人账号会将获取到的消息通过apikey发到图灵机器人网站上, 然后图灵机器人背后使用机器学习+大数据分析相结合的人工智能技术得出消息的回复, 并把该回复响应给机器人账号,进而呈现在你的屏幕上, 这就是整个聊天机器人的工作原理

> 注册好图灵机器人账号后, 它会免费提供给我们每天100次的调用次数, 也就是说他可以每天跟我们对话100次, 如果你觉得少了也可以升级到收费版, 调用次数会大幅提升.

微信机器人
=========

参考[项目](https://github.com/pig6/wxrobot), 该项目使用python3来运行的, 因此我们先安装python3, 去[python官网](https://www.python.org/downloads/)下载, 这里我选择Windows版本, 安装的时候有个地方`add to path` 需要勾选, 其他都点下一步就安装好了.
验证是否安装成功, 在CMD命令行窗口里输入:

```bash
python -V
```

如果输出版本号, 则表示安装成功

[从github下载项目源码](https://github.com/pig6/wxrobot)

下载后进入项目目录,当前目录下应该可以看到`robot.py`文件, 如果是下载zip包则先解压缩

安装依赖文件, 打开CMD窗口输入下面命令:

```bash
pip3 install -U wxpy -i "https://pypi.doubanio.com/simple/"
```

等依赖安装结束后, 启动项目:

```bash
python robot.py
```

此时会出现一个登陆用的二维码, 用手机微信扫码登陆, 登陆成功后会提示登陆成功, 这样一个微信机器人就创建成功了, 试着发消息给这个微信号, 和他聊聊天吧!

**注意**: 

- 如果在扫码登陆的时候报错, 很可能是因为你使用了新申请的微信, 腾讯为了安全考虑做出的新号登录限制, 一个号是否可以用来作为机器人账号, 可以先试着登陆[网页版微信](https://wx.qq.com/)成功与否来判断, 因为该项目的原理就是使用了网页端微信的api来收发消息的
- 该项目里使用了原作者自己提供的图灵机器人apikey, 可能由于使用次数达到限制而无法自动回复, 此时我们应该使用自己图灵账号的apikey了, 打开 `config.py` , 修改里面 `tuling_api_key` 这一项的值为自己的apikey.

QQ机器人
=======

酷Q
---

酷Q是一款免费的qq机器人, 以前是基于webqq、smartQQ协议做的自动收发消息功能, 但现在腾讯已经放弃了网页版的qq, 也就是以前的webqq、smartQQ协议都不行了。后来听说酷Q按照安卓的qq协议反编译出的，因此最新版应该是基于安卓协议的。

下载Windows版安装
---------------

从[酷Q官网](https://cqp.cc/t/23253)下载软件压缩包，直接解压运行即可，此时会要求登录qq账号来作为机器人账号，建议使用小号来登录。登录完成后，会有提示下一步如何操作，根据提示完成后就可以大致明白如何使用QQ机器人的聊天等功能了。如需启用图灵机器人, 设置图灵的apikey后就可以聊天了.

这是最基本的功能，他还有一个强大之处在于，他的插件扩展功能更强大

在酷Q的[应用官网](https://cqp.cc/b/app)，插件在酷Q官网叫应用，选择一个应用下载吧（需要注册登录），下载完后缀名是cpk的文件后，直接放入酷Q安装目录的app文件夹中，重启酷Q就可以加载进去，然后就可以体验机器人的乐趣了。

安装Linux版
----------

一般也不会挂着机器人在Windows上，所以如果我们有linux服务器，就比较好挂着机器人了。

这里使用的Docker安装酷Q，详细查看[官网介绍](https://cqp.cc/t/34558)

首先确保Linux已经装好Docker了，接下来拉取镜像运行就可以了

```bash
docker pull coolq/wine-coolq
```

然后运行 酷Q 镜像：

```bash
docker run --name=coolq --rm -p 9000:9000 -v /root/coolq-data:/home/user/coolq -e VNC_PASSWD=123456 -e COOLQ_ACCOUNT=123456 coolq/wine-coolq
```

运行后，会看到控制台中输出一系列日志。当你看到 `[CQDaemon] Started CoolQ`  时，说明已启动成功。
此时，在浏览器中访问 `http://你的服务器IP:9000` 即可看到远程操作登录页面，输入密码123456，即可看到 酷Q Air 的登录界面啦。
在登录后，右键点击悬浮窗 -> 昵称 -> 勾选「自动登录」，即可保证 酷Q 能自动登录。

这时候如果关闭linux界面酷Q就会停了, 所以我们需要后台运行酷Q:

```bash
docker run --name=coolq -d -p 9000:9000 --restart always -v /root/coolq-data:/home/user/coolq -e VNC_PASSWD=123456 -e COOLQ_ACCOUNT=123456 coolq/wine-coolq
```

查看运行状态:

```bash
docker logs coolq
```

启动/停止服务

```bash
docker start coolq
docker stop coolq
```

如果想安装插件, 把插件放入挂载的文件`/root/coolq-data/app`, 重启酷Q即可