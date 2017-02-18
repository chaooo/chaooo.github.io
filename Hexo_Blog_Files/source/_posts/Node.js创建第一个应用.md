---
title: Node.js创建第一个应用(1)
date: 2016-06-27 11:55:07
tags: node
categories: nodeJS学习笔记
---

### 1、Node.js创建第一个应用

#### 1.1、创建服务器

使用 `http.createServer()` 方法创建服务器，并使用 `listen` 方法绑定 8888 端口。 函数通过 `request`, `response` 参数来接收和响应数据。实例如下，在项目的根目录下创建一个叫 server.js 的文件，并写入以下代码：
<!-- more --> 
``` javascript
  var http = require("http");//引入require模块
  http.createServer(function(require, response){
    //发送 HTTP 头部
    //HTTP 状态值：200：OK
    //内容类型：text/plain
    response.writeHead(200, {'Content-Type':'text/plain'});
    //发送响应数据：“Hello World”
    response.end('Hello World\n');
  }).listen(8888);
  //终端打印如下信息
  console.log('Server running at http://127.0.0.1:8888/');
```
使用 node 命令执行以上的代码：
```
  node server.js
  Server running at http://127.0.0.1:8888/
```
接下来，打开浏览器访问 `http://127.0.0.1:8888/`，你会看到一个写着 "Hello World" 的网页。