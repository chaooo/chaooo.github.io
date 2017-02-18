---
title: Node.js 路由(8)
date: 2016-06-28 18:55:08
tags: node
categories: nodeJS学习笔记
---

### 8、Node.js 路由
我们要为路由提供请求的`URL`和其他需要的`GET`及`POST`参数，随后路由需要根据这些数据来执行相应的代码。因此，我们需要查看`HTTP`请求，从中提取出请求的`URL`以及`GET/POST`参数。我们需要的所有数据都会包含在`request`对象中，该对象作为`onRequest()`回调函数的第一个参数传递。但是为了解析这些数据，我们需要额外的`Node.JS`模块，它们分别是`url`和`querystring`模块。
<!-- more -->
``` javascript
                       url.parse(string).query
                                               |
               url.parse(string).pathname      |
                           |                   |
                           |                   |
                         ------ -------------------
    http://localhost:8888/start?foo=bar&hello=world
                                    ---       -----
                                     |          |
                                     |          |
                  querystring(string)["foo"]    |
                                                |
                             querystring(string)["hello"]
```
当然我们也可以用`querystring`模块来解析`POST`请求体中的参数，稍后会有演示。
现在我们来给`onRequest()`函数加上一些逻辑，用来找出浏览器请求的`URL`路径：
``` javascript 
  var http = require("http");
  var url = require("url");
  function start() {
    function onRequest(request, response) {
      var pathname = url.parse(request.url).pathname;
      console.log("Request for " + pathname + "received.");
      response.writeHead(200, {"Content-Type": "text/plain"});
      response.write("Hello World");
      response.end();
    }
    http.createServer(onRequest).listen(8888);
    console.log("Server has started.");
  }
  exports.start = start;
```
现在我们可以来编写路由了，建立一个名为`router.js`的文件:
``` javascript
  function route(pathname){
    console.log("About to route a request for " + pathname);
  }
  exports.route = route;
```
在添加更多的逻辑以前，我们先来看看如何把路由和服务器整合起来(我们将使用依赖注入的方式较松散地添加路由模块)。首先，我们来扩展一下服务器的`start()`函数，以便将路由函数作为参数传递过去：
``` javascript
  var http = require("http");
  var url = require("url");
  function start() {
    function onRequest(request, response) {
      var pathname = url.parse(request.url).pathname;
      console.log("Request for " + pathname + "received.");
      route(pathname);//路由函数
      response.writeHead(200, {"Content-Type": "text/plain"});
      response.write("Hello World");
      response.end();
    }
    http.createServer(onRequest).listen(8888);
    console.log("Server has started.");
  }
  exports.start = start;
```
同时，我们会相应扩展`index.js`，使得路由函数可以被注入到服务器中：
``` javascript
  var server = require("./server");
  var router = require("./router");
  server.start(router.route);
```
现在启动应用（`node index.js`），随后请求一个URL，你将会看到应用输出相应的信息，这表明我们的HTTP服务器已经在使用路由模块了，并会将请求的路径传递给路由：
``` javascript
  node index.js
  Request for /foo received.
  About to route a request for /foo
  //以上输出已经去掉了比较烦人的`/favicon.ico`请求相关的部分。
```
