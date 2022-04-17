---
title: 「环境配置」Tomcat9安装及多实例多应用配置(CentOS 8)
date: 2020-11-20 18:11:23
tags: [环境配置, CentOS, Tomcat9]
categories: 环境配置
---

### 1. Tomcat9安装
使用wget获取：
``` shell
wget https://mirrors.aliyun.com/apache/tomcat/tomcat-9/v9.0.41/bin/apache-tomcat-9.0.41.tar.gz
```
下载之后如果没有进入别的文件夹，压缩包一般是在/root下面。进入/root下面进行解压：<!-- more -->
``` shell
tar xzf apache-tomcat-9.0.41.tar.gz
```
然后复制到指定的文件夹下(在/data/apps下面创建了一个tomcat的文件夹)：
``` shell
mkdir -p /data/apps
mv apache-tomcat-9.0.41 /data/apps/tomcat
```
然后配置jvm内存参数（也可以不用配置）：
``` shell
vim /data/apps/tomcat/bin/setenv.sh
```
在setenv.sh中写入如下语句：
``` shell
JAVA_OPTS='-Djava.security.egd=file:/dev/./urandom -server -Xms512m -Xmx1024m -Dfile.encoding=UTF-8'
```

开放端口
``` shell
#开放8080端口
firewall-cmd --add-port=8080/tcp --permanent && firewall-cmd --reload
#重新加载防火墙规则
firewall-cmd --reload
```

启动/停用
``` shell
#启动
cd /data/apps/tomcat/bin && sh startup.sh
#停用
cd /data/apps/tomcat/bin && sh shutdown.sh
```
通过浏览器访问 【ip】:8080



### 2. Tomcat多实例配置
Tomcat多实例部署的好处在于升级方便，配置及安装文件间互不影响。
```
    安装路径                       实例位置
【CATALINA_HOME】            【CATALINA_BASE】
        |————bin     <-------------|
        |————lib     <-------------|
                                   |————（conf,webapps,logs,temp,work）

```

创建一个实例存放路径
``` shell
mkdir -p /data/runtime/tomcat-instance
```

新建两个tomcat实例，把安装路径下的`conf,webapps,logs,temp,work`文件拷贝到实例中：
``` shell
cd /data/runtime/tomcat-instance
mkdir tomcat1 tomcat2
# 
cd /data/apps/tomcat/
cp conf/ webapps/ temp/ logs/ work/ -rt /data/runtime/tomcat-instance/tomcat1
cp conf/ webapps/ temp/ logs/ work/ -rt /data/runtime/tomcat-instance/tomcat2
```

新建 Tomcat实例启动/停止脚本
``` shell
cd /data/runtime/tomcat-instance
vim tomcat1.sh
vim tomcat2.sh
```

tomcat1.sh
``` shell
#!/bin/bash

. /etc/rc.d/init.d/functions

source /etc/profile

export CATALINA_BASE=/data/runtime/tomcat-instance/tomcat1
export CATALINA_HOME=/data/apps/tomcat
export CATALINA_PID=$CATALINA_BASE/CATALINA_PID
export JAVA_OPTS="-server -Xms256m -Xmx512m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m"

sname=tomcat1
mark=$CATALINA_BASE/temp
pid=`ps -ef | grep "$mark" | grep -v "grep $mark" | awk -F " " '{print $2}'`

function start()
{
  if [ -f "$CATALINA_PID" ]; then
    rm -rf $CATALINA_PID
  fi
  echo "$sname start ......"
  $CATALINA_HOME/bin/catalina.sh start 2>/dev/null
  pid=`ps -ef | grep "$mark" | grep -v "grep $mark" | awk -F " " '{print $2}'`
  echo "PID:                   $pid"
}

function stop()
{
  echo "$sname stop ......"
  if [ $pid ]; then
    kill -9 $pid
    rm -f $CATALINA_PID
  fi
}

function status()
{
  if [ $pid ]; then
    echo "$sname run ......"
  else
    echo "$sname stop ......"
  fi
  echo `ps -ef | grep "$CATALINA_BASE" | grep -v "grep $CATALINA_BASE"`
}

case $1 in
start)
  start
  ;;
stop)
  stop
  ;;
restart)
  stop
  start
  ;;
status)
  status
  ;;
*)
  echo "Option in (start|stop|restart|status)"
  ;;
esac
```

tomcat2.sh参考tomcat1.sh


赋予权限
``` shell
chmod 777 tomcat1.sh tomcat2.sh
```

配置实例`server.xml`端口
+ Server Port：该端口用于监听关闭tomcat的shutdown命令，默认为8005
+ Connector Port：该端口用于监听HTTP的请求，默认为8080
+ AJP Port：该端口用于监听AJP（ Apache JServ Protocol ）协议上的请求，通常用于整合Apache Server等其他HTTP服务器，默认为8009
+ Redirect Port：重定向端口，出现在Connector配置中，如果该Connector仅支持非SSL的普通http请求，那么该端口会把 https 的请求转发到这个Redirect Port指定的端口，默认为8443；

这里`tomcat1`实例保持默认，把`tomcat2`实例的`Server Port`改为了`8006`，`Connector Port`改为了 `8081`:

``` shell
cd /data/runtime/tomcat-instance/tomcat2
vim conf/server.xml
# 8081端口
firewall-cmd --add-port=8081/tcp --permanent && firewall-cmd --reload
```


分别在 tomcat1、tomcat2 的 webapps/ROOT 目录下放入了页面文件。

启动两个实例
``` shell
cd /data/runtime/tomcat-instance
./tomcat1.sh start
./tomcat2.sh start
```

通过浏览器：`【ip】:8080/`，`【ip】:8081/`，如：192.168.2.100:8080


### 3. nginx使用https代理tomcat
修改nginx配置文件，在server中添加如下内容：

``` shell
location /tomcat1/ {
     proxy_pass http://127.0.0.1:8080/;
          proxy_set_header   Host     $host:$server_port;
          proxy_set_header   X-Real-IP        $remote_addr;
          proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
          proxy_set_header   Scheme  $scheme;
}
location /tomcat2/ {
     proxy_pass http://127.0.0.1:8081/;
          proxy_set_header   Host     $host:$server_port;
          proxy_set_header   X-Real-IP        $remote_addr;
          proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
          proxy_set_header   Scheme  $scheme;
}
```

通过浏览器访问：`【ip】/tomcat1/`，`【ip】/tomcat2/`，如：192.168.2.100/tomcat1/

- tips: `Tomcat`默认上传到服务器中文件的权限是`- -rw-r-----`640权限，其他人不可读。
到`Tomcat`中`bin`目录下面，修改`catalina.sh`中`UMASK="0027"`为`UMASK="0022"`。
``` shell
# Set UMASK unless it has been overridden
if [ -z "$UMASK" ]; then
    UMASK="0022"
fi
umask $UMASK
```

