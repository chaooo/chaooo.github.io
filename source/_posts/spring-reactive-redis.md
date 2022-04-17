---
title: 「Spring Reactive Stack」响应式方式访问Redis
date: 2021-03-04 19:01:42
tags: [后端开发, Spring, SpringReactive, Redis]
categories: SpringReactive
---

`Spring Data Redis`中同时支持了`Jedis`客户端和`Lettuce`客户端。但是仅`Lettuce`是支持`Reactive`方式的操作；这里选择默认的`Lettuce`客户端。<!-- more -->

1. 创建`Maven`项目，并在`pom.xml`导入依赖:
``` xml
<!-- reactive redis依赖包（包含Lettuce客户端） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

2. 配置文件`application.yml`
``` ymal
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password: 123456
    #Redis数据库索引（默认为0）
    database: 0
    #连接超时时间（毫秒）
    timeout: 5000
```

3. 注入配置类：
``` java
@Configuration
public class ReactiveRedisConfig {
    @Bean
    public ReactiveRedisTemplate reactiveRedisTemplate(ReactiveRedisConnectionFactory factory) {
        return new ReactiveRedisTemplate<>(factory, RedisSerializationContext.string());
    }
}
```

4. 简单的RedisService封装
``` java
@Service
@AllArgsConstructor
public class RedisService {

    private final ReactiveRedisTemplate<String, String> redisTemplate;

    public Mono<String> get(String key) {
        return key==null ? null : redisTemplate.opsForValue().get(key);
    }
    public Mono<Boolean> set(String key, String value) {
        return redisTemplate.opsForValue().set(key, value);
    }
    public Mono<Boolean> set(String key, String value, Long time) {
        return redisTemplate.opsForValue().set(key, value, Duration.ofSeconds(time));
    }
    public Mono<Boolean> exists(String key) {
        return redisTemplate.hasKey(key);
    }
    public Mono<Long> remove(String key) {
        return redisTemplate.delete(key);
    }
}
```

5. 测试
``` java
@SpringBootTest
class ReactiveRedisTest {
    @Resource
    private RedisService redisService;

    @Test
    void test1() {
        // 保存5分钟
        redisService.set("test1", "test1_value", 5 * 60L).subscribe(System.out::println);
        redisService.get("test1").subscribe(System.out::println);
    }
}
```

测试运行结果：
``` bash
true
test1_value
```

> 本文使用Spring Boot版本：2.4.3
