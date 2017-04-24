---
title: Yet another introduction to service worker
date:
tags: service worker, push, fetch
---

#### 主要内容
- service worker的简介
- service worker中的fetch和push实践

#### 什么是service worker
- service worker首先是一种[web worker](http://www.html5rocks.com/en/tutorials/workers/basics/)
- **拥有拦截网页请求，push notification**，sync等能力，本文主要讲前两者

#### service worker的lifecycle
> html5rocks中的一篇[tutorials](http://www.html5rocks.com/en/tutorials/service-worker/introduction/)
![service worker lifecycle](https://www.skyitachi.cn/oss/pic/sw-lifecycle-small.png)

service worker主要状态有6种parsed, installing, installed, activating, activated, 和redundant.
> 引用[service worker state explained](https://bitsofco.de/the-service-worker-lifecycle/)
![service worker state](https://bitsofco.de/content/images/2016/07/Lifecycle-3.png)

> [service worker interface](https://w3c.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-interface)
![service worker interface](https://www.skyitachi.cn/oss/pic/sw-interface.png)

#### ServiceWorker和ServiceWorkerRegistration状态说明
- ServiceWorker本身有6种状态，而ServiceWorkerRegistration也有`installing`,`waiting`, `active`这三种状态
- ServiceWorkerRegistration的状态取决于ServiceWorker的不同状态, 考虑以下一种情况
  > 当现有的ServiceWorker已经运行了之后，打开其他tab时，这个worker在注册时有可能已经是active状态了，所以ServiceWorkerRegistration本身也就是active了，不需要再install

#### fetch实践
1. register
```javascript
// app.js
navigator.serviceWorker.register("./ctrl.js")
  .then(function (register) {
    if (register.installing) {
      console.log("Service Worker: in the installing state");
    } else if (register.waiting) {
      console.log("Service Worker: in the waiting state");
    } else if (register.active) {
      console.log("Service Worker: in the active state");
    }
    console.log("Service Worker: ", register.scope);
  })
  .catch(function (err) {
    console.log("serviceWorker registration failed: ", err);
  });
```
2. install
```javascript
// ctrl.js, 将需要缓存的静态资源加入到cache storage中
var urlsToCache = [
  "/static/index.js", 
  "/static/index.css"
];
var CACHE_NAME = "page-cache-v1";
this.addEventListener("install", function (event) {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function (cache) {
        return cache.addAll(urlsToCache);
      })
  );
});
```
3. activate
```javascript
this.addEventListener("activate", function (event) {
  // do some cache strategy, 这里是只允许CACHE_NAME的缓存存在
  var cacheWhiteList = [CACHE_NAME];
  event.waitUntil(
    caches.keys().then(function (cacheNames) {
      return Promise.all(
        cacheNames.map(function (cacheName) {
          if (cacheWhiteList.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```
4. fetch
```javascript
// 这里就是拦截请求的关键了，官方说法是change network stack default behavior, 即使是offline的时候也是可触发fetch的
this.addEventListener("fetch", function (event) {
  event.respondWith(
    caches.match(event.request)
      .then(function (response) {
        if (response) {
          console.log("SW: cache hit");
          return response;
        } else {
          console.log("SW: cache does not hit");
          return fetch(event.request);
        }
      })
  );
});
```
说明一下： fetch只拦截get请求且是https的，实践中localhost的也是可行的

#### push实践
> chrome中push依赖gcm，点击[如何申请](https://developers.google.com/web/fundamentals/getting-started/push-notifications/step-04?hl=en), 同时需要配置manifest.json，此文中也有说明，当然得自备梯子

首先简要看下push的整个流程
![push.png](https://www.skyitachi.cn/oss/pic/sw-push.png)
图中黑色线代表初始化设置阶段，起点是service worker的registration,
蓝色线代表服务端发送push事件的整个流程, 起点是your server
> 下文中假设已经申请好gcm相关和设置好manifest

1. register
```javascript
navigator.serviceWorker.register("./ctrl.js")
  .then(function (registration) {
    // 检测一下权限
    if (Notification.permission === "denied") {
      console.warn("no permission");
      return;
    }
    registration.pushManager.subscribe({
      userVisibleOnly: true
    }).then(function(sub) {
      // send subId to your server
    });
  });
```
2. 监听push事件
```javascript
this.addEventListener("push", event => {
  const msgUrl = "your server message url";
  event.waitUntil(
    fetch(msgUrl)
      .then(response => {
        if (response.status === 200) {
          return response.json();
        }
      })
      .then(data => {
        // show chrome browser notification
        return this.registration.showNotification(data.title, {
          body: data.message,
          icon: data.ico,
          tag: data.tag
        });
      })
  );
});
```
3. 发送push事件
```javascript
// your_server.js
var gcm = { api_key: "your api key" };
var subId = "your subId";
var send = function example() {}; // your server send response method
request({
  url: "https://android.googleapis.com/gcm/send",
  method: "post",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `key= ${gcm.api_key}`
  },
  body: JSON.stringify({
    "registration_ids": [subId]
  })
}, function (err, response, body) {
  if (err) {
    send(err.message);
  } else {
    send({
      title: "msg_title",
      body: "msg body",
      icon: "icon_url",
      tag: "msg_tag"
    });
  }
});
```

#### 总结
- 关于service worker中fetch和push的使用我这里只是简单的介绍了一下，没有提到比如scope和service worker如何更新等方面, 这篇文章也是我自己学习service worker的一些总结，很明显有些不足之处，网上的service worker教程大多比较零散，这里会集中一些资料让大家方便查阅

#### 参考资料
 - https://bitsofco.de/the-service-worker-lifecycle/
 - https://github.com/w3c/ServiceWorker/blob/master/explainer.md
 - https://jakearchibald.github.io/isserviceworkerready/resources.html
 - https://w3c.github.io/ServiceWorker/spec/service_worker/index.html
 - http://www.html5rocks.com/en/tutorials/service-worker/introduction/
 - https://developers.google.com/web/fundamentals/getting-started/push-notifications/?hl=en
