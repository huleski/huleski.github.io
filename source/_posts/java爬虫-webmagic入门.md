---
title: java爬虫-webmagic入门
categories: 爬虫
tags: 爬虫
date: 2019-06-14 18:48:13
---

webmagic简介
-----------

![webmagic](https://camo.githubusercontent.com/8b3a6d93d95d39147ef93f5021f6b69255bda888/687474703a2f2f7765626d616769632e696f2f696d616765732f6c6f676f2e6a706567)

[官方网站](http://webmagic.io/) 

> webmagic是一个开源的Java垂直爬虫框架，目标是简化爬虫的开发流程，让开发者专注于逻辑功能的开发。webmagic的核心非常简单，但是覆盖爬虫的整个流程，也是很好的学习爬虫开发的材料。

webmagic的主要特色：

- 完全模块化的设计，强大的可扩展性。
- 核心简单但是涵盖爬虫的全部流程，灵活而强大，也是学习爬虫入门的好材料。
- 提供丰富的抽取页面API。
- 无配置，但是可通过POJO+注解形式实现一个爬虫。
- 支持多线程。
- 支持分布式。
- 支持爬取js动态渲染的页面。
- 无框架依赖，可以灵活的嵌入到项目中去。

该项目是参考了:

[python爬虫 scrapy](https://github.com/scrapy/scrapy)

[Java爬虫 Spiderman](http://git.oschina.net/l-weiwei/spiderman)

快速开始
-------

webmagic使用maven管理依赖，在项目中添加对应的依赖即可使用webmagic：

```xml
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-core</artifactId>
    <version>0.7.3</version>
</dependency>
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-extension</artifactId>
    <version>0.7.3</version>
</dependency>
```

**注意: ** 这里都是参考官方文档, 但实际上maven库的包源有bug, 不过作者已经在源代码里修复了, 需要copy源代码重新编译打包

克隆源代码 

```bash
git clone https://github.com/code4craft/webmagic.git
```

或者直接下载zip源码压缩包, 下载完解压即可

下载完成后导入到开发工具中, 重新 install webmagic-core模块即可

创建第一个爬虫:

```java
public class GithubRepoPageProcessor implements PageProcessor {

    // 部分一：抓取网站的相关配置，包括编码、抓取间隔、重试次数等
    private Site site = Site.me().setRetryTimes(3).setSleepTime(1000);

    @Override
    // process是定制爬虫逻辑的核心接口，在这里编写抽取逻辑
    public void process(Page page) {
        // 部分二：定义如何抽取页面信息，并保存下来
        page.putField("author", page.getUrl().regex("https://github\\.com/(\\w+)/.*").toString());
        page.putField("name", page.getHtml().xpath("//h1[@class='entry-title public']/strong/a/text()").toString());
        if (page.getResultItems().get("name") == null) {
            //skip this page
            page.setSkip(true);
        }
        page.putField("readme", page.getHtml().xpath("//div[@id='readme']/tidyText()"));

        // 部分三：从页面发现后续的url地址来抓取
        page.addTargetRequests(page.getHtml().links().regex("(https://github\\.com/[\\w\\-]+/[\\w\\-]+)").all());
    }

    @Override
    public Site getSite() {
        return site;
    }

    public static void main(String[] args) {

        Spider.create(new GithubRepoPageProcessor())
                //从"https://github.com/code4craft"开始抓
                .addUrl("https://github.com/code4craft")
                //开启5个线程抓取
                .thread(5)
                //启动爬虫
                .run();
    }
}
```

点击运行, 就能看到爬虫工作了

如果运行的时候报错 `javax.net.ssl.SSLException: Received fatal alert: protocol_version` 那是你没有编译源码打包.

代码中注释已经很详细了, 相信大家也大概明白爬取的过程了. 更详细的爬虫教程看作者的[官方教程](http://webmagic.io/docs/)