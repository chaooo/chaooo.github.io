---
title: 「Docker」Docker容器与镜像的使用
date: 2020-05-07 11:56:53
tags: [Docker, 环境配置, Linux]
categories: 环境配置
---


## 1. Docker客户端
docker客户端非常简单，我们可以直接输入`docker`命令来查看到Docker客户端的所有命令选项。<!-- more -->

``` shell
[root@localhost jenkins]# docker
```

可以通过命令`docker <command> --help`更深入的了解指定的Docker命令使用方法。
例如我们要查看`docker stats`指令的具体使用方法：

``` shell
[root@localhost jenkins]# docker stats --help
Usage:	docker stats [OPTIONS] [CONTAINER...]
Display a live stream of container(s) resource usage statistics
Options:
  -a, --all             Show all containers (default shows just running)
      --format string   Pretty-print images using a Go template
      --no-stream       Disable streaming stats and only pull the first result
      --no-trunc        Do not truncate output
```


## 2. Docker镜像(Image)使用
当运行容器时，使用的镜像如果在本地中不存在，docker就会自动从docker镜像仓库中下载，默认是从[Docker Hub](https://hub.docker.com)公共镜像源下载。
1. 查看现有的镜像(Image)
``` shell
[root@localhost jenkins]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        10 months ago       13.3kB
```

> * 各个选项说明:
>    + REPOSTITORY：表示镜像的仓库源
>    + TAG：镜像的标签(同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本)
>    + IMAGE ID：镜像ID
>    + CREATED：镜像创建时间
>    + SIZE：镜像大小

2. 获取一个新的镜像
``` shell
[root@localhost jenkins]# docker pull <REPOSITORY:TAG>
```

> 使用 REPOSTITORY:TAG 来定义不同的镜像，如ubuntu仓库源15.10的版本：`ubuntu:15.10`。


### 2.1 查找镜像
我们可以从[Docker Hub](https://hub.docker.com)网站来搜索镜像。
也可以使用`docker search`命令来搜索镜像。比如我们需要一个httpd的镜像来作为我们的web服务。
``` shell
[root@localhost jenkins]# docker search httpd
NAME                                    DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
httpd                                   The Apache HTTP Server Project                  3248                [OK]                
centos/httpd                                                                            33                                      [OK]
```

> - NAME：镜像仓库源的名称
> - DESCRIPTION：镜像的描述
> - OFFICIAL：是否docker官方发布


### 2.2 创建镜像
* 当我们从docker镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。
    1. 使用`Dockerfile`指令来创建一个新的镜像
    2. 从已经创建的容器中更新镜像，并且提交这个镜像

1. 需要创建一个Dockerfile文件，文件名必须是Dockerfile
``` shell
[root@localhost jenkins]# vim Dockerfile
FROM centos
RUN yum -y install httpd
RUN yum -y install net-tools
RUN yum -y install elinks
CMD ["/bin/bash"]
```

2. 构建镜像，使用`docker build`进行镜像的构建，最后需要指定Dockerfile文件所在路径；
    + 看到最后输出两条Successfully则构建成功。
    + 它会根据文件中写的内容，使用centos镜像实例化一个容器，进入容器中执行三个yum命令
``` shell
[root@localhost jenkins]# docker build -t jenkins/centos-http-net /home/jenkins
Successfully built 09266c896243
Successfully tagged jenkins/centos-http-net:latest
```

3. 查看已经构建好的镜像
``` shell
[root@localhost jenkins]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        10 months ago       13.3kB
jenkins/centos-http-net   latest    09266c896243   10 seconds ago   581MB
```

3. 镜像构建过程
在构建命令执行时输出的一大堆信息中，是执行Dockerfile中的每一行，最关键的几行信息如下
``` shell
Step 1/5 : FROM centos  # 调用centos
 5e35e350aded   # centos镜像id
  
Step 2/5 : RUN yum install httpd -y
 Running in a16ddf07c140  # 运行一个临时容器来执行install httpd
Removing intermediate container a16ddf07c140  # 完成后删除临时的容器id
 b51207823459  # 生成一个镜像
  
Step 3/5 : RUN yum install net-tools -y
 Running in 459c8823018a # 运行一个临时容器执行install net-tools
Removing intermediate container 459c8823018a # 完成后删除临时容器id
 5b6c30a532d4  # 再生成一个镜像
 
Step 4/5 : RUN yum install elinks -y
 Running in a2cb490f9b2f  # 运行一个临时容器执行install elinks
Removing intermediate container a2cb490f9b2f # 完成后删除临时容器id
 24ba4735814b # 生成一个镜像
 
Step 5/5 : CMD ["/bin/bash"]
 Running in 792333c88ba8  # 运行临时容器，执行/bin/bash
Removing intermediate container 792333c88ba8  # 完成后删除临时容器id
 09266c896243  # 生成镜像
Successfully built 09266c896243  # 最终成功后的镜像id就是最后生成的镜像id
```
每一步生成一个镜像，都属于一个docker commit的执行结果
在这个过程中一共生成了三个镜像层，都会被存储在graph中，包括层与层之间的关系，查看docker images中生成的镜像id是否为最后生成的镜像id，FROM和CMD都不算做镜像层

通过`docker history`也可以看到简单的构建过程，看到的是三个yum就是形成的三个镜像层
``` shell
[root@localhost jenkins]# docker history chai/centos-http-net:latest 
IMAGE         CREATED          CREATED BY                                      SIZE    COMMENT
09266c896243  17 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
24ba4735814b  17 minutes ago   /bin/sh -c yum install elinks -y                121MB               
5b6c30a532d4  18 minutes ago   /bin/sh -c yum install net-tools -y             112MB               
b51207823459  18 minutes ago   /bin/sh -c yum install httpd -y                 145MB               
5e35e350aded  4 months ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>     4 months ago     /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
<missing>     4 months ago     /bin/sh -c #(nop) ADD file:45a381049c52b5664…   203MB
```

## 3. Docker容器(Container)
镜像(Image)的概念更多偏向于一个环境包，这个环境包可以移动到任意的Docker平台中去运行；
而容器(Container)就是你运行环境包的实例，换句话说container是images的一种具体表现形式。
+ 容器相关命令：
``` shell
docker container ls，列出现在正在运行的容器
docker container ls -a，列出所有的容器，包括没有在运行的
docker ps -a，列出所有的容器，包括没有在运行的
docker run -it centos，交互式运行容器的方法，这样的话运行之后contianer不会退出，但是这个对话关闭这个，contianer还是就会退出了
docker container rm container_id，可以删除一个容器（注意，这里的id可以是完整的容器id，也可以是缩写，只要是可以和别的id区分开的就可以了）
docker rm container_id，默认就是去删除container的，所以是和docker container rm id是一样的
docker images，和docker image ls是一样的，显示本机的所有镜像
docker image rm image_id，删除image，可以是image_id或者image_name
docker rmi id，是和docker image rm id一样的，是简写
docker container ls -aq，只会显示出容器的id，在批量删除容器的时候有用
docker rm $(docker container ls -aq)，删除所有的容器
docker rm $(docker ps -aq)，也是删除所有的容器
docker rm $(docker container ls -f "status=exited" -q)，删除所有不在运行的容器
```

## 4. Dockerfile文件语法
常用构建镜像指令
``` shell
FROM  # 指定base镜像
MAINTAINER # 指定镜像作者，后面根任意字符串
COPY # 把文件从host复制到镜像内
  COPY src dest
  COPY ["src","dest"]
  src:只能是文件
ADD # 用法和COPY一样，唯一不同时src可以是压缩包，表示解压缩到dest位置，src也可以是目录
ENV # 设置环境变量可以被接下来的镜像层引用，并且会加入到镜像中
  ENV MY_VERSION 1.3
  RUN yum -y install http-$MY_VERSION
  # 当进入该镜像的容器中echo $MY_VERSION会输出1.3
EXPOSE # 指定容器中的进程监听的端口（接口），会在docker ps -a中的ports中显示
  EXPOSE 80
VOLUME # 容器卷，后面会讲到，把host的路径mount到容器中
  VOLUME /root/htdocs /usr/local/apahce2/htdocs
WORKDIR # 为后续的镜像层设置工作路径
        # 如果不设置，Dockerfile文件中的每一条命令都会返回到初始状态
        # 设置一次后，会一直在该路经执行之后的分层，需要WORKDIR /回到根目录
CMD # 启动容器后默认运行的命令，使用构建完成的镜像实例化为容器时，进入后默认执行的命令
    # 这个命令会被docker run启动命令替代
    # 如：docker -it --rm centos echo "hello"
    # echo "hello"会替代CMD运行的命令
  CMD ["nginx", "-g", "daemon off"]  # 该镜像实例化后的容器，进入后运行nginx启动服务
ENTRYPOINT # 容器启动时运行的命令，不会被docker run的启动命令替代
```

### 4.1 RUN/CMD/ENTRYPOINT区别
1. `RUN`：执行命令并创建新的镜像层，主要用于安装软件包
2. `ENTRYPOINT`和`CMD`都算作是启动指令，也就是必须启动容器才会去执行的指令，一般用来启动运行程序使用；当ENTRYPOINT和CMD同时存在时，ENTRYPOINT生效。
3. `ENTRYPOINT`和`CMD`使用格式：
    + shell格式： 会始终调用一个shell程序去执行命令，如`ENTRYPOINT echo "hello $name"`
    + exec格式：`CMD ["命令", "选项", "参数"]`、`ENTRYPOINT ["命令", "选项", "参数"]`
        + exec格式下无法去调用ENV定义的变量，如果非要让exec格式去读取变量的话，它的命令的位置就要使用一个shell环境。因为变量的读取就是使用shell去读取的。如：ENTRYPOINT ["/bin/sh", "-c", "echo hello,$变量名"]
        + 当使用exec格式时，ENTRYPOINT的第一个参数被识别为命令，CMD的参数按顺序变为ENTRYPOINT命令的参数

