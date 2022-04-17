---
title: 「SpringCloud」OpenFeign服务间调用
date: 2021-11-12 16:01:00
tags: [后端开发, SpringCloud, OpenFeign]
categories: SpringCloud
---

### 1. Spring Cloud OpenFeign简介
`OpenFeign`是`SpringCloud`提供的一个声明式的伪`Http`客户端，它使得调用远程服务就像调用本地服务一样简单，只需要创建一个接口并添加一个注解即可。
`OpenFeign`是`SpringCloud`在`Feign`的基础上支持了`Spring MVC`的注解，并通过动态代理的方式产生实现类来做负载均衡并进行调用其他服务。<!-- more -->

#### 1.1 OpenFeign使用流程：
1. 引入`Spring Cloud OpenFeign`的依赖
2. 启动类上添加注解`@EnableFeignCleints`
3. 按照`Feign`的规则定义接口并添加`@FeignClient`注解
4. 在需要使用`Feign`接口的类里注入，直接调用接口方法


### 2. OpenFeign的使用
1. 在`pom.xml`文件中添加依赖：

``` xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. 在启动类上，加上`@EnableFeignCleints`注解：

``` java
// basePackages 是Feign接口定义的路径
@EnableFeignClients(basePackages = {"com.XXX.feign"})
@EnableDiscoveryClient
@SpringBootApplication
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```

3. 按照`Feign`的规则定义接口：

``` java
// 括号内是远程调用微服务在注册中心的服务名
@FeignClient("XXX-CLOUD-TOOLS")
public interface ToolsFeign {
    /**
     * 远程调用TOOLS服务接口“/tools/sms/send”--请求发短信
     */
    @PostMapping("/tools/sms/send")
    String getSendSms(@RequestParam String mobile, @RequestParam String content);
}
```

4. 调用`Feign`接口方法

``` java
@Slf4j
@Service
public class RemoteCallServiceImpl implements RemoteCallService {
    /**
     * 在需要使用`Feign`接口的类里注入ToolsFeign
     */
    @Resource
    private ToolsFeign toolsFeign;

    /**
     * 调用 ToolsFeign 发送短信
     */
    @Override
    public String remoteSendSms(String mobile, String content) {
        return toolsFeign.getSendSms(mobile, content);
    }
}
```

5. 远程服务接口：

``` java
@RestController
@RequiredArgsConstructor
@RequestMapping("/tools")
public class SmsApi {
    /**
     * 发送短信
     *
     * @param smsDto {
     *                mobile  手机号
     *                content 短信内容
     *               }
     */
    @PostMapping("/sms/send")
    public ResponseJson<String> sendSms(SmsDto smsDto){
        String result = SmsUtil.sendSms(smsDto.getMobile(), smsDto.getContent());
        JSONObject json = (JSONObject) JSONObject.parse(result);
        if (null != json && json.getInteger("code") == 0) {
            return ResponseJson.success(0, "发送成功",null);
        } else {
            return ResponseJson.error(0, "发送失败", result);
        }
    }
}
```


### 3. OpenFeign的核心工作原理：
1. 通过`@EnableFeignCleints`触发`Spring`应用程序对`classpath`中`@FeignClient`修饰类的扫描
2. 解析到`@FeignClient`修饰类后，`Feign`框架通过扩展`SpringBeanDeifinition`的注册逻辑，最终注册一个`FeignClientFacotoryBean`进入`Spring`容器
3. `Spring`容器在初始化其他用到`@FeignClient`接口的类时，获得的是`FeignClientFacotryBean`产生的一个代理对象`Proxy`.
4. 基于`java`原生的动态代理机制，针对`Proxy`的调用，都会被统一转发给`Feign`框架所定义的一个`InvocationHandler`，由该`Handler`完成后续的`HTTP`转换，发送，接收，翻译`HTTP`响应的工作


### 4. OpenFeign日志
`Feign` 和 `RestTemplate` 不一样 ，对请求细节封装的更加彻底，不管是请求还是请求的参数，还是响应的状态都看不到，想要看到请求的细节需要通过`Feign`的日志，我们可以通过配置来调整日志级别，从而了解`OpenFeign`中`Http`请求的细节。即对`OpenFeign`远程接口调用的情况进行监控和日志输出。

#### 4.1 日志级别
- `NONE`：默认级别，不显示日志
- `BASIC`：仅记录请求方法、`URL`、响应状态及执行时间
- `HEADERS`：除了`BASIC`中定义的信息之外，还有请求和响应头信息
- `FULL`：除了`HEADERS`中定义的信息之外，还有请求和响应正文及元数据信息


#### 4.2 配置日志bean
``` java
@Configuration
public class OpenFeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

#### 4.3 开启日志
在`YMAL`配置文件中中指定监控的接口，以及日志级别
``` ymal
logging:
  level:
    com.XXX.feign.ToolsFeign: debug  # 以什么级别监控哪个接口
```

![](up-509245506c54819c3220067d999b6df8a85.webp)
