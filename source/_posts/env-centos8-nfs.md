---
title: 「环境配置」CentOS 8 安装和配置 NFS 服务器
date: 2020-08-10 18:11:23
tags: [环境配置, CentOS, NFS]
categories: 环境配置
---


网络文件系统（NFS）是一个分布式文件系统协议，它允许你通过网络共享远程文件夹。

NFS 协议默认是不加密的，不提供用户身份鉴别。服务端通过限定客户端的 IP 地址和端口来限制访问。<!-- more -->

* NFS 特点：
    + 基于TCP/IP协议，服务于linux之间资源共享
    + 将远程主机上共享资源挂载到本地目录，使得像使用本地文件一样方便。


### 1. 建立 NFS 服务器
#### 1.1 安装 NFS 服务端（CentOS8中默认安装了nfs-utils软件包）
1. 用rpm检查是否有nfs-utils的包已安装：

``` shell
[root@localhost ~]# rpm -qa | grep nfs-utils
nfs-utils-2.3.3-31.el8.x86_64
```

2. 如果没有安装 执行如下命令安装：

``` shell
[root@localhost ~]# dnf install nfs-utils
```

3. 启用并启动 NFS 服务：

``` shell
[root@localhost ~]# systemctl enable --now nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
```

4. 默认情况下，在 CentOS8 上，NFS3 和 NFS4 都可以用，NFS2 被禁用。想要验证，运行下面的cat命令：

``` shell
[root@localhost ~]# cat /proc/fs/nfsd/versions
-2 +3 +4 +4.1 +4.2
```

NFS 服务器配置选项在`/etc/nfsmount.conf`和`/etc/nfs.conf`文件中。默认的设置足够满足我们的要求。


#### 1.2 创建文件系统
1. 创建共享目录，并开放目录权限

``` shell
[root@localhost ~]# mkdir /nfs_database
[root@localhost ~]# chmod 777 /nfs_database
[root@localhost ~]# cd /nfs_database/ && echo "this is nfs database test !!" > nfs_test.txt
cd /nfs_database/ && echo "this is nfs database test chmod 777 /nfs_database" > nfs_test.txt
[root@localhost nfs_database]# ls
nfs_test.txt
```

2. 编辑NFS服务程序配置文件(允许IP地址`192.168.2.*`的所有主机访问NFS共享资源文件夹)

``` shell
[root@localhost ~]# vim /etc/exports
/nfs_database 192.168.2.* (rw,no_root_squash,async) 
```

配置完成后，使nfs配置生效，使用exportfs实用程序有选择地导出目录，而无需重新启动NFS服务

``` shell
#使用exportfs实用程序有选择地导出目录，而无需重新启动NFS服务
[root@localhost ~]# exportfs -rv
#查看当前配置为nfs共享的目录及其状态
[root@localhost ~]# exportfs -v
```

| NFS配置文件参数 | 作用 |
|---|---|
|ro				|只读（read only）											|
|rw				|读写（read write）											|
|root_squash	|当NFS客户端以root管理员访问时，映射为NFS服务器匿名用户			|
|no_root_squash	|当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员		|
|all_squash	|表示客户机所有用户访问时，映射为NFS服务器匿名用户		|
|sync			|同时将数据写入到内存与硬盘中，保证不丢失数据					|
|async			|优先将数据保存到内存，然后再写入硬盘，效率更高，但可能丢失数据	|


3. 启动和启用rpcbind服务程序

``` shell
# 安装rpcbind
[root@localhost ~]# yum install -y rpcbind
# 重启并启用rpc服务程序
[root@localhost ~]# systemctl restart rpcbind && systemctl enable rpcbind
# 重启并启用NFS服务程序
[root@localhost ~]# systemctl restart nfs-server && systemctl enable nfs-server
```

#### 1.3 防火墙设置
将该服务添加到防火墙中进行放行。

部署nfs服务不仅需要nfs服务软件包，还需要rpc-bind服务和mountd服务。
因为nfs服务需要向客户端广播地址和端口信息，nfs客户端需要使用mount对远程nfs服务器目录进行挂载。

``` shell
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-service=nfs
success
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-service=rpc-bind
success
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-service=mountd
success
[root@localhost ~]# firewall-cmd --reload
success
```


### 2. 客户端配置

1. 查询远程nfs服务器是否能够连通

``` shell
[root@localhost ~]# showmount -e 192.168.2.100
Export list for 192.168.2.100:
/nfs_database (everyone)
```

| showmount参数 | 作用 |
|---|---|
|-e		|显示NFS服务器共享列表		|
|-a		|显示本地挂载的文件资源情况	|
|-v		|显示版本号					|


2. 创建本地nfs专用共享目录

``` shell
[root@localhost ~]# mkdir /nfs_database
[root@localhost ~]# chmod 777 /nfs_database
```

3. 将远程nfs服务器共享目录挂载到本地创建的nfs共享目录

``` shell
[root@localhost ~]# mount -t nfs 192.168.2.100:/nfs_database /nfs_database
```

+ mount参数：
    * `-t`：使用TCP协议
    * `nfs`：nfs服务
    * `192.168.2.100:/nfs_database`：远程nfs服务器资源共享目录
    * `/nfs-database`：本地资源共享目录

4. 本机查看共享文件

``` shell
[root@localhost ~]# cd /nfs_database
[root@localhost nfs_database]# ll
总用量 4
-rw-r--r-- 1 root root 50 12月 17 14:52 nfs_test.txt
```
