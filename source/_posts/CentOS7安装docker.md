---
title: CentOS7安装docker
categories: docker
tags: docker
date: 2021-07-08 20:11:49
---

Docker 官方建议要 CentOS7.0 及以上系统版本，本文介绍 Docker CE 在CentOS7下的安装使用。

## Device Mapper

Docker默认使用AUFS作为存储驱动，但是AUFS并没有被包括在Linux的主线内核中。

```bash
# 安装
yum install -y device-mapper
# 重新加载dm_mod内核模块
modprobe dm_mod
```

## 环境准备

```bash
# 卸载旧版本
yum remove docker \ docker-client \ docker-client-latest \ docker-common \ docker-latest \ docker-latest-logrotate \ docker-logrotate \ docker-selinux \ docker-engine-selinux \ docker-engine

# 安装编译环境
yum -y install gcc gcc-c++

# 安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2

# 设置stable镜像仓库, 下面二选一
1. yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
2. yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新yum软件包索引
yum makecache fast
```

## 安装配置

```bash
#从高到低列出Docker-ce的版本
yum list docker-ce.x86_64  --showduplicates | sort -r    

# 安装最新版
yum -y install docker-ce

# 启动docker
systemctl start docker

# 让docker开机自启动
systemctl enable docker

# 查看docker版本
docker -v
```

### 修改容器默认存储路径

> docker安装之后默认的服务数据存放根路径为/var/lib/docker目录下，var目录默认使用的是根分区的磁盘空间；所以这是非常危险的事情；随着我们镜像、启动的容器实例开始增多的时候，磁盘所消耗的空间也会越来越大，所以我们必须要做数据迁移和修改docker服务的默认存储位置路径；有多种方式是可以修改docker默认存储目录路径的，但是最好是在docker安装完成后，第一时间便修改docker的默认存储位置路径为其他磁盘空间较大的目录(一般企业中为/data目录)，规避迁移数据过程中所造成的风险。

```bash
# 创建docker容器存放的路径
mkdir -p /home/data/docker

# 停止docker
systemctl stop docker

# 迁移数据到新目录
rsync -avz /var/lib/docker/ /home/data/docker/
```

编辑docker配置文件 `vim /etc/docker/daemon.json`, 指定存储路劲

```json
"graph":"/home/data/docker"
```

### 镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决。

Docker国内镜像：

- 网易加速器：http://hub-mirror.c.163.com
- 官方中国加速器：https://registry.docker-cn.com
- ustc的镜像：https://docker.mirrors.ustc.edu.cn

编辑docker配置文件 `vim /etc/docker/daemon.json`, 加入以下配置

```json
"registry-mirrors": ["https://hub-mirror.c.163.com"]
```

### 开启docker API
在 `/etc/docker/daemon.json` 文件中添加 

```json
"hosts":["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
```

配置好后docker却异常报错了, 这里需要修改启动文件 `/usr/lib/systemd/system/docker.service`

```bash
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock 注释掉改为下面这样
ExecStart=/usr/bin/dockerd
```

然后执行以下步骤即可

```bash
systemctl daemon-reload
systemctl reset-failed docker.service
systemctl restart docker
```

## 安装docker-compose

```bash
# 获取脚本
curl -L "https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 添加可执行权限
chmod +x /usr/local/bin/docker-compose
```
