---
title: docker搭建GitLab-CE中文版详细教程
categories: GitLab
tags: 
  - GitLab
  - docker
date: 2019-03-19 19:56:29
---

我是在本机局域网搭建的gitlab,  切记**服务器配置不能太低**, gitlab比较耗资源, 这也是它功能强大的来源, 搭建环境:
- 系统: centOS7, 8G内存, i5处理器
- ip: 10.12.2.22
- 预留端口: 8090

## 1.获取镜像

``` 
docker pull beginor/gitlab-ce:11.0.1-ce.0
```

查看镜像, 有1.5G大小

![查看gitlab镜像](http://pubgmjp23.bkt.clouddn.com/15856745-4bcae9df30615e9f.png)

## 2.运行镜像
由于是docker镜像运行, 所以我们需要把gitlab的配置, 数据, 日志存到容器外面, 即将其挂载到宿主机。
先准备三个目录：
```
mkdir -p /home/software/gitlab/etc
mkdir -p /home/software/gitlab/logs
mkdir -p /home/software/gitlab/data
```

准备好这三个目录之后， 就可以开始运行 Docker 镜像了。完整的运行命令如下 ( [查看更多详细配置](https://docs.gitlab.com/omnibus/docker/) )：
```
docker run \
--detach \
--publish 8443:443 \    # 映射https端口, 不过本文中没有用到
--publish 8090:80 \      # 映射宿主机8090端口到容器中80端口
--publish 8022:22 \      # 映射22端口, 可不配
--name gitlab \            
--restart always \
--hostname 10.12.2.22 \    # 局域网宿主机的ip, 如果是公网主机可以写域名
-v /home/software/gitlab/etc:/etc/gitlab \    # 挂载gitlab的配置文件
-v /home/software/gitlab/logs:/var/log/gitlab \    # 挂载gitlab的日志文件
-v /home/software/gitlab/data:/var/opt/gitlab \    # 挂载gitlab的数据
-v /etc/localtime:/etc/localtime:ro \    # 保持宿主机和容器时间同步
--privileged=true beginor/gitlab-ce    # 在容器中能以root身份执行操作
```

这个时候已经搭建完了, 查看一下 ( 启动需要几分钟, 我大概启动了2分钟多 ) :

![查看gitlab运行状态](http://pubgmjp23.bkt.clouddn.com/15856745-ffc482d0a6a1fb2d.png)

看到状态显示为 `healthy` 就代表已经启动了, 这时候去访问 http://10.12.2.22:8090
第一次访问时，将被重定向到密码重置屏幕, 默认帐户的用户名是root, 登录后, 您可以更改用户名

![登录界面](http://pubgmjp23.bkt.clouddn.com/15856745-d70d7d9e3de6d8e9.png)

## 3.配置gitlab
要能充分使用gitlab, 必须配置邮件发送功能, 修改配置文件 gitlab.rb (启动镜像后产生的文件), 这里我配置的是QQ邮箱 ( [查看其它邮箱配置](https://docs.gitlab.com/omnibus/settings/smtp.html#smtp-settings) )
```
vim /home/software/gitlab/etc/gitlab.rb
```
在文件的最后加上配置: 
```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "fuck@qq.com"
gitlab_rails['smtp_password'] = "授权码"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = 'fuck@qq.com'
```

网上很多教程说要配置external_url, 我按照加了配置后gitlab反而异常了, 不管它, 用默认的就好了, 保存退出, 再另外开一个终端, 进入容器:
```
docker exec -it gitlab /bin/bash
```

此时已经进入docker容器了, 容器中执行命令重新配置gitlab:
```
gitlab-ctl reconfigure            # 重新配置
```

现在可以测试邮件是否配置正确了, 同样容器中执行:
```
gitlab-rails console   		# 进入邮件控制台, 稍等一会才能进入
Notify.test_email('baka@qq.com', 'Message Subject', 'Message Body').deliver_now  	# 发送测试邮件
```

![测试gitlab邮件发送](http://pubgmjp23.bkt.clouddn.com/15856745-8672d7423144b20a.png)

![邮件已经成功收到](http://pubgmjp23.bkt.clouddn.com/15856745-c84daee944ffb48e.png)

邮件配置已经完成了, 现在需要配置项目路径 (如果你预留的gitlab映射端口是80的话, 你已经配置完了), 在宿主机中 (容器外面) 修改文件gitlab.yml, 如果port不对, 要改过来
```
vim /home/software/gitlab/data/gitlab-rails/etc/gitlab.yml
```

![修改ip端口](http://pubgmjp23.bkt.clouddn.com/15856745-1ede73a0e00ee5a3.png)

改完之后在容器中重启gitlab就配置完成了。 注意: 此时**不能**再重新配置(gitlab-ctl reconfigure), 否则可能会改变刚修改的gitlab.yml文件
```
gitlab-ctl restart     # 重启gitlab
```

**gitlab常用命令**
```

# 重新应用gitlab的配置
gitlab-ctl reconfigure
 
# 重启gitlab服务
gitlab-ctl restart
 
# 查看gitlab运行状态
gitlab-ctl status
 
#停止gitlab服务
gitlab-ctl stop
 
# 查看gitlab运行日志
gitlab-ctl tail
```

**gitlab邮件没收到? 看这里**
如果没收到邮件, 有两个可能 :
- 电脑没网络    (我就是犯了这个愚蠢的错误, 主机改成静态ip后没网了也不知道, 折腾了很久才发现)
- 邮件被当做垃圾邮件了    ( 去垃圾箱看看吧 -_-! )
- 邮箱密码错了, 注意是授权码哦


**gitlab搭建前后对比**
- 搭建前, 内存只占用了500多M
- 搭建后, 内存就占用了5G

![搭建gitlab后内存使用飙升](http://pubgmjp23.bkt.clouddn.com/15856745-030a1594e4a710ac.png)

