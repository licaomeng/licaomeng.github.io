---
layout: post
title:  "Nginx初探"
date:   2017-03-16 23:06:19
categories: Nginx
---
前段时间在帮手公司做微信公众号，使用到了Nginx做反向代理，发现Nginx真的是个好东西。

##我为什么使用Nginx？

我们的微信公众号开发是使用前后端分离的架构，后端使用java做WebService, 前端就是用html5 + jQuery。下面就来说下web 服务器选择的几个阶段。

**Phase 1: tomcat + ecstatic server（微信公众测试号）**

Tomcat几乎成了java web的首选web服务器，这里依旧使用tomcat做web服务器。由于是前后端分离，前端也需要单独部署，这里使用的是node 的 ecstatic server，使用npm工具可以获得。由于微信要求开发者用来接收微信消息和事件的接口URL只能以http://或https://开头，也就是说只支持80以及443端口。这样tomcat就不能使用我们常用的8080端口，而要改成80端口（在server.xml中修改）。ecstatic server使用9000端口，就可以是前后端分别开发，分开部署。

**Phase 2: tomcat**

在Phase 1，我们使用的是微信测试号，Phase 2我们使用官方认证的公众号。于是，在迁移的过程中就发现了一些问题。由于我们的公众号使用了JSSDK，需要配置安全域名，在Phase 1是这样写的：XXX.XXX.XXX.XXX:9000，但是到了Phase 2这样写就是不允许的了，不支持使用ip以及端口，只可以是通过ICP备案验证的域名。所以这里不能再使用9000端口了。所以Phase 2前端server最后使用了tomcat。这样前后端都使用tomcat作为server。具体做法是在server.xml中添加一行：
<Context path="/" docBase="C:/ YOUR-FRONT-END-PATH " reloadable="true" debug="0" crossContext="true"/>

**Phase 3: Nginx + tomcat**

Phase 2似乎已经大功告成了，但是这里仍然有很大的问题。如果我们修改了前端代码，必须重启tomcat才能够生效。既然这样还谈什么前后端分离，这样就不太好了。那怎么才能避免tomcat的无畏重启呢？当然前后端可以部署到不同的server上面自然不会互相影响，可我们只有一台server的情况下，怎样才可以做到呢？这个时候Nginx就横空出世了。基本思想就是客户端只和Nginx打交道，而Nginx作为我们tomcat的代理。tomcat部署后端java代码使用8080端口，而前端就直接把Nginx作为静态资源服务器。而反向代理Nginx使用80端口。下面是Nginx配置文件的关键片段：

```
location / {
			proxy_pass http://127.0.0.1:8080;
}
location ~ \.(html|js|css|png|gif)$ {    
			root C:/YOUR-FRONT-END-PATH;
			expires 24h;
}
```

上面分别是java后端以及前端的配置。注意这里有一个坑，http://127.0.0.1:8080不要写成localhost:8080, 这样会导致前端文件加载慢，甚至不能加载。这个坑真是害人不浅啊。
至此，已经完成了我对Nginx的第一次接触。
周末偶然在书店看到一本关于Nginx的书《深入理解Nginx》,看了一下午，燃起了我对于阅读Nginx源码的热情。Github上面有Nginx的源码：https://github.com/nginx/nginx。另外发现淘宝的tengine团队也有很好的讲解Nginx源码的资料：http://tengine.taobao.org/book/index.html。准备下一阶段有计划的开始借助资料阅读Nginx源码。


