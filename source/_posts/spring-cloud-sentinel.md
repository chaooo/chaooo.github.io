---
title: 「SpringCloud」OpenFeign整合Sentinel实现熔断降级
date: 2021-11-23 17:01:00
tags: [后端开发, SpringCloud, Sentinel]
categories: SpringCloud
---

### 1. Sentinel简介
[Sentinel](https://github.com/alibaba/Sentinel) 是阿里开源的项目，提供了流量控制、熔断降级、系统负载保护等多个维度来保障服务之间的稳定性。

Sentinel 分为两个部分:
- 核心库（`Java` 客户端）不依赖任何框架/库，能够运行于所有 `Java` 运行时环境，同时对 `Dubbo / Spring Cloud` 等框架也有较好的支持。
- 控制台（`Dashboard`）基于 `Spring Boot` 开发，打包后可以直接运行，不需要额外的 `Tomcat` 等应用容器。<!-- more -->

这里仅介绍`Sentinel`核心库 与 `Spring Cloud OpenFeign`整合使用。

### 2. Sentinel与OpenFeign整合
基于上一篇[OpenFeign服务间调用](https://my.oschina.net/chaoo/blog/5308587)
#### 2.1 在`pom.xml`文件中添加依赖

``` xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>2021.1</version>
</dependency>
```

#### 2.2 在`YMAL`配置文件中添加如下配置
``` ymal
# 配置文件打开 Sentinel 对 Feign 的支持
feign:
  sentinel:
    enabled: true
```

> `Feign` 对应的接口中的资源名策略定义：`httpmethod:protocol://requesturl`。`@FeignClient` 注解中的所有属性，`Sentinel` 都做了兼容。
> 如：ToolsFeign 接口中方法 getSendSms 对应的资源名为 POST:http://XXX-CLOUD-TOOLS/tools/sms/send。

#### 2.3 修改`OpenFeign`调用远程服务，`@FeignClient`属性中配置降级回调类
`@FeignClient`属性中的`fallback`和`fallbackFactory`
- `fallback`：定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，`fallback`指定的类必须实现`@FeignClient`标记的接口。
- `fallbackFactory`：工厂类，用于生成`fallback`类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码。

> 注：同一个`@FeignClient`里，`fallback` 和 `fallbackFactory` 不能同时使用。

##### 2.3.1 使用fallback示例
1. 按照`Feign`的规则定义接口，使用`fallback`属性：

``` java
/**
 * value是远程调用微服务在注册中心的服务名。
 * fallback：定义容错的类，当远程调用的接口失败或者超时的时候，会调用对应接口的容错逻辑，
 * fallback指定的类必须实现@FeignClient标记的接口。
 */
@FeignClient(value = "XXX-CLOUD-TOOLS", fallback = ToolsFeignFallback.class)
public interface ToolsFeign {
    /**
     * 远程调用TOOLS服务接口“/tools/sms/send”--请求发短信
     */
    @PostMapping("/tools/sms/send")
    String getSendSms(@RequestParam String mobile, @RequestParam String content);
}
```

2. 编写降级回调类`ToolsFeignFallback.calss`

``` java
@Component
public class ToolsFeignFallback implements ToolsFeign{
    @Override
    public String getSendSms(String mobile, String content) {
        return "接口容错-fallback";
    }
}
```

##### 2.3.2 使用fallbackFactory示例
1. 按照`Feign`的规则定义接口，使用`fallbackFactory`属性：

``` java
/**
 * value是远程调用微服务在注册中心的服务名。
 * fallbackFactory：工厂类，用于生成fallback类实例，
 * 通过此属性可以实现每个接口通用的容错逻辑，以达到减少重复的代码。
 */
@FeignClient(value = "XXX-CLOUD-TOOLS", fallbackFactory = ToolsFeignFallbackFactory.class)
public interface ToolsFeign {
    /**
     * 远程调用TOOLS服务接口“/tools/sms/send”--请求发短信
     */
    @PostMapping("/tools/sms/send")
    String getSendSms(@RequestParam String mobile, @RequestParam String content);
}
```

2. 编写降级回调工厂类`ToolsFeignFallbackFactory.calss`

``` java
@Component
public class ToolsFeignFallbackFactory implements FallbackFactory<ToolsFeignFallback> {
	@Override
	public ToolsFeignFallback create(Throwable throwable) {
		return new ToolsFeignFallback(throwable);
	}
}
```

3. 编写降级回调类`ToolsFeignFallback.calss`

``` java
/**
 * sentinel 降级处理
 */
public class ToolsFeignFallback implements ToolsFeign{
	private Throwable throwable;
	ToolsFeignFallback(Throwable throwable) {
		this.throwable = throwable;
	}
    /**
     * 调用服务提供方的接口
     */
    @Override
    public String getSendSms(String mobile, String content) {
        return "接口容错-fallback" + throwable.getMessage();
    }
}
```


### 3. Sentinel 熔断降级
#### 3.1 Sentinel 熔断策略
- **慢调用比例** (`SLOW_REQUEST_RATIO`)：选择以慢调用比例作为阈值，需要设置允许的慢调用 `RT`（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（`HALF-OPEN` 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。
- **异常比例** (`ERROR_RATIO`)：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（`HALF-OPEN` 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 `0% - 100%`。
- **异常数** (`ERROR_COUNT`)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（`HALF-OPEN` 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

> 注意异常降级仅针对业务异常，对 `Sentinel` 限流降级本身的异常（`BlockException`）不生效。为了统计异常比例或异常数，需要通过 `Tracer.trace(ex)` 记录业务异常。
> 开源整合模块，如 `Sentinel Dubbo Adapter`, `Sentinel Web Servlet Filter` 或 `@SentinelResource` 注解会自动统计业务异常，无需手动调用。


#### 3.2 Sentinel熔断降级规则说明
熔断降级规则（DegradeRule）包含下面几个重要的属性：

| Field | 说明 | 默认值 |
| :----: | :---- | :---- |
| resource | 资源名，即规则的作用对象 | -- |
| grade | 熔断策略，支持 慢调用比例/异常比例/异常数策略 | 慢调用比例 |
| count | 慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值 | -- |
| timeWindow | 熔断时长，单位为 s | -- |
| minRequestAmount | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入） | 5 |
| statIntervalMs | 统计时长（单位为 ms），如 60*1000 代表分钟级（1.8.0 引入） | 1000 ms |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入） | -- |


#### 3.3 熔断器事件监听
Sentinel 支持注册自定义的事件监听器监听熔断器状态变换事件（state change event）。示例：
``` java
EventObserverRegistry.getInstance().addStateChangeObserver("logging", (prevState, newState, rule, snapshotValue) -> {
    if (newState == State.OPEN) {
        // 变换至 OPEN state 时会携带触发时的值
        System.err.println(String.format("%s -> OPEN at %d, snapshotValue=%.2f", prevState.name(), TimeUtil.currentTimeMillis(), snapshotValue));
    } else {
        System.err.println(String.format("%s -> %s at %d", prevState.name(), newState.name(), TimeUtil.currentTimeMillis()));
    }
});
```

