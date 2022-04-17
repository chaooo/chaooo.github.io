---
title: 「Jenkins」Jenkins自动部署Jar到远程服务器
date: 2020-04-10 19:30:53
tags: [Jenkins, 环境配置, Linux]
categories: 环境配置
---

### 1. 配置远程服务器SSH免密登录
1. 本地客户端生成公私钥（一路回车默认即可），会在用户目录.ssh文件夹下创建公私钥<!-- more -->

``` shell
[localuser@localhost .ssh]$ ssh-keygen
[localuser@localhost .ssh]$ ls
id_rsa  id_rsa.pub
```

2. 上传公钥到服务器，这里远程服务器地址为：192.168.2.200，用户为：testuser

``` shell
ssh-copy-id -i ~/.ssh/id_rsa.pub testuser@192.168.2.200
```

上面这条命令会在远程服务器的`~/.ssh`目录生成`authorized_keys`，里面是id_rsa.pub(公钥)内容。
> 若目标服务器已经存在了`authorized_keys`，则可拷贝公钥内容追加到`authorized_keys`内容的末尾。

3. 测试免密登录，本地客户端通过ssh连接远程服务器，就可以免密登录了。

``` shell
[localuser@localhost ~]$ ssh testuser@192.168.2.200
Last login: Tue Nov 17 20:57:25 2020 from 192.168.2.202
[testuser@caimeidev1 ~]$ exit
logout
Connection to 192.168.2.200 closed.
```


### 2. 登录Jenkins客户端并配置
先安装插件：`Git Paramater`，这里只演示部署，因为已在本地打好包推送到了git服务器。
1. 新建Item， 输入任务名称：`MavenTest`(自己定义)，选择自由项目，点击`确定`。
2. 勾选`This project is parameterized`(参数化构建)
    + 选择`Choice Parameter`,添加打包环境参数(名称：buildEnv，选项：beta和prod)；
    + 选择`Git Parameter`,定义参数名称:`gitBranch`，参数类型选择`分支`；
    + ![](up-60e4e7fe99f99698de5e5488e94b969eccb.webp)
3. 源码管理，选择Git，填写仓库地址(`Repository URL`)和选择凭据(`Credentials`)。
    + ![](up-499a162af32384b469b4b80cc0632f54f73.webp)
4. 构建环境，勾选`Add timestamps to the Console Output`，加上时间戳。
5. Post Steps，选择执行shell脚本Execute shell，输入：
    + ![](up-3551b07c426b8380a725353feb341453c0b.webp)

``` shell
cd /home/jenkins/shell
# 对应两个构建参数$buildEnv，$gitBranch
./demo-deploy-$buildEnv.sh demo-0.0.1-SNAPSHOT.jar  ${gitBranch}
```


### 3. 编写部署脚本
可参考[shell脚本部署Java应用](https://my.oschina.net/chaoo/blog/4721418)
1.  本地客户端

``` shell
[localuser@localhost ~]$ cd /home/jenkins/shell
# 推送不同服务器用不同shell脚本，beta:demo-deploy-beta.sh，product:demo-deploy-prod.sh，对应$buildEnv参数
[localuser@localhost ~]$ vim demo-deploy-beta.sh
```

``` shell
#!/usr/bin/bash
fileName=$1
gitBranch=$2
if [ -z "$fileName" ]; then
    echo "文件名不能为空"
    exit 0
fi
echo "准备发布【$gitBranch】分支，到【Beta：192.168.2.200】"
echo "开始拷贝jar文件【$fileName】到远程服务器"
scp /home/jenkins/root/workspace/MavenTest/target/$fileName testuser@192.168.2.200:/usr/local/test/$fileName.prev
# 捕获上一条命令的输出$? (if 0 正常 else 错误)
if [ "$?" == "0" ]; then
    echo "文件传输结束，准备启动远程服务器的部署脚本"
    ssh testuser@192.168.2.200 /usr/local/test/demo-deploy.sh  $fileName
else
   echo "拷贝文件错误"
   exit 0
fi
```

``` shell
# 添加执行权限
[localuser@localhost ~]$ chmod +x demo-deploy-beta.sh
```


2. 远程服务器
``` shell
[localuser@localhost ~]$ cd /usr/local/test
[localuser@localhost ~]$ vim demo-deploy.sh
```

``` shell
#!/bin/env bash
echo "服务器开始部署服务"
projectname="demo-0.0.1-SNAPSHOT"

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
    # 获取当前时间
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

``` shell
# 添加执行权限
[localuser@localhost ~]$ chmod +x demo-deploy.sh
```


### 4. 部署测试
`Build with Parameters` -> 选择要部署的`git分支${gitBranch}`和`环境参数${buildEnv}`

![](up-66c8b64a303730fb9c192fd57f769efdd5d.webp)

构建完成后点击`Build History`(构建历史)里的构建版本号，点击`控制台输出`查看日志

![](up-3b4db51b047564d03e104526cc387f2a883.webp)
