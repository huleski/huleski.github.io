---
title: Springboot解决跨域
categories: Springboot
tags: 
    - Springboot
    - 跨域
date: 2019-09-07 14:28:47
---

跨域访问概念
---------

> CORS（Cross Origin Resource Sharing）跨域资源共享：表示 JavaScript 代码所在的机器和后端 api 所在的机器不是同一台的情况下实现资源访问。

广义的跨域

> 资源跳转： A链接、重定向、表单提交
> 
> 资源嵌入： `<link>、<script>、<img>、<frame>`等dom标签
> 
> 脚本请求： js发起的ajax请求、dom和js对象的跨域操作等

在前后端分离的项目中，前端一般是 SPA （Single Page Application）类型的应用，所有的 JavaScript 代码都会“下载”到用户机器的浏览器中，后端 api 在服务器端以单个机器或者集群的形式存在。

同源策略

> 同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。

常见跨域场景
----------

URL| 说明 | 是否允许通信
---| ------ | ----------
http://www.domain.com/a.js<br>http://www.domain.com/b.js<br>http://www.domain.com/lab/c.js|同一域名，不同文件或路径|允许
http://www.domain.com:8000/a.js<br>http://www.domain.com/b.js | 同一域名，不同端口|不允许
http://www.domain.com/a.js<br>https://www.domain.com/b.js|同一域名，不同协议|不允许
http://www.domain.com/a.js<br>http://192.168.4.12/b.js|域名和域名对应相同ip|不允许
http://www.domain.com/a.js<br>http://x.domain.com/b.js<br>http://domain.com/c.js|主域相同，子域不同|不允许
http://www.domain1.com/a.js<br>http://www.domain2.com/b.js|不同域名|不允许

跨域解决方案
---------

```text
1、 通过jsonp跨域
2、 document.domain + iframe跨域
3、 location.hash + iframe
4、 window.name + iframe跨域
5、 postMessage跨域
6、 跨域资源共享（CORS）
7、 nginx代理跨域
8、 nodejs中间件代理跨域
9、 WebSocket协议跨域
```

1. 通过jsonp跨域

jquery ajax:

```javascript
$.ajax({
    url: 'http://www.domain2.com:8080/login',
    type: 'get',
    dataType: 'jsonp',  // 请求方式为jsonp
    jsonpCallback: "handleCallback",    // 自定义回调函数名
    data: {}
});
```

vue.js

```javascript
this.$http.jsonp('http://www.domain2.com:8080/login', {
    params: {},
    jsonp: 'handleCallback'
}).then((res) => {
    console.log(res); 
})
```

jsonp缺点：只能实现get一种请求。

2. 跨域资源共享（CORS）

普通跨域请求：只服务端设置Access-Control-Allow-Origin即可，前端无须设置，若要带cookie请求：前后端都需要设置。

原生ajax

```javascript
// 前端设置
xhr.withCredentials = true;
```

jquery

```javascript
$.ajax({
    ...
   xhrFields: {
       withCredentials: true    // 前端设置是否带cookie
   },
   crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
    ...
});
```

axios

```javascript
axios.defaults.withCredentials = true
```

java后台设置 (springboot)

```java
@Configuration
public class FilterConfig {

    /**
     * 支持跨域请求
     * @author Holeski
     * @date 2019/8/28 9:48
     */
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedOrigin("*");
        config.setAllowCredentials(true);
        config.addAllowedMethod("*");
        config.addAllowedHeader("*");

        UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
        configSource.registerCorsConfiguration("/**", config);
        return new CorsFilter(configSource);
    }
}
```

nginx反向代理

```bash
#proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;

    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        #当前端只跨域不带cookie时，可以为 *
        add_header Access-Control-Allow-Origin http://www.domain1.com;  
        add_header Access-Control-Allow-Credentials true;
    }
}
```

遇到一个非常奇怪的bug是, 在本地按照springboot的方式设置好了, 调试也没问题, 打好jar包放到服务器上就跨域失败, 后来捣鼓了一天找到一个方法:

```properties
spring.mvc.dispatch-options-request=true
```

设置完就好了, 可是查看源码发现他的默认值就是`true`。。。。。。后面有时间再仔细看看这个问题
