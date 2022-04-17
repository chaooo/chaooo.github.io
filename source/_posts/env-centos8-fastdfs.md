---
title: 「环境配置」Centos8 安装 FastDFS 6.06
date: 2020-08-02 18:11:23
tags: [环境配置, CentOS, FastDFS]
categories: 环境配置
---


FastDFS是一款开源的分布式文件系统，功能主要包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了文件大容量存储和高性能访问的问题。FastDFS特别适合以文件为载体的在线服务，如图片、视频、文档等等。<!-- more -->

FastDFS为互联网应用量身定做，解决大容量文件存储问题，追求高性能和高扩展性。FastDFS可以看做是基于文件的key value存储系统，key为文件ID，value为文件内容，因此称作分布式文件存储服务更为合适。

* FastDFS特点如下： 
    1. 分组存储，简单灵活；
    2. 对等结构，不存在单点；
    3. 文件ID由FastDFS生成，作为文件访问凭证。FastDFS不需要传统的name server或meta server；
    4. 大、中、小文件均可以很好支持，可以存储海量小文件；
    5. 一台storage支持多块磁盘，支持单盘数据恢复；
    6. 提供了nginx扩展模块，可以和nginx无缝衔接；
    7. 支持多线程方式上传和下载文件，支持断点续传；
    8. 存储服务器上可以保存文件附加属性。

[【官方GitHub地址】](https://github.com/happyfish100/fastdfs)


### 1. 准备安装文件
* 安装文件：fastdfs,libfastcommon,fastdfs-nginx-module
* 文件存放位置：/usr/local/src

``` shell
cd /usr/local/src
wget -c "https://github.com/happyfish100/fastdfs/archive/V6.06.tar.gz" -O fastdfs-6.06.tar.gz
wget -c "https://github.com/happyfish100/libfastcommon/archive/V1.0.43.tar.gz" -O libfastcommon-1.0.43.tar.gz
wget -c "https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.22.tar.gz" -O fastdfs-nginx-module-1.22.tar.gz
wget -c http://nginx.org/download/nginx-1.18.0.tar.gz
```


### 2. 编译安装

准备编译环境
``` shell
yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
tar -zxvf libfastcommon-1.0.43.tar.gz
tar -zxvf fastdfs-6.06.tar.gz
tar -zxvf fastdfs-nginx-module-1.22.tar.gz
```

#### 2.1 编译安装libfatscommon
``` shell
cd /usr/local/src/libfastcommon-1.0.43
./make.sh && ./make.sh install
```

检查(出现libfastcommon.so即成功)
``` shell
ls /usr/lib64|grep libfastcommon
ls /usr/lib|grep libfastcommon
```


#### 2.2 编译安装fastdfs
``` shell
cd /usr/local/src/fastdfs-6.06
./make.sh && ./make.sh install
```

检查
``` shell
ls /usr/bin|grep fdfs
```


### 3. 配置Tracker(FastFDS跟踪器)
1. 启用并修改配置文件

``` shell
cd /etc/fdfs/
cp tracker.conf.sample tracker.conf
vim /etc/fdfs/tracker.conf

# 配置文件生效
disabled = false
# 提供服务端口
port = 22122
# Tracker 数据和日志目录地址（根目录必须存在，子目录会自动创建）
base_path = /fastdfs/tracker
# HTTP 服务端口 默认8080，建议修改防止冲突
http.server_port = 8080
```

3. 创建Tracker基础数据目录，即base_path对应的目录

``` shell
mkdir -p /fastdfs/tracker
```

4. 启动Tracker服务

``` shell
# 启动
systemctl start fdfs_trackerd
# 重启
systemctl restart fdfs_trackerd
# 查看服务状态
systemctl status fdfs_trackerd
# 检查服务是否启动
ps -ef | grep fdfs
# 22122端口正在被监听，则算Tracker服务安装成功
netstat -tulnp | grep fdfs
# 关闭
systemctl stop fdfs_trackerd
```

5. 设置开机启动

``` shell
systemctl enable fdfs_trackerd.service
```


### 4. 配置Storage(FastFDS存储器)
1. 启用并修改配置文件

``` shell
cd /etc/fdfs/
cp storage.conf.sample storage.conf
vim /etc/fdfs/storage.conf

# Storage 数据和日志目录地址（根目录必须存在，子目录会自动创建）
base_path = /fastdfs/storage/base
# 逐一配置 store_path_count 个路径，索引号基于 0
# 如果不配置，那就和base_path一样
store_path0 = /fastdfs/storage
# tracker_server 的列表，多个时，每个写一行（会主动连接 tracker_server）
tracker_server = ip:22122
# HTTP 服务端口 默认8888，建议修改防止冲突
http.server_port = 8888
```

2. 创建Tracker基础数据目录，即base_path对应的目录

``` shell
# 对应base_path
mkdir -p  /fastdfs/storage/base
# 对应store_path0，有多个要创建多个
mkdir -p  /fastdfs/storage
```

3. 启动Storage服务

``` shell
# 启动
systemctl start fdfs_storaged
# 重启
systemctl restart fdfs_storaged
# 查看服务状态
systemctl status fdfs_storaged
# 检查服务是否启动
ps -ef | grep fdfs
# 22122端口正在被监听，则算Tracker服务安装成功
netstat -tulnp | grep fdfs
# 关闭
systemctl stop fdfs_storaged
```

4. 查看Storage和Tracker是否在通信

``` shell
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

5. 设置开机自启

``` shell
systemctl enable fdfs_storaged.service
```

### 5. 客户端配置
1. 启用修改Client配置文件

``` shell
cd /etc/fdfs/
cp client.conf.sample client.conf
vim /etc/fdfs/client.conf

# Client 的数据和日志目录
base_path= /fastdfs/client
# Tracker 端口
tracker_server=ip:22122
```

2. 创建Client基础数据目录
``` shell
mkdir -p /fastdfs/client
```



### 6. 配置Nginx模块
安装配置fastfds-nginx-module模块

1. 进入 /usr/local/src/fastdfs-nginx-module-1.22/src/

``` shell
cd /usr/local/src/fastdfs-nginx-module-1.22/src/
```

2. 编辑配置文件（FastDFS 服务脚本设置的 bin 目录是 /usr/local/bin， 但实际命令安装在 /usr/bin/ 下）

``` shell
vim config
# 将config文件中的/usr/local替换成/usr
:%s+/usr/local+/usr
```

3. 进入 nginx 解压目录，添加fastdfs-nginx-module

``` shell
cd /mnt/newdatadrive/apps/nginx-1.18.0
./configure --add-module=/mnt/newdatadrive/apps/fastdfs/fastdfs-nginx-module-1.22/src/ --with-http_stub_status_module
```

若是SSL(https)，没有SSL证书忽略此步
``` shell
./configure --add-module=/mnt/newdatadrive/apps/fastdfs/fastdfs-nginx-module-1.22/src/ --with-http_ssl_module
```

4. 编译安装

``` shell
make && make install
```

5. 复制并修改fastdfs-ngin-module中的配置文件

``` shell
cp /usr/local/src/fastdfs-nginx-module-1.22/src/mod_fastdfs.conf /etc/fdfs/
vi /etc/fdfs/mod_fastdfs.conf

connect_timeout=10
tracker_server=ip:22122
url_have_group_name = true
store_path0=/fastdfs/storage
```

6. 进入fastdfd源码conf目录，将http.conf，mime.types两个文件拷贝到/etc/fdfs/目录下

``` shell
cd /usr/local/src/fastdfs-6.06/conf/
cp http.conf mime.types /etc/fdfs/
```

7. 创建一个软连接，在/fastdfs/storage文件存储目录下创建软连接，将其链接到实际存放数据 的目录（可以省略）

``` shell
ln -s /fastdfs/storage/data/ /fastdfs/storage/data/M00
```


8. 编辑Nginx配置

``` shell
vi /usr/local/nginx/conf/nginx.conf

server {
    listen       80;# 建议修改，防止冲突
    server_name  ip;
    location ~/group([0-9])/M00 {
            root  /fastdfs/storage/data;
            ngx_fastdfs_module;
    }
}
```

9. 启动Ngnix

``` shell
/usr/local/nginx/sbin/nginx
```


### 7. 测试

``` shell
# 下载测试图片到本地
cd /usr/local/src
wget -c "https://www.caimei365.com/img/base/placeholder.png" -O test.png
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/test.png
# 得到返回文件名
group1/M00/00/00/wKgCZF_YlreAcZCZAAHolZLymZE514.png
```

与ip拼接后浏览器访问：http://192.168.2.100/group1/M00/00/00/wKgCZF_YlreAcZCZAAHolZLymZE514.png


