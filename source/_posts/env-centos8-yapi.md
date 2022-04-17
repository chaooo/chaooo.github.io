---
title: 「环境配置」Centos8安装部署Node+MongDB+YApi(接口管理)
date: 2020-07-15 18:11:23
tags: [环境配置, CentOS, YApi]
categories: 环境配置
---

YApi:权限管理、Mock服务、可视化接口管理、数据导入（支持postman），其依赖NodeJS+MongDB。<!-- more -->

### 1. 安装NodeJS

1. 下载NodeJS稳定版14.15.1

``` shell
[root@localhost ~]# mkdir -p /local/node-server
[root@localhost ~]# cd /local/node-server
[root@localhost node-server]# wget https://nodejs.org/dist/v14.15.1/node-v14.15.1-linux-x64.tar.xz
```

2. 解压下载好的node包到安装目录下

``` shell
[root@localhost node-server]# tar xvf node-v14.15.1-linux-x64.tar.xz
```

3. `node -v`查看版本号

``` shell
[root@localhost node-server]# node -v
bash: node: 未找到命令...
[root@localhost node-server]# /local/node-server/node-v14.15.1-linux-x64/bin/node -v
v14.15.1
```

4. 创建软链接，就可以全局使用node和npm命令

``` shell
[root@localhost node-server]# ln -s /local/node-server/node-v14.15.1-linux-x64/bin/node /usr/local/bin/node
[root@localhost node-server]# ln -s /local/node-server/node-v14.15.1-linux-x64/bin/npm /usr/local/bin/npm
[root@localhost node-server]# node -v
v14.15.1
[root@localhost node-server]# npm -v
6.14.8
```


### 2. 安装MongDB
1. 创建yum源文件

``` shell
[root@localhost ~]# vim /etc/yum.repos.d/mongodb-org-4.2.repo
# 输入如下内容
[mongodb-org-4.2]  
name=MongoDB Repository  
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/  
gpgcheck=1  
enabled=1  
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

2. 安装mongodb

``` shell
[root@localhost ~]# yum -y install mongodb-org
```

3. 查看安装目录

``` shell
[root@localhost ~]# whereis mongod
mongod: /usr/bin/mongod /etc/mongod.conf /usr/share/man/man1/mongod.1
```

4. 编辑配置文件`/etc/mongod.conf`(根据自己需要进行修改bindip地址，可监听127.0.0.1或内网地址。如果需要绑定多个ip )

``` shell
[root@localhost ~]# vim /etc/mongod.conf
bindIp: 127.0.0.1,192.168.2.101
```

4. 启动Mongodb

``` shell
# 启动mongodb
systemctl start mongod.service

# 停止mongodb
systemctl stop mongod.service

# 查询 mongodb 状态：
systemctl status mongod.service

# 设置为开机启动
systemctl enable mongod.service
```

如果在不同服务器下访问或者修改端口需要配置防火墙或者阿里云服务器安全组件 默认为27017 如修改可在/etc/mongod.conf下修改端口，
到此安装完成。

5. 启动 mongo shell

``` shell
[root@localhost ~]# mongo
MongoDB shell version v4.2.11
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
# 其他信息省略...

> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> exit
bye
```


### 3. 安装YApi
1. 开始安装

``` shell
mkdir yapi && cd yapi
wget http://registry.npm.taobao.org/yapi-vendor/download/yapi-vendor-1.9.2.tgz
tar -zxvf yapi-vendor-1.9.2.tgz
```

2. 拷贝依赖package至vendors

``` shell
[root@localhost yapi]# ll
drwxr-xr-x. 9 root root    4096 11月 20 16:18 package
-rw-r--r--. 1 root root 9403779 5月  29 14:14 yapi-vendor-1.9.2.tgz
[root@localhost yapi]# mv package vendors
[root@localhost yapi]# ll
drwxr-xr-x. 9 root root    4096 11月 20 16:18 vendors
-rw-r--r--. 1 root root 9403779 5月  29 14:14 yapi-vendor-1.9.2.tgz
```

3. 安装依赖

``` shell
[root@localhost yapi]# cd vendors/
[root@localhost vendors]# npm install --production --registry https://registry.npm.taobao.org
```

4. 拷贝配置并修改(MongoDB地址、端口、用户)

``` shell
[root@localhost vendors]# cd ../
[root@localhost yapi]# cp vendors/config_example.json ./config.json
[root@localhost yapi]# vim config.json
```

5. 安装服务

``` shell
[root@localhost yapi]# cd vendors/
[root@localhost vendors]# npm run install-server
初始化管理员账号成功,账号名："admin@admin.com"，密码："ymfe.org"
```

6. 启动

``` shell
[root@localhost vendors]# node server/app.js
log: -------------------------------------swaggerSyncUtils constructor-----------------------------------------------
log: 服务已启动，请打开下面链接访问: 
http://127.0.0.1:3000/
```

7. 安装pm管理服务启动

``` shell
[root@localhost vendors]# npm install pm2 -g
# 配置服务启动项
[root@localhost vendors]# pm2 start "/local/yapi/vendors/server/app.js" --name yapi
# 查看服务信息
[root@localhost vendors]# pm2 info yapi
# 停止服务
[root@localhost vendors]# pm2 stop yapi
# 重启服务
[root@localhost vendors]# pm2 restart yapi
```

> 若找不到pm2命令，可先把node安装路径/bin添加到环境变量。

8. 设置开机启动

运行 pm2 startup，即在/etc/init.d/目录下生成pm2-root的启动脚本，且自动将pm2-root设为服务。
``` shell
pm2 startup
```

运行 pm2 save，会将当前pm2所运行的应用保存在/root/.pm2/dump.pm2下，当开机重启时，运行pm2-root服务脚本，并且到/root/.pm2/dump.pm2下读取应用并启动。
``` shell
pm2 save
```

9. 还可以配合IDEA插件`Api Generator`使用
    + Preferences → Plugins → Marketplace → 搜索“Api Generator” → 安装该插件 → 重启IDE
    + [Api Generator使用教程](http://forgus.vicp.io/2019/10/28/Api_Generator_introduction/)
