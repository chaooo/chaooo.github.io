---
title: sublime text3个人习惯配置
date: 2016-06-29 09:41:48
tags: sublime
categories: 前端工具
---

### 1、安装

分别在官网下载并安装 [nodejs](https://nodejs.org/en/download/) 和 [sublime text3](https://www.sublimetext.com/3)。

### 2、sublime text3注册：
<!-- more --> 
点击菜单【help】－>【Enter License】，粘贴下面注册码(亲测v3103可用 ):
``` bash
—– BEGIN LICENSE —–
Ryan Clark
Single User License
EA7E-812479
2158A7DE B690A7A3 8EC04710 006A5EEB
34E77CA3 9C82C81F 0DB6371B 79704E6F
93F36655 B031503A 03257CCC 01B20F60
D304FA8D B1B4F0AF 8A76C7BA 0FA94D55
56D46BCE 5237A341 CD837F30 4D60772D
349B1179 A996F826 90CDB73C 24D41245
FD032C30 AD5E7241 4EAA66ED 167D91FB
55896B16 EA125C81 F550AF6B A6820916
—— END LICENSE ——
```

### 3、安装package control组件，用于管理所有插件

按ctrl + ~调出控制台(或点击菜单栏的【View】->【Show Console】)，在Console窗口中输入以下代码，按回车键：
``` bash
import urllib.request,os,hashlib; h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
等待安装完毕，重启Sublime Text3。
按快捷键：Ctrl+Shift+P，调出界面，在其中输入：install，第一个选项即是Package Control：

### 4、用Package Control安装插件

按快捷键：Ctrl+Shift+P，调出界面，按照个人习惯安装插件（以下是我使用的插件）
`Material Theme`
`Emmet`
`CSS Format`
`CSScomb`
`jsFormat`
`AutoFileName`
`Autoprefixer`
`Doc Blockr`
`SublimeLinter`
`SublimeLinter-jshint`
`SublimeLinter-csslint`
`Color Highlighter`
`BracketHighlighter`

### 5、配置nodejs

##### 方法1
(1)下载sublime Text的[nodejs插件](https://github.com/tanepiper/SublimeText-Nodejs)
(2)下载后解压:直接改名为nodejs 放到 Preferences–>浏览程序包Browse Packages所在的文件夹
(3)修改配置:打开Nodejs文件夹，找到文件“Nodejs.sublime-build”， 拖拽到sublime，显示：
``` json
{
  "cmd": ["node", "$file"],
  "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
  "selector": "source.js",
  "shell":true,
  "encoding": "cp1252", 
  "windows": 
    { 
        "cmd": ["taskkill /F /IM node.exe & node", "$file"] 
    },
  "linux":
    {
        "cmd": ["killall node; node", "$file"]
    },
    "osx":
    {
  "cmd": ["killall node; node $file"]
    }
}
```
(4)修改为：
``` json
{
  "cmd": ["node", "$file"],
  "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
  "selector": "source.js",
  "shell":true,
  "encoding": "utf-8",
  "windows":
    {
      "cmd": ["taskkill /F /IM node.exe", ""],
      "cmd": ["node", "$file"]
    },
  "linux":
    {
        "cmd": ["killall node; node", "$file"]
    },
    "osx":
    {
  "cmd": ["killall node; node $file"]
    }
}
```
(5)完成:随便写一段nodejs代码，ctrl+B运行
(6)注意：在手动解压sublime Text插件后，需要在preference->package settings->package control的user setting下添加installed packages中的“Nodejs”，不然重启sublime Text 会被删除Nodejs插件。

##### 方法2
首先需要先安装[nodejs](https://nodejs.org/en/download/)。
(1)运行Sublime,菜单上找到Tools ---> Build System ---> new Build System
(2)输入：
{
  "cmd": ["node", "$file"],
  "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
  "selector": "source.js",
  "shell":true,
  "encoding": "utf-8",
  "windows":
    {
      "cmd": ["taskkill /F /IM node.exe", ""],
      "cmd": ["node", "$file"]
    }
}
(3)保存文件为NodeJs.sublime-build
(4)菜单上找到Tools ---> Build System --->选择 NodeJs
(5)安装sublime插件 JavaScript & NodeJs Snippets
(6)新建test.js文件，输入 console.log('Hello Node.js'); 按快捷键 Ctrl + B 运行，成功输出