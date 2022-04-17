---
title: 「环境配置」物理机安装CentOS 8
date: 2020-03-15 23:08:46
tags: [环境配置, CentOS]
categories: 环境配置
---


CentOS 8 所需的最低硬件配置:
- 2GB RAM
- 64位x86 / 2GHz或以上的 CPU
- 20GB 硬盘空间
<!-- more -->

### 1. 下载 CentOS 8 ISO 文件
在 CentOS 官方网站 https://www.centos.org/download/ 下载 CentOS 8 ISO 文件。


### 2. 创建 CentOS 8 启动介质（USB 或 DVD）
下载 CentOS 8 ISO 文件之后，将 ISO 文件烧录到 USB 移动硬盘或 DVD 光盘中，作为启动介质。
常用的烧录工具如：UltraISO。
然后重启系统，在 BIOS 中设置为从上面烧录好的启动介质启动。


### 3. 选择“安装 CentOS Linux 8.0”选项
当系统从 CentOS 8 ISO 启动介质启动之后，就可以看到启动选择界面。选择“Install CentOS Linux 8”（安装 CentOS Linux 8）选项并按回车。


### 4. 安装错误处理
实体机安装，写入镜像到U盘会出现找不到U盘的情况，报错是：`/dev/root does not exist`。
界面尾部出现 `dracut:/#` 时，键入 `ls /dev` 查看启动U盘所在盘符。
``` shell
dracut:/# ls /dev
```
![](up-e46cee7c94277328598af51d821b95afaea.webp)

前缀`sd`表示磁盘，如sda、sdb、sdc；sda后面的4表示磁盘a的分区，因为物理机磁盘删除了分区，是未分区状态，所以sda4就是U盘了。
确定好sda是启动U盘后，键入 `reboot` 重启。
进入到开机系统选择界面，根据提示 按键盘 `e` 或 `tab`。
界面会出现：`initrd=initrd.img inst.stage2=hd:LABEL=CentOSx86_64 rd.live.check quiet`，
修改为U盘的分区（sda4）：`initrd=initrd.img inst.stage2=hd:/dev/sda4 quiet`。
然后保存后按`回车键`安装。


### 5. 开始安装CentOS 8
根据图像安装向导的引导进行配置。
设置`root`用户的密码，创建一个本地用户。
在安装完成后，安装向导会提示重启系统。
> 注意：重启完成后，记得要把安装介质断开，并将 BIOS 的启动介质设置为硬盘。

同意 `CentOS 8` 的许可证，使用刚创建的本地用户登录。
开始使用 `CentOS LinuxStart Using CentOS Linux`。


### 6. 安装配置JDK
使用 yum 直接安装，环境变量会自动配置好。
1. 检查 yum 中有没有 java1.8 包
``` shell
yum list java-1.8*
```

2. 开始安装java1.8
``` shell
yum install java-1.8.0-openjdk* -y
```

3. 用验证`java -version`
``` shell
[root@localhost ~]$ java -version
openjdk version "1.8.0_272"
OpenJDK Runtime Environment (build 1.8.0_272-b10)
OpenJDK 64-Bit Server VM (build 25.272-b10, mixed mode)
```


### 7. 安装配置Git
1. 使用Yum安装Git
``` shell
yum install git
```

2. 用验证`git --version`
``` shell
[root@localhost ~]$ git --version
git version 2.18.4
```

3. 配置Git
``` shell
[root@localhost ~]$ git config --global user.name "yourname"
[root@localhost ~]$ git config --global user.email "youremail@email.com"
```

4. 验证配置
``` shell
[root@localhost ~]$ git config --list
user.name=yourname
user.email=youremail@email.com
```

5. 生成git授权证书
键入命令`ssh-keygen -t rsa -C "youremail@email.com"`一路回车键就好:
``` shell
[root@localhost ~]$ ssh-keygen -t rsa -C "youremail@email.com"
```

6. 拷贝公钥(私钥:`id_rsa`，公钥:`id_rsa.pub`)到git服务器，如gitee添加SSH公钥。
``` shell
[root@localhost ~]$ cd ~/.ssh/
[root@localhost .ssh]# ls
id_rsa  id_rsa.pub
[root@localhost .ssh]# cat id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD8blWzO+L+H8h8GkNOEEfLBQGCszU= youremail@email.com
```

7. 添加公钥完成后进行测试SSH链接(gitee为例)
键入命令`ssh -T git@gitee.com`，根据提示键入`yes`确认，当终端提示`hi/welcome/successfully`等表示链接成功。


### 8. 安装配置Maven
1. 下载并解压Maven
``` shell
[root@localhost ~]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.zip
[root@localhost ~]# unzip apache-maven-3.6.3-bin.zip
# 把maven安装到/usr/local/下
[root@localhost apache-maven-3.6.3]# mv apache-maven-3.6.3 /usr/local/apache-maven-3.6.3
[root@localhost ~]# cd /usr/local/apache-maven-3.6.3
[root@localhost apache-maven-3.6.3]# pwd
/usr/local/apache-maven-3.6.3
```

2. 修改系统配置`/etc/profile`，添加Maven环境变量
``` shell
[root@localhost apache-maven-3.6.3]# vim /etc/profile
# 在文件末尾添加：
export MAVEN_HOME=/usr/local/apache-maven-3.6.3
export PATH=$MAVEN_HOME/bin:$PATH
```

重新加载系统配置：`. /etc/profile`
``` shell
[root@localhost apache-maven-3.6.3]# . /etc/profile
```

3. 验证Maven
``` shell
[root@localhost apache-maven-3.6.3]# mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/local/apache-maven-3.6.3
Java version: 1.8.0_272, vendor: Red Hat, Inc., runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.272.b10-1.el8_2.x86_64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "4.18.0-193.el8.x86_64", arch: "amd64", family: "unix"
```

### 9. Centos8开放防火墙端口
1. 查看防火墙某个端口是否开放

``` shell
firewall-cmd --query-port=3306/tcp
```

2. 开放防火墙端口3306

``` shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

3. 查看防火墙状态

``` shell
systemctl status firewalld
```

4. 关闭防火墙

``` shell
systemctl stop firewalld
```

5. 打开防火墙

``` shell
systemctl start firewalld
```

6. 开放一段端口

``` shell
firewall-cmd --zone=public --add-port=40000-45000/tcp --permanent
```

7. 查看开放的端口列表

``` shell
firewall-cmd --zone=public --list-ports
```

8. 重启防火墙

``` shell
firewall-cmd --reload  # 重启防火墙(修改配置后要重启防火墙)
```
