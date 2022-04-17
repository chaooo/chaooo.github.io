---
title: 「Spring」面向切面编程(AOP模块)
date: 2018-04-21 22:48:36
tags: [后端开发, Spring]
categories: Spring
---

AOP（Aspect Oriented Programming）：面向切面编程，
它是面向对象基础上发展来的技术，是面向对象更高层次的应用，
它可以在不修改原有代码的情况给组件增强功能。
<!-- more -->


### 1. AOP涉及到的概念
- Aspect：切面，用来封装共通业务逻辑；其类叫切面类，其创建的对象叫切面对象。
- JoinPoint：连接点，用来封装切面所要嵌入的位置信息的对象，（主要封装了方法信息）
- Pointcut：切点，是一堆连接点的集合，后面会使用切点表达式来表述切点
- Target：目标，要被切入共通业务逻辑的对象
- Proxy：代理，被增强之后的目标对象就是代理
- Advice：通知，时机，切面逻辑在目标方法执行之前调用，执行之后调用，目标方法前后，目标方法最终，目标方法出现异常


### 2. 编写AOP程序步骤
1. 编写一个Sevice类，里面有登录和注册两个方法，然后使用Spring容器获取Service类对应的对象，调用登录和注册方法
2. 在不修改登录和注册原有代码的情况下，让两个方法调用前输出`******`
    1. 添加aop的jar包到lib
    2. 编写一个类，定义共同业务逻辑
    3. 配置aplicationContext.xml，创建切面对象
    4. 配置aop:config，切面-->通知-->切点


### 3. 切点表达式
1. Bean限定表达式
    - `bean("容器内组件id")`，支持通配符*，如：`bean("*Dao")`，`bean("acc*")`
2. 类型限定表达式
    - `within("包名.类型")`，要求表达式最后一部分必须是类型，如：`com.dao.impl.类型`，`com.dao.impl.*`，`com.dao..*`
3. 方法限定表达式
    - `execution("表达式")`，可以有 权限修饰 返回值类型 方法名(参数类型)throws 异常，必须有:`返回值类型 方法名()`


### 4. 通知的五种类型
1. `<aop:before`：前置通知，目标方法执行之前调用
2. `<aop:after-returning`：后置通知，目标方法执行之后调用（目标方法出异常，通知方法无法执行）
3. `<aop:after-throwing`：异常通知，目标方法出异常才调用
4. `<aop:after`：最终通知，目标方法之后**一定**会执行
5. `<aop:around`：环绕通知，目标方法执行前后都调用


### 5. 标注形式AOP步骤
1. 建项目，添加jar包(ioc,aop)，src下添加配置文件
2. 编写一个Sevice类，里面有登录和注册两个方法
3. 开启组件扫描，在类上打对应标注，创建Spring容器 测试逻辑
4. 定义一个切面类，定义切面方法，并在容器中使用标注@Component创建切面对象
5. 开启标注形式aop：`<aop:aspectj-autoproxy proxy-target-class="true|false" />`
6. 使用切面对应的标注以及通知对应的标注结合切点表达式完成aop： `@Aspect，@Before...`


### 6. AOP 通知对应的标注
1. `@Before`：前置通知，目标方法执行之前调用
2. `@AfterReturning`：后置通知，目标方法执行之后调用（目标方法出异常，通知方法无法执行）
3. `@AfterThrowing`：异常通知，目标方法出异常才调用
4. `@After`：最终通知，目标方法之后**一定**会执行
5. `@Around`：环绕通知，目标方法执行前后都调用


### 7. @Around具体用法
@Around既可以在目标方法之前织入增强动作，也可以在执行目标方法之后织入增强动作；
``` java
@Around("within(com..*)")
public Object showAfterDate(ProceedingJoinPoint pjp) throws Throwable {
    System.out.println("开始时间：" + new Date().getTime());
    Object obj = pjp.proceed();
    System.out.println("结束时间：" + new Date().getTime());
    System.out.println("执行时间："date2.getTime() - date.getTime());
    return obj;
}
```
>虽然Around功能强大，但通常需要在线程安全的环境下使用。因此，如果使用普通的Before、AfterReturing增强方法就可以解决的事情，就没有必要使用Around增强处理了。


### 8. 异常通知
JoinPoint可以获取出异常的方法
``` java
@AfterThrowing(value="within(com..*)", throwing="e")
public void processException(JoinPoint jp, Exception e) {
    System.out.println("捕获到异常" + jp.getSignature() + ":\n「" + e +"」");
}
```
