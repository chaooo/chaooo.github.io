---
title: Node.js Stream流(5)
date: 2016-06-28 9:50:03
tags: node
categories: nodeJS学习笔记
---


### 5、Node.js Stream(流)
`Stream` 是一个抽象接口，`Node` 中有很多对象实现了这个接口。例如，对`http` 服务器发起请求的`request` 对象就是一个 `Stream`，还有`stdout`（标准输出）。
Stream有四种流类型：
  `Readable` //可读操作。
  `Writable` //可写操作。
  `Duplex`   //可读写操作。
  `Transform`//操作被写入数据，然后读出数据。
所有的`Stream`对象都是`EventEmitter`的实例。
<!-- more -->
常用事件：
      `data`  //当有数据可读时触发。
      `end  ` //没有更多数据可读时触发。
      `error` //在接收和写入过程中发生错误时触发。
      `finish`//所有数据已被写入到底层系统时触发。
#### 从流中读取数据实例
创建input.txt文件，内容自定。
创建main.js文件：
``` javascript
  var fs = require("fs");
  var data = '';
  //创建可读流
  var readerStream = fs.createReadStream('input.txt');
  //设置编码为 utf8。
  readerStream.setEncoding('UTF8');
  //处理流事件-->data,end,and errror
  readerStream.on('data',function(chunk){
    data += chunk;
  });
  readerStream.on('end',function(){
    console.log(data);
  });
  readerStream.on('error',function(err){
    console.log(err.stack);
  });
  console.log("程序执行完毕");
```
#### 写入流实例
创建main.js文件：
``` javascript
  var fs = require("fs");
  var data = '我是被写入的数据';
  //创建一个可以写入的流，写入到output.txt中
  var writerStream = fs.createWriteStream('output.txt');
  //使用utf8编码写入数据
  writerStream.write(data,'UTF8');
  //标记文件末尾
  writerStream.end();
  //处理流事件-->finish, errror
  readerStream.on('finish',function(){
    console.log("写入完成。");
  });
  readerStream.on('error',function(err){
    console.log(err.stack);
  });
  console.log("程序执行完毕");
```
####　管道流实例
创建input.txt文件，内容自定。
创建main.js文件：
``` javascript
  var fs = require("fs");
  //创建一个可读流
  var readerStream = fs.createReadStream('input.txt');
  //创建一个可写流
  var writerStream = fs.createWriteStream('output.txt');
  //管道读写操作，读取input.txt内容，并写入到output.txt文件中。
  readerStream.pipe(writerStream);
  console.log("程序执行完毕");
```
####　链式流实例
创建compress.js文件：
``` javascript
  var fs = require("fs");
  var zlib = require('zlib');
  //压缩input.txt文件为input.txt.gz
  fs.createReadStream('input.txt').pipe(zlib.createGzip()).pipe(fs.createWriteStream('input.txt.gz'));
  console.log("文件压缩完成。");
  //执行完以上操作后，我们可以看到当前目录下生成了 input.txt 的压缩文件 input.txt.gz。接下来，让我们来解压该文件
  //创建 decompress.js 文件:
  var fs = require("fs");
  var zlib = require('zlib');
  //解压input.txt.gz文件为input.txt
  fs.createReadStream('input.txt.gz').pipe(zlib.createGunzip()).pipe(fs.createWriteStream('input.txt'));
  console.log("文件解压完成。"); 
```
