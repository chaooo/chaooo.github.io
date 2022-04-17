---
title: 「Spring Security」安全架构与认证鉴权原理
date: 2021-11-29 14:01:00
tags: [后端开发, SpringSecurity, 安全认证]
categories: 安全认证
---

### 1. Spring Security Servlet 安全架构
`Spring Security` 设计的 `Servlet` 安全从架构上分为三个层次，分别是「认证」、「鉴权」、「入侵防护」。通过**过滤器机制**将安全逻辑应用到 `Servlet` 项目。

请求的接收和处理是通过一个一个的过滤器顺序执行实现的，过滤器是 `Servlet` 项目处理请求的基础。

`Spring` 将自己体系内的过滤器交由「过滤器代理`FilterChainProxy`」管理，`FilterChainProxy` 同样也是一个过滤器，被封装在 `Spring` 的「过滤器委托代理`DelegatingFilterProxy`」中。
`Spring Security` 在 `FilterChainProxy` 中加入了「安全过滤器链`SecurityFilterChain`」实现安全保护功能。<!-- more -->

其过程如图：
![](up-0190e15403889a4a54ded3892f2a2d7cd18.webp)

安全过滤器链（`SecurityFilterChain`）的特点：
- 为所有 `Spring Security` 支持的 `Servlet` 指明了起点；
- 对于一些后台操作，可以提升执行效率；
- 在 `Servlet` 容器中，过滤器的选择是由 `URL` 决定的，如此便可针对不同 `URL` 指定相互独立的安全策略。

#### 1.1 安全过滤器 Filter
`Spring Security` 内置了 `33` 种安全过滤器，每个过滤器有固定的顺序及应用场景；内置过滤器的参数设置通过 `HttpSecurity` 类相应的配置方法完成。

在认证与授权中关键的三个过滤器：
1. `UsernamePasswordAuthenticationFilter`：该过滤器用于拦截我们表单提交的请求（默认为/login），进行用户的认证过程。
2. `FilterSecurityInterceptor`：该过滤器主要用来进行授权判断。
3. `ExceptionTranslationFilter`：该过滤器主要用来捕获处理`spring security`抛出的异常，异常主要来源于`FilterSecurityInterceptor`。

`Spring Security` 的认证、授权异常在过滤器校验过程中产生，并在 `ExceptionTranslationFilter` 中接收并进行处理，
1. `ExceptionTranslationFilter` 过滤器首先像其他过滤器一样，调用过滤器链的执行方法 `FilterChain.doFilter(request, response)` 启动过滤处理；
2. 如果当前的用户没有通过认证或者因为其他原因在执行过程中抛出了 AuthenticationException 异常，此时将开启「认证流程」：
    - 清空 `SecurityContextHolder` 对象；
    - 并将原始请求信息「`request`」保存到 `RequestCache` 对象中；
    - 使用 `AuthenticationEntryPoint` 对象存储的认证地址，向客户端索要身份证明。例如，使用浏览器登录的用户，将浏览器地址重定向到 /login 或者回传一个 WWW-Authenticate 认证请求头。
3. 如果当前用户身份信息已确认，但是没有访问权限，则会产生 `AccessDeniedException` 异常，然后访问被拒绝。继续执行拒绝处理 `AccessDeniedHandler`。


#### 1.2 自定义过滤器 Filter
在 `HttpSecurity` 对象中增加自定义 `Filter` 可用于实现认证方式的扩展等场景，扩展 `Filter` 需要实现 `javax.servlet.Filter` 接口；并且需要指定新过滤器的位置。

例如，扩展自定义接口 SimpleFilter。
1. 自定义接口类

``` java
public class SimpleFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("In SimpleFilter");
    }
}
```

2. 加入到指定位置，比如加在 UsernamePasswordAuthenticationFilter 之前

``` java
http.addFilterBefore(new SimpleFilter(), UsernamePasswordAuthenticationFilter.class);
```

### 2. Spring Security 认证
#### 2.1 Spring Security 基本认证组件

<table>
    <thead><tr><th>组别</th><th>组件名</th><th>简述</th></tr></thead>
    <tbody>
        <tr><td rowspan="4">存储单元</td><td>Authentication</td><td>维护用户用于认证的信息</td></tr>
        <tr><td>GrantedAuthority</td><td>认证用户的权限信息比如角色、范围等等</td></tr>
        <tr><td>SecurityContextHolder</td><td>用于维护 SpringContext</td></tr>
        <tr><td>SecurityContext</td><td>用来存储当前认证用户的信息</td></tr>
        <tr><td rowspan="4">认证管理</td><td>AuthenticationManager</td><td>SpringSecurity 向外提供的用于认证的 API 集合</td></tr>
        <tr><td>ProviderManager</td><td>AuthenticationManager 的常见实现类</td></tr>
        <tr><td>AuthenticationProvider</td><td>用于 ProviderManager 提供认证实现</td></tr>
        <tr><td>AuthenticationEntryPoint</td><td>用于获取用户认证信息</td></tr>
        <tr><td>流程管理</td><td>AbstractAuthenticationProcessingFilter</td><td>是认证过滤器的基础，用于组合认证流程</td></tr>
    </tbody>
</table>


#### 2.2 存储单元
![](up-911c8967517a976252c03eb5d98f9a97151.webp)

1. `SecurityContextHolder` 对象是整个 `Spring Security` 体系的核心，它维护着 `SecurityContext` 对象。它是唯一的。
2. `SecurityContext` 对象用于衔接 `SecurityContextHolder` 和 `Authentication` 对象，是对 `Authentication` 的外层封装。
3. `Authentication` 是用户的认证信息。

`Authentication`对象有三个核心属性：
- **principal**：用户的身份信息；
- **credentials**：用户的认证凭据，比如密码，通常情况下，当用户完成认证后，此项内容就会被清空；
- **authorities**：用户的权限，用于更高层次的鉴权功能，通常包括角色、使用范围等信息。该属性基本由 `GrantedAuthority` 实现。`GrantedAuthority` 是在前述 `Authentication` 对象中所指的权限信息。在开发过程中，可以通过 `Authentication.getAuthorities()` 方法获取。权限信息通常包括角色、范围，或者其他扩展内容。

`Authentication` 两个主要作用：
1. 为 `AuthenticationManager` 对象提供用于认证的信息载体；
2. 用于获取某个用户的基本信息。


#### 2.3 认证管理
1. `AuthenticationManager` 为 `Spring` 过滤器提供认证支持 `API`。`AuthenticationManager` 的实现形式并没有严格限制，通常情况下使用 `ProviderManager`。
2. `ProviderManager` 是 `AuthenticationManager` 的最常用的实现类，它包含了一系列的 `AuthenticationProvider` 对象，用以判断认证流程是否完成、认证结构是否成功。
3. `AuthenticationProvider`：每个 `ProviderManager` 可以包含多个 `AuthenticationProvider` ，每个 `AuthenticationProvider` 提供一种认证类型，例如：`DaoAuthenticationProvider` 可以完成「用户名 / 密码」的认证，`JwtAuthenticationProvider` 用于完成 JWT 方式的认证。
4. `AuthenticationEntryPoint` 在当一个请求包含的认证信息不全时，比如未认证终端访问受保护资源时发挥作用，如跳转到登录页面、返回认证要求等。


#### 2.4 流程管理
`AbstractAuthenticationProcessingFilter` 是所有认证过滤器的基类。
1. 当用户提交认证信息，AbstractAuthenticationProcessingFilter 首先从请求信息（例如用户名、密码）中创建 Authentication 对象；
2. 将 Authentication 对象传递给 AuthenticationManager 对象，用于后续认证；
3. 如果认证失败，则执行失败流程：
    - 清空 SecurityContextHolder 对象；
    - 触发 RememberMeServices.loginFail 方法；
    - 触发 AuthenticationFailureHandler。
4. 如果认证成功，则执行成功流程：
    - SessionAuthenticationStrategy 登记新的登录；
    - 将 Authentication 对象设置到 SecurityContextHolder 对象中，并将 SecurityContext 对象保持到 Session 中；
    - 调用 RememberMeServices.loginSuccess 方法；
    - ApplicationEventPublisher 发起事件 InteractiveAuthenticationSuccessEvent


### 3. Spring Security 鉴权
Spring Security 包含**确认身份**和**确认身份的可执行操作**两部分，前者为认证(`Authentication`)，后者即为鉴权(`Authorization`)；

#### 3.1 权限
`Spring Security` 的权限默认是以字符串形式存储的权限信息，比如角色名称、功能名称等；

在用户身份信息得到确认后，`Authentication` 中会存储一系列的 `GrantedAuthority` 对象，这些对象用来判断用户可以使用哪些资源。

`GrantedAuthority` 对象通过 `AuthenticationManager` 插入到 `Authentication` 对象中，并被 `AccessDecisionManager` 使用，判断其权限。

`GrantedAuthority` 是一个接口，其仅包含一个 `getAuthority()` 方法，返回一个字符串值，该值作为权限的描述，当权限较为复杂，该方法需要返回 null，此时 `AccessDecisionManager` 会根据 `getAuthority()` 返回值情况判断是否要进行特殊处理。

`SimpleGrantedAuthority` 是 `GrantedAuthority` 的一个基础实现类，可以满足一般的业务需求。


#### 3.2 鉴权
- 前置鉴权：由 `AccessDecisionManager` 对象判断其是否允许继续执行； 权限判断发生在方法被调用前，或者 WEB 请求之前。不满足抛出 `AccessDeniedException` 异常。
- 后置鉴权：通过 `AfterInvocationManager` 进行管理；后置鉴权在资源被访问后，根据权限的判定来修改返回的内容，或者返回 `AccessDeniedException`。

前置鉴权 `AccessDecisionManager` 对象由 `AbstractSecurityInterceptor` 发起调用，其职责是给出资源是否能被访问的最终结果；

- `AccessDecisionManager` 包含三个主要方法：
    - `boolean supports(ConfigAttribute attribute);`：判断配置属性是否可被访问；
    - `boolean supports(Class clazz);`：判断安全对象的类型是否支持被访问；
    - `void decide(Authentication authentication, Object secureObject,Collection<ConfigAttribute> attrs) throws AccessDeniedException;`：通过认证信息、安全对象、权限信息综合判断安全对象是否允许被访问。

`Spring Security` 内置了以「**投票**」为判定方法的鉴权策略。`Spring Security` 的鉴权策略可以由用户自己实现。

**投票策略**下，`AccessDecisionManager` 控制着一系列的 `AccessDecisionVoter` 实例，判断权限是否满足，如果不满足抛出 `AccessDeniedException` 异常。
- `AccessDecisionVoter` 也包含三个方法：
    - `boolean supports(ConfigAttribute attribute);`：判断配置属性是否支持；
    - `boolean supports(Class clazz);`：判断类型是否支持；
    - `int vote(Authentication authentication, Object object, Collection<ConfigAttribute> attrs);`：根据认证信息对安全资源进行投票。

投票鉴权分为三类：
- 基于角色的投票：`RoleVoter`；
- 基于认证信息的投票：`AuthenticatedVoter`，主要区分认证用户、匿名用户等；
- 自定义投票策略。


#### 3.3 Servlet 请求鉴权流程
`Servlet` 鉴权主要围绕着 `FilterSecurityInterceptor` 类展开，该类作为一个安全过滤器，被放置在 `FilterChainProxy` 中。

具体流程如下：
1. `FilterSecurityInterceptor` 从 `SecurityContextHolder` 中获取 `Authentication` 对象；
2. `FilterSecurityInterceptor` 从 `HttpServletRequest`、`HttpServletREsponse`、 `FilterChain` 中创建 `FilterInvocation` 对象；
3. 将创建的 `FilterInvocation` 对象传递给 `SecurityMetadataSource` 用来获取 `ConfigAttribute` 对象集合；
4. 最后，将 `Authentication`、`FilterInvocation` 和 `ConfigAttribute` 对象传递给 `AccessDecisionManager` 实例验证权限：
    - 如果验证失败，将抛出 `AccessDeniedException` 异常，并由 `ExceptionTranslationFilter` 接收并处理；
    - 如果验证通过，`FilterSecurityInterceptor` 将控制权交还给 `FilterChain`，使程序继续执行。
