---
title: 「SpringCloud」Config配置中心
date: 2021-11-09 18:01:00
tags: [后端开发, SpringCloud, Config]
categories: SpringCloud
---

### 1. Spring Cloud Config简介
`Spring Cloud Config`可以为微服务架构中的应用提供集中化的外部配置支持，它分为服务端和客户端两个部分。
服务端被称为分布式配置中心，它是个独立的应用，可以从配置仓库获取配置信息并提供给客户端使用。
客户端可以通过配置中心来获取配置信息，在启动时加载配置。
`Spring Cloud Config`默认采用`Git`来存储配置信息，所以天然就支持配置信息的版本管理，并且可以使用`Git`客户端来方便地管理和访问配置信息。<!-- more -->


### 2. 准备工作
因为`config server`是需要到`git`上拉取配置文件的，所以还需要在远程的`git`上新建一个存放配置文件的仓库，
如下仓库中存放客户端配置文件：

``` bash
application-beta.yml
application-dev.yml
application-prod.yml
```


### 3. 配置中心服务端搭建
1. 搭建服务端配置中心（`config-server`），在`pom.xml`文件中添加依赖：

``` xml
<!--配置服务-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<!--注册到注册中心-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2. 在服务端编辑`application.yml`配置文件内容如下：

``` ymal
server:
  port: 8009
# 指定当前服务的名称，这个名称会注册到注册中心
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git: #配置存储配置信息的Git仓库
          uri: http://git.xxx.com/config-files.git # 远程git仓库的地址
          username: username                       # git账户名
          password: password                       # git密码
          clone-on-start: true                     # 开启启动时直接从git获取配置
          search-paths: /**                        # 指定搜索根路径下的目录，若有多个路径使用逗号隔开

# 指定服务注册中心的地址
eureka:
  instance:
    prefer-ip-address: true                                       # 是否使用 ip 地址注册
    instance-id: ${spring.cloud.client.ip-address}:${server.port} # ip:port
  client:
    service-url:                                                  # 设置服务注册中心地址
      defaultZone: http://localhost:18000/eureka/
```

3. 在启动类上，加上`@EnableConfigServer`注解，声明这是一个`config-server`。代码如下：

``` java
/**
 * `@EnableDiscoveryClient`: 声明一个可以被发现的客户端
 * `@EnableConfigServer`: 声明一个Config配置服务
 */
@EnableConfigServer
@EnableDiscoveryClient
@SpringBootApplication
public class ConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class, args);
    }
}
```

4. 启动项目，访问`http://localhost:8009/master/application-dev.yml`，可以看到能够访问到客户端配置文件的内容。


### 4. 客户端获取配置
1. 在`pom.xml`文件中添加依赖：

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

2. 编辑`bootstrap.yml`配置文件内容如下：

``` ymal
server:
  port: 18001

spring:
  cloud:
    config:            #Config客户端配置
      profile: dev                 #启用配置后缀名称
      label: master                #分支名称
      uri: http://localhost:8009   #配置中心地址
      name: application            #配置文件名称
```

启动服务，配置中心读取的配置生效。


### 5. 获取配置信息的基本规则
```
# 获取配置信息
/{label}/{name}‐{profile}
# 获取配置文件信息
/{label}/{name}‐{profile}.yml
```

- `name` : 文件名，一般以服务名(`spring.application.name`)来命名，如果配置了`spring.cloud.config.name`，则为该名称.
- `profiles` : 一般作为环境标识，对应配置文件中的`spring.cloud.config.profile`
- `lable` : 分支（`branch`），指定访问某分支下的配置文件，对应配置文件中的`spring.cloud.config.label`，默认值是在服务器上设置的(对于基于`git`的服务器，通常是“`master`”)

#### 5.1 Maven的Profile管理
`Maven`提供了`Profile`切换功能(多环境`dev,beta,prod`)， 如下`pom.xml`：
``` xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <!-- 环境标识，需要与配置文件的名称相对应 -->
            <activatedProperties>dev</activatedProperties>
        </properties>
        <activation>
            <!-- 默认环境 -->
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>beta</id>
        <properties>
            <activatedProperties>beta</activatedProperties>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <activatedProperties>prod</activatedProperties>
        </properties>
    </profile>
</profiles>
```

配置文件`bootstrap.yml`：
``` ymal
spring:
  cloud:
    config:                             # Config客户端配置
      profile: @activatedProperties@    # 启用配置后缀名称
      label: master                     # 分支名称
      uri: http://localhost:8009        # 配置中心地址
      name: application                 # 配置文件名称
```

`SpringBoot`一把情况下会遵从你选的环境将`@activatedProperties@`替换掉。

但`SpringCloud`比较特殊，使用配置中心后客户端不会再使用`application.yml`，而是使用`bootstrap.yml`。但是`Maven`不认`bootstrap.yml`里的`@activatedProperties@`。

**解决**：在`pom.xml`的`build`标签里添加如下代码，用于过滤`yml`文件：
``` xml
<build>
    <finalName>${project.artifactId}</finalName>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <!--可以在此配置过滤文件  -->
            <includes>
                <include>**/*.yml</include>
            </includes>
            <!--开启filtering功能  -->
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

#### 5.2 将不同服务的配置文件放到以服务名命名的目录下
因为`config server`默认情况下只会搜索`git`仓库根路径下的配置文件，所以我们还需要加上一个配置项：`search-paths`，
该配置项用于指定`config server`搜索哪些路径下的配置文件，需要注意的是这个路径是相对于`git`仓库的，并非是项目的路径。
```
spring:
  cloud:
    config:
      server:
        git:
          ...
          search-paths: /**  # 指定搜索根路径下的所有目录，若有多个路径使用逗号隔开
```
