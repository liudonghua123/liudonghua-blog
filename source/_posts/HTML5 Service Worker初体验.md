---
title: HTML5 Service Worker初体验
tags:
  - HTML5
  - Service Worker
id: 723
categories:
  - 学习
date: 2015-04-22 09:39:58
---

Chrome 42稳定版发布了！根据Chrome博客公布的消息主要对包括新应用、扩展和包括Push API在内的Web Platform APIs进行重要改善，最大的亮点无疑是本地推送通知，只要得到用户许可，即使Chrome浏览器已经关闭，消息也可以推送给用户。

<!--more-->

这个功能是不是很酷！！！
这里面就用到了Service worker这个HTML5的新东西，看上去是不是和Web worker有些类似，来看看别人是怎么说的
> Service Worker是基于Web Worker的事件驱动的，他们执行的机制都是新开一个线程去处理一些额外的，以前不能直接处理的任务。对于Web Worker，我们可以使用它来进行复杂的计算，因为它并不阻塞浏览器主线程的渲染。而Service Worker，我们可以用它来进行本地缓存，相当于一个本地的proxy
> ## What are service workers?
> 
> It's a new API to enable offline Web apps. (Like AppCache but that doesn't suck.)
看一个来自serviceworker.org的简单例子吧！
/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <script>
        // scope defaults to "/*"
        navigator.serviceWorker.register("/service-worker.js").then(function(serviceWorker) {
            console.log("success!");
            // To use the serviceWorker immediately,
            // you might call window.location.reload()
        });
    </script>
  </head>
  <body>
  </body>
</html>
```

/service-worker.js

```javascript
this.oninstall = function(e) {
    // Create a cache of resources.
    var resources = new Cache();
    var visited = new Cache();
    // Fetch them.
    e.waitUntil(resources.add(
        "/index.html",
        "/fallback.html",
        "/css/base.css",
        "/js/app.js",
        "/img/logo.png"
    ).then(function() {
        // Add caches to the global caches.
        return Promise.all([
            caches.set("v1", resources),
            caches.set("visited", visited)
        ]);
    }));
};

this.onfetch = function(e) {
    e.respondWith(
        // Check to see if request is found in cache
        caches.match(e.request).catch(function() {
            // It's not? Prime the cache and return the response.
            return caches.get("visited").then(function(visited) {
                return fetch(e.request).then(function(response) {
                    visited.put(e.request, response);
                    // Don't bother waiting, respond already.
                    return response;
                });
            });
        }).catch(function() {
            // Connection is down? Simply fallback to a pre-cached page.
            return caches.match("/fallback.html");
        });
    );
};
```

从[这里](https://jakearchibald.github.io/isserviceworkerready/)可以了解到还是Chrome 40+对Service Worker支持的比较完善！
[![Is ServiceWorker ready](/resources/2015/04/Is-ServiceWorker-ready.jpg)](/resources/2015/04/Is-ServiceWorker-ready.jpg)

参考资料
[初识ServiceWorker](http://www.foreverpx.cn/2014/09/28/%E5%88%9D%E8%AF%86ServiceWorker/)
[游戏规则改变者 Service Worker](http://www.w3.org/2014/Talks/0825-xiaoqian-offline/#/)
[Push Notifications on the Open Web](http://updates.html5rocks.com/2015/03/push-notificatons-on-the-open-web)
[W3C Service Workers Draft](http://www.w3.org/TR/2014/WD-service-workers-20140508/#introduction)
[serviceworker.org](http://www.serviceworker.org/#)
[W3C HTML5中文兴趣小组技术分享: ECMAScript 6规范导读](http://www.chinaw3c.org/archives/575/)
[service-worker-wiki](https://github.com/sandropaganotti/service-worker-wiki)
[ServiceWorkersDemos](https://github.com/w3c-webmob/ServiceWorkersDemos)