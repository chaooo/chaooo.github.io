---
title: 「环境配置」CentOS 8 常用软件安装(MySQL Nginx SVN Redis)
date: 2020-07-18 18:11:23
tags: [环境配置, CentOS, Nginx, MySQL, Redis, SVN]
categories: 环境配置
---

### 1. CentOS 8 安装（MySQL8.0/MySQL5.7)
#### 1.1 安装 MySQL 8.0
##### 1.1.1 使用最新的包管理器安装MySQL
```
dnf install @mysql
```

<!-- more -->

##### 1.1.2 配置表大小写不敏感
在首次启动之前要配置表大小写不敏感，这是和 MySQL 7 不一样的地方。

mysql的配置文件是 /etc/my.cnf，它 include 了 /etc/my.cnf.d 目录下的配置，所以在 /etc/my.cnf.d/mysql-server.cnf 配置文件里`[mysqld]`下面的配置`lower_case_table_names=1`:

```
[mysqld]
lower_case_table_names=1
```


##### 1.1.3 设置开机启动
安装完成后，运行以下命令来启动MySQL服务并使它在开机时自动启动：
```
systemctl enable --now mysqld
```

检查MySQL服务器是否正在运行：
```
systemctl status mysqld
```

##### 1.1.4 添加密码及安全设置
运行mysql_secure_installation脚本，该脚本执行一些与安全性相关的操作并设置MySQL根密码
```
mysql_secure_installation
```

步骤如下：
1.  要求你配置VALIDATE PASSWORD component（验证密码组件）： 输入y ，回车进入该配置
    *   选择密码验证策略等级， 我这里选择0 （low），回车
    *   输入新密码两次
    *   确认是否继续使用提供的密码？输入y ，回车
    *   移除匿名用户？ 输入y ，回车
    *   不允许root远程登陆？ 我这里需要远程登陆，所以输入n ，回车
2.  移除test数据库？ 输入y ，回车
3.  重新载入权限表？ 输入y ，回车

``` shell
[root@localhost ~]# mysql_secure_installation

Securing the MySQL server deployment.
Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:
LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 0
Please set the password for root here.

New password: 
Re-enter new password: 

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.

Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```

##### 1.1.5 配置远程登陆用户
1. 本机登录MySQL:

```
mysql -uroot -p<上面步骤中设置的密码>
```

2. 创建用户和授权

在mysql8.0创建用户和授权和之前不太一样了，其实严格上来讲，也不能说是不一样,只能说是更严格,mysql8.0需要先创建用户和设置密码,然后才能授权；
将root用户的host字段设为'%'，意为接受root所有IP地址的登录请求：
```
#先创建一个用户
use mysql;
create user 'developer'@'%' identified by '05bZ/OxTB:X+yd%1';
 
#再进行授权
grant all privileges on *.* to 'developer'@'%' with grant option;
flush privileges;
# 若用SQLyog连接MySQL8.0出现错误2058，执行如下命令
ALTER USER 'developer'@'%' IDENTIFIED WITH mysql_native_password BY '05bZ/OxTB:X+yd%1';
```

3. 开放防火墙端口3306

MySQL默认监听3306端口，设置完成后输入exit退出mysql，回到终端shell界面，接着开启系统防火墙的3306端口：
```
firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --reload
```

##### 1.1.6 关闭MySQL主机查询dns
MySQL会反向解析远程连接地址的dns记录，如果MySQL主机无法连接外网，则dns可能无法解析成功，导致第一次连接MySQL速度很慢，所以在配置中可以关闭该功能。
[参考文档](https://www.cnblogs.com/liruning/p/7111015.html)
打开`/etc/my.cnf`文件，添加以下配置：
```
[mysqld]
skip-name-resolve
```

##### 1.1.7 重启服务
```
systemctl restart mysqld
```

本机测试安装后，MySQL8.0默认已经是utf8mb4字符集，所以字符集不再修改。



#### 1.2 CentOS 8 安装配置 MySQL 5.7
##### 1.2.1 添加MySQL存储库
禁用MySQL默认的存储库：
```
dnf remove @mysql
dnf module reset mysql &&  dnf module disable mysql
```

CentOS 8没有MySQL5.7存储库，创建一个新的存储库文件。
```
vi /etc/yum.repos.d/mysql-community.repo
```
将以下数据粘贴到文件中。
```
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=0

[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/7/$basearch/
enabled=1
gpgcheck=0

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/7/$basearch/
enabled=1
gpgcheck=0
```

##### 1.2.2 在CentOS 8 上安装MySQL 5.7
禁用MySQL 8 存储库：
```
dnf config-manager --disable mysql80-community
```

然后启用MySQL 5.7存储库：
```
dnf config-manager --enable mysql57-community
```

然后在CentOS 8 上安装MySQL 5.7：
```
dnf install mysql-community-server
```

按y开始安装。

安装完成后检查软件包的转速详细信息，以确认它是5.7。
```
rpm -qi mysql-community-server 
```


##### 1.2.3 在CentOS 8 / RHEL 8上配置MySQL 5.7
安装后，启动mysqld服务。
```
systemctl enable --now mysqld.service
```

2.2 –复制为root用户生成的随机密码
```
grep 'A temporary password'  /var/log/mysqld.log
```

若没有生成临时密码，执行下面命令
``` shell
rm -rf /var/lib/mysql
systemctl restart mysqld
grep 'temporary password' /var/log/mysqld.log
2020-12-11T09:41:41.459519Z 1 [Note] A temporary password is generated for root@localhost: #4q0R3/#qmC0
```

运行mysql_secure_installation脚本，该脚本执行一些与安全性相关的操作并设置MySQL根密码，
可以参考【***1.1.3 添加密码及安全设置***】及之后的操作。




### 2. CentOS 8 安装配置 Nginx
#### 2.1 下载安装
官网下载：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)
或者直接在linux执行命令：`wget http://nginx.org/download/nginx-1.18.0.tar.gz`

``` shell
# 安装依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
# 解压缩
tar -zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0/
# 执行配置【没有SSL证书】
./configure
#  执行配置【有SSL证书】
./configure --prefix=/usr/local/nginx --with-http_ssl_module
# 编译安装(默认安装在/usr/local/nginx)
make
make install
```

防火墙配置，nginx默认监听80端口
``` shell
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
```
> SSL (sslkey存放路径：`/usr/local/nginx/conf`)
``` conf
server {
    listen 443 ssl;
    server_name 【域名】;
    ssl_certificate      sslkey/_.caimei365.com_bundle.crt;
    ssl_certificate_key  sslkey/_.caimei365.com.key;
    ssl_session_timeout  5m;
    ssl_protocols        TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers          AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
    ssl_prefer_server_ciphers on;
}
```

#### 2.2 Nginx验证与配置
- 测试配置文件：`/usr/local/nginx/sbin/nginx -t`
- nginx主配置文件：`/usr/local/nginx/conf/nginx.conf`
- nginx日志文件：`/usr/local/nginx/logs/access.log`
- 启动Nginx：`/usr/local/nginx/sbin/nginx`

加入环境变量
``` shell
vim /etc/profile
```

最尾输入以下内容：
``` shell
export PATH=$PATH:$JAVA_HOME/bin:/usr/local/nginx/sbin
```

使用`source /etc/profile`命令使配置文件生效。


#### 2.3 Centos 8 配置Nginx开机自启动
1. 进入到/lib/systemd/system/目录，编辑nginx.service

```
[root@localhost ~]# cd /lib/systemd/system/
[root@localhost system]# vim nginx.service

[Unit]
Description=nginx service
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=yes

[Install]
WantedBy=multi-user.target
```

> - `[Unit]`:服务的说明：
>        - Description:描述服务
>        - After:描述服务类别
> - `[Service]`服务运行参数的设置：
>        - Type=forking是后台运行的形式
>        - ExecStart为服务的具体运行命令
>        - ExecReload为重启命令
>        - ExecStop为停止命令
>        - PrivateTmp=True表示给服务分配独立的临时空间
>        - 注意：`[Service]`的启动、重启、停止命令全部要求使用绝对路径
> - `[Install]`运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3


2. 加入开机自启动

``` shell
systemctl enable nginx.service
```

3. 常用命令
``` shell
# 启动nginx服务
systemctl start nginx.service
# 停止服务
systemctl stop nginx.service
# 重新启动服务
systemctl restart nginx.service
# 查看所有已启动的服务
systemctl list-units --type=service
# 查看服务当前状态
systemctl status nginx.service
# 设置开机自启动
systemctl enable nginx.service
# 停止开机自启动
systemctl disable nginx.service
```



### 3. CentOS 8 安装配置 SVN

1. 安装
``` shell
# 安装SVN
[root@localhost ~]# yum -y install subversion
# 创建目录
[root@localhost ~]# mkdir -p /var/svn/svnrepos
# 创建版本库(codes为自定义仓库目录)
[root@localhost ~]# svnadmin create /var/svn/svnrepos/codes
```

2. 配置
    + authz：负责账号权限的管理，控制账号是否读写权限
    + passwd：负责账号和密码的用户名单管理
    + svnserve.conf：svn服务器配置文件
``` shell
[root@localhost ~]# cd /var/svn/svnrepos/codes/conf
[root@localhost conf]# ls
authz  passwd  svnserve.conf
```

添加账户，并赋予读写权限，在authz末尾加入：
``` shell
[root@localhost conf]# vim authz
[/]
admin=rw
test1=rw
test2=rw
```

设置账户密码：
``` shell
[root@localhost conf]# vim passwd
[users]
# harry = harryssecret
# sally = sallyssecret
admin = 123456
test1 = 123456
test2 = 123456
```

设置svn服务器配置文件
``` shell
[root@localhost conf]# vim svnserve.conf
[general]
anon-access = read
auth-access = write

password-db = passwd

authz-db = authz

realm = My First Repository
```

3. 设置开机自启
修改/etc/sysconfig/svnserve 将OPTIONS修改为自己的库版本保存目录（保留引号和-r）
```
vim /etc/sysconfig/svnserve
OPTIONS="-r /var/svn/svnrepos"
```

开机自启
``` shell
systemctl enable svnserve.service
```

4. 启动服务
``` shell
svnserve -d -r /var/svn/svnrepos
```

默认端口是3690
需要修改监听端口或者监听IP可以通过修改--listen-port和 --listen-host来进行修改

可以通过svn协议进行访问：`svn://[ip]:[port]/codes`


### 4. CentOS 8 安装和配置 Redis

1. Redis版本5.0.x包含在默认的CentOS 8存储库中，查询可用的redis安装包：

```  shell
[root@localhost ~]# dnf list redis
上次元数据过期检查：3:04:47 前，执行于 2020年12月21日 星期一 23时03分10秒。
可安装的软件包
redis.x86_64      5.0.3-2.module_el8.2.0+318+3d7e67ea     AppStream
```

2. 执行安装：

``` shell
[root@localhost ~]# dnf install redis
```

3. 安装完成后，启用并启动Redis服务：

``` shell
[root@localhost ~]# systemctl enable --now redis
Created symlink /etc/systemd/system/multi-user.target.wants/redis.service → /usr/lib/systemd/system/redis.service.
```

4. 检查Redis服务器是否正在运行
``` shell
[root@localhost ~]# systemctl status redis
```

5. 配置Redis远程访问

修改Redis配置文件：

``` bash
[root@localhost ~] vim /etc/redis.conf
# 查找 /bind 找到：bind 127.0.0.1并注释，使其它ip地址也可访问
# bind 127.0.0.1
# 
# 查找 /requirepass 去掉注释#，并把foobared 替换为密码，例如：qwe123
# requirepass foobared
requirepass qwe123
```

防火墙配置，Redis默认监听6379端口

``` shell
firewall-cmd --zone=public --add-port=6379/tcp --permanent && firewall-cmd --reload
```

重新启动Redis服务以使更改生效：

``` bash
[root@localhost ~] systemctl restart redis
```

6. 其他服务器远程访问测试

``` bash
redis-cli -h 192.168.2.100 -p 6379 -a qwe123
192.168.2.100:6379>
```

