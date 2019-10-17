---
title: nginx设置反向代理访问pixiv图片
categories: nginx
tags: nginx
date: 2019-10-17 19:53:19
---

反向代理
-------

正常情况下, pixiv 的图片服务器域名为 `i.pximg.net`，因为有防盗链保护，只要 `Referer` 是空值或不是來自 pixiv 的域名就会返回`403`。

使用nginx反向代理只需要將 `www.pixiv.net` 设置到 `Referer`中就可以直接访问图片了

nginx配置
--------

例如在自己的电脑上安装好nginx后, 修改配置文件, 加上以下配置

```conf
proxy_cache_path D:\logs levels=1:2 keys_zone=pximg:10m max_size=10g inactive=7d use_temp_path=off;

server {
    server_name  localhost;
    listen 80;
    access_log off;

    location / {
        proxy_cache pximg;
        proxy_pass https://i.pximg.net;
        proxy_cache_revalidate on;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_lock on;
        add_header X-Cache-Status $upstream_cache_status;
        proxy_set_header Host i.pximg.net;
        proxy_set_header Referer "https://www.pixiv.net/";
        proxy_set_header User-Agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.84 Safari/537.36";

        proxy_cache_valid 200 7d;
        proxy_cache_valid 404 5m;
    }
}
```

修改完后重启nginx, 随便访问一张pixiv图片, 只需把`https`换成`http`, 并且把`i.pximg.net`域名换成`localhost`即可, 例如: 

- pixiv网站上原始链接 (直接访问会返回403): https://i.pximg.net/img-original/img/2017/12/20/00/12/19/66360679_p0.png
- 经过我们的nginx反向代理 (可以正常访问)：http://localhost/img-original/img/2017/12/20/00/12/19/66360679_p0.png

这样我们就能绕过pixiv的防盗链从而直接访问pixiv图片了, 其实网上有一个公开的pixiv反向代理域名`i.pixiv.cat`, 在访问pixiv图片时, 只需将`i.pximg.net` 更換成 `i.pixiv.cat` 就可以使用

<style>
table th:first-of-type {
    width: 50%;
}
</style>

直接访问图片   |   反向代理访问图片
---------     |   -------------
![直接访问](https://i.pximg.net/img-original/img/2017/12/20/00/12/19/66360679_p0.png)    |   ![反向代理访问](https://i.pixiv.cat/img-original/img/2017/12/20/00/12/19/66360679_p0.png)
