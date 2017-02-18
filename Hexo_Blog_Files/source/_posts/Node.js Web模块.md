---
title: Node.js Web 模块(14)
date: 2016-06-29 20:51:36
tags: node
categories: nodeJS学习笔记
---


### 14、Node.js Web 模块
使用 Node 创建 Web 服务器
Node.js 提供了 http 模块，http 模块主要用于搭建 HTTP 服务端和客户端，使用 HTTP 服务器或客户端功能必须调用 http 模块，代码如下：
``` javascript
  var http = require('http');
```
以下是演示一个最基本的 HTTP 服务器架构(使用8081端口)，创建 server.js 文件，代码如下所示：
<!-- more -->
``` javascript
  var http = require('http');
  var fs = require('fs');
  var url = require('url');
  //创建服务器
  http.createServer(function(request,response){
    //解析请求，包括文件名
    var pathname = url.parse(request.url).pathname;
    //输出请求文件名
    console.log("Request for " + pathname + " received.");
    //从文件系统中读取请求的文件内容
    fs.readFile(pathname.substr(1), function (err, data){
      if (err) {
        console.log(err);
        //HTTP 状态码：404：NOT FOUND
        //Content Type：text/plain
        response.writeHead(404, {'Content-Type': 'text/html'});
      } else{
        //HTTP 状态码：200：OK
        //Content Type：text/plain
        response.writeHead(200, {'Content-Type': 'text/html'});
        //响应文件内容
        response.write(data.toString());
      };
      //发送响应数据
      response.end();
    });
  }).listen(8081);
```
控制台会输出以下信息
``` javascript
  console.log('Server running at http://127.0.0.1:8081/');
```
接下来我们在该目录下创建一个 index.htm 文件，代码如下：
``` html
        <html>
        <head>
        <title>Sample Page</title>
        </head>
        <body>
            Hello World!
        </body>
        </html>
```
执行 server.js 文件：
```
  node server.js
  Server running at http://127.0.0.1:8081/
```

#### 使用 Node 创建 Web 客户端
Node 创建 Web 客户端需要引入 http 模块，创建 client.js 文件，代码如下所示：
``` javascript
  var http = require('http');
  //用于请求选项
  var optionss = {
    host:'localhost',
    port:'8081'
    path:'/index.htm'
  };
  //处理响应的回调函数
  var callback = function(response){
    //不断更新数据
    var body = '';
    response.on('data', function(data){
      body += data;
    });
    response.on('end', function(){
      //数据接收完成
      console.log(body);
    });
  }
  //向服务器端发送请求
  var req = http.request(options, callback);
  req.end();
```
新开一个终端，执行 client.js 文件，输出结果如下：
```
  node client.js
```
``` html
  <html>
    <head>
      <title>Sample Page</title>
    </head>
    <body>
      Hello World!
    </body>
  </html>
```
执行 server.js 的控制台输出信息如下：
```
  Server running at http://127.0.0.1:8081/
  Request for /index.htm received.   # 客户端请求信息
```
