---
title: Java接口幂等性设计及实例
date: 2021-02-17 14:12:42
tags: [后端开发, Spring, 幂等, SpringBoot]
categories: Spring
---


**幂等性**：多次调用方法或者接口不会改变业务状态，可以保证重复调用的结果和单次调用的结果一致。
`select`和`delete`操作具有天然幂等性：`select`多次结果总是一致，`delete`第一次执行后继续再执行也不会对数据有影响；
一般没有幂等性而出现异常的操作：`insert`操作，`update`操作，混合类型操作(同时包含增删改等)。<!-- more -->

### 1. 使用幂等的场景
1. 前端重复提交：前端瞬时点击多次造成表单重复提交；
2. 接口超时重试：接口可能会因为某些原因而调用失败，出于容错性考虑会加上失败重试的机制。如果接口调用一半，再次调用就会因为脏数据的存在而出现异常。
3. 消息重复消费：在使用消息中间件来处理消息队列，且手动`ack`确认消息被正常消费时。如果消费者突然断开连接，那么已经执行了一半的消息会重新放回队列。被其他消费者重新消费时就会导致结果异常，如数据库重复数据，数据库数据冲突，资源重复等。
4. 请求重发：网络抖动引发的`nginx`重发请求，造成重复调用；

### 2. 幂等性设计 
1. `update`操作
    + 根据唯一业务`id`去更新数据。
    + 使用乐观锁(增加版本号或修改时间字段)。
2. `insert`操作
    + 若该操作具有唯一业务号，则可通过数据库层面的唯一/联合唯一索引来限制重复数据；或通过分布式锁来保证接口幂等性。
    + 若该操作没有唯一业务号，可以使用`Token`机制，保证幂等性。
3. 混合操作（一个接口包含多种操作）
    + 使用`Token`机制，或使用`Token` + 分布式锁的方案来解决幂等性问题。

### 3. 解决方案
#### 3.1 Token机制实现
通过`Token` 机制实现接口的幂等性，这是一种比较通用性的实现方法。

具体流程步骤：
1. 客户端会先发送一个请求去获取`Token`，服务端会生成一个全局唯一的`ID`作为`Token`保存在`Redis`中，同时把这个`ID`返回给客户端；
2. 客户端第二次调用业务请求的时候必须携带这个`Token`；
3. 服务端会校验这个 `Token`，如果校验成功，则执行业务，并删除`Redis`中的 `Token`；
4. 如果校验失败，说明`Redis`中已经没有对应的 `Token`，则表示重复操作，直接返回指定的结果给客户端。

#### 3.2 基于MySQL实现
通过`MySQL`唯一索引的特性实现接口的幂等性。

具体流程步骤：
1. 建立一张去重表，其中某个字段需要建立唯一索引；
2. 客户端去请求服务端，服务端会将这次请求的一些信息插入这张去重表中；
3. 因为表中某个字段带有唯一索引，如果插入成功，证明表中没有这次请求的信息，则执行后续的业务逻辑；
4. 如果插入失败，则代表已经执行过当前请求，直接返回。

#### 3.3 基于Redis实现
通过`Redis`的`SETNX`命令实现接口的幂等性。

> `SETNX key value`：当且仅当`key`不存在时将`key`的值设为`value`；若给定的`key`已经存在，则`SETNX`不做任何动作。设置成功时返回`1`，否则返回`0`。

具体流程步骤：
1. 客户端先请求服务端，会拿到一个能代表这次请求业务的唯一字段；
2. 将该字段以`SETNX`的方式存入`Redis`中，并根据业务设置相应的超时时间；
3. 如果设置成功，证明这是第一次请求，则执行后续的业务逻辑；
4. 如果设置失败，则代表已经执行过当前请求，直接返回。


### 4. 实例：自定义注解实现API幂等处理（基于Redis实现）
#### 4.1 引入redis支持
1. `pom.xml`引入`Redis`的依赖
``` xml
<!-- redis依赖包 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <!-- 排除lettuce包，使用jedis代替-->
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
<!-- aop切面 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
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
    #连接池最大连接数（使用负值表示没有限制）
    jedis:
      pool:
        max-active: 50
        #连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1
        #连接池中的最大空闲连接
        max-idle: 20
        #连接池中的最小空闲连接
        min-idle: 2
    #连接超时时间（毫秒）
    timeout: 5000
```

3. 测试`Redis`连接
``` java
@SpringBootTest
class RedisTest {

    @Resource
    private RedisTemplate<String,String > redisTemplate;

    @Test
    void simpleTest() {
        ValueOperations<String,String> valueOperations = redisTemplate.opsForValue();
        String key = "RedisTemplateTest-simpleTest-001";
        valueOperations.set(key,key+key);
        System.out.println(valueOperations.get(key));
    }
}
```

#### 4.2 编码实现
1. 添加幂等异常

``` java
package com.example.demo;
/**
 * 处理幂等相关异常
 *
 * @author : Charles
 * @date : 2021/3/1
 */
public class IdempotentException extends RuntimeException {
    public IdempotentException(String message) {
        super(message);
    }
    @Override
    public String getMessage() {
        return super.getMessage();
    }
}
```

2. 自定义幂等注解

``` java
package com.example.demo;
import java.lang.annotation.*;
/**
 * 自定义幂等注解
 *
 * @author : Charles
 * @date : 2021/3/1
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Idempotent {
    /**
     * 前缀属性，作为redis缓存Key的一部分。
     */
    String prefix() default "idempotent_";
    /**
     * 需要的参数名数组
     */
    String[] keys();
    /**
     * 幂等过期时间（秒），即：在此时间段内，对API进行幂等处理。
     */
    int expire() default 3;
    /**
     * 提示错误码，也可自定义为字符串直接提醒
     */
    int errorCode() default 1001;
}
```

3. 幂等切面

``` java
package com.example.demo;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.core.LocalVariableTableParameterNameDiscoverer;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;
import org.springframework.stereotype.Component;
import redis.clients.jedis.commands.JedisCommands;
import redis.clients.jedis.params.SetParams;

import javax.annotation.Resource;
import java.lang.reflect.Method;


/**
 * 幂等切面
 *
 * @author : Charles
 * @date : 2021/3/1
 */
@Slf4j
@Aspect
@Component
@ConditionalOnClass(RedisTemplate.class)
public class IdempotentAspect {

    private static final String LOCK_SUCCESS = "OK";

    @Resource
    private RedisTemplate<String,String> redisTemplate;

    /**
     * 切入点,根据自定义Idempotent实际路径进行调整
     */
    @Pointcut("@annotation(com.example.demo.Idempotent)")
    public void executeIdempotent() {
    }

    @Around("executeIdempotent()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        // 获取参数对象列表
        Object[] args = joinPoint.getArgs();
      	//获取方法
        Method method = ((MethodSignature) joinPoint.getSignature()).getMethod();
        // 得到方法名
        String methodName = method.getName();
        // 获取参数数组
        String[] parameters = new LocalVariableTableParameterNameDiscoverer().getParameterNames(method);

        //获取幂等注解
        Idempotent idempotent = method.getAnnotation(Idempotent.class);

        // 初始化springEL表达式解析器实例
        ExpressionParser parser = new SpelExpressionParser();
        // 初始化解析内容上下文
        EvaluationContext context = new StandardEvaluationContext();
        // 把参数名和参数值放入解析内容上下文里
        for (int i = 0; i < parameters.length; i++) {
            if (args[i] != null) {
            	// 添加解析对象目标
                context.setVariable(parameters[i], args[i]);
            }
        }
        // 解析定义key对应的值，拼接成key
        StringBuilder idempotentKey = new StringBuilder(idempotent.prefix() + ":" + methodName);
        for (String s : idempotent.keys()) {
            // 解析对象
            Expression expression = parser.parseExpression(s);
            idempotentKey.append(":").append(expression.getValue(context));
        }
        // 通过 setnx 确保只有一个接口能够正常访问
        String result = redisTemplate.execute(
            (RedisCallback<String>) connection -> (
                (JedisCommands) connection.getNativeConnection()
            ).set(
                idempotentKey.toString(),
                idempotentKey.toString(),
                new SetParams().nx().ex(idempotent.expire())
            )
        );

        if (LOCK_SUCCESS.equals(result)) {
            return joinPoint.proceed();
        } else {
            log.error("API幂等处理, key=" + idempotentKey);
            throw new IdempotentException("API幂等处理, key=" + idempotentKey);
        }
    }
}
```

#### 4.3 幂等注解的使用

1. 接口添加`@Idempotent`注解
``` java
@Idempotent(prefix="idempotent", keys={"#id", "#str"}, expire=5)
@PostMapping("/test")
public String testApi(Integer id, String str) {
    return "测试幂等API:" + id + str;
}
```

2. 连续调用API测试
``` bash
curl -X POST "http://localhost:8080/test?id=1002&str=TestIdempotentParameterString"
```

3. 测试结果

第一次调用`/test`，会正常返回`测试幂等API:1002TestIdempotentParameterString`；
并且在`5`秒内，`Redis`会存在`key`为`idempotent:testApi:1002:TestIdempotentParameterString`的唯一值。
```
127.0.0.1:6379> keys "idempotent:testApi:1002:TestIdempotentParameterString"
1) "idempotent:testApi:1002:TestIdempotentParameterString"
```

再次调用`/test`，会返回异常。
``` json
{
    "timestamp": "2021-03-01T06:42:14.282+00:00",
    "path": "/test",
    "status": 500,
    "error": "Internal Server Error",
    "message": "API幂等处理, key=idempotent:testApi:1002:TestIdempotentParameterString",
    "requestId": "b42b9639-8",
    "trace": "com.example.demo.IdempotentException: API幂等处理, key=idempotent:testApi:1002:TestIdempotentParameterString\r\n\tat com.example.demo.IdempotentAspect.around ... 中间省略 ... (FastThreadLocalRunnable.java:30)\r\n\t\tat java.lang.Thread.run(Thread.java:748)\r\n"
}
```
