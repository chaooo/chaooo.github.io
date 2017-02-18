---
title: 好用的Web包管理器-Bower
date: 2016-08-12 15:32:41
tags: bower
categories: 前端工具
---

Bower是twitter推出的客户端包管理工具，用于命令行操作包的搜索、下载、更新、卸载(如jQuery、Bootstrap、JavaScript、HTML、CSS之类的网络资源)。Bower对包结构没有强制规范，可以很方便获取各种Web模块文件，但bower本身不存储模块文件和模块版本信息，模块发布者通过register方式将模块可访问的公开的git地址记录在bower的数据库中，而所有版本都是通过代码库的tag来决定的。
<!-- more --> 
### 开始之前

在安装[bower](https://bower.io/)之前，必须确认你已经安装了[Node.js](http://nodejs.org/)和[Git](http://git-scm.com/)。

### 1.安装Bower
使用npm，打开终端，键入：
``` bash
npm install -g bower   #全局安装bower
```
移步[这里](https://github.com/bower/bower/wiki/Troubleshooting)查看不同平台上安装的问题。

### 2.使用Bower
使用help命令查看帮助。   
``` bash
bower help

Usage:
    bower <command> [<args>] [<options>]
Commands:
    cache                   Manage bower cache
    help                    Display help information about Bower
    home                    Opens a package homepage into your favorite browser
    info                    Info of a particular package
    init                    Interactively create a bower.json file
    install                 Install a package locally
    link                    Symlink a package folder
    list                    List local packages - and possible updates
    login                   Authenticate with GitHub and store credentials
    lookup                  Look up a package URL by name
    prune                   Removes local extraneous packages
    register                Register a package
    search                  Search for a package by name
    update                  Update a local package
    uninstall               Remove a local package
    unregister              Remove a package from the registry
    version                 Bump a package version
Options:
    -f, --force             Makes various commands more forceful
    -j, --json              Output consumable JSON
    -l, --loglevel          What level of logs to report
    -o, --offline           Do not hit the network
    -q, --quiet             Only output important information
    -s, --silent            Do not output anything, besides errors
    -V, --verbose           Makes output more verbose
    --allow-root            Allows running commands as root
    -v, --version           Output Bower version
    --no-color              Disable colors
See 'bower help <command>' for more information on a specific command.
```
### 3.安装包到本地
通过命令bower install安装软件包默认到bower_components/目录。
``` bash
bower install <package>    #package为包名
```
想要下载的包可以是GitHub上的短链接（如jquery/jquery）、.git 、一个URL或者其它.
``` bash
bower install  # 通过 bower.json 文件安装
bower install jquery   # 通过在github上注册的包名安装
bower install desandro/masonry   # GitHub短链接
bower install git://github.com/user/package.git   # Github上的 .git
bower install http://example.com/script.js   # URL
```
安装选项
``` bash
    -F, --force-latest: Force latest version on conflict
    -p, --production: Do not install project devDependencies
    -S, --save: Save installed packages into the project’s bower.json dependencies
    -D, --save-dev: Save installed packages into the project’s bower.json devDependencies
    -E, --save-exact: Configure installed packages with an exact version rather than semver
```

### 4.用bower.json文件来管理依赖
发布项目的时候没有必要把所有依赖的库发布上去，只需在根目录生成一个bower.json文件即可，别人使用时在根目录执行`bower install`就可根据bower.json来安装依赖的包。
在项目中执行
``` bash
bower init
```
会提示你输入一些基本信息，根据提示按回车或者空格即可，然后会生成一个bower.json文件，用来保存该项目的配置.
如果想保存依赖信息(dependencies)到你的bower.json文件，安装包时，命令后面跟上`--save`即可。

### 5.使用下载好的包
对于已经下载下来的包，默认在当前目录的bower_components文件夹。你可以直接在项目里引用。例如：
``` html
<link rel="stylesheet" href="bower_components/bootstrap/dist/css/bootstrap.min.css">
<script type="text/javascript" src="bower_components/jquery/dist/jquery.min.js"></script>
```

### 6.更新包
若下载的包升级了，只需执行`update`命令即可更新，例如：
``` bash 
bower update jquery
```
这样就可以自动升级到最新版的jquery了。
更新选项
``` bash
    -F, --force-latest: Force latest version on conflict
    -p, --production: Do not install project devDependencies
    -S, --save: Update dependencies in bower.json
    -D, --save-dev: Update devDependencies in bower.json
```

### 7.搜索包
``` bash
bower search               #搜索所有包
bower search <packageName> #搜索指定名称的包
```
或者可以在[这里:https://bower.io/search/](https://bower.io/search/)搜索喜欢的包.

### 8.卸载包
``` bash
bower uninstall <name> [<name> ..] [<options>]
```
卸载选项
``` bash
    -S, --save: Remove uninstalled packages from the project’s bower.json dependencies
    -D, --save-dev: Remove uninstalled packages from the project’s bower.json devDependencies
```
