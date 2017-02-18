---
title: Node.js模块系统(6)
date: 2016-06-28 11:32:07
tags: node
categories: nodeJS学习笔记
---

### 6、Node.js模块系统
Node.js 提供了`exports` 和 `require` 两个对象，其中 `exports` 是模块公开的接口，`require` 用于从外部获取一个模块的接口，即所获取模块的 `exports` 对象。
<!-- more -->
实例：
``` javascript
  //hello.js
  function Hello(){
    var name;
    this.setName = function(thyName){
      name = thyName;
    };
    this.sayHello = function(){
      console.log('Hello '+ name);
    };
  };
  module.exports = Hello;
  //main.js
  var Hello = require('./hello');
  hello = new Hello();
  hello.setName('BYVoid');
  hello.sayHello();
```
#### 服务端的模块放在哪里
我们已经在代码中使用了模块了。像这样：
``` javascript
  var http = require("http");
  ...
  http.createServer(...);
```
Node.js中自带了一个叫做"http"的模块，我们在我们的代码中请求它并把返回值赋给一个本地变量。这把我们的本地变量变成了一个拥有所有 `http` 模块所提供的公共方法的对象。
Node.js 的 `require`方法中的文件查找策略如下：
```
开始require-->
  if (在文件模块缓存区中) {
    返回exports.
  } else{
    if (是原生模块) {
      if (在原生模块缓存区中) {
        返回exports.
      } else{
        加载原生模块-->缓存原生模块-->返回exports.
      };
    } else{
      查找文件模块-->根据扩展名载入文件模块-->缓存文件模块-->返回exports.
    };
  };
```
#### 从文件模块缓存中加载
尽管原生模块与文件模块的优先级不同，但是都不会优先于从文件模块的缓存中加载已经存在的模块。
#### 从原生模块加载 */
原生模块的优先级仅次于文件模块缓存的优先级。`require`方法在解析文件名之后，优先检查模块是否在原生模块列表中。以http模块为例，尽管在目录下存在一个`http/http.js/http.node/http.json`文件，`require("http")`都不会从这些文件中加载，而是从原生模块中加载。原生模块也有一个缓存区，同样也是优先从缓存区加载。如果缓存区没有被加载过，则调用原生模块的加载方式进行加载和执行。
#### 从文件加载 */
当文件模块缓存中不存在，而且不是原生模块的时候，`Node.js`会解析`require`方法传入的参数，并从文件系统中加载实际的文件，加载过程中的包装和编译细节在前一节中已经介绍过，这里我们将详细描述查找文件模块的过程，其中，也有一些细节值得知晓。
`require`方法接受以下几种参数的传递:
  `http、fs、path`等，原生模块。
  `./mod`或`../mod`，相对路径的文件模块。
  `/pathtomodule/mod`，绝对路径的文件模块。
  `mod`，非原生模块的文件模块。