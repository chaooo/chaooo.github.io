---
title: 「Shell」Shell脚本部署Java应用
date: 2020-06-10 15:12:46
tags: [环境配置, Linux, Shell]
categories: 环境配置
---

Shell脚本部署Java应用
<!-- more -->

### 1. 根据PID获取进程信息
``` shell
#!/bin/bash
#
# Function: 根据用户输入的PID，过滤出该PID所有的信息
#

read -p "请输入要查询的PID: " P
n=`ps -aux| awk '$2~/^'$P'$/{print $11}'|wc -l`
if [ $n -eq 0 ];then
 echo "该PID不存在！"
 exit
fi
echo "--------------------------------"
echo "进程PID: $P"
echo "进程命令：`ps -aux| awk '$2~/^'$P'$/{print $11}'`"
echo "进程所属用户: `ps -aux| awk '$2~/^'$P'$/{print $1}'`"
echo "CPU占用率：`ps -aux| awk '$2~/^'$P'$/{print $3}'`%"
echo "内存占用率：`ps -aux| awk '$2~/^'$P'$/{print $4}'`%"
echo "进程开始运行的时刻：`ps -aux| awk '$2~/^'$P'$/{print $9}'`"
echo "进程运行的时间：`ps -aux| awk '$2~/^'$P'$/{print $10}'`"
echo "进程状态：`ps -aux| awk '$2~/^'$P'$/{print $8}'`"
echo "进程虚拟内存：`ps -aux| awk '$2~/^'$P'$/{print $5}'`"
echo "进程共享内存：`ps -aux| awk '$2~/^'$P'$/{print $6}'`"
echo "--------------------------------"
```

> 若执行出现权限不够，为shell文件增加执行权限:`chmod +x test.sh`


### 2. 使用Shell脚本部署Jar包
将跳板机上(或本地服务器)的jar包文件拷贝到发布服务器，然后通过发布服务器上的脚本实现旧jar包的备份，新jar包的启动。
* 实现部署的操作：拷贝jar包到服务器 -> 备份旧服务jar包 -> 启动新服务jar包
* 使用命令：`./begin.sh demo-0.0.1-SNAPSHOT.jar`

2. 跳板机(本地服务器)的脚本begin.sh

``` shell
#!/bin/bash
fileName=$1
if [ -z "$fileName" ]; then
    echo "文件名不能为空"
    exit 0
fi
echo "开始拷贝jar文件【$fileName】到192.168.2.100"
scp $fileName cmuser@192.168.2.100:/usr/local/test/$fileName.prev
echo "文件传输结束，准备启动192.168.2.100的部署脚本"
ssh cmuser@192.168.2.100 /usr/local/test/demo-deploy.sh $fileName
```

通过上面的跳板机上的代码，我们知道跳板机最终会调用发布服务器上`/usr/local/test/demo-deploy.sh`这个shell脚本命令。

3. 发布服务器脚本demo-deploy.sh

``` shell
#!/usr/bin/env bash
echo "服务器开始部署服务"
projectname="demo"

# 打开文件所属的目录，不然远程执行会找不到当前目录
cd /usr/local/test

# 新的jar包会当成参数传过来
newJar=$1
echo "新的jar为：$newJar" 
# 如果新的jar包为空则退出
if [ -z "$newJar" ]; then
    echo "新的jar不能为空"
    exit 0
fi

# 获取旧的jar包名称，当然可能是空的，也可能跟当前名称一致
oldJar=$(ps -ef | grep ${projectname}|grep -v 'demo-deploy.sh'|grep -v grep|awk '{print $10}'|cut -d '/' -f 2)
echo "当前运行的旧的jar包为：$oldJar" 
#如果新的jar包为空则退出
if [ -z "$oldJar" ]; then
    echo "没有启动的demo服务"
else
    # 如果旧的进程还在就将旧的进程杀掉
    oldId=`ps -ef|grep ${projectname}|grep -v "$0"|grep -v "grep"|awk '{print $2}'`
    echo "$oldId"
    echo "kill old process start ..."
    for id in $oldId
    do
        kill -9 $id
        echo "killed $id"
    done
    echo "kill old process end"
    # 获取当前时间戳
    suffix=".bak-"`date '+%s%3N'`;
    echo $suffix;
    # 将旧的jar包进行备份
    mv $oldJar ${oldJar}${suffix}
fi

# 开始启动新的进程
mv ${1}.prev ${1}
nohup java -jar ${1} > run.txt 2>&1 &
echo "服务启动查看进程:"
echo `ps -ef | grep ${projectname}|grep -v 'demo-deploy.sh'|grep -v grep`
```

4. 演示
``` shell
# 跳板机
[cmuser@localhost test]$ ./begin.sh demo-0.0.1-SNAPSHOT.jar
开始拷贝jar文件【demo-0.0.1-SNAPSHOT.jar】到192.168.2.100
cmuser@192.168.2.100's password: 
demo-0.0.1-SNAPSHOT.jar                                                                                                                                                                                 100%   20MB  11.2MB/s   00:01    
文件传输结束，准备启动192.168.2.100的部署脚本
cmuser@192.168.2.100's password: 
服务器开始部署服务
新的jar为：demo-0.0.1-SNAPSHOT.jar
当前运行的旧的jar包为：demo-0.0.1-SNAPSHOT.jar
2581
kill old process start ...
killed 2581
kill old process end
.bak-20201117
服务启动查看进程:
```

``` shell
# 发布服务器(部署前pid:2581)
[cmuser@192.168.2.100 test]$ ps -ef | grep demo
cmuser    2581     1 16 17:03 ?        00:00:06 java -jar demo-0.0.1-SNAPSHOT.jar

[cmuser@192.168.2.100 test]$ ll
total 40260
-rw-rw-r--. 1 cmuser  cmuser  20606107 Nov 18 15:45 demo-0.0.1-SNAPSHOT.jar
-rw-rw-r--. 1 cmuser  cmuser  20606111 Nov 18 15:06 demo-0.0.1-SNAPSHOT.jar.bak-1605685530423
-rwxrwxr-x. 1 jenkins jenkins     1356 Nov 17 17:56 demo-deploy.sh
-rwxrwxrwx. 1 jenkins jenkins     1207 Nov 18 15:45 run.txt

# 发布服务器(部署后:2653)
[cmuser@192.168.2.100 test]$ ps -ef | grep demo
cmuser    2653     1 59 17:04 ?        00:00:07 java -jar demo-0.0.1-SNAPSHOT.jar
```
