---
title: 「SpringCloud」Gateway网关
date: 2021-10-29 15:26:23
tags: [后端开发, SpringCloud, Gateway]
categories: SpringCloud
---

### 1. Gateway简介

Spring Cloud Gateway 是基于 Spring5.0、SpringBoot2.0 和 Project Reactor 开发的网关，旨在提供一种简单而有效的方式来对API进行路由，基于过滤器链的方式提供：安全，监控/埋点，和限流。

Spring Cloud Gateway 基于 Spring Boot2.x、Spring WebFlux 和 Project Reactor构建，属于异步非阻塞模型。<!-- more -->

### 1.1 核心概念

路由（Route）：路由是网关最基础的部分，路由信息由ID、目标URI、一组断言和一组过滤器组成。如果断言路由为真，则说明请求的 URI 和配置匹配。

断言（Predicate）：Java8 中的断言函数。Spring Cloud Gateway 中的断言函数输入类型是 Spring 5.0 框架中的 ServerWebExchange。Spring Cloud Gateway 中的断言函数允许开发者去定义匹配来自于 Http Request 中的任何信息，比如请求头和参数等。

过滤器（Filter）：使用特定工厂构建的 Spring Framework GatewayFilter 实例。过滤器将会对请求和响应进行处理。

### 1.2 工作流程

![](up-d4f455febd78f36b2be8fcc90839349977e.webp)

客户端向 Spring Cloud Gateway 发出请求。 由网关处理程序 Gateway Handler Mapping 映射确定请求与路由匹配，则将其发送到网关 Web 处理程序 Gateway Web Handler。 Web 处理程序通过指定的过滤器链将请求发送到我们实际的服务执行业务逻辑，然后返回。 过滤器被虚线分隔的原因是过滤器可以在发送代理请求之前和之后运行逻辑。执行所有 pre 过滤器逻辑，然后发出代理请求；发出代理请求后，将运行 post 过滤器逻辑。

### 2. Gateway网关搭建

1.  在Spring Cloud父工程中创建module
2.  导入依赖

```xml
<!-- 引入spring cloud gateway依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!-- 引入SpringCloud Eureka client依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

1.  启动类上加注解`@EnableDiscoveryClient`注册到Eureka

```java
// @EnableEurekaClient: 声明一个Eureka客户端，只能注册到Eureka Server
// @EnableDiscoveryClient: 声明一个可以被发现的客户端，可以是其他注册中心
@EnableDiscoveryClient
@SpringBootApplication
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}

```

1.  配置文件application.yml

```ymal
server:
  port: 18001

# 指定当前服务的名称，这个名称会注册到注册中心
spring:
  application:
    name: @artifactId@
  cloud:              # spring cloud gateway 路由配置方式
    gateway:
      discovery:      # 是否与服务发现组件进行结合，通过 serviceId 转发到具体的服务实例。
        locator:      # 默认为false，设为true便开启通过服务中心的自动根据 serviceId 创建路由的功能。
          enabled: true
        lowerCaseServiceId: true # 将请求路径的服务名配置改成小写
      routes:
        - id: user-server                 # 自定义的路由 ID，保持唯一性
          uri: lb://XXXX-cloud-user       # 从注册中心获取服务，且以lb(load-balance)负载均衡方式转发
          predicates:
            - Path=/user/**               # 将以/user/开头的请求转发到uri为lb://XXXX-cloud-user的地址上
        - id: product-server
          uri: lb://XXXX-cloud-product
          predicates:
            - Path=/product/**            # 将以/product/开头的请求转发到uri为lb://XXXX-cloud-product的地址上

# 指定服务注册中心的地址
eureka:
  instance:
    prefer-ip-address: true       # 是否使用 ip 地址注册
    instance-id: ${spring.cloud.client.ip-address}:${server.port} # ip:port
  client:
    service-url:                  # 设置服务注册中心地址
      defaultZone: http://localhost:18000/eureka/

```

### 3. 路由配置

#### 3.1 路由配置方式

1.  基础URI路由配置

```ymal
spring:
  cloud:
    gateway:
      routes:
        - id: product-service         # 自定义的路由 ID，保持唯一
          uri: http://localhost:9004  # 目标服务地址
          predicates:                 # 断言（判断条件）：Predicate接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。
            - Path=/product/**

```

> 单个URI的地址的schema协议，一般为http或者https协议。 和注册中心相结合的路由配置的schema协议部分为自定义的lb:类型，表示从微服务注册中心（如Eureka）订阅服务，并且进行服务的路由。

1.  基于代码的路由配置

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
 
@SpringBootApplication
public class GatewayApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
 
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()   // 参数分别为：路由 ID，断言（判断条件），目标服务地址
                .route("product-service", r -> r.path("/product/**").uri("http://localhost:9004"))
                .build();
    }
}

```

#### 3.2 路由匹配规则

Spring Cloud Gateway 是通过 Spring WebFlux 的 HandlerMapping 做为底层支持来匹配到转发路由，Spring Cloud Gateway 内置了很多 Predicates 工厂，这些 Predicates 工厂通过不同的 HTTP 请求参数来匹配，多个 Predicates 工厂可以组合使用。

路由谓词工厂（Route Predicate Factories）

<table>
<thead><tr><th>类型</th><th>路由谓词</th><th>路由谓词工厂</th><th>描述</th></tr></thead>
<tbody>
  <tr><td rowspan="3">时间相关</td><td>After</td><td>AfterRoutePredicateFactory</td><td>在某个时间之后的请求才会被转发，如：`- After=2017-01-20T17:42:47.789-07:00[America/Denver]`</td></tr>
  <tr><td>Before</td><td>BeforeRoutePredicateFactory</td><td>在某个时间之前的请求才会被转发，如：`- Before=2017-01-20T17:42:47.789-07:00[America/Denver]`</td></tr>
  <tr><td>Between</td><td>BetweenRoutePredicateFactory</td><td>在某个时间段之间的才会被转发，如：`- Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]`</td></tr>
  <tr><td>Cookie相关</td><td>Cookie</td><td>CookieRoutePredicateFactory</td><td>`- Cookie=chocolate, ch.p`名为chocolate的表单或者满足正则ch.p的表单才会被匹配到进行请求转发</td></tr>
  <tr><td rowspan="2">Header相关</td><td>Header</td><td>HeaderRoutePredicateFactory</td><td>`- Header=X-Request-Id, \d+`携带参数X-Request-Id或者满足\d+的请求头才会匹配</td></tr>
  <tr><td>Host</td><td>HostRoutePredicateFactory</td><td>`- Host=**.somehost.org,**.anotherhost.org`当主机名为somehost.org或anotherhost.org的时候才会被转发</td></tr>
  <tr><td rowspan="5">请求相关</td><td>Method</td><td>MethodRoutePredicateFactory</td><td>`- Method=GET,POST`只有GET和POST方法才会匹配转发请求</td></tr>
  <tr><td>Path</td><td>PathRoutePredicateFactory</td><td>`- Path=/red/{segment},/blue/{segment}`当请求的路径为/red/、/blue/开头的时才会被转发</td></tr>
  <tr><td>Query</td><td>QueryRoutePredicateFactory</td><td>`- Query=green`只要请求中包含green参数即可</td></tr>
  <tr><td>RemoteAddr</td><td>RemoteAddrRoutePredicateFactory</td><td>`- RemoteAddr=192.168.1.1/24`主机IP</td></tr>
  <tr><td>Weight</td><td>WeightRoutePredicateFactory</td><td>`- Weight=group1, 2`权重是按组计算的, 两个参数：group 和 weight（int）</td></tr>
</tbody>
</table>

```ymal
spring:
  cloud:
    gateway:
      routes:
      - id: predicate_route_test
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
        - Method=GET,POST
        - Host=**.somehost.org,**.anotherhost.org
        - Header=X-Request-Id, \d+

```

### 4. 过滤器规则

Spring Cloud Gateway 除了具备请求路由功能之外，也支持对请求的过滤。

-   Spring Cloud Gateway的Filter的生命周期："pre"和"post"。
    -   PRE：这种过滤器在请求被路由之前调用。我们可以利用这种过滤器实现身份认证、在集群中选择请求的微服务、记录调试信息等。
    -   POST：这种过滤器在路由到微服务以后执行。这种过滤器可以用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等等。
-   Spring Cloud Gateway的Filter从作用范围可以分为两种：局部过滤器（GatewayFilter）和 全局过滤器（GlobalFilter）。
    -   GatewayFilter：应用到当个路由或者一个分组的路由上。
    -   GlobalFilter：应用到所有的路由上。

#### 4.1 局部过滤器

局部过滤器（GatewayFilter），是针对单个路由的过滤器。可以对访问的URL过滤，进行切面处理。Spring Cloud Gateway 包含许多内置的 GatewayFilter 工厂。

| 过滤器工厂 | 作用 | 参数 |
| --- | --- | --- |
| AddRequestHeader | 为原始请求添加Header | Header的名称及值 |
| AddRequestParameter | 为原始请求添加请求参数 | 参数名称及值 |
| AddResponseHeader | 为原始响应添加Header | Header的名称及值 |
| DedupeResponseHeader | 剔除响应头中重复的值 | 需要去重的Header名称及去重策略 |
| CircuitBreaker | 为路由引入CircuitBreaker的断路器保护 | CircuitBreakerCommand的名称 |
| MapRequestHeader | 将fromHeader的值更新到toHeader | fromHeader名称, toHeader名称 |
| FallbackHeaders | 为fallbackUri的请求头中添加具体的异常信息 | Header的名称 |
| PrefixPath | 为原始请求路径添加前缀 | 前缀路径 |
| PreserveHostHeader | 为请求添加一个 preserveHostHeader=true的属 性，路由过滤器会检查该属性以 决定是否要发送原始的Host | 无 |
| RequestRateLimiter | 用于对请求限流，限流算法为令 牌桶 | keyResolver、 rateLimiter、 statusCode、 denyEmptyKey、 emptyKeyStatus |
| Redirect | 将原始请求重定向到指定的URL | http状态码及重定向的 url |
| RemoveRequestHeader | 为原始请求删除某个Header | Header名称 |
| RemoveResponseHeader | 为原始响应删除某个Header | Header名称 |
| RemoveRequestParameter | 为原始请求删除请求参数 | 参数名称 |
| RewritePath | 重写原始的请求路径 | 原始路径正则表达式以 及重写后路径的正则表 达式 |
| RewriteLocationResponseHeader | 重写响应头中 Location 的值 | 输入四个参数：stripVersionMode、locationHeaderName、hostValue、protocolsRegex |
| RewriteResponseHeader | 重写原始响应中的某个Header | Header名称，值的正 则表达式，重写后的值 |
| SaveSession | 在转发请求之前，强制执行 WebSession::save操作 | 无 |
| SecureHeaders | 为原始响应添加一系列起安全作 用的响应头 | 无，支持修改这些安全 响应头的值 |
| SetPath | 修改原始的请求路径 | 修改后的路径 |
| SetRequestHeader | 重置请求头的值 | Header的名称及值 |
| SetResponseHeader | 修改原始响应中某个Header的值 | Header名称，修改后 的值 |
| SetStatus | 修改原始响应的状态码 | HTTP 状态码，可以是 数字，也可以是字符串 |
| StripPrefix | 用于截断原始请求的路径 | 使用数字表示要截断的 路径的数量 |
| Retry | 针对不同的响应进行重试 | retries、statuses、 methods、series |
| RequestSize | 设置允许接最大请求包的大小。如果请求包大小超过设置的 值，则返回 413 Payload Too Large | 请求包大小，单位为字 节，默认值为5M |
| SetRequestHost | 用指定的值替换现有的host header | 指定的Host |
| ModifyRequestBody | 在转发请求之前修改原始请求体内容 | 修改后的请求体内容 |
| ModifyResponseBody | 修改原始响应体的内容 | 修改后的响应体内容 |

> 每个过滤器工厂都对应一个实体类，并且这些类的名称必须以GatewayFilterFactory结尾，这是Spring Cloud Gateway的一个约定，例如AddRequestHeader对一个的实体类为AddRequestHeaderGatewayFilterFactory。

4.1.1 限流过滤器RequestRateLimiter

Spring Cloud Gateway官方提供了基于令牌桶的限流支持。 基于其内置的过滤器工厂RequestRateLimiterGatewayFilterFactory实现。 在过滤器工厂中是通过Redis和Lua脚本结合的方式进行流量控制。

1.  首先引入Redis依赖：

```xml
<!-- reactive redis依赖包（包含Lettuce客户端） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>

```

1.  配置文件

```yaml
spring:
  application:
    name: api-gateway-server
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true               # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
          lower-case-service-id: true # 微服务名称以小写形式呈现
      routes:       # 配置路由： 路由id，路由到微服务的uri,断言（判断条件）
        - id: product-service       # 路由id
          uri: lb://service-product # 路由到微服务的uri。 lb://xxx，lb代表从注册中心获取服务列表，xxx代表需要转发的微服务的名称
          predicates:               # 断言（判断条件）
            - Path=/product-service/**
          filters:   # 配置路由过滤器
            - name: RequestRateLimiter                 # 使用的限流过滤器是Spring Cloud Gateway提供的
              args:
                key-resolver: '#{@pathKeyResolver}'   # 使用SpEL从容器中获取对象
                redis-rate-limiter.replenishRate: 1   # 令牌桶每秒填充平均速率，允许用户每秒处理多少个请求
                redis-rate-limiter.burstCapacity: 3   # 令牌桶的上限，令牌桶的容量，允许在一秒钟内完成的最大请求数

```

1.  配置Redis中key的解析器

```java
import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;
import java.util.Objects;

@Configuration
public class KeyResolverConfig {
    /**
     * 基于请求路径的限流
     */
    //@Bean
    public KeyResolver pathKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getPath().toString());
    }

    /**
     * 基于请求参数的限流
     * 请求/abc?userId=1
     */
    //@Bean
    public KeyResolver useKeyResolver() {
        return exchange -> Mono.just(Objects.requireNonNull(exchange.getRequest().getQueryParams().getFirst("userId")));
    }

    /**
     * 基于请求IP地址的限流
     */
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(Objects.requireNonNull(exchange.getRequest().getHeaders().getFirst("x-Forwarded-For")));
    }
}

```

> Spring Cloud Gateway目前提供的限流还是比较简单的，在实际开发中我们的限流策略会有很多种情况， 比如：对不同接口的限流，被限流后的友好提示，这些可以通过自定义RedisRateLimiter来实现自己的限流策略。

#### 4.2 全局过滤器

全局过滤器（GlobalFilter）作用于所有路由，Spring Cloud Gateway定义了Global Filter接口，用户可以自定义实现自己的Global Filter。 通过全局过滤器可以实现对权限的统一校验，安全性校验等功能。

| 过滤器工厂 | 描述 |
| --- | --- |
| ForwardRoutingFilter | 它会从exchange.getRequiredAttribute(GATEWAY\_REQUEST\_URL\_ATTR);获取路由配置的URI，如果这个URI是forward模式，过滤器会将请求转发到DispatcherHandler，然后匹配到网关本的请求路径之中，原来请求的URI将被forward的URI覆盖，原始的请求URI被存储到exchange的ServerWebExchangeUtils.GATEWAY\_ORIGINAL\_REQUEST\_URL_ATTR属性之中。 |
| LoadBalancerClientFilter | 它是用来处理负载均衡的过滤器。在网关后面的服务可以启动多个服务实例，这个过滤器就是把请求根据均衡规则路由到某台服务实例上面。它从exchange的ServerWebExchangeUtils.GATEWAY\_REQUEST\_URL\_ATTR属性中获取URI，如果这个URI的scheme是“lb”，如：lb://myserivce，它会使用spring cloud 的LoadBalancerClient解析myservice服务名，获取一个服务实例的host和port，并替换原来的客户端请求。原来请求的url会存储在exchange的ServerWebExchangeUtils.GATEWAY\_ORIGINAL\_REQUEST\_URL\_ATTR属性中。这个过滤器也会从exchange中获取ServerWebExchangeUtils.GATEWAY\_SCHEME\_PREFIX\_ATTR属性值，如果它的值也是“lb”，也会使用相同的规则路由。 |
| NettyRoutingFilter | 这是一个优先级最低的过滤器，如果从exchange的ServerWebExchangeUtils.GATEWAY\_REQUEST\_URL\_ATTR获取的URL的scheme是https或http，它将使用Netty的HttpClient创建向下执行的请求代理，请求返回的结果将存储在exchange的ServerWebExchangeUtils.CLIENT\_RESPONSE_ATTR属性中，过滤器链后面的过滤器可以从中获取返回的结果。（还有一个测试使用的过滤器，WebClientHttpRoutingFilter，它和NettyRoutingFilter的功能一样，但是不使用netty）。 |
| NettyWriteResponseFilter | 它的优先级是最高的，它是“post”类型的过滤器。如果在exchange中ServerWebExchangeUtils.CLIENT\_RESPONSE\_ATTR的属性存在HttpClientResponse，它会在所有的其它的过滤器执行完成之后运行，将响应的数据发送给网关的客户端。 |
| RouteToRequestUrlFilter | 它的作用是把浏览器的URL请求的Path路径添加到路由的URI之中，比如浏览器请求网关的URL是：`http://localhost:8080/app-a/app/balance`，路由的URI配置是：`uri: lb://app-a`，那么添加之后的路由的URI是：`lb://app-a/app/balance`，并将它存储在exchange的ServerWebExchangeUtils.GATEWAY\_REQUEST\_URL_ATTR属性之中。 |
| WebsocketRoutingFilter | 它是用来路由WebScoket请求，在exchange的ServerWebExchangeUtils.GATEWAY\_REQUEST\_URL_ATTR的URI中，如果scheme是ws或wss，它会使用Spring Web Socket 模块转发WebSocket请求。WebSockets可以使用路由进行负载均衡，比如：lb:ws://serviceid。 |
| GatewayMetricsFilter | 它用来统计一些网关的性能指标。需要添加spring-boot-starter-actuator的项目依赖。 |

在网关路由 ServerWebExchange 后，它将通过在 exchange 添加一个 gatewayAlreadyRouted 属性，从而将exchange标记为 routed 。一旦请求被标记为 routed ，其他路由过滤器将不会再次路由请求，而是直接跳过，防止重复的路由操。可以使用便捷方法将 exchange 标记为 routed ，或检查 exchange 是否是 routed。

```java
ServerWebExchangeUtils.isAlreadyRouted   //检查是否已被路由
ServerWebExchangeUtils.setAlreadyRouted  //设置routed状态

```

#### 4.3 自定义全局过滤器鉴权

鉴权逻辑： ①当客户端第一次请求服务的时候，服务端对用户进行信息认证（登录）。 ②认证通过，将用户信息进行加密形成token，返回给客户端，作为登录凭证。 ③以后每次请求，客户端都携带认证的token。 ④服务端对token进行解密，判断是否有效。

对于验证用户是否已经登录授权的过程可以在网关层统一校验。校验的标准就是请求中是否携带token凭证以及token的正确性。

这里代码实现仅判断是否携带token凭证：`TokenFilter.java`

```java
import org.apache.commons.lang.StringUtils;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class TokenFilter implements GlobalFilter, Ordered {

    /**
     * 是否携带token凭证
     * 对请求参数中的access-token进行判断
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        String token = request.getHeaders().getFirst("access-token");
        //如果token为空
        if (StringUtils.isEmpty(token)) {
            //设置Http的状态码
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            //请求结束
            return exchange.getResponse().setComplete();
        }
        //如果token存在，继续执行
        return chain.filter(exchange);
    }

    /**
     * 指定过滤器的执行顺序，返回值越小，优先级越高
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```
