---
title: gitlab版本升级
categories: gitlab
tags: gitlab
date: 2021-07-08 18:02:05
---

以前用的gitlab-ce是别人构建的中文版镜像, 版本太低, 作者现在也没有再维护更新, 而官方最新版现在已支持中文, 所以打算升级gitlab。

当前的gitlab版本：V10.7.5，准备更新到最新版V14.0.4

[Gitlab官方升级版本指南](https://docs.gitlab.com/ee/update/index.html)


## 数据备份和恢复（可选）

为了防止更新失败后能够复原，可以备份数据（实际上我没有备份上来就是干。。。）

### 备份

进入docker容器输入命令备份数据

```bash
gitlab-rake gitlab:backup:create
```

完成后会在 `/var/opt/gitlab/backups/` 文件夹下生成备份文件 `1572606813_gitlab_backup.tar`， 其中，`1572606813` 是备份版本号后面会用到，然后将文件从容器中复制出来留作备份。

### 恢复

等gitlab的docker镜像启动后将备份文件复制进容器到gitlab的备份目录 `backup` 文件夹下, 在gitlab容器执行命令： 

```bash
gitlab-rake gitlab:backup:restore BACKUP=备份版本号
```

还原备份，这里实际执行：`gitlab-rake gitlab:backup:restore BACKUP=1572606813` 

## 版本升级

由于不同版本之间的差异导致不能直接升级到指定版本, 必须经过中间版本过渡升级到指定版本, 下面是升级版本路线([最新升级路线官方文档](https://docs.gitlab.com/ee/update/#upgrade-paths)):

> 8.11.Z -> 8.12.0 -> 8.17.7 -> 9.0.13 -> 9.5.10 -> 10.0.7 -> 10.8.7 -> 11.0.6 -> 11.11.8 -> 12.0.12 -> 12.1.17 -> 12.10.14 -> 13.0.14 -> 13.1.11 -> 13.8.8 -> 13.12.15 -> 14.0.12 -> 14.3.6 -> 14.9.5 -> 14.10.Z -> 15.0.Z -> 15.1.Z -> 15.4.0 -> latest 15.Y.Z

升级过程就是启动不同的docker版本, 由低到高逐步升级成功
