---
title: 跨域和CORS
date: 2018-09-02 17:50:21
categories:
  - JavaScript
tags: 
- 跨域
---
# 跨域和CORS

最近的一个项目在对接的时候，会报出以下错误：

> <font color=red>OPTIONS **\*** 401()</font>  
> <font color=red>Failed to load **\***: Response to preflight request doesn't pass access conreol check : No 'Access-Conrtrl-Allow-Origin' header is present on the requested resoutce. Origin 'http://localhost:10001' is not allowed access.</font>

直译过来是 OPTIONS 请求错误，response 的 header 里面没有 'Access-Conrtrl-Allow-Origin'字段。

如果在开发过程中看到这个错误的话就可以直接截图给后端让他改了（虽然产生原因在浏览器……

<!--more-->

# 同源策略

同源是指三个相同：

- 协议
- 域名
- 端口

如果这三个中间有任何一个不相同，则称之为不同源。
在浏览器中，如果源地址和目标地址不同源，则以下三种行为会受到同源策略限制：

- Cookie、LocalStorage 和 IndexDB 读取
- Dom 获取
- AJAX 请求

# CORS

但是在平时的开发过程中，本地使用的服务器是 127.0.0.1，当然和后端的服务器不同源。这时候就不可避免地进行跨域请求。
目前在浏览器中对不同源的目标发送 XMLHttpRequest 或者 fetch 都会自动地进行 CORS（Cross-Origin-Resource-Share, 跨域资源共享)，用户和开发者使用起来和正常请求没有差别。

## 简单请求

上面的错误就是 CORS 的其中一个过程，如果将要发出的跨域请求满足以下条件，那么就属于 CORS 定义下的“简单请求”：

1. 请求方法是以下三种方法之一：

   - HEAD
   - GET
   - POST

2. HTTP 的头信息不超出以下几种字段：

   - Accept
   - Accept-Language
   - Content-Language
   - Last-Event-ID
   - Content-Type：只限于三个值 application/x-www-form-urlencoded、multipart/form-data、text/plain

简单请求和普通的请求区别是会在头部加上一个 Origin 字段，如果指定的源不在许可的范围内，就不会返回一个包含'Access-Control-Allow-Origin'的字段，浏览器会报出上面的第二个错误。

## 非简单请求

上面的第一个错误：OPTIONS 401()，显示是 OPTIONS 方法的请求返回了一个 Unauthorized 状态码。
如果浏览器发出的跨域请求时非简单请求，那么就会先发出一个"preflight"(预检请求)，来检验源是否在服务器允许 CORS 的源列表内，如果返回的头部包含 Access-Control-Allow-Origin 字段而且值包含该源，那么浏览器会正常发出该请求。

那么现在错误原因已经很清晰了，首先由于 127.0.0.1 和服务器"不同源"，浏览器自动执行 CORS，非简单请求先发出 OPTIONS 预检，服务器返回的头部没有 Access-Control-Allow-Origin 字段，因此 options 报 401 错误。

解决办法：截图给后端（设置一下 Access-Control-Allow-Origin 添加 127.0.0.1 或者\*（通配符）

有关于跨域的其他解决方案 JSONP、WebSocket，OPTIONS 请求返回的其他头部字段，请继续阅读参考资料中的博文。

# 参考资料

1. [HTTP 访问控制（CORS）—— MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)
2. [浏览器的同源策略——ruanyifeng](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)
3. [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
4. [HTTP 状态码——Wikipedia](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)


<small>朱耀华_20180902</small>
