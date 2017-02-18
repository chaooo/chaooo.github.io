---
title: Node.js GET/POST请求(12)
date: 2016-06-29 15:38:20
tags: node
categories: nodeJS学习笔记
---

### 12、Node.js GET/POST请求
#### 获取GET请求内容
由于GET请求直接被嵌入在路径中，URL是完整的请求路径，包括了?后面的部分，因此你可以手动解析后面的内容作为GET请求的参数。node.js中url模块中的parse函数提供了这个功能。
``` javascript
  var http = require('http');
  var url = require('url');
  var util = require('util');
  http.createServer(function(req, res){
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end(util.inspect(url.parse(req.url, true)));
  }).listen(3000);
```
在浏览器中访问`http://localhost:3000/user?name=w3c&email=w3c@w3cschool.cc` 然后查看返回结果: 
<!-- more -->
``` javascript
  Url {
    protocol: null,
    slashes: null,
    auth: null,
    host: null,
    port: null,
    hostname: null,
    hash: null,
    search: '?name=w3c&email=w3c@w3cschool.cc',
    query: { name: 'w3c', email: 'w3c@w3cschool.cc' },
    pathname: '/user',
    path: '/user?name=w3c&email=w3c@w3cschool.cc',
    href: '/user?name=w3c&email=w3c@w3cschool.cc' }
```
#### 获取POST请求内容
POST请求的内容全部的都在请求体中，http.ServerRequest并没有一个属性内容为请求体，原因是等待请求体传输可能是一件耗时的工作。比如上传文件，而很多时候我们可能并不需要理会请求体的内容，恶意的POST请求会大大消耗服务器的资源，所有node.js默认是不会解析请求体的， 当你需要的时候，需要手动来做。
``` javascript
  var http = require('http');
  var querystring = require('querystring');
  var util = require('util');
  http.createServer(function(req, res){
    var post = '';//定义了一个post变量，用于暂存请求体的信息
    req.on('data',function(chunk){//通过req的data事件监听函数，每当接受到请求体的数据，就累加到post变量中
      post += chunk;
    });
    req.on('end',function(){//在end事件触发后，通过querystring.parse将post解析为真正的POST请求格式，然后向客户端返回。
      post = querystring.parse(post);
      res.end(util.inspect(post));
    });
  }).listen(3000);
```