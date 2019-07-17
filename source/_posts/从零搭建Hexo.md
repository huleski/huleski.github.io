---
title: 从零搭建Hexo
categories: Hexo
tags: Hexo
date: 2019-07-10 11:15:12
---

参考文章:
-------

- [使用hexo快速搭建个人博客](https://blog.yupaits.com/tools/hexo-blog.html#%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)
- [Hexo+Next 添加菜单分类页面](https://hoxis.github.io/Hexo+Next%20%E6%96%B0%E5%A2%9E%E8%8F%9C%E5%8D%95%E5%88%86%E7%B1%BB%E9%A1%B5%E9%9D%A2.html)
- [NexT主题官网](http://theme-next.iissnan.com/getting-started.html#menu-settings)
- [Hexo进阶高级教程](https://segmentfault.com/a/1190000010058060)
- [hexo搭建个人博客--NexT主题优化](https://segmentfault.com/a/1190000013660164)
- [Hexo+NexT优化部署](https://www.jianshu.com/p/d34e9531cfce)
- [asdfv1929 's Home](https://asdfv1929.github.io/photos/)
- [NexT主题的优化定制修改指南](https://blog.csdn.net/u012195214/article/details/79204088)
- [Hexo提交百度和Google收录站点](https://www.93bok.com/Hexo%E6%8F%90%E4%BA%A4%E7%99%BE%E5%BA%A6%E5%92%8CGoogle%E6%94%B6%E5%BD%95%E7%AB%99%E7%82%B9/)
- [Hexo NexT 主题6.x版本的使用配置与美化](https://xian6ge.cn/posts/5b8c41e7/)

碰到的问题
---------

- 升级主题到Next6, 字体样式失效, 图标也丢失, 启动还会报错 

```bash
localhost/:1 Refused to apply style from 'http://localhost:4000/lib/font-awesome/css/font-awesome.min.css?v=4.6.2' 
because its MIME type ('text/html') is not a supported stylesheet MIME type, and strict MIME checking is enabled.
```

在hexo中有自带`font-awesome`字体的, 但是升级到hexo6时就没有了, 把next目录下（next -> source -> lib）的 `font-awesome` 文件夹复制到next6同样的目录下, 解决

![font文件夹位置](http://pubgmjp23.bkt.clouddn.com/%5BD60F5%7D%29IHOGB5%5BW9$T%29F8R.png)
