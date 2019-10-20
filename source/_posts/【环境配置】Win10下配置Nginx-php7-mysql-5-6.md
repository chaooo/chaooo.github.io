---
layout: '[post]'
title: 【环境配置】Win10下配置Nginx+PHP-7+MySQL-5.6
date: 2018-10-22 11:39:12
tags: 环境配置
categories: 环境配置
---


### 1. 软件下载
  - Windows操作系统。
  - Nginx，下载地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)。
  - PHP，下载地址：[http://php.net/downloads.php](http://php.net/downloads.php)（nginx下php是以FastCGI的方式运行，所以我们下载非线程安全也就是nts的php包）。
  - MySQL，下载地址：[https://www.mysql.com/downloads/](https://www.mysql.com/downloads/)。（选择社区版`Community`->`MySQL Community Server`->`MySQL Community Server 5.6`，根据Windows系统选择对应zip包）。
<!-- more -->
### 2. 软件安装
在C盘新建安装目录`C:\PHP`。
#### 2.1 Nginx安装
  Nginx本身就是绿色软件，下载zip安装包解压到`C:\PHP`，打开目录`C:\PHP\nginx-1.15.8`双击nginx.exe就可以运行，然后在浏览器打开[http://127.0.0.1](http://127.0.0.1)，出现欢迎界面表示NGINX正常工作。
  确认NGINX正常工作后在任务管理器中结束nginx.exe任务。
#### 2.2 PHP安装
  把PHP的zip安装包解压到`C:\PHP`，解压后PHP安装目录为：`C:\PHP\php-7.3.2`。
  cmd进行到安装目录，输入php.exe -v,正常会显示版本信息。
  将`C:\PHP\php-7.3.2`加入系统环境变量。
#### 2.3 准备网站根目录
  准备一个文件夹，作为网站的根目录，这个在下面的配置文件中会多次用到，我把`C:\PHP\web`作为我的网站根目录。
  在根目录`C:\PHP\web`下新建一个info.php文件，输入如下内容：
    ``` php
    <?php
        phpinfo();
    ?>
    ```
#### 2.4 让nginx识别PHP
  配置PHP (`C:\PHP\php-7.3.2`)
  在PHP根目录下找到php.ini-development文件，编辑器打开nginx.conf:
  在PHP根目录下修改配置文件`C:\PHP\php-7.3.2\php.ini-development`并另存为`php.ini`,在其中修改或添加配置：
  ```
  cgi.fix_pathinfo=1
  ```
  配置nginx conf(`C:\PHP\nginx-1.15.8\conf`)
  在Nginx根目录下找到conf目录，编辑器打开`C:\PHP\nginx-1.15.8\confnginx.conf`:
    ```
    error_log  logs/error.log; #打开error_log
    http {

        # ...

        server {

            # ...

            location / {
                root     C:\PHP\web; #配置根目录
                index   index.html index.htm index.php;
            }

            # ...

            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            # 打开下面几行注释
            location ~ \.php$ {
                root             C:\PHP\web; #配置根目录
                fastcgi_pass     127.0.0.1:9000;
                fastcgi_index    index.php;
                #重要: 把下面 /scripts 修改成 $document_root
                fastcgi_param    SCRIPT_FILENAME  $document_root$fastcgi_script_name; 
                include          fastcgi_params;
            }

            # ...
        }
    }
    ```
#### 2.5 运行与测试
nginx是一个反向代理的web服务器，因此它其实必须依赖一个真正的web服务器才能执行动态的网页内容，因此这里php就是使用fastcgi来充当这个真正的web服务器，它运行在9000端口上，这也是为什么`nginx.conf`中有这样一句`fastcgi_pass 127.0.0.1:9000;`。
1. 在任务管理器中结束nginx.exe任务，然后到`C:\PHP\nginx-1.15.8`目录双击nginx.exe开启服务。
2. 在命令行中，cd到php的home目录`C:\PHP\php-7.3.2`，然后执行如下命令：
  ``` cmd
  php-cgi.exe -b 127.0.0.1:9000 -c php.ini
  ```
3. 打开浏览器，输入 [http://127.0.0.1/info.php](http://127.0.0.1/info.php)，这时候可以看到phpinfo页面：页面内容包含了PHP 当前状态的大量信息，包含了 PHP 编译选项、启用的扩展、PHP 版本、服务器信息和环境变量（如果编译为一个模块的话）、PHP环境变量、操作系统版本信息、path 变量、配置选项的本地值和主值、HTTP 头和PHP授权信息(License)。

#### 2.6 MySQL安装
  把MySQL的zip安装包解压到`C:\PHP`，解压后PHP安装目录为：`C:\PHP\mysql-5.6.43-winx64`。
  将`C:\PHP\mysql-5.6.43-winx64\bin`加入系统环境变量。
  修改配置文件`C:\PHP\mysql-5.6.43-winx64\my-default.ini`并另存为`my.ini`,在其中修改或添加配置 （my.ini文件的编码必须是英文编码（如windows中的ANSI），不能是UTF-8或GBK等）：
    ```
    basedir=C:\PHP\mysql-5.6.43-winx64       #mysql所在目录
    datadir=C:\PHP\mysql-5.6.43-winx64\data  #mysql所在目录\data
    ```
  以管理员身份运行cmd,到安装目录的bin下，输入`mysqld -install`：
    ```
    C:\PHP\mysql-5.6.43-winx64\bin> mysqld -install
    Service successfully installed.
    ```
  输入命令:`mysql --version`,正常会显示版本信息。
  输入命令:`net start mysql`启动服务(停止命令：net stop mysql):
    ```
    C:\PHP\mysql-5.6.43-winx64\bin>net start mysql
    MySQL 服务正在启动 ..
    MySQL 服务已经启动成功。
    ```
  服务启动成功之后，输入命令：`mysql -u root -p`（第一次登录没有密码，直接按回车过）:
    ```
    C:\PHP\mysql-5.6.43-winx64\bin>mysql -u root -p
    Enter password:
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 1
    Server version: 5.6.43 MySQL Community Server (GPL)
    Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    mysql>
    ```
  如出现`mysql>`,即登录成功。
  输入命令`exit`,退出登录。
    ```
    mysql> exit
    Bye
    C:\PHP\mysql-5.6.43-winx64\bin>
    ```
### 3. 制作自动启动脚本
控制台就一直开着，很不方便。这个时候可以使用 [RunHiddenConsole.zip](https://link.jianshu.com?t=http://www.inbeijing.org/wp-content/uploads/2015/06/RunHiddenConsole.zip) 来得管理服务的启动与关闭。
#### 3.1 启动脚本
在目录`C:\PHP`下新建一个`start.bat`作为启动脚本文件：
``` bat
:启动脚本
@echo off
set php_home=./php-7.3.2
set nginx_home=./nginx-1.15.8

REM Windows 下无效
REM set PHP_FCGI_CHILDREN=5

REM 每个进程处理的最大请求数，或设置为 Windows 环境变量
set PHP_FCGI_MAX_REQUESTS=1000
echo Starting PHP FastCGI...
RunHiddenConsole %php_home%/php-cgi.exe -b 127.0.0.1:9000 -c %php_home%/php.ini
echo FastCGI 启动成功
echo.
echo Starting nginx...
RunHiddenConsole %nginx_home%/nginx.exe -p %nginx_home%
echo nginx 启动成功
echo.
:echo 15秒后自动退出
:ping 0.0.0.0  -n 15 > null
:请按任意键继续. . .
pause
```
#### 3.2 停止脚本
在目录`C:\PHP`下新建一个`stop.bat`作为停止脚本文件：
``` bat
:停止脚本
@echo off
echo Stopping nginx...  
taskkill /F /IM nginx.exe > nul
echo nginx 已停止
:换行
echo.
echo Stopping PHP FastCGI...
taskkill /F /IM php-cgi.exe > nul
echo FastCGI 已停止
:请按任意键继续. . .
pause
```
#### 3.3 重启脚本
在目录`C:\PHP`下新建一个`restart.bat`作为重启脚本文件：
``` bat
:停止脚本
@echo off
echo Stopping nginx...  
taskkill /F /IM nginx.exe > nul
echo nginx 已停止
:换行
echo.
echo Stopping PHP FastCGI...
taskkill /F /IM php-cgi.exe > nul
echo FastCGI 已停止
echo.

:启动脚本
@echo off
set php_home=./php-7.3.2
set nginx_home=./nginx-1.15.8

REM Windows 下无效
REM set PHP_FCGI_CHILDREN=5

REM 每个进程处理的最大请求数，或设置为 Windows 环境变量
set PHP_FCGI_MAX_REQUESTS=1000
echo Starting PHP FastCGI...
RunHiddenConsole %php_home%/php-cgi.exe -b 127.0.0.1:9000 -c %php_home%/php.ini
echo FastCGI 启动成功
echo.
echo Starting nginx...
RunHiddenConsole %nginx_home%/nginx.exe -p %nginx_home%
echo nginx 启动成功
echo.
:echo 15秒后自动退出
:ping 0.0.0.0  -n 15 > null
:请按任意键继续. . .
pause
```
### 4.最后
我的根目录结构
``` cmd
C:\PHP>dir
 驱动器 C 中的卷是 系统
 卷的序列号是 09C1-B27D

 C:\PHP 的目录

2019/02/22  15:46    <DIR>          .
2019/02/22  15:46    <DIR>          ..
2019/02/22  11:23    <DIR>          mysql-5.6.43-winx64
2018/12/25  17:54    <DIR>          nginx-1.15.8
2019/02/21  15:59    <DIR>          php-7.3.2
2019/02/22  15:41               758 restart.bat
2010/10/26  11:43             1,536 RunHiddenConsole.exe
2019/02/22  15:41               549 start.bat
2019/02/22  15:41               227 stop.bat
2019/02/21  16:56    <DIR>          web
               4 个文件          3,070 字节
               6 个目录 100,959,772,672 可用字节
```
