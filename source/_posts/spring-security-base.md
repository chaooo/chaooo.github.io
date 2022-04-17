---
title: 「Spring Security」Spring Security 基础入门示例
date: 2021-11-25 15:01:00
tags: [后端开发, SpringSecurity, 安全认证]
categories: 安全认证
---

`Spring Security` 是一个功能强大且高度可定制的身份验证和访问控制的安全框架。它是 `Spring` 应用程序在安全框架方面的公认标准。

其核心特性包括：认证和授权、常规攻击防范、与 `Servlet` 接口集成、与 `Spring MVC` 集成等。

常规攻击防范在 `Spring Security` 安全框架中是默认开启的，常见的威胁抵御方式有：防止伪造跨站请求（`CSRF`），安全响应头（`HTTP Response headers`），`HTTP`通讯安全等<!-- more -->

### 1. 入门示例
新建 `SpringBoot` 项目，在 `pom.xml` 中增加 `Spring Security` 依赖：
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

> 只要加入依赖，项目的所有接口都会被自动保护起来。

创建一个 Controller：
``` java
@RestController
public class HelloApi {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```
导入`spring-boot-starter-security`启动后，`Spring Security`已经生效，默认拦截全部请求，如果用户没有登录，跳转到内置登录页面。

访问`/hello`接口 ，需要登录之后才能访问。

![](up-44bc65394ab67d8d0445bc7a0dc9913df6a.webp)

默认配置下，会自动生成一个 `user` 用户，并分配其随机密码，密码可以从控制台的日志信息中找到：

![](up-2a23b6505fa1faae06a8f1f25f910f979b7.webp)


### 2. Spring Security默认配置项
`Spring Boot` 引入 `Spring Security` 启动后，将会自动开启如下配置项：
- 默认开启一系列基于 `springSecurityFilterChain` 的 `Servlet` 过滤器，包含了几乎所有的安全功能，例如：保护系统 `URL`、验证用户名、密码表单、重定向到登录界面等；
- 创建 `UserDetailsService` 实例，并生成随机密码，用于获取登录用户的信息详情；
- 将安全过滤器应用到每一个请求上。

除此之外，`Spring Security` 还有一些其他可配置的功能：
- 限制所有访问必须首先通过认证；
- 生成默认登录表单；
- 创建用户名为 `user` 的可以通过表单认证的用户，并为其初始化密码；
- 使用 `BCrypt` 方式加密密码；
- 提供登出的能力；
- 保护系统不受 `CSRF` 攻击；
- 会话固定保护；
- 集成安全消息头；
- 提供一些默认的 `Servlet` 接口，如：「getRemoteUser」、「getUserPrincipal」、「isUserInRole」、「login」和「logout」。


### 3. 配置文件定义用户信息
也可以直接在 `application.yml` 配置文件中配置用户的基本信息：
``` ymal
spring:
  security:
    user:
      name: user
      password: 123
```
配置完成后，重启项目，就可以使用这里配置的用户名/密码登录了。


### 4. Java代码定义用户信息
如果需要自定义逻辑时，只需要实现`UserDetailsService`接口即可。
``` java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    @Resource
    private PasswordEncoder encoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 判断用户名是否存在(用户名也可查询数据库)，如果不存在抛出UsernameNotFoundException
        if(!username.equals("admin")){
            throw new UsernameNotFoundException("用户名不存在");
        }
        // 把查询出来的密码进行解析,或直接把password放到构造方法中。
        // 理解:password就是数据库中查询出来的密码，查询出来的内容不是123
        String password = encoder.encode("123");

        return new User(username,password, AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```
返回值 `UserDetails` 是一个接口，要想返回 `UserDetails` 的实例就只能返回接口的实现类。
这里使用 `Spring Security` 中提供的 `org.springframework.security.core.userdetails.User`。

`Spring Security` 要求：当进行自定义登录逻辑时容器内必须有 `PasswordEncoder` 实例。

客户端密码和数据库密码是否匹配是由 `Spring Security` 去完成的，但 `Security` 中没有默认密码解析器。所以当自定义登录逻辑时要求必须给容器注入`PaswordEncoder`的`bean`对象。
``` java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder getPasswordEncoder(){
        /*
         * BCryptPasswordEncoder是spring security官方推荐的解析器，
         * 它是对bcrypt强散列方法的具体实现，基于Hash算法实现的单向加密。
         * 可以通过strength控制加密强度，默认为10，长度越长安全性越高。
         */
        return new BCryptPasswordEncoder();
    }

}
```
重启项目后，在浏览器中输入账号：`admin`，密码：`123`，登录后就可以访问接口了。
