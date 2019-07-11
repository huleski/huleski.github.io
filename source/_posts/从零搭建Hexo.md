---
title: 搭建Hexo碰到的问题
categories: Hexo
tags: Hexo
date: 2019-07-10 11:15:12
---

- 升级主题到Next6, 字体样式失效, 图标也丢失, 启动还会报错 

```bash
localhost/:1 Refused to apply style from 'http://localhost:4000/lib/font-awesome/css/font-awesome.min.css?v=4.6.2' 
because its MIME type ('text/html') is not a supported stylesheet MIME type, and strict MIME checking is enabled.
```

在hexo中有自带`font-awesome`字体的, 但是升级到hexo6时就没有了, 把next目录下（next -> source -> lib）的 `font-awesome` 文件夹复制到next6同样的目录下, 解决

![font文件夹位置](http://pubgmjp23.bkt.clouddn.com/%5BD60F5%7D%29IHOGB5%5BW9$T%29F8R.png)