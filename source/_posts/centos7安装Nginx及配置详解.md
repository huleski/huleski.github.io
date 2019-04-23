---
title: centos7安装Nginx及配置详解
categories: Nginx
tags: 
    - Nginx
    - centos7
date: 2019-02-25 14:41:56
---

Nginx是一款轻量级的网页服务器、反向代理服务器。相较于Apache、lighttpd具有占有内存少，稳定性高等优势。**它主要的用途是提供反向代理服务**。

安装所需环境
----

1. gcc 安装

安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
```
yum install gcc-c++
```

2. PCRE pcre-devel 安装

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
```
yum install -y pcre pcre-devel
```

3. zlib 安装

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
```
yum install -y zlib zlib-devel
```

4. OpenSSL 安装

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
```
yum install -y openssl openssl-devel
```

安装Nginx
-----

- 直接下载.tar.gz安装包，地址：https://nginx.org/en/download.html
- 使用wget命令下载（推荐）:
```
wget -c https://nginx.org/download/nginx-1.14.2.tar.gz
```

- 解压
```
tar -zxvf nginx-1.14.2.tar.gz
```

- 执行以下命令配置安装
```
cd nginx-1.14.2         # 进入Nginx目录
./configure             # 使用默认配置
make && make install    # 编辑安装

```

- Nginx默认配置
```
nginx path prefix: "/usr/local/nginx"
nginx binary file: "/usr/local/nginx/sbin/nginx"
nginx modules path: "/usr/local/nginx/modules"
nginx configuration prefix: "/usr/local/nginx/conf"
nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
nginx pid file: "/usr/local/nginx/logs/nginx.pid"
nginx error log file: "/usr/local/nginx/logs/error.log"
nginx http access log file: "/usr/local/nginx/logs/access.log"
nginx http client request body temporary files: "client_body_temp"
nginx http proxy temporary files: "proxy_temp"
nginx http fastcgi temporary files: "fastcgi_temp"
nginx http uwsgi temporary files: "uwsgi_temp"
nginx http scgi temporary files: "scgi_temp"
```

- 查找安装路径：
```
whereis nginx
```

安装完成, 以下是Nginx常用命令:
```
/usr/local/nginx/sbin/nginx –t                          # 测试配置文件是否正常
/usr/local/nginx/sbin/nginx                             # 启动Nginx
/usr/local/nginx/sbin/nginx -s stop                     # 停止Nginx
/usr/local/nginx/sbin/nginx -s reload                   # 重新加载配置文件
```

- 开机自启动

编辑文件 rc.local
```
vim /etc/rc.local
```

在最下面增加一行:
```
/usr/local/nginx/sbin/nginx
```

设置执行权限：
```
chmod 755 /etc/rc.local
```
Nginx安装完毕, 打开浏览器访问 `http://localhost`查看是否安装成功

配置Nginx
--------

Nginx配置文件nginx.conf大致分为以下几块:
```
main
events   {
  ....
}
http        {
  ....
  upstream myproject {
    .....
  }
  server  {
    ....
    location {
        ....
    }
  }
  server  {
    ....
    location {
        ....
    }
  }
  ....
}
```

nginx配置文件主要分为六个区域： 
`main(全局设置)、events(nginx工作模式)、http(http设置)、 `

`sever(主机设置)、location(URL匹配)、upstream(负载均衡服务器设置)。`

- main模块

 main模块是一个全局的设置：
 
```
user nobody nobody;
worker_processes 2;
error_log  /usr/local/var/log/nginx/error.log  notice;
pid        /usr/local/var/run/nginx/nginx.pid;
worker_rlimit_nofile 1024;
```

`user` 指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行。

`worker_processes` 指定了Nginx要开启的子进程数。每个Nginx进程平均耗费10M~12M内存。根据经验，一般指定1个进程就足够了，如果是多核CPU，建议指定和CPU的数量一样的进程数即可。我这里写2，那么就会开启2个子进程，总共3个进程。

`error_log` 用来定义全局错误日志文件。日志输出级别有debug、info、notice、warn、error、crit可供选择，其中，debug输出日志最为最详细，而crit输出日志最少。

`pid` 用来指定进程id的存储文件位置。

`worker_rlimit_nofile` 用于指定一个nginx进程可以打开的最多文件描述符数目，这里是65535，需要使用命令“ulimit -n 65535”来设置。

- events 模块

events模块来用指定nginx的工作模式和连接数上限
```
events {
    use epoll; #linux平台
    worker_connections  1024;
}
```
`use` 用来指定Nginx的工作模式。Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll。其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在Mac中。

`worker_connections` 用于定义Nginx每个进程的最大连接数，即接收前端的最大请求数，默认是1024。最大客户端连接数由`worker_processes`和`worker_connections`决定，即`Max_clients=worker_processes*worker_connections`，在作为反向代理时，Max_clients变为：`Max_clients = worker_processes * worker_connections/4 (注: 可能有出入)` 。
进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令“ulimit -n 65536”后worker_connections的设置才能生效。

- http 模块
http模块是最核心的模块了，它负责HTTP服务器相关属性的配置，它里面的server和upstream子模块至关重要，等到反向代理和负载均衡以及虚拟目录等会仔细说。

```
http{
    include mime.types;     #文件扩展名与文件类型映射表

    default_type application/octet-stream;      #默认文件类型

    charset utf-8;         #默认编码

    #设置日志的格式
    log_format  main    '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';


    #服务器名字的hash表大小
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
    server_names_hash_bucket_size 128;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    client_header_buffer_size 32k;

    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    large_client_header_buffers 4 64k;

    #设定通过nginx上传文件的大小
    client_max_body_size 8m;

    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    #sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile on;

    #开启目录列表访问，合适下载服务器，默认关闭。
    autoindex on;

    #此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    tcp_nopush on;
     
    tcp_nodelay on;

    #长连接超时时间，单位是秒
    keepalive_timeout 120;

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k;    #最小压缩文件大小
    gzip_buffers 4 16k;    #压缩缓冲区
    gzip_http_version 1.0;    #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2;    #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;    #压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;

    #开启限制IP连接数的时候需要使用
    #第一个参数：$binary_remote_addr 表示通过remote_addr这个标识来做限制，“binary_”的目的是缩写内存占用量，是限制同一客户端ip地址
    #第二个参数：zone=one:10m表示生成一个大小为10M，名字为one的内存区域，用来存储访问的频次信息
    #第三个参数：rate=1r/s表示允许相同标识的客户端的访问频次，这里限制的是每秒1次，还可以有比如30r/m的
    limit_zone crawler $binary_remote_addr 10m;](limit_req_zone binary_remote_addr zone=one:10m rate=1r/s;

    # 第一个参数：zone=one 设置使用哪个配置区域来做限制，与上面limit_req_zone 里的name对应
    # 第二个参数：burst=5，这个配置的意思是设置一个大小为5的缓冲区当有大量请求过来时，超过了访问频次限制的请求可以先放到这个缓冲区内
    # 第三个参数：nodelay，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排队
    server {
        location /search/ {
            limit_req zone=one burst=5 nodelay;
        }
    }
}
```
`include` 用来设定文件的mime类型, 类型在配置文件目录下的mime.type文件定义，来告诉nginx识别文件类型。

`default_type` 设定了默认的类型为二进制流，也就是当文件类型未定义时使用这种方式，例如在没有配置asp 的locate 环境时，Nginx是不予解析的，此时，用浏览器访问asp文件就会出现下载了。

`log_format` 用于设置日志的格式，和记录哪些参数，这里设置为main，刚好用于access_log来记录这种类型。

main的类型日志如下：也可以增删部分参数。
```
127.0.0.1 - - [21/Apr/2015:18:09:54 +0800] "GET /index.php HTTP/1.1" 200 87151 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.76 Safari/537.36"
```
`access_log` 用来纪录每次的访问日志的文件地址，后面的main是日志的格式样式，对应于log_format的main。

`sendfile` 参数用于开启高效文件传输模式。将tcp_nopush和tcp_nodelay两个指令设置为on用于防止网络阻塞。

`keepalive_timeout` 设置客户端连接保持活动的超时时间。在超过这个时间之后，服务器会关闭该连接。

其他参数后面介绍

- server 模块

sever 模块是http的子模块，它用来定一个虚拟主机，基本的配置:
```
server {
    #监听端口
    listen 80;

    #域名可以有多个，用空格隔开
    server_name www.w3cschool.cn w3cschool.cn;
    index index.html index.htm index.php;
    root /data/www/w3cschool;

    #对******进行负载均衡
    location ~ .*.(php|php5)?$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
        
    #图片缓存时间设置
    location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$ {
        expires 10d;
    }
        
    #JS和CSS缓存时间设置
    location ~ .*.(js|css)?$ {
        expires 1h;
    }
    #在需要使用负载均衡的server中增加 proxy_pass http://bakend/;
}
```
`server` 标志定义虚拟主机开始。 

`listen` 用于指定虚拟主机的服务端口。 

`server_name` 用来指定IP地址或者域名，多个域名之间用空格分开。 

`root` 表示在这整个server虚拟主机内，全部的root web根目录。注意要和locate {}下面定义的区分开来。 

`index` 全局定义访问的默认首页地址。注意要和locate {}下面定义的区分开来。 

`access_log` 用来指定此虚拟主机的访问日志存放路径，最后的main用于指定访问日志的输出格式。

- location 模块

location 用来定位/解析URL，所以，它也提供了强大的正则匹配功能，也支持条件判断匹配，用户可以通过location指令实现Nginx对动、静态网页进行过滤处理。
```
#对 "/" 启用反向代理
location / {
    proxy_pass http://127.0.0.1:88;
    proxy_redirect off;
    proxy_set_header X-Real-IP $remote_addr;
        
    #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
    #以下是一些反向代理的配置，可选。
    proxy_set_header Host $host;

    #允许客户端请求的最大单文件字节数
    client_max_body_size 10m;

    #缓冲区代理缓冲用户端请求的最大字节数，
    #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
    #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
    client_body_buffer_size 128k;

    #表示使nginx阻止HTTP应答代码为400或者更高的应答。
    proxy_intercept_errors on;

    #后端服务器连接的超时时间_发起握手等候响应超时时间
    #nginx跟后端服务器连接超时时间(代理连接超时)
    proxy_connect_timeout 90;

    #后端服务器数据回传时间(代理发送超时)
    #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
    proxy_send_timeout 90;

    #连接成功后，后端服务器响应时间(代理接收超时)
    #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
    proxy_read_timeout 90;

    #设置代理服务器（nginx）保存用户头信息的缓冲区大小
    #设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
    proxy_buffer_size 4k;

    #proxy_buffers缓冲区，网页平均在32k以下的设置
    #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
    proxy_buffers 4 32k;

    #高负荷下缓冲大小（proxy_buffers*2）
    proxy_busy_buffers_size 64k;

    #设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
    #设定缓存文件夹大小，大于这个值，将从upstream服务器传
    proxy_temp_file_write_size 64k;
}

#本地动静分离反向代理配置
#所有jsp的页面均交由tomcat或resin处理
location ~ .(jsp|jspx|do)?$ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://127.0.0.1:8080;
}
    
#所有静态文件由nginx直接读取不经过tomcat或resin
location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|
pdf|xls|mp3|wma)$ {
    expires 15d;
}
```
`location ~` 开启正则匹配。

后面详细介绍location的匹配规则

- upstream 模块

upstream 模块用来作负载均衡
```
upstream backend{
    ip_hash;
    server 192.168.12.1:80;
    server 192.168.12.2:80 down;
    server 192.168.12.3:8080  max_fails=3  fail_timeout=20s;
    server 192.168.12.4:8080;
}
```
`upstream` 指令指定了一个负载均衡器的名称backend。这个名称可以任意指定，在后面需要的地方直接调用即可。

`ip_hash` 是其中的一种负载均衡调度算法。紧接着就是各种服务器了。用server关键字表识，后面接ip。

**Nginx的负载均衡模块目前支持以下几种调度算法**:

1. `weight` 轮询（默认）。每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。weight。指定轮询权值，weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下。
2. `ip_hash`。每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题。
3. `fair`。依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。
4. `url_hash`。按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx 的hash软件包。
5. `least_conn` 下一个请求将被分派到活动连接数量最少的服务器

在HTTP Upstream模块中，可以通过server指令指定后端服务器的IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。常用的状态有：

`down`，表示当前的server暂时不参与负载均衡。

`backup`，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。

`max_fails`，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。

`fail_timeout`，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

**注意** 当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。

location匹配规则
--------------
语法:
```
location [=|~|~*|^~] /uri/ {...}
```
符号    |   含义
--------|-------
=       |  表示精确匹配
^~       | 表示 URI 以某个常规字符串开头。Nginx 不对 URL 做编码，因此请求为 /static/20%/aa，可以被 ^~ /static/ /aa 匹配到
~       | 表示区分大小写的正则匹配
~*       | 表示不区分大小写的正则匹配
/       | 通用匹配，任何请求都会匹配

多个 location 配置的情况下匹配顺序为：
- 首先匹配 =
- 其次匹配 ^~
- 其次是按文件中顺序的正则匹配
- 最后是交给 /
- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

若规则如下:
```
location = / {
    #规则A
}

location = /login {
    #规则B
}

location ^~ /static/ {
    #规则C
}

location ~ \.(gif|jpg|png|js|css)$ {
    #规则D
}

location ~* \.png$ {
    #规则E
}

location / {
    #规则F
}
```
- 访问根目录 /， 比如 http://localhost/ 将匹配规则 A
- 访问 http://localhost/login 将匹配规则 B，http://localhost/register 则匹配规则 F
- 访问 http://localhost/static/a.html 将匹配规则 C
- 访问 http://localhost/a.png 符合规则 D 和规则 E，但是只匹配较前的D，
- 访问 http://localhost/static/c.png 则优先匹配到规则 C
- 访问 http://localhost/a.PNG 则匹配规则 E，而不会匹配规则 D，因为规则 E 不区分大小写。
- 访问 http://localhost/category/id/1111 则最终匹配到规则 F，这个时候 nginx 可以作为反向代理服务器。

server_name参数的配置
-------------------------

nginx中的server_name指令主要用于配置基于名称虚拟主机.

如果server_name配置了多个参数, 在接到请求后的匹配顺序如下：

1. 确切的server_name匹配：

```yml
server {
    listen       80;
    server_name  www.google.com google.com;
    ...
}
```

2. 以*通配符开始的最长字符串：

```yml
server {
    listen       80;
    server_name  *.google.com;
    ...
}
```

3. 以*通配符结束的最长字符串：

```yml
server {
    listen       80;
    server_name  www.*;
    ...
}
```

注意: 通配符名字只可以在名字的起始处或结尾处包含一个星号，并且星号与其他字符之间用点分隔。所以，`www.*.example.org` 和 `w*.example.org` 都是非法的。

有一种形  如“.example.org”的特殊通配符，它可以既匹配确切的名字“example.org”，又可以匹配一般的通配符名字“*.example.org”。

4. 匹配正则表达式：

```yml
server {
    listen       80;
    server_name  ~^(?<www>.+)\.google\.com$;
    ...
}
```

nginx将按照1,2,3,4顺序对server name进行匹配, 而与配置的排版先后顺序无关, 只要有一项匹配后就会停止搜索。