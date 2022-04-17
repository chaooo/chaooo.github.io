---
title: 「Docker」Docker简介与安装（Linux环境centos）
date: 2020-05-06 11:56:53
tags: [Docker, 环境配置, Linux]
categories: 环境配置
---

Docker 是一个开源的应用容器引擎，基于Go语言并遵从Apache2.0协议开源。
Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。
容器是完全使用沙箱机制，相互之间不会有任何接口（类似于app）,更重要的是容器性能开销极低。<!-- more -->

### 1. Docker解决的问题
* 解决运行环境和配置问题，方便发布，方便做持续集成。
    + 由于不同的机器有不同的操作系统，以及不同的库和组件，在将一个应用部署到多台机器上需要进行大量的环境配置操作。
    + `Docker`主要解决环境配置问题，它是一种虚拟化技术，对进程进行隔离，被隔离的进程独立于宿主操作系统和其它隔离的进程。
    + 使用`Docker`可以不修改应用程序代码，不需要开发人员学习特定环境下的技术，就能够将现有的应用程序部署在其它机器上。
* 使用`Docker`的好处：
    + 部署方便且安全，隔离性好，快速回滚，成本低
* `Docker`的应用场景：
    + Web 应用的自动化打包和发布。
    + 自动化测试和持续集成、发布。
    + 在服务型环境中部署和调整数据库或其他的后台应用。


### 2. Docker架构
Docker使用客户端-服务器(C/S)架构模式，使用远程API来管理和创建Docker容器。
Docker容器通过Docker镜像(Images)来创建，容器与镜像的关系类似于面向对象编程中的对象与类。
* Docker镜像(Images)
    + 用于创建Docker容器(Container)的模板。
* Docker容器(Container)
    + 独立运行的一个或一组应用。
* Docker客户端(Client)
    + 通过命令行或者其他工具使用[Docker API](https://docs.docker.com/reference/api/docker_remote_api)与Docker的守护进程通信。
* Docker主机(Host)
    + 一个物理或者虚拟的机器用于执行Docker守护进程和容器。
* Docker仓库(Registry)
    + 用来保存镜像(Images)，可以理解为代码控制中的代码仓库，[Docker Hub](https://hub.docker.com)提供了庞大的镜像集合供使用。
* Docker Machine
    + 一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker。


### 3. CentOS下安装Docker
要求CentOS 6.5或更高的版本，要求系统为64位、系统内核版本为 2.6.32-431 或者更高版本。
这里演示环境CentOS 8.2。
* 添加Docker存储库
    + 首先，必须添加一个外部存储库以获得Docker CE。这里使用官方的Docker CE CentOS存储库。

1. 下载docker-ce的repo
``` shell
[root@localhost jenkins]# curl https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1919  100  1919    0     0   3408      0 --:--:-- --:--:-- --:--:--  3402
```

2. 安装依赖
``` shell
[root@localhost jenkins]# yum install https://download.docker.com/linux/fedora/32/x86_64/stable/Packages/containerd.io-1.3.7-3.1.fc32.x86_64.rpm
```

3. 安装docker-ce
``` shell
[root@localhost jenkins]# yum install docker-ce
```

4. 启动docker
``` shell
[root@localhost jenkins]# systemctl start docker
```

5. 检查docker服务是否正常运行：
``` shell
[root@localhost jenkins]# systemctl status docker
```

6. 测试运行 hello-world
    + 由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行。
``` shell
[root@localhost jenkins]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
