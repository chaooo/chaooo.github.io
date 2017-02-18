---
title: NodeJs学习笔记(2)
date: 2016-06-27 12:55:00
tags: node
categories: nodeJS学习笔记
---

### 2、Node.js 事件循环

Node.js 有多个内置的事件，我们可以通过引入 `events` 模块，并通过实例化 `EventEmitter` 类来绑定和监听事件，如下实例：
<!-- more -->
``` javascript
  //引入events模块
  var events = require('events');
  //创建eventEmitter对象
  var eventEmitter = new events.EventEmitter();
  //创建时间处理程序
  var connectHander = function connected() {
    console.log('连接成功。');
    //触发data_received事件
    eventEmitter.emit('data_received');
  }
  //绑定connection事件处理程序
  eventEmitter.on('connection', connectHandler);
  //使用匿名函数绑定data_received事件
  eventEmitter.on('data_received', function(){
    console.log('数据接收成功。');
  });
  //触发connection事件
  eventEmitter.emit('connection');
  console.log("程序执行完毕。");
```

##### node应用程序如何工作

创建一个input.txt文件，内容如下：
``` 
  Hello World;
```
创建main.js文件，代码如下：
``` javascript
  var fs = require("fs");
  fs.readFile('input.txt', function (err, data) {
    if (err) {
      console.log(err.stack);
    }
    console.log(data.toString());
  });
  console.log("程序执行完毕");
```
以上程序中 `fs.readFile()` 是异步函数用于读取文件。 如果在读取文件过程中发生错误，错误 err 对象就会输出错误信息。如果没发生错误，readFile 跳过 err 对象的输出，文件内容就通过回调函数输出。执行以上代码，执行结果如下：
``` 
  Hello World;
```
接下来我们删除 input.txt 文件，执行结果如下所示：
``` 
  程序执行完毕
  Error: ENOENT, open 'input.txt'
```