---
title: 「Jenkins」使用Maven构建Java应用程序
date: 2020-04-08 12:30:53
tags: [Jenkins, 环境配置, Linux]
categories: 环境配置
---

### 1. 添加git凭据
若安装Jenkins时安装的推荐插件，git已经默认安装了，我们添加git凭据：
1. `系统管理(Manage Jenkins)` -> `Manage Credentials` -> `全局` -> `添加凭据`。
    * 类型选择`SSH Username with private key`，往下滑到`Private Key`，并勾选`Enter directly`，在`Enter directly`的key区域点击`Add`，粘贴本机`git私钥`，点击确定。
    * 类型选择`Username with password`，填写git用户名和密码，点击确定。
<!-- more -->

> 本机Git私钥获取：
``` shell
[root@localhost ~]# cat .ssh/id_rsa
```

> git公钥私钥生成详细细节参考[物理机安装CentOS 8.0](https://my.oschina.net/chaoo/blog/4709339)的安装配置Git。

### 2. Jenkins配置`Maven`与`Publish Over SSH`插件
1. 在插件管理搜索`Maven`，找到`Maven Integration`并勾选，然后点击`直接安装`。
2. 接下来同样方法安装`Publish Over SSH`插件(用来通过`ssh命令`发送Maven的构建)。
3. 配置全局变量，添加要部署的远程用户：
    * `系统管理(Manage Jenkins)` -> 点击`Configure System` -> 往下滑到`SSH Server`；
    * 在`SSH Servers`这里点击`新增`，填写用户信息(Name:此配置名，HostNmae:要连接的SSH主机名或IP地址，UserName:远程用户名)；
    * 点击`高级`配置密码，勾选`Use password authentication, or use a different key`，在`Passphrase / Password`输入远程用户密码；
    * 点击`Test Configuration`测试能否连通，最后点击`保存`。
4. 配置mavne的jenkins本地仓库：
    * `系统管理(Manage Jenkins)` -> 点击`Configure System` -> 找到`Maven项目配置`；
    * `Local Maven Repository` -> 选择`Local to the workspace` -> `保存`。
5. 全局工具配置：
    * `系统管理(Manage Jenkins)` -> 点击`Global Tool Configuration`；
    * `Maven配置` -> `默认setting/全局setting`选择`Settings file in filesystem` -> 填写本机安装Maven的setting路径，如`/usr/local/apache-maven-3.6.3/conf/settings.xml`
    * `JDK` -> `JDK安装` -> `新增JDK` -> `JAVA_HOME`填写java安装目录，如`/usr/lib/jvm/java-openjdk`
    * `Git` -> `Git installations` -> `Path to Git executable`填写`/usr/bin/git`(可用which -a git查看)
    * `Maven` -> `Maven安装` -> `新增Maven` -> `MAVEN_HOME`填写Maven安装目录，如`/usr/local/apache-maven-3.6.3`


### 3. 使用Maven构建Java应用程序
1. 新建Item， 输入任务名称，选择Maven项目，点击“确定”
2. 源码管理，选择Git，填写仓库地址(`Repository URL`)和选择凭据(`Credentials`)
3. 构建环境，勾选`Add timestamps to the Console Output`，加上时间戳
4. Build，`Goals and options`根据自己情况自行修改：`clean package -pl demo -am -Dmaven.test.skip=true -P beta`
    * -pl 选项后可跟随{groupId}:{artifactId}或者所选模块的相对路径(多个模块以逗号分隔)，这里只想打包demo模块
    * -am 表示同时处理选定模块所依赖的模块
    * -P 打包的环境
5. Post Steps，发布步骤，这里可以选择执行shell脚本Execute shell（发布到和Jenkins是同一台服务器），Send files or execute commands over SSH（不同服务器）。

### 4. Maven构建配置【Post Steps】
1. 若发布到和Jenkins是同一台服务器，下拉框列表选择`Execute shell`，然后填写shell脚本；
2. 若不在同一服务器，下拉框选择`Send files or execute commands over SSH`，配置`SSH Server`
    + `Source files`填写Maven本地打包后Jar包路径，如：`target/demo-0.0.1-SNAPSHOT.jar`
    + `Remove prefix`去除前缀，如：`target/`
    + `Remote directory`拷贝到Linux服务器的路径，如：`/home/jenkins/test`
    + `Exec command`Jar包拷贝后，执行脚本运行Jar包。
3. shell脚本详情(填写时去掉注释)：
``` shell
#后台执行
BUILD_ID=DONTKILLME

# 进入到项目
cd /home/jenkins/test

# 找到原进程,kill
Project_name=test
pid=$(ps -ef | grep java| grep $Project_name|awk -F '[ ]+' '{print $2}')
kill -9 $pid

# 启动jar
nohup java -jar $Project_name.jar > run.txt &
```
