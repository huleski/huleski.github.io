---
title: nginx配置目录浏览及美化主题
categories: nginx
tags: nginx
date: 2021-11-30 19:21:52
---

nginx可以配置浏览目录服务, 这样用户可以直接在浏览器中查看文件目录, 并且可以下载文件, 非常方便

## ngx_http_autoindex_module 模块

目录浏览服务由ngx_http_autoindex_module模块提供

命令|默认值|值域|作用域|例子
----|-----|----|-----|-----
autoindex|off|on：开启目录浏览；<br>off：关闭目录浏览|http, server, location|autoindex on; 打开目录浏览功能
autoindex_format|	html	|html、xml、json、jsonp 分别用这几个风格展示目录	|http, server, location	|autoindex_format html; 以网页的风格展示目录内容。该属性在1.7.9及以上适用
autoindex_exact_size	|on	|on：展示文件字节数；<br>off：以可读的方式显示文件大小	|http, server, location	|autoindex_exact_size off; 以可读的方式显示文件大小，单位为 KB、MB 或者 GB，autoindex_format为html时有效
autoindex_localtime	|off|	on、off：是否以服务器的文件时间作为显示的时间	|http, server, location	|autoindex_localtime on; 以服务器的文件时间作为显示的时间,autoindex_format为html时有效

修改nginx配置

```bash
server {
    listen       10000;
    server_name  localhost;

    location / {
        root   /home;
        # 开启目录浏览 
        autoindex on; 
        # 以可读的方式显示文件大小
        autoindex_exact_size off;
        # 以服务器的文件时间作为显示的时间
        autoindex_localtime on;
        # 展示中文文件名
        charset utf-8,gbk;
    }
}
```

执行命令 `nginx -s reload` 重载配置, 打开浏览器访问 `http://localhost:10000` 就可以在浏览器中访问 `/home` 目录中的文件了, 但是nginx原始的目录很简陋, 不是正常人看的, 可以美化一下

## 安装Nginx FancyIndex模块

先安装所需模块 `FancyIndex`, 在nginx的源码文件夹下下载插件: 

```bash
git clone https://github.com/aperezdc/ngx-fancyindex.git ngx-fancyindex
```
查看已安装的nginx完整编译参数:

```bash
nginx -V
```

控制台会打印出参数:

```bash
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_xslt_module=dynamic
...(略)
```

`configure arguments:`后面一大串的都是编译参数, 复制下来, 执行:

```bash
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-stream_ssl_preread_module --with-http_addition_module --with-http_xslt_module=dynamic
...(略)   --add-module=ngx-fancyindex-0.4.2
```

也就是 `./configure` 跟着上面复制的一大串再加上 `--add-module=ngx-fancyindex-0.4.2` 执行就可以了, 如果执行过程中有报错缺少什么依赖组件就网上搜一下, 少什么安装什么, 转好了再执行直到成功

再执行编译:
```
make
```

再执行:

```bash
2>&1 ./nginx -V | tr ' ' '\n'|grep fan
```

如果看到输出 `ngx-fancyindex` 就说明编译好了

先备份原来的nginx文件(如果有问题可以还原) 

```bash
# /usr/sbin/nginx 是原执行文件
mv /usr/sbin/nginx /usr/sbin/nginx.bak
```

把编译目录中`objs`文件中的nginx移过去就安装完成了:

```bash
mv objs/nginx /usr/sbin/
```

## 选择Fancy Index主题

在 `/home` 目录中下载Fancy Index主题:

```bash
git clone https://github.com/lanffy/Nginx-Fancyindex-Theme.git
```

修改nginx配置

```bash
server {
    listen       10000;
    server_name  localhost;

    location / {
        root   /home;
        # 开启目录浏览 
        autoindex on; 
        # 以可读的方式显示文件大小
        autoindex_exact_size off;
        # 以服务器的文件时间作为显示的时间
        autoindex_localtime on;
        # 展示中文文件名
        charset utf-8,gbk;
        # 新增目录美化配置
        include /home/Nginx-Fancyindex-Theme/fancyindex.conf;
    }
}
```

重启Nginx即可

```bash
nginx -s reload
```