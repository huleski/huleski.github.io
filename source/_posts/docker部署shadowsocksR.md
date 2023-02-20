---
title: docker部署shadowsocksR
categories: docker
tags: 
    - docker
    - shadowsocksR
date: 2023-02-17 15:08:55
---

> 前期准备
>
> 系统: CentOS7
>
> 环境：docker 17.0
>
> 一台在 境外/国外 的云服务器

一键部署:

```bash
docker run -d --name ssr \
  -p 38989:8989 -p 38989:8989/udp malaohu/ssr-with-net-speeder \
  -s 0.0.0.0 -p 8989 \
  -k 123456 \
  -m rc4-md5 \
  -o http_simple \
  -O auth_sha1_v4
```

```text
解释一下
服务器端口：38989
密码：123456
加密：rc4-md5
协议：auth_sha1_v4  这里需要注意，auth_sha1这个协议已经被此软件弃用了，注意有_v4
混淆：http_simple
```

最后, 需要放开云服务器的38989端口访问权限(tcp和udp), 可以访问了

# 血泪总结(写于搭建SSR后的第3天)

阿里云香港服务器公网ip被墙了，服务器已作废，***，退钱！ 

####不用用阿里云！

### 不要用阿里云！

## 不要用阿里云！
 
#这里不是家！！！ T_T

