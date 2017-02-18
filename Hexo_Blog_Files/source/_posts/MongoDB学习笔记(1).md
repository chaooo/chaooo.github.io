---
title: MongoDB学习笔记
date: 2016-07-29 10:22:14
tags: mongodb
categories: 数据库
---

## part1 安装配置

### 一、安装：

在mongodb官网下载对应自己电脑系统的安装包，地址为： [http://www.mongodb.org/downloads](http://www.mongodb.org/downloads)。
<!-- more --> 
1、以Windows64bit为例，下载.msi文件双击安装。
2、安装过程中，点击 "Custom(自定义)" 按钮来设置安装目录(D:\MongoDB\bin)。
3、创建数据目录(D:\MongoDB\data\db),MongoDB默认数据目录\data\db。
4、连接数据库(命令行win+r cmd,到D:\MongoDB\bin目录下，执行代码：mongod --dbpath D:\MongoDB\data\db)
``` bash
  D:
  cd D:\MongoDB\bin
  mongod --dbpath D:\MongoDB\data\db
```
5、启动 MongoDB JavaScript 工具(D:\MongoDB\bin目录下,打开mongo,会看到：)
``` bash
  MongoDB shell version: 3.2.4  //mongodb版本
  connecting to: test  //默认shell连接的是本机localhost 上面的test库
```
此时就可以操作数据库了。

### 二、将MongoDB服务器作为Windows服务运行

1、在D:\MongoDB目录下创建mongodb.config,写入如下：
``` bash
  ## 数据库文件目录
  dbpath=D:/MongoDB/data
  ## 日志目录
  logpath=D:/MongoDB/log/mongo.log
  diaglog=3
```
2、常规命令(cmd管理员):
```
  D:
  cd D:\MongoDB\bin
  mongod --config D:\MongoDB\mongodb.config 
```
3、若常规方式失败，则sc方式(cmd管理员)：
```
  D:
  cd D:\MongoDB\bin
  sc create mongodb binPath= "D:\MongoDB\bin\mongod.exe --service --config=D:\mongoDB\mongodb.config" 
```
访问地址：localhost:27017测试是否启动成功