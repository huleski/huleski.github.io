---
title: 使用frp实现内网穿透
categories: frp
tags: frp
date: 2021-05-24 17:23:16
---

FRP作用
------

> FRP是一个内网穿透工具
FRP让本地局域网的机器可以暴露到公网，简单的说就是在世界的任何地方，你可以访问家里开着的那台电脑!
FRP 支持 TCP、UDP、HTTP、HTTPS， 就是说不仅仅限于本地web服务器可以暴露，整台机器都可以暴露。
> 
> 1.在办公室访问家里的电脑，反之亦然（可以使用NAS+FRP实现私有云盘）
> 
> 2.自己电脑上的项目，方便发给客户朋友演示。比如我做了个小网站，发给朋友看看未上线版本，发个url给他就好了。
> 
> 3.调试一些需要远程调用的程序，远程调用比如微信的API 回调接口。 因为我有了外网地址就不需要部署在公网服务器，直接进行本地调试。

下载安装
-------

[下载地址](https://github.com/fatedier/frp/releases)获取压缩包 `https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_linux_amd64.tar.gz`

在安装的时候注意一下, 我们需要在公网服务器上安装服务端, 然后在本地电脑安装客户端, 这个压缩包里面已经同时包含了服务端和客户端, 安装都是一样的(都是Linux环境), 解压即可

```bash
tar -zxvf frp_0.33.0_linux_amd64.tar.gz
mv frp_0.33.0_linux_amd64 /usr/local/frp
cd /usr/local/frp
```

文件介绍
-------

```txt
frpc                    # 客户端二进制文件
frpc_full.ini           # 客户端配置文件完整示例
frpc.ini                # 客户端配置文件
frps                    # 服务端二进制文件
frps_full.ini           # 服务端配置文件完整示例
frps.in1                # 服务端配置文件
```

配置
----

服务端需要在安装在有公网的服务器上(例如公网ip:233.233.233.233), 配置文件在安装目录下的`frps.ini`, 编辑为以下内容

```bash
bind_port = 7001
vhost_http_port = 7002
vhost_https_port = 7003
```

启动服务端

```bash
nohup ./frps -c ./frps.ini &
```

或者将frps注册为系统服务, 编辑文件 `vim /usr/lib/systemd/system/frps.service`

```bash
[Unit]
Description=frp server
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/frp/frps -c /usr/local/frp/frps.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID

[Install]
WantedBy=multi-user.target
```

然后就可以用系统命令管理服务了

```bash
systemctl enable frps
systemctl start  frps
systemctl status frps
```


客户端需要安装在本地需要映射服务的电脑上, 配置文件在安装目录下的`frpc.ini`, 编辑为以下内容

```bash
[common]
server_addr = 233.233.233.233
server_port = 7001

[abc]
type = tcp
local_ip = 127.0.0.1
local_port = 2000
remote_port = 2000
```

启动客户端

```bash
nohup ./frpc -c ./frpc.ini &
```

现在本地服务就能在外网通过`233.233.233.233:2000`访问到了。

如果有独立域名, 可以在服务端配置nginx反向代理转发域名到`localhost:2000`即可通过域名访问。

将frpc注册为系统服务, 编辑文件 `vim /usr/lib/systemd/system/frpc.service`, 内容: 

```bash
[Unit]
Description=Frp Client Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/frp/frpc -c /usr/local/frp/frpc.ini
ExecReload=/usr/local/frp/frpc reload -c /usr/local/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```