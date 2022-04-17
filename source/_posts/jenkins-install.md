---
title: 「Jenkins」安装Jenkins（Linux环境centos）
date: 2020-04-04 17:56:53
tags: [Jenkins, 环境配置, Linux]
categories: 环境配置
---

### 1. 下载Jenkins
这里选择清华大学的Jenkins镜像源站下载稳定`2.249.3`版本(war包)
[https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.249.3/](https://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.249.3/)

<!-- more -->

创建Jenkins用户来操作Jenkins：
``` shell
sudo useradd -mU jenkins -s /bin/bash  # 创建jenkins用户并添加同名组、创建用户目录,默认shell为bash
sudo passwd jenkins                    # 重置密码
New password: 
Retype new password: 
su jenkins                             # 切换用户
cd ~                                   # 进入/home/jenkins目录
```

将**jenkins.war**上传到`/home/jenkins`下



### 2. 启动Jenkins
后台运行Jenkins
``` shell
nohup java -DJENKINS_HOME=/home/jenkins/root -jar /home/jenkins/jenkins.war --httpPort=8888 &
```

我这里的主机IP：`192.168.2.100`，端口号指定了`8888`，防火墙需要开放`8888`端口。
- `Centos6/7` 配置`iptables`规则：
``` shell
# 编辑配置文件
vim /etc/sysconfig/iptables
# 在文件中间添加iptables规则
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8888 -j ACCEPT
# 重启防火墙
service iptables restart
```

- `Centos8` 配置`firewalld`：
``` shell
firewall-cmd --query-port=8888/tcp                          # 查询端口是否开放
firewall-cmd --add-port=8888/tcp --permanent             #永久添加8080端口例外(全局)
systemctl stop firewalld.service             #停止防火墙
systemctl start firewalld.service            #启动防火墙
```

访问 Jenkins生成更新目录，我这里访问：`192.168.2.100:8888`。
等待初始化完成出现`解锁Jenkins`时，先不急填密码，先把插件源换掉，这样提升安装速度和降低失败率。



### 3. 修改默认插件源
若一直停留在Please wait while Jenkins is getting ready to work页面，那么可能是你的网络连不到Jenkins官方仓库上。
可以先进入`/home/jenkins/root`目录，打开`hudson.model.UpdateCenter.xml`将`url`中的 
`https://updates.jenkins.io/update-center.json`更改为清华大学的镜像地址:`https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`
``` shell
sed -i "s/https:\/\/updates.jenkins.io\/update-center.json/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins\/updates\/update-center.json/g" /home/jenkins/root/hudson.model.UpdateCenter.xml
```

重启Jenkins
``` shell
ps -ef|grep jenkins   #定位Jenkins的进程号pid
kill -9 <pid>         #杀jenkins进程
# 重启Jenkins
nohup java -DJENKINS_HOME=/home/jenkins/root -jar /home/jenkins/jenkins.war --httpPort=8888 &
```

修改默认Jenkins插件源与连接检测位置
``` shell
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /home/jenkins/root/updates/default.json
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /home/jenkins/root/updates/default.json
```

重启Jenkins，使插件源生效
``` shell
ps -ef|grep jenkins   #定位Jenkins的进程号pid
kill -9 <pid>         #杀jenkins进程
# 重启Jenkins
nohup java -DJENKINS_HOME=/home/jenkins/root -jar /home/jenkins/jenkins.war --httpPort=8888 &
```



### 4. 解锁Jenkins并创建管理员用户
1. 解锁Jenkins
``` shell
cat /home/jenkins/root/secrets/initialAdminPassword
```

复制输出的密码，访问`192.168.2.100:8888`，粘贴到管理员密码框中，继续。
安装推荐的插件就可以。

2. 创建管理员用户
等待插件安装完成 -> 创建新管理员账户 -> 一路保存并完成。

3. 重启Jenkins
因为更新了管理员用户，需要重启下Jenkins服务。

4. 登录Jenkins
访问`192.168.2.100:8888`，输入刚才创建的账号与密码登录。



### 5. 设置开机启动Jenkins

1. 到`/home/jenkins/shell`目录下创建启动脚本`jenkins.sh`

``` shell
cd /home/jenkins/shell
vim  jenkins.sh
```

脚本`jenkins.sh`:

``` shell
#!/usr/bin/bash

# 导入环境变量
export JENKINS_HOME=/home/jenkins/

cd $JENKINS_HOME

pid=`ps -ef | grep jenkins.war | grep -v 'grep'| awk '{print $2}'`
if [ "$1" = "start" ];then
if [ -n "$pid" ];then
    echo 'jenkins is running...'
else
    # java启动服务 配置java安装根路径,和启动war包存的根路径
    nohup java -DJENKINS_HOME=$JENKINS_HOME/root -jar $JENKINS_HOME/jenkins.war --httpPort=8888 >/dev/null 2>&1 &
    echo "服务启动查看进程:"
    echo `ps -ef | grep jenkins.war | grep -v 'jenkins.sh'|grep -v grep`
fi
elif [ "$1" = "stop" ];then
    exec ps -ef | grep jenkins | grep -v grep | awk '{print $2}'| xargs kill -9
    echo 'jenkins is stop...'
else
    echo 'Please input like this:"./jenkins.sh start" or "./jenkins stop"'
fi
```

2. 切换root用户，并添加可执行权限

``` shell
su root
# 添加可执行权限
chmod +x /home/jenkins/shell/jenkins.sh
```

3. 到 /lib/systemd/system 服务注册目录下创建 jenkins.service

``` shell
[Unit]
Description=Jenkins
After=network.target
 
[Service]
Type=forking
User=jenkins
Group=jenkins
ExecStart=/home/jenkins/shell/jenkins.sh start
ExecReload=/home/jenkins/shell/jenkins.sh reload
ExecStop=/home/jenkins/shell/jenkins.sh stop
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```

4. 设置开机启动

``` shell
# 刷新配置
systemctl daemon-reload
# 设置开机启动
systemctl enable jenkins.service
```

5. 查看设置开机启动的服务列表

``` shell
systemctl list-units --type=service
```


> 到此，安装Jenkins已全部完成。

