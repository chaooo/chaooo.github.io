---
title: 【安全认证】Shiro安全框架入门
date: 2019-12-22 20:44:40
tags: [后端开发, 安全认证]
categories: 安全认证
---


### 1. 初识Shiro
`Apache Shiro`是一个强大易用的Java安全框架，提供了认证、授权、加密、会话管理、与Web集成、缓存等。
- 具体来说，满足对如下元素的支持：
    + 用户，角色，权限(仅仅是操作权限，数据权限必须与业务需求紧密结合)，资源(url)。
    + 用户分配角色，角色定义权限。
    + 访问授权时支持角色或者权限，并且支持多级的权限定义。<!-- more -->

- Shiro作为一个完善的权限框架，可以应用在多种需要进行身份认证和访问授权的场景，例如：`独立应用`、`web应用`、`spring框架中集成`等。


### 2. Shiro整体架构
在shiro架构中，有3个最主要的组件：`Subject`，`SecurityManager`，`Realm`。

![](http://cdn.chaooo.top/Java/Shiro.png)

1. **`Subject`**(如图上层部分)："操作用户(**主体**)"，本质上就是当前访问用户的抽象描述。
2. **`SecurityManager`**(如图中层部分)：是Shiro架构中最核心的组件(**控制器**)，通过它可以协调其他组件完成用户认证和授权。
    + `Authenticator`：认证器，协调一个或者多个Realm，从Realm指定的数据源取得数据之后进行执行具体的认证。
    + `Authorizer`：授权器，用户访问控制授权，决定用户是否拥有执行指定操作的权限。
    + `Session Manager`：Session管理器，Shiro自己实现了一套Session管理机制。
    + `Session DAO`：实现了Session的操作，主要有增删改查。
    + `CacheManager`：缓存管理器，缓存角色数据和权限数据等。
    + `Pluggable Realms`：数据库与数据源之间的一个桥梁。Shiro获取认证信息、权限数据、角色数据 通过Realms来获取。
    + `Cryptography`：是用来做加解密，能非常快捷的实现数据加密。
3. **`Realm`**(如图下层部分)：定义了访问数据的方式，用来连接不同的**数据源**，如：LDAP，关系数据库，配置文件等等。


### 3. Shiro认证与授权
#### 3.1 Shiro认证
【创建`SecurityManager`】>【主体提交请求】>【`SecurityManager`调用`Authenticator`去认证】>【`Realm`验证】
+ 操作用户（主体）提交请求到Security Manager调用Authenticator去认证,Authenticator通过Pluggable Realms去获取认证信息，Pluggable Realms是从下面的数据源（数据库）中去获取的认证信息，然后用通过Pluggable Realms从数据库中获取的认证信息和主体提交过来的认证数据做比对。

``` java
/**
 * Shiro认证 测试
 */
public class AuthenticationTest {
    // 构建一个简单的数据源
    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();
    @Before
    public void addUser(){
        // 参数分别为：用户名，密码，权限...
        simpleAccountRealm.addAccount("chaooo", "123456", "admin","user");
    }
    /**
     * 认证测试方法
     */
    @Test
    public void testAuthentication(){
        // 1.构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);
        // 2. 主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();
        // 3. 调用Subject.login(token)方法开始用户认证流程
        UsernamePasswordToken token = new UsernamePasswordToken("chaooo", "123456");
        try {
        　　subject.login(token);
        } catch (UnknownAccountException e) {
        　　logger.error(String.format("用户不存在: %s", username), e);
        } catch (IncorrectCredentialsException e) {
        　　logger.error(String.format("密码不正确: %s", username), e);
        } catch (ConcurrentAccessException e) {
        　　logger.error(String.format("用户重复登录: %s", username), e);
        } catch (AccountException e) {
        　　logger.error(String.format("其他账户异常: %s", username), e);
        }
    }
}
```


#### 3.2 Shiro授权
shiro访问授权有3种实现方式：**`api`调用**，**`java`注解**，**`jsp`标签**。
1. 通过api调用实现:【创建`SecurityManager`】>【主体授权】>【`SecurityManager`调用`Authorizer`授权】>【`Realm`获取角色权限数据】
    + 大体上和认证操作一样，也是通过Pluggable Realms从下面的数据源（数据库）中去获取权限数据,角色数据。

``` java
// 在执行访问授权验证之前，必须执行用户认证
// 角色验证
Subject subject = SecurityUtils.getSubject();
if(subject.hasRole("admin")) {
　　//用户属于角色admin
}else{
　　//用户不属于角色admin
}
// subject.checkRoles("admin","user");同时check多个角色
// 权限验证
String perm = "log:manage:*";
if(subject.isPermitted(perm)) {
　　logger.info(String.format("用户： %s 拥有权限：%s", name, perm));
}else {
　　logger.error(String.format("用户：%s 没有权限：%s", name, perm));
}
```

2. 在spring框架中可以通过java注解

``` java
@RequiresPermissions(value={"log:manage:*"})
public ModelAndView home(HttpServletRequest req) {
　　ModelAndView mv = new ModelAndView("home");
　　return mv;
}
```

3. 在JSP页面中还可以直接使用jsp标签

``` xml
<!-- 使用shiro标签 -->
<shiro:hasPermission name="log:manage:*">
　　<a href="<%=request.getContextPath()%>/user/home">操作日志审计</a><br/>
</shiro:hasPermission>
```


#### 3.3 Quickstart
1. 新建一个`Maven`项目，`pom`导入`jar`包:`shiro-all`、`slf4j-api`、`slf4j-log4j12`、`log4j`;
2. `classpath`下新建`shiro.ini`配置文件:

``` ini shiro.ini
# -----------------------------------------------------------------------------
# Users and their assigned roles
#
# Each line conforms to the format defined in the
# org.apache.shiro.realm.text.TextConfigurationRealm#setUserDefinitions JavaDoc
# -----------------------------------------------------------------------------
[users]
# user 'root' with password 'secret' and the 'admin' role
root = secret, admin
# user 'guest' with the password 'guest' and the 'guest' role
guest = guest, guest
# user 'chaooo' with password '123456' and roles 'user' and 'guest'
chaooo = 123456, user, guest

# -----------------------------------------------------------------------------
# Roles with assigned permissions
# 
# Each line conforms to the format defined in the
# org.apache.shiro.realm.text.TextConfigurationRealm#setRoleDefinitions JavaDoc
# -----------------------------------------------------------------------------
[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
# The 'schwartz' role can do anything (*) with any lightsaber:
user = user:*
# The 'goodguy' role is allowed to 'query' (action) the user (type) with license plate 'zhangsan' (instance specific id)
guest = user:query:zhangsan
```

3. 启动运行Quickstart

``` java Quickstart.java
public class Quickstart {
        
    private static final transient Logger log = LoggerFactory.getLogger(Quickstart.class);
        
    public static void main(String[] args) {
        // 构建SecurityManager环境
        DefaultSecurityManager securityManager = new DefaultSecurityManager();
        IniRealm iniRealm = new IniRealm("classpath:shiro.ini");
        securityManager.setRealm(iniRealm);
        SecurityUtils.setSecurityManager(securityManager);
        
        // get the currently executing user:
        // 获取当前的 Subject
        Subject currentUser = SecurityUtils.getSubject();
        
        // Do some stuff with a Session (no need for a web or EJB container!!!)
        // 测试使用 shiro的Session
        Session session = currentUser.getSession();
        session.setAttribute("someKey", "aValue");
        String value = (String) session.getAttribute("someKey");
        if (value.equals("aValue")) {
            log.info("---> Retrieved the correct value! [" + value + "]");
        }
        
        // let's login the current user so we can check against roles and permissions:
        // 测试当前的用户是否已经被认证. 即是否已经登录.
        // 调动 Subject 的 isAuthenticated()
        if (!currentUser.isAuthenticated()) {
            // 把用户名和密码封装为 UsernamePasswordToken 对象
            UsernamePasswordToken token = new UsernamePasswordToken("chaooo", "123456");
            // rememberme
            token.setRememberMe(true);
            try {
                // 执行登录.
                currentUser.login(token);
            }
            // 若没有指定的账户, 则 shiro 将会抛出 UnknownAccountException 异常.
            catch (UnknownAccountException uae) {
                log.info("----> There is no user with username of " + token.getPrincipal());
                return;
            }
            // 若账户存在, 但密码不匹配, 则 shiro 会抛出 IncorrectCredentialsException 异常。
            catch (IncorrectCredentialsException ice) {
                log.info("----> Password for account " + token.getPrincipal() + " was incorrect!");
                return;
            }
            // 用户被锁定的异常 LockedAccountException
            catch (LockedAccountException lae) {
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            }
            // ... catch more exceptions here (maybe custom ones specific to your application?
            // 所有认证时异常的父类.
            catch (AuthenticationException ae) {
                //unexpected condition?  error?
            }
        }
        
        //say who they are:
        //print their identifying principal (in this case, a username):
        log.info("----> User [" + currentUser.getPrincipal() + "] logged in successfully.");

        //test a role:
        // 测试是否有某一个角色. 调用 Subject 的 hasRole 方法.
        if (currentUser.hasRole("admin")) {
            log.info("----> May the Admin be with you!");
        } else {
            log.info("----> Hello, mere mortal.");
            return;
        }
        
        //test a typed permission (not instance-level)
        // 测试用户是否具备某一个行为. 调用 Subject 的 isPermitted() 方法。
        if (currentUser.isPermitted("user:query, edit")) {
            log.info("----> You are permitted to 'query' and 'edit' 'user'");
        } else {
            log.info("Sorry, you don't have permission");
        }
        
        //a (very powerful) Instance Level permission:
        // 测试用户是否具备某一个行为. 资源标识符:操作:对象实例ID
        if (currentUser.isPermitted("user:query:zhangsan")) {
            log.info("----> You are permitted to 'delete' 'user' 'zhangsan'");
        } else {
            log.info("Sorry, you don't have permission!");
        }
        
        //all done - log out!
        // 执行登出. 调用 Subject 的 Logout() 方法.
        System.out.println("---->" + currentUser.isAuthenticated());
        currentUser.logout();
        System.out.println("---->" + currentUser.isAuthenticated());
        System.exit(0);
    }
}

```



### 4. 在SpringMVC框架中集成Shiro
#### 4.1 配置Maven依赖
``` xml
<!-- shiro配置 -->
<dependency>
　　<groupId>org.apache.shiro</groupId>
   <artifactId>shiro-core</artifactId>
   <version>${version.shiro}</version>
</dependency>
<!-- Enables support for web-based applications. -->
<dependency>
　　<groupId>org.apache.shiro</groupId>
   <artifactId>shiro-web</artifactId>
   <version>${version.shiro}</version>
</dependency>
<!-- Enables AspectJ support for Shiro AOP and Annotations. -->
<dependency>
　　<groupId>org.apache.shiro</groupId>
   <artifactId>shiro-aspectj</artifactId>
   <version>${version.shiro}</version>
</dependency>
<!-- Enables Ehcache-based famework caching. -->
<dependency>
　　<groupId>org.apache.shiro</groupId>
   <artifactId>shiro-ehcache</artifactId>
   <version>${version.shiro}</version>
</dependency>
<!-- Enables Spring Framework integration. -->
<dependency>
　　<groupId>org.apache.shiro</groupId>
   <artifactId>shiro-spring</artifactId>
   <version>${version.shiro}</version>
</dependency>
```

+ `Shiro`使用了日志框架`slf4j`，因此需要对应配置指定的日志实现组件，如：`log4j`，`logback`等。
    + 在此，以使用`log4j`为日志实现为例：

``` xml
<!--
shiro使用slf4j作为日志框架，所以必需配置slf4j。
同时，使用log4j作为底层的日志实现框架。
-->
<dependency>
　　<groupId>org.slf4j</groupId>
　　<artifactId>slf4j-api</artifactId>
　　<version>1.7.25</version>
</dependency>
<dependency>
　　<groupId>org.slf4j</groupId>
　　<artifactId>slf4j-log4j12</artifactId>
　　<version>1.7.25</version>
</dependency>
<dependency>
　　<groupId>log4j</groupId>
　　<artifactId>log4j</artifactId>
　　<version>1.2.17</version>
</dependency>
```


#### 4.2 集成Shiro
在`Spring`框架中集成`Shiro`，本质上是与`Spring IoC`容器和`Spring MVC`框架集成。
##### 4.2.1 `Shiro`与`Spring IoC`容器集成
+ `Spring IoC`容器提供了一个非常重要的功能，就是依赖注入，将`Bean`的定义以及`Bean`之间关系的耦合通过容器来处理。
+ 也就是说，在`Spring`中集成`Shiro`时，`Shiro`中的相应`Bean`的定义以及他们的关系也需要通过`Spring IoC`容器实现。
+ `Shiro`提供了与`Web`集成的支持，其通过一个`ShiroFilter`入口来拦截需要安全控制的`URL`，然后进行相应的控制。
+ `ShiroFilter`类是安全控制的入口点，其负责读取配置（如`ini`配置文件），然后判断`URL` 是否需要登录/权限等工作。
    + [urls] 部分的配置，其格式是：`url = 拦截器[参数], 拦截器[参数]`
+ `shiro`中默认的过滤器：

|默认拦截器名|拦截器类与说明（括号里的表示默认值）|
|-----------|--------------------------------|
|<span style="white-space:nowrap;">身份验证相关</span>||
|authc|org.apache.shiro.web.filter.authc.FormAuthenticationFilter<br>基于表单的拦截器；如"/**=authc"，如果没有登录会跳到相应的登录页面登录；主要属性：usernameParam：表单提交的用户名参数名（ username）；  passwordParam：表单提交的密码参数名（password）； rememberMeParam：表单提交的密码参数名（rememberMe）；  loginUrl：登录页面地址（/login.jsp）；successUrl：登录成功后的默认重定向地址； failureKeyAttribute：登录失败后错误信息存储key（shiroLoginFailure）；|
|authcBasic|org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter<br>Basic HTTP身份验证拦截器，主要属性：applicationName：弹出登录框显示的信息（application）；|
|logout|org.apache.shiro.web.filter.authc.LogoutFilter<br>退出拦截器，主要属性：redirectUrl：退出成功后重定向的地址（/）;示例"/logout=logout"|
|user|org.apache.shiro.web.filter.authc.UserFilter<br>用户拦截器，用户已经身份验证/记住我登录的都可；示例"/**=user"|
|anon|org.apache.shiro.web.filter.authc.AnonymousFilter<br>匿名拦截器，即不需要登录即可访问；一般用于静态资源过滤；示例"/static/**=anon"|
|授权相关||
|roles|org.apache.shiro.web.filter.authz.RolesAuthorizationFilter<br>角色授权拦截器，验证用户是否拥有所有角色；主要属性：loginUrl：登录页面地址（/login.jsp）；unauthorizedUrl：未授权后重定向的地址；示例"/admin/**=roles[admin]"|
|perms|org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter<br>权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例"/user/**=perms["user:create"]"|
|port|org.apache.shiro.web.filter.authz.PortFilter<br>端口拦截器，主要属性：port（80）：可以通过的端口；示例"/test= port[80]"，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样|
|rest|org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter<br>rest风格拦截器，自动根据请求方法构建权限字符串（GET=read, POST=create,PUT=update,DELETE=delete,HEAD=read,TRACE=read,OPTIONS=read, MKCOL=create）构建权限字符串；示例"/users=rest[user]"，会自动拼出"user:read,user:create,user:update,user:delete"权限字符串进行权限匹配（所有都得匹配，isPermittedAll）；|
|ssl|org.apache.shiro.web.filter.authz.SslFilter<br>SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口（443）；其他和port拦截器一样；|
其他||
|noSessionCreation|org.apache.shiro.web.filter.session.NoSessionCreationFilter<br>不创建会话拦截器，调用 subject.getSession(false)不会有什么问题，但是如果 subject.getSession(true)将抛出 DisabledSessionException异常；|

+ `URL`匹配模式：url模式使用Ant 风格模式
    + Ant 路径通配符支持`?`、`*`、`**`，注意通配符匹配不包括目录分隔符“/”：
    + `?`：匹配一个字符，如/admin? 将匹配/admin1，但不匹配/admin 或/admin/；
    + `*`：匹配零个或多个字符串，如/admin 将匹配/admin、/admin123，但不匹配/admin/1；
    + `**`：匹配路径中的零个或多个路径，如/admin/** 将匹配/admin/a 或/admin/a/b
+ `URL`匹配顺序：URL 权限采取**第一次匹配优先**的方式，即从头开始使用第一个匹配的url模式对应的拦截器链。如：
    + /bb/**=filter1
    + /bb/aa=filter2
    + /**=filter3
    + 如果请求的url是“/bb/aa”，因为按照声明顺序进行匹配，那么将使用filter1 进行拦截，所以通配符一般写在靠后。

``` xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
  <property name="securityManager" ref="securityManager"/>
  <property name="loginUrl" value="/index"/>
  <property name="successUrl" value="/home"/>
  <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
  <!-- The 'filters' property is not necessary since any declared javax.servlet.Filter bean  -->
  <!-- defined will be automatically acquired and available via its beanName in chain        -->
  <!-- definitions, but you can perform instance overrides or name aliases here if you like: -->
  <!-- <property name="filters">
      <util:map>
          <entry key="logout" value-ref="logoutFilter" />
      </util:map>
  </property> -->
  <property name="filterChainDefinitions">
      <value>
          # some example chain definitions:
          # /admin/** = authc, roles[admin]
          # /docs/** = authc, perms[document:read]
          /login = anon
          /logout = anon
          /error = anon
          /** = user
          # more URL-to-FilterChain definitions here
      </value>
  </property>
</bean>
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
  <!-- Single realm app.  If you have multiple realms, use the 'realms' property instead. -->
  <property name="realm" ref="myRealm" />
  <!-- By default the servlet container sessions will be used.  Uncomment this line
       to use shiro's native sessions (see the JavaDoc for more): -->
  <!-- <property name="sessionMode" value="native"/> -->
</bean>
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
<!-- Define the Shiro Realm implementation you want to use to connect to your back-end -->
<!-- security datasource: -->
<bean id="myRealm" class="org.apache.shiro.realm.jdbc.JdbcRealm">
  <property name="dataSource" ref="dataSource"/>
  <property name="permissionsLookupEnabled" value="true"/>
</bean>
<!-- Enable Shiro Annotations for Spring-configured beans.  Only run after -->
<!-- the lifecycleBeanProcessor has run: -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor"/>
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
  <property name="securityManager" ref="securityManager"/>
</bean>
```


##### 4.2.2 与`Spring MVC`集成
+ 跟在普通`Java Web`应用中使用`Shiro`一样，集成`Shiro`到`Spring MVC`时，实际上就是通过在`web.xml`中添加指定`Filter`实现。配置如下：

``` xml
<!-- The filter-name matches name of a 'shiroFilter' bean inside applicationContext.xml -->
<!-- DelegatingFilterProxy作用是自动到Spring 容器查找名字为shiroFilter（filter-name）的bean并把所有Filter 的操作委托给它。 -->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<!-- Make sure any request you want accessible to Shiro is filtered. /* catches all -->
<!-- requests.  Usually this filter mapping is defined first (before all others) to -->
<!-- ensure that Shiro works in subsequent filters in the filter chain:             -->
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> `Spring`中集成`Shiro`的原理就是：通过在`web.xml`中配置的`Shiro Filter`与`Spring IoC`中定义的相应的`Shiro Bean`定义建立关系，从而实现在`Spring`框架集成`Shiro`。


#### 4.3 数据源配置
在`Shiro`中，`Realm`定义了访问数据的方式，用来连接不同的数据源，如：LDAP，关系数据库，配置文件等。
- 以`org.apache.shiro.realm.jdbc.JdbcRealm`为例，将用户信息存放在关系型数据库中。
- 在使用`JdbcRealm`时，必须要在关系型数据库中存在3张表，分别是
    + `users`表，存放认证用户基本信息，在该表中必须存在2个字段：`username`，`password`。
    + `roles_permissions`表，存放角色和权限定义，在该表中必须存在2个字段：`role_name`，`permission`。
    + `user_roles`表，存放用户角色对应关系，在该表中必须存在2个字段：`username`，`role_name`。
- 实际上，在更加复杂的应用场景下，通常需要扩展`JdbcRealm`。


#### 4.4 认证
在`Shiro`中，认证即执行用户登录，读取指定`Realm`连接的数据源，以验证用户身份的有效性与合法性。
- 在shiro中，用户需要提供principals （身份）和credentials（证明）给shiro，从而应用能验证用户身份：
    + principals：身份，即主体的标识属性，可以是任何属性，如用户名、邮箱等，唯一即可。一个主体可以有多个principals，但只有一个Primary principals，一般是用户名/邮箱/手机号。
    + credentials：证明/凭证，即只有主体知道的安全值，如密码/数字证书等。
    + 最常见的principals 和credentials 组合就是用户名/密码了
- 身份认证流程：
    1. 首先调用Subject.login(token) 进行登录，其会自动委托给SecurityManager
    2. SecurityManager负责真正的身份验证逻辑；它会委托给Authenticator 进行身份验证；
    3. Authenticator 才是真正的身份验证者，ShiroAPI 中核心的身份认证入口点，此处可以自定义插入自己的实现；
    4. Authenticator 可能会委托给相应的AuthenticationStrategy进行多Realm 身份验证，默认ModularRealmAuthenticator会调用AuthenticationStrategy进行多Realm 身份验证；
    5. Authenticator 会把相应的token 传入Realm，从Realm 获取身份验证信息，如果没有返回/抛出异常表示身份验证失败了。此处可以配置多个Realm，将按照相应的顺序及策略进行访问。
        + Realm：一般继承AuthorizingRealm（授权）即可；其继承了AuthenticatingRealm（即身份验证），而且也间接继承了CachingRealm（带有缓存实现）

``` java
Subject subject = SecurityUtils.getSubject();
if(!subject.isAuthenticated()) {
　　UsernamePasswordToken token = new UsernamePasswordToken(name, password);
    try {
    　　subject.login(token);
    } catch (UnknownAccountException e) {
    　　logger.error(String.format("用户不存在: %s", token.getPrincipal()), e);
    } catch (IncorrectCredentialsException e) {
    　　logger.error(String.format("密码不正确: %s", token.getPrincipal()), e);
    } catch (ConcurrentAccessException e) {
    　　logger.error(String.format("用户重复登录: %s", token.getPrincipal()), e);
    } catch (AccountException e) {
    　　logger.error(String.format("其他账户异常: %s", token.getPrincipal()), e);
    }
}
```


#### 4.5 授权
Shiro作为权限框架，仅仅只能控制对资源的操作权限，并不能完成对数据权限的业务需求。
- 而对于Java Web环境下Shiro授权，包含两个方面的含义。
    + 其一，对于前端来说，用户只能看到他对应访问权限的元素。
    + 其二，当用户执行指定操作（即：访问某个uri资源）时，需要验证用户是否具备对应权限。
- 对于第一点，在Java Web环境下，通过Shiro提供的JSP标签实现。
- 对于第二点，与在非Java Web环境下一样，需要在后端调用API进行权限（或者角色）检验。
- 在Spring框架中集成Shiro，还可以直接通过Java注解方式实现
- `Permissions`：
    + 规则：`资源标识符：操作：对象实例ID`,即对哪个资源的哪个实例可以进行什么操作.其默认支持通配符权限字符串，: 表示资源/操作/实例的分割；, 表示操作的分割，* 表示任意资源/操作/实例。如：`user:edit:manager`
        - 也可以使用通配符来定义，如：`user:edit:*`、`user:*:*`、`user:*:manager`
        - 部分省略通配符：缺少的部件意味着用户可以访问所有与之匹配的值，比如：`user:edit`等价于`user:edit:*`、`user`等价于`user:*:*`
        - 注意：通配符只能从字符串的结尾处省略部件，也就是说`user:edit`并**不等价**于`user:*:edit`
- 授权流程:
    1. 首先调用Subject.isPermitted*/hasRole* 接口，其会委托给SecurityManager，而SecurityManager接着会委托给Authorizer；
    2. Authorizer是真正的授权者，如果调用如isPermitted(“user:view”)，其首先会通过PermissionResolver把字符串转换成相应的Permission 实例；
    3. 在进行授权之前，其会调用相应的Realm 获取Subject 相应的角色/权限用于匹配传入的角色/权限；
    4. Authorizer 会判断Realm 的角色/权限是否和传入的匹配，如果有多个Realm，会委托给ModularRealmAuthorizer进行循环判断，如果匹配如isPermitted*/hasRole* 会返回true，否则返回false表示授权失败。
        + `ModularRealmAuthorizer`进行多Realm 匹配流程：
            1. 首先检查相应的Realm 是否实现了实现了Authorizer；
            2. 如果实现了Authorizer，那么接着调用其相应的`isPermitted*/hasRole*`接口进行匹配；
            3. 如果有一个Realm匹配那么将返回true，否则返回false。

##### 4.5.1 Shiro标签
1. `<shiro:guest></shiro:guest>`:用户没有身份验证时显示相应信息，即游客访问信息
2. `<shiro:user></shiro:user>`:用户已经经过认证/记住我登录后显示相应的信息。
3. `<shiro:authenticated></shiro:authenticated>`:用户已经身份验证通过，即Subject.login登录成功，**不是记住我登录的**
4. `<shiro:notAuthenticated></shiro:notAuthenticated>`标签：用户未进行身份验证，即没有调用Subject.login进行登录，**包括记住我自动登录**的也属于未进行身份验证。
5. `<shiro:pincipal></shiro:pincipal>`：**显示用户身份信息**，默认调用`Subject.getPrincipal()`获取，即Primary Principal。
6. **`<shiro:hasRole></shiro:hasRole>`**标签：如果当前Subject 有角色将显示body 体内容
7. `<shiro:hasAnyRoles></shiro:hasAnyRoles>`标签：如果当前Subject有任意一个角色（或的关系）将显示body体内容
8. `<shiro:lacksRole></shiro:lacksRole>`：如果当前Subject 没有角色将显示body 体内容
9. **`<shiro:hasPermission></shiro:hasPermission>`**：如果当前Subject 有权限将显示body体内容
10. `<shiro:lacksPermission></shiro:lacksPermission>`：如果当前Subject没有权限将显示body体内容

``` xml
<!-- 在jsp页面中引入shiro标签库 -->
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>
<!-- 权限控制 -->
<shiro:hasRole name="admin">
　　<a>用户管理</a>
</shiro:hasRole>
<shiro:hasPermission name="winnebago:drive:eagle5">
　　<a>操作审计</a>
</shiro:hasPermission>
```


##### 4.5.2 调用API进行权限（或者角色）检验 
``` java
String roleAdmin = "admin";
Subject currentUser = SecurityUtils.getSubject();
if(!currentUser.hasRole(roleAdmin)) {
　　//todo something
}
```


##### 4.5.3 Shiro权限注解
- `@RequiresAuthentication`：表示当前Subject已经通过login 进行了身份验证；即Subject. isAuthenticated() 返回true
- `@RequiresUser`：表示当前Subject 已经身份验证或者通过记住我登录的。
- `@RequiresGuest`：表示当前Subject没有身份验证或通过记住我登录过，即是游客身份。
- `@RequiresRoles(value={“admin”, “user”}, logical= Logical.AND)`：表示当前Subject 需要角色admin 和user
- `@RequiresPermissions(value={“user:a”, “user:b”}, logical= Logical.OR)`：表示当前Subject 需要权限user:a或user:b。
- 通过自定义拦截器可以扩展功能，例如：动态url-角色/权限访问控制的实现、根据Subject 身份信息获取用户信息绑定到Request（即设置通用数据）、验证码验证、在线用户信息的保存等

``` java
@Controller
public class HomeController {
　　@RequestMapping("/home")
　　@RequiresPermissions(value={"log:manage:*"})
　　public ModelAndView home(HttpServletRequest req) {
　　　　ModelAndView mv = new ModelAndView("home");
　　　　return mv;
　　}
}
```


#### 4.6 Spring集成Shiro注意事项
1. `Spring 4.2.0 RELEASE`**`+`** 与 `Spring 4.1.9 RELEASE`**`-`**版本，配置方式有所不同。
2. 虽然`shiro`的注解定义是在`Class`级别的，但是实际验证只能支持方法级别：`@RequiresAuthentication`、`@RequiresPermissions`、`@RequiresRoles`。


### 5. Shiro会话管理
Shiro提供了完整的企业级会话管理功能，不依赖于底层容器（如web容器tomcat），不管JavaSE还是JavaEE环境都可以使用，提供了会话管理、会话事件监听、会话存储/持久化、容器无关的集群、失效/过期支持、对Web 的透明支持、SSO 单点登录的支持等特性。

#### 5.1 会话相关的API
- Subject.getSession()：即可获取会话；其等价于Subject.getSession(true)，即如果当前没有创建Session 对象会创建一个；Subject.getSession(false)，如果当前没有创建Session 则返回null
-  session.getId()：获取当前会话的唯一标识
-  session.getHost()：获取当前Subject的主机地址
-  session.getTimeout() & session.setTimeout(毫秒)：获取/设置当前Session的过期时间
-  session.getStartTimestamp() & session.getLastAccessTime()：获取会话的启动时间及最后访问时间；如果是 JavaSE 应用需要自己定期调用 session.touch() 去更新最后访问时间；如果是 Web 应用，每次进入 ShiroFilter 都会自动调用 session.touch() 来更新最后访问时间。
-  session.touch() & session.stop()：更新会话最后访问时间及销毁会话；当Subject.logout()时会自动调用 stop 方法来销毁会话。如果在web中，调用 HttpSession. invalidate()也会自动调用Shiro Session.stop 方法进行销毁Shiro 的会话
- session.setAttribute(key, val) & session.getAttribute(key) & session.removeAttribute(key)：设置/获取/删除会话属性；在整个会话范围内都可以对这些属性进行操作

#### 5.2 会话监听器
会话监听器(SessionListiner):会话监听器用于监听会话创建、过期及停止事件

#### 5.3 SessionDao
- AbstractSessionDAO 提供了 SessionDAO 的基础实现，如生成会话ID等
- CachingSessionDAO 提供了对开发者透明的会话缓存的功能，需要设置相应的 CacheManager
- MemorySessionDAO 直接在内存中进行会话维护
- EnterpriseCacheSessionDAO 提供了缓存功能的会话维护，默认情况下使用 MapCache 实现，内部使用ConcurrentHashMap 保存缓存的会话。 

#### 5.4 数据表
``` sql
create table sessions (
    id varchar(200),
    session varchar(2000),
    constraint pk_sessions primary key(id)
) charset=utf8 ENGINE=InnoDB;
```

#### 5.5 会话验证
- Shiro 提供了会话验证调度器，用于定期的验证会话是否已过期，如果过期将停止会话
- 出于性能考虑，一般情况下都是获取会话时来验证会话是否过期并停止会话的；但是如在 web 环境中，如果用户不主动退出是不知道会话是否过期的，因此需要定期的检测会话是否过期，Shiro 提供了会话验证调度器SessionValidationScheduler
- Shiro 也提供了使用Quartz会话验证调度器：QuartzSessionValidationScheduler


### 6. Shiro缓存
- CacheManagerAware 接口
    + Shiro 内部相应的组件（DefaultSecurityManager）会自动检测相应的对象（如Realm）是否实现了CacheManagerAware 并自动注入相应的CacheManager。
- Realm 缓存
    + Shiro 提供了 CachingRealm，其实现了CacheManagerAware 接口，提供了缓存的一些基础实现；
    + AuthenticatingRealm 及 AuthorizingRealm 也分别提供了对AuthenticationInfo 和 AuthorizationInfo 信息的缓
存。
- Session 缓存
    + 如 SecurityManager 实现了 SessionSecurityManager，其会判断 SessionManager 是否实现了acheManagerAware 接口，如果实现了会把CacheManager 设置给它。
    + SessionManager 也会判断相应的 SessionDAO（如继承自CachingSessionDAO）是否实现了CacheManagerAware，如果实现了会把 CacheManager设置给它
    + 设置了缓存的 SessionManager，查询时会先查缓存，如果找不到才查数据库。
- RememberMe
    + Shiro 提供了记住我（RememberMe）的功能，比如访问如淘宝等一些网站时，关闭了浏览器，下次再打开时还是能记住你是谁，下次访问时无需再登录即可访问，基本流程如下：
        1. 首先在登录页面选中 RememberMe 然后登录成功；如果是浏览器登录，一般会把 RememberMe 的Cookie 写到客户端并保存下来；
        2. 关闭浏览器再重新打开；会发现浏览器还是记住你的；
        3. 访问一般的网页服务器端还是知道你是谁，且能正常访问；
        4. 但是比如我们访问淘宝时，如果要查看我的订单或进行支付时，此时还是需要再进行身份认证的，以确保当前用户还是你。
- 认证和记住我
    + subject.isAuthenticated() 表示用户进行了身份验证登录的，即使有 Subject.login 进行了登录；
    + subject.isRemembered()：表示用户是通过记住我登录的，此时可能并不是真正的你（如你的朋友使用你的电脑，或者你的cookie 被窃取）在访问的
    + 两者二选一，即 subject.isAuthenticated()==true，则subject.isRemembered()==false；反之一样。
- 建议
    + 访问一般网页：如个人在主页之类的，我们使用user 拦截器即可，user 拦截器只要用户登录(isRemembered() || isAuthenticated())过即可访问成功；
    + 访问特殊网页：如我的订单，提交订单页面，我们使用authc 拦截器即可，authc 拦截器会判断用户是否是通过Subject.login（isAuthenticated()==true）登录的，如果是才放行，否则会跳转到登录页面叫你重新登录。
- 实现
    + 如果要自己做RememeberMe，需要在登录之前这样创建Token：UsernamePasswordToken(用户名，密码，是否记住我)，且调用UsernamePasswordToken 的：token.setRememberMe(true); 方法


> 参考文章：
> - [细说shiro之一：shiro简介](https://www.cnblogs.com/nuccch/p/6775855.html)
> - [细说shiro之五：在spring框架中集成shiro](https://www.cnblogs.com/nuccch/p/6775855.html)