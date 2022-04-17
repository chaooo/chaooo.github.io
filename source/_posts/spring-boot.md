---
title: 「Spring」SpringBoot入门
date: 2018-06-06 23:06:01
tags: [后端开发, Spring, SpringBoot]
categories: Spring
---

Spring Boot 是由 Pivotal 团队提供的全新框架。Spring Boot 是所有基于 Spring Framework 5.0 开发的项目的起点。Spring Boot 的设计是为了简化新 Spring 应用的初始搭建以及开发过程，并且尽可能减少你的配置文件。
<!-- more -->
从最根本上来讲，Spring Boot 就是一些库的集合，它能够被任意项目的构建系统所使用。它使用 “习惯优于配置” （项目中存在大量的配置，此外还内置一个习惯性的配置）的理念让你的项目快速运行起来。用大佬的话来理解，就是 spring boot 其实不是什么新的框架，它默认配置了很多框架的使用方式，就像 maven 整合了所有的 jar 包，spring boot 整合了所有的框架，总结一下及几点：
1. 为所有 Spring 开发提供一个更快更广泛的入门体验。
2. 零配置。无冗余代码生成和XML 强制配置，遵循“约定大于配置” 。
3. 集成了大量常用的第三方库的配置， Spring Boot 应用为这些第三方库提供了几乎可以零配置的开箱即用的能力。
4. 提供一系列大型项目常用的非功能性特征，如嵌入式服务器、安全性、度量、运行状况检查、外部化配置等。
5. Spring Boot 不是Spring 的替代者，Spring 框架是通过 IOC 机制来管理 Bean 的。Spring Boot 依赖 Spring 框架来管理对象的依赖。Spring Boot 并不是Spring 的精简版本，而是为使用 Spring 做好各种产品级准备。


### 1. 项目管理工具Maven的基本使用
Maven是一个使用java编写的开源的项目管理工具，可以方便灵活的控制项目，不必浪费时间去在不同的环境中配置依赖的jar包，而专心于业务逻辑。

#### 1.1 配置Maven的系统环境变量
1. 下载并解压到目录，如`D:\apache-maven-3.6.1`
2. 添加新的系统环境变量MAVEN_HOME=安装的目录：`MAVEN_HOME=D:\apache-maven-3.6.1`
3. 添加`%MAVEN_HOME%\bin`到系统PATH变量.
4. 测试Maven配置是否成功，打开命令行窗口，输入`mvn -v`，如果有maven 版本信息输出则证明配置成功，否则请查看自己配置路径等是否正确。

> 注意：安装Maven前请确保已安装JDK并成功配置其环境变量。


#### 1.2 maven中的术语
- **maven插件**：maven主要定义了项目对象模型的生命周期。实际上每个任务都是交由插件完成的。maven的生命周期与插件目标相互绑定，来完成每个具体的任务。
- **maven坐标**：就是对项目的定位。groupId：组id，机构名。artifactId：构建id ，产品名或者产品的id。version ：版本号。
- **坐标形式**：groupId + artifactId+ version
- **maven仓库**：存放maven共享构建的位置。
    1. 本地仓库：localRepository（使用`conf/settings.xml`设置）
    2. 私服仓库：部署在局域网中的仓库，方便整个团队的开发使用。
    3. 中央仓库：远程仓库下载地址：http://repo1.maven.org/maven2

``` xml
<!-- conf/settings.xml设置本地仓库路径 -->
<settings ...
    <localRepository>D:/apache-maven-3.6.1/.m2/repository</localRepository>
...
```

#### 1.3 maven构建的生命周期
清除--> 编译--> 测试--> 报告--> 打包(jar\war)--> 安装--> 部署
1. 清除：`mvn clean`
2. 编译：`mvn compile`
3. 测试：`mvn test`
4. 打包：`mvn package`
5. 安装：`mvn install`
6. 部署：`mvn deploy`

#### 1.4 MAVEN优点
1. 模块化项目
    - 项目非常大时，可借助Maven将一个项目拆分成多个工程，最好是一个模块对应一个工程，利于分工协作。而且模块可以通信。
2. 实现Jar包共享
    - 借助Maven，可将jar包仅仅保存在“仓库”中，有需要该文件时，就引用该文件接口，不需要复制文件过来占用空间。
3. jar包的依赖
    - 借助Maven可以以规范的方式下载jar包，因为所有的知名框架或第三方工具的jar包已经按照统一的规范存放到了Maven的中央仓库中。
4. jar包的自动导入
    - 通过xml定义引入jar包，Maven会自动导入jar包及其依赖jar包进来。

#### 1.5 MAVEN工具
- 可以命令行使用，也可以结合Eclipse和Idea使用
- 简化项目搭建、编译、打包、发布等工作



### 2. SpringBoot基础
- SpringBoot是对**Spring框架的封装**，用于**简化**Spring应用搭建和开发过程。
- SpringBoot是pivotal公司产品、SpringCloud也是。

#### 2.1 SpringBoot典型特点：
- 去除XML配置，完全采用Java配置方式
- 内置tomcat服务器
- 利用自动配置创建很多对象（DataSource、JdbcTemplate、DispatcherServlet等）
- 提供一系列启动器（jar包集合）
- 采用properties或yml做配置文件
- 应用采用jar包发布

#### 2.2 SpringBoot程序构成
- 创建工程，导入boot启动器（jar包）
- `spring-boot-starter` (核心、包含ioc、yml、自动配置、Log日志)
- `spring-boot-starter-parent`（包含参数设置、文件编码、jdk版本等）
- `spring-boot-starter-jdbc`（包含连接池、jdbcTemplate等）
- `spring-boot-starter-web`（包含mvc、restful、tomcat等）
- `spring-boot-starter-test`（包含junit、spring-test等）
- 添加配置文件`application.properties`或`application.yml`

#### 2.3 SpringBoot配置文件
application.properties
``` properties
spring.datasource.username=root
spring.datasource.password=123456
server.port=8888
```

application.yml
``` yml
spring:
 datasource:
  username: root
  password: 123456
server:
 port: 8888
```


#### 2.4 SpringBoot启动类
定义启动类，通过main方法启动
``` java
@SpringBootApplication
public class Xxxx{
    public static void main(String[] args){
        SpringApplication.run(Xxxx.class);
    }
}
```

#### 2.5 SpringBoot数据库访问
在pom.xml定义spring-boot-starter-jdbc、mysql驱动包
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
        <version>2.0.5.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
</dependencies>
```

在application.properties定义数据库连接参数
``` properties
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql://localhost:3306/ydma
spring.datasource.driverClassName=com.mysql.jdbc.Driver
```

定义启动类，内部会根据自动配置机制生成DataSource和JdbcTemplate
``` java
@SpringBootApplication
public class RunBoot {
    public static void main(String[] args) throws SQLException {
        //ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        ApplicationContext ctx = 
                SpringApplication.run(RunBoot.class, args);
        DataSource ds = ctx.getBean(DataSource.class);
        System.out.println(ds.getConnection());
        JdbcTemplate template = ctx.getBean(JdbcTemplate.class);
        System.out.println(template);
        String sql = "insert into paper_score (total_score,my_score,user_id) values (?,?,?)";
        Object[] params = {100,90,1};
        template.update(sql,params);
    }
}
//提示：DataSource和JdbcTemplate都是基于自动配置机制产生，直接注入使用即可。
```

#### 2.6 打包发布SpringBoot程序：
1. 在pom.xml定义spring-boot-maven-plugin插件
``` xml
<build>
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
</plugins>
</build>
```

2. 点击工程右键选择run as- maven build ...
3. 执行完毕后会在项目target目录下生成一个jar包，该包就是发布包
    - 可以采用java -jar xxxx.jar命令启动

> 提示：eclipse设置jdk必须指向到JDK路径，不要JRE路径。




### 3. SpringBoot启动过程
1. 调用SpringApplication的静态的run方法启动
2. 静态的run方法调用SpringApplication对象的run方法
    - (SpringApplication对象创建时加载spring.factories文件中Initializer和Application Listeners组件，判断程序类型servlet、reactive、default)
3. 对象的run方法会创建Spring的ApplicationContext容器对象
    - 获取启动Listener组件
    - 获取environment环境参数
    - 获取启动Logo信息Banner
    - 根据程序类型不同创建不同类型的ApplicationContext对象
    - 将Listener、environment、banner设置到ApplicationContext容器对象中
    - 为ApplicationContext容器对象加载程序中各种Bean组件
    - 开始执行启动任务ApplicationRunner、CommandLineRunner等
    - 返回ApplicationContext容器对象

### 4. @SpringBootApplication作用
SpringApplication.run方法在启动中，加载一个带有@SpringBootApplication标记的参数，该标记具有以下几种功能。

#### 4.1 @SpringBootConfiguration（SpringBoot Bean定义）
- spring中bean定义`<bean id="" class="">`
- SpringBoot通过`@Bean、@Primary`标记定义。

案例代码：
``` java
@SpringBootConfiguration//开启Bean定义功能
public class BeanConfiguration {
    @Bean//将返回的UserDao对象放入Spring容器，默认方法名为id
    public UserDao userdao() {
        return new UserDao();
    }
    @Bean("dao2")//将返回的UserDao对象放入Spring容器，指定id为dao2
    @Primary//默认注入该对象
    public UserDao userdao1() {
        return new UserDao();
    }
    @Bean("userService")
    public UserService userService() {
        return new UserService();
    }
}
```
> @SpringBootConfiguration标记是对Spring的@Configuration封装，所以直接用@Configuration也可以。

#### 4.2 @ComponentScan（SpringBoot组件扫描）
- spring中组件扫描`<context:component-scan base-package=""/>`
- SpringBoot通过@ComponentScan
    1. 扫描指定包路径组件，带@Controller、@Service、@Repository、@Component注解标记组件
        + `@ComponentScan(basePackages= {"cn.xdl.dao","cn.xdl.service"})`
    2. 扫描cn.xdl包及子包下的组件
        + `@ComponentScan(basePackages="cn.xdl")`
    3. 扫描当前包及子包下的组件
        + `@ComponentScan`
    4. 扫描当前包及子包组件，并且将DeptService组件纳入
        + `@ComponentScan(includeFilters= {@Filter(type=FilterType.ASSIGNABLE_TYPE,classes=DeptService.class)})`
    5. 扫描当前包及子包组件，带有@Controller、@Service...、@MyComponent注解有效
        + `@ComponentScan(includeFilters= {@Filter(type=FilterType.ANNOTATION,classes=MyComponent.class)})`

#### 4.3 @EnableAutoConfiguration（SpringBoot自动配置）
自动配置机制是SpringBoot框架特有功能，能在启动后自动创建一些常用对象，例如DataSource、JdbcTemplate等。
- 自动配置原理：
    1. 在xxx-autoconfigure.jar包中META-INF目录下有一个spring.factories文件，其中定义了大量的XxxAutoConfiguration配置组件。当开启@EnableAutoConfiguration标记时，标记内部会触发AutoConfigurationImportSelector组件调用SpringFactoriesLoader加载spring.factories文件。
    2. 自动配置组件就是采用@Configuration+@Bean+@Primary标记事先定义好的配置组件，通过Boot启动自动去spring.factories文件加载，然后在Spring容器中创建出约定对象。

``` java
DataSourceAutoConfiguration//创建dataSource对象
JdbcTemplateAutoConfiguration//创建jdbcTemplate
DispatcherServletAutoConfiguration//创建DispatcherServlet对象
RedisAutoConfiguration//创建RedisTemplate对象
```

- 通过自动配置机制创建DataSource对象
    1. 引入spring-boot-starter-jdbc（hikari）、驱动包
    2. 在application.properties文件追加db参数
    3. 在启动类使用@EnableAutoConfiguration标记
        + DataSourceAutoConfiguration默认会创建Hikari、tomcat、dbcp2连接池对象，优先级hikari最高，依次tomcat、dbcp2.
        + 如果通过spring.datasource.type属性指定其他类型连接池组件，SpringBoot可以按指定类型创建连接池。
``` properties
spring.datasource.type=org.apache.commons.dbcp2.BasicDataSource
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
```

#### 4.4 MAVEN如何排除某个jar包（扩展）
在引入spring-boot-starter-jdbc启动器时，由于jar包依赖会自动引入HikariCP，可以通过< exclusion>标记排除依赖。
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

