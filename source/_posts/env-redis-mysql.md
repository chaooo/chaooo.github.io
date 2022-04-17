---
title: 「环境配置」Redis与MySQL多实例配置
date: 2020-02-11 23:08:46
tags: [环境配置, Redis, MySQL]
categories: 环境配置
---

最近由于工作的需要，需要在同一台服务器上搭建两个`Redis`与`MySQL`的实例。
多实例：就是在一台机器上面开启多个不同的端口(如`Redis`用`6379`/`6380`，`MySQL`用`3306`/`3307`等)，运行多个服务进程；公用一套安装程序，使用不同的配置文件，数据文件。
<!-- more -->

### 1. Redis多实例配置
#### 1.1 查看主机Redis信息
1. 用`ps`命令查看`Redis`进程

``` bash
[root@localhost ~] ps -ef |grep redis
root      1706     1  0  2019 ?        04:12:09 /usr/local/bin/redis-server *:6379                    
root     18174  2560  0 15:35 pts/0    00:00:00 grep redis
```

2. 查找配置文件位置

``` bash
[root@localhost ~] locate redis.conf
/etc/redis.conf
```

#### 1.2 拷贝配置文件并修改
1. 拷贝`redis.conf`并命名为`redis6380.conf`，并修改参数

``` bash
[root@localhost ~] cp /etc/redis.conf /etc/redis6380.conf
[root@localhost ~] vim /etc/redis6380.conf
# 查找 /pidfile 找到pid位置
# pidfile /var/run/redis.pid        #修改pid，每个实例需要运行在不同的pid
pidfile /var/run/redis6380.pid
# 
# 查找 /port 6379 找到端口位置
# port 6379                         #修改端口
port 6380
#                      
# 查找 /dir 找到数据目录位置
# dir /mnt/newdatadrive/data/redis  #修改数据存放目录
dir /mnt/newdatadrive/data/redis6380
# 
# 已开启Redis持久化
appendonly yes
```

2. 准备上面配置的文件

``` bash
[root@localhost ~] mkdir –p /mnt/newdatadrive/data/redis6380
[root@localhost ~] cp /var/run/redis.pid /var/run/redis6380.pid
```


#### 1.3 启动测试
1. 启动`6380`端口`Redis`服务，并查看`Redis`进程

``` bash
[root@localhost ~] /usr/local/bin/redis-server /etc/redis6380.conf
[root@localhost ~] ps -ef |grep redis
root      1706     1  0  2019 ?        04:12:00 /usr/local/bin/redis-server *:6379         
root     15967     1  0 12:16 ?        00:00:00 /usr/local/bin/redis-server *:6380             
root     15994  8014  0 12:16 pts/2    00:00:00 grep redis
```

2. 测试登录`Redis`客户端

``` bash
[root@localhost ~] redis-cli -p 6380
127.0.0.1:6380> QUIT     #退出
```

3. 停止`6380`端口的`Redis`服务

``` bash
redis-cli -p 6380 shutdown
```


#### 1.4 Redis数据迁移
1. 登录原`Redis`客户端(`6379`)

``` bash
[root@localhost ~] redis-cli -p 6379
127.0.0.1:6379> SAVE             #数据备份
127.0.0.1:6379> CONFIG GET dir   #查看Redis数据目录
1) "dir"
2) "/mnt/newdatadrive/data/redis"
127.0.0.1:6379> QUIT             #退出
```

2. 拷贝数据文件`appendonly.aof`和`dump.rdb`到`6380`

``` bash
# 查看6379的数据文件
[root@localhost ~] cd /mnt/newdatadrive/data/redis && ll
total 55176
-rw-r--r-- 1 root root 55411226 Feb 11 09:25 appendonly.aof
-rw-r--r-- 1 root root  1017181 Feb 11 12:28 dump.rdb
# 拷贝到6380
[root@localhost ~] \cp /mnt/newdatadrive/data/redis/appendonly.aof /mnt/newdatadrive/data/redis6380/appendonly.aof
[root@localhost ~] \cp /mnt/newdatadrive/data/redis/dump.rdb /mnt/newdatadrive/data/redis6380/dump.rdb
```

3. 启动`6380`端口`Redis`服务，导入`AOF`数据文件

``` bash
[root@localhost ~] /usr/local/bin/redis-server /etc/redis6380.conf
[root@localhost ~] redis-cli -p 6380 --pipe < /mnt/newdatadrive/data/redis6380/appendonly.aof
```

4. 登录`Redis`查看数据

``` bash
[root@localhost ~] redis-cli -p 6380
127.0.0.1:6380>   #输入具体命令查看数据
```

#### 1.5 配置远程可访问
1. 修改配置文件`redis6380.conf`

``` bash
[root@localhost ~] vim /etc/redis6380.conf
# 查找 /bind 找到：bind 127.0.0.1并注释，其它ip地址也可访问
# bind 127.0.0.1
# 
# 查找 /requirepass 去掉注释#，并把foobared 替换为密码，例如：password123456
# requirepass foobared
requirepass password123456
```

2. 开启防火墙的端口号规则（安全组），将`6380`端口号开通

``` bash
[root@localhost ~] /sbin/iptables -I INPUT -p tcp --dport 6380 -j ACCEPT
```

3. 修改完成后，要在服务里重启`Redis`服务才能使设置生效

``` bash
/usr/local/bin/redis-server /etc/redis6380.conf
```

4. 测试远程访问

``` bash
C:\Users\zc> redis-cli -h 192.168.111.226 -p 6380 -a password123456
192.168.111.226:6380>
```

5. 停止`6380`的`Redis`服务也需要密码

``` bash
[root@localhost ~] redis-cli -p 6380 -a password123456 shutdown
```



### 2. MySQL多实例配置
#### 2.1 查看主机MySQL信息
1. 查看现有`MySQL`数据库实例占用端口

``` bash
[root@localhost ~] netstat -anp | grep mysqld
tcp6       0      0 :::3306                 :::*                    LISTEN      1089/mysqld         
unix  2      [ ACC ]     STREAM     LISTENING     20497    1089/mysqld          /var/lib/mysql/mysql.sock
```

> **须先关闭单实例，跟多实例会有冲突**
> - 备份数据：`[root@localhost ~] mysqldump -P 3306 -u root -p --all-databases > /home/backup/data3306.bak`
> - 停止单实例服务：`[root@localhost ~] service mysqld stop`

2. 查找配置文件位置

``` bash
[root@localhost ~] locate my.cnf
/etc/my.cnf
/etc/my.cnf.d
```

#### 2.2 添加一个3307端口的实例
1. 拷贝`my.cnf`并命名为`my3307.cnf`，并修改参数，主要修改port,sockt,datadir

``` bash
[root@localhost ~] cp /etc/my.cnf /etc/my3307.cnf
[root@localhost ~] vi /etc/my3307.cnf
[mysqld]
# server端字符集
character-set-server=utf8
collation-server=utf8_general_ci
user=root
# 修改端口
port=3307
# 修改数据存放目录
datadir=/var/lib/mysql3307
# 客户端连接socket
socket=/var/lib/mysql/mysql3307.sock
# 修改日志文件
log-error=/var/log/mysqld3307.log
# 修改pid，每个实例需要运行在不同的pid
pid-file=/var/run/mysqld/mysqld3307.pid
# 解决问题：TIMESTAMP with implicit DEFAULT value is deprecated
explicit_defaults_for_timestamp=true
# skip_grant_tables
[mysql]
socket=/var/lib/mysql/mysql3307.sock
default-character-set=utf8
[mysql.server]
default-character-set=utf8
[mysql_safe]
default-character-set=utf8
[client]
socket=/var/lib/mysql/mysql3307.sock
default-character-set=utf8
```

2. 初始化数据库

``` bash
# 写入host避免反解析报错
[root@localhost ~] echo "127.0.0.1   `hostname`" >> /etc/hosts && cat /etc/hosts
[root@localhost ~] mysqld --defaults-file=/etc/my3307.cnf --initialize-insecure
```

3. 启动`3307`端口`MySQL`服务，并查看`MySQL`进程

``` bash
[root@localhost ~] mysqld --defaults-file=/etc/my3307.cnf --user=root &
```

4. 登录`MySQL`

``` bash
# 多实例为root增加密码
[root@localhost ~] mysqladmin -u root -S /var/lib/mysql/mysql3307.sock password '123qwe'
# 登录
[root@localhost ~] mysql -S /var/lib/mysql/mysql3307.sock -p
```

5. 停止本实例`MySQL`服务

``` bash
[root@localhost ~] mysqladmin -u root -S /var/lib/mysql/mysql3307.sock shutdown
```


#### 2.3 再添加一个3308端口的实例
1. 拷贝`my.cnf`并命名为`my3308.cnf`，并修改参数，主要修改port,sockt,datadir

``` bash
[root@localhost ~] cp /etc/my.cnf /etc/my3308.cnf
[root@localhost ~] vi /etc/my3308.cnf
[mysqld]
# server端字符集
character-set-server=utf8
collation-server=utf8_general_ci
user=root
# 修改端口
port=3308
# 修改数据存放目录
datadir=/var/lib/mysql3308
# 客户端连接socket
socket=/var/lib/mysql/mysql3308.sock
# 修改日志文件
log-error=/var/log/mysqld3308.log
# 修改pid，每个实例需要运行在不同的pid
pid-file=/var/run/mysqld/mysqld3308.pid
# 解决问题：TIMESTAMP with implicit DEFAULT value is deprecated
explicit_defaults_for_timestamp=true
# skip_grant_tables
[mysql]
socket=/var/lib/mysql/mysql3308.sock
default-character-set=utf8
[mysql.server]
default-character-set=utf8
[mysql_safe]
default-character-set=utf8
[client]
socket=/var/lib/mysql/mysql3308.sock
default-character-set=utf8
```

2. 初始化数据库

``` bash
[root@localhost ~] mysqld --defaults-file=/etc/my3308.cnf --initialize-insecure
```

3. 启动`3308`端口`MySQL`服务

``` bash
[root@localhost ~] mysqld --defaults-file=/etc/my3308.cnf --user=root &
```

4. 登录`MySQL`

``` bash
# 多实例为root增加密码
[root@localhost ~] mysqladmin -u root -S /var/lib/mysql/mysql3308.sock password '123qwe'
# 登录
[root@localhost ~] mysql -S /var/lib/mysql/mysql3308.sock -p
```

5. 停止本实例`MySQL`服务

``` bash
[root@localhost ~] mysqladmin -u root -S /var/lib/mysql/mysql3308.sock shutdown
```


#### 2.4 实例3307开启远程访问
1. 开启`3307`端口防火墙

``` bash
[root@localhost ~] /sbin/iptables -I INPUT -p tcp --dport 3307 -j ACCEPT
```

2. 测试远程访问

``` bash
C:\Users\zc>mysql -h 192.168.111.227 -P 3307 -u root -p
Enter password: ******
```

