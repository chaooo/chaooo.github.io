---
title: 「Spring」事务管理
date: 2018-04-15 22:47:30
tags: [后端开发, Spring]
categories: Spring
---

事务的基本概念：事务指的是逻辑上的一组操作，这组操作要么全部成功，要么全部失败。
<!-- more -->


### 1. 事务的特性(ACID)
+ 事务的特性：**原子性、一致性、隔离性、持久性**。
+ 原子性（Atomicity）：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
+ 一致性（Consistency）：事务前后数据的完整性必须保持一致。
+ 隔离性（Isolation）：多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰，多个并发事务之间数据要相互隔离（数据库中相应的数据隔离级别，通过它避免事务间的冲突）。
+ 持久性（Durability）:一个事务一旦被提交，它对数据库中数据的改变是永久性的，即使数据库发生故障也不应该对其有任何影响。


### 2. Spring提供事务管理的3个接口：
1. **PlatformTransactionManager**：事务管理器，用来管理事务的接口，定义了事务的提交、回滚等方法。
2. **TransactionDefinition**：事务定义信息（隔离级别、传播行为、是否超时、是否只读）。
3. **TransactionStatus**：事务具体运行状态（事务是否提交，事务是否有保存点，事务是否是新事物等状态）。

> Spring事务管理时，这三个接口是有联系的，Spring首先会根据事务定义信息TransactionDefinition获取信息,然后由事务管理器PlatformTransactionManager进行管理，在事务管理过程中，会产生一个事务的状态，这个状态就保存在事务具体运行状态TransactionStatus中了。


### 3. TransactionDefinition接口
TransactionDefinition定义事务隔离级别(Isolation)、定义事务传播行为(Propagation)
+ 如果不考虑隔离性,就会引发安全问题：脏读、不可重复读、以及虚读或者叫做幻读。
+ 事务的传播行为：解决业务层方法之间相互调用时,使用何种事务的问题。


#### 3.1 安全问题
1. 脏读：一个事务读取了另一个事务改写但还未提交的数据，如果这些数据被回滚，则读到的数据是无效的。
2. 不可重复读：同一事务中，多次读取同一数据返回的结果有所不同（读取到另一个事务已经提交的更新的数据）。
3. 幻读：一个事务读取了几行记录后，另一个事务插入一些记录，幻读就发生了。再后来的查询中，第一个事务就会发现有些原来没有的记录。


#### 3.2 事务的隔离级别(Isolation)：
1. **`READ_UNCOMMITED`**(读未提交)：允许读取未提交的改变了的数据（最低级别），可能导致脏读、不可重复读、幻读等。
2. **`READ_COMMITED`**(读提交)：允许在并发事务提交后读取，可防止脏读，但可能导致不可重复读、幻读。
3. **`REPEATABLE_READ`**(可重复读)：多次读取相同字段是一致的,除非数据被事务本身改变，可防止脏读、不可重复读，但可能导致幻读。
4. **`SERIALIZABLE`**(序列化)：事务是串行的,完全服从ACID的级别隔离，确保不发生脏读、不可重复读、幻读等。这在所有的隔离基本中是最慢的，它是典型的通过完全锁定在事务中涉及的数据表来完成的。
5. `DEFAULT`(Spring提供)：使用数据库默认的隔离级别（Mysql默认采用`REPEATABLE_READ`隔离级别，Oracle默认采用`READ_COMMITTED`隔离级别）。


#### 3.3 事务的传播特性(Propagation)：
1. 第一类：运行在同一个事务
    + **`REQUIRED`**：默认，支持当前事务，如果当前没有事务，就新建一个事务。
    + `SUPPORTS`：支持当前事务，如果当前没有事务，就不使用事务(以非事务方式执行)
    + `MANDATORY`：支持当前事务，如果当前没有事务，就抛出异常
2. 第二类：运行在不同事务
    + **`REQUIRES_NEW`**：新建事务，如果当前存在事务，把当前事务挂起
    + `NOT_SUPPORTED`：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
    + `NEVER`：以非事务方式执行，如果当前存在事务，则抛出异常
3. 第三类：嵌套执行--即外层事务如果失败，内层事务要么回滚到保存点要么回滚到初始状态
    + **`NESTED`**：如果当前事务存在，则嵌套事务执行


### 4. TransactionStatus接口
平台事务管理器(PlatformTransactionManager)会根据TransactionDefinition中定义的事务信息(包括隔离级别、传播行为)来进行事务的管理,在管理的过程中事务可能产生了保存点或事务是新的事务等情况,那么这些信息都会记录在TransactionStatus的对象中。


### 5. PlatformTransactionManager接口（事务管理器）
该接口有许多实现类例如：DataSourceTransactionManager、HibernateTransactionManager等。


#### 5.1 Spring支持两种方式事务管理：
1. 编程式事务管理
    + 手动编写代码进行事务管理，通过TransactionTemlate手动管理事务（很少使用）
2. 声明式事务管理
    + 基于TransactionProxyFactoryBean的方式（很少使用）
    + 基于AspectJ的xml方式，配置稍复杂,但清晰可见事务使用范围（经常使用）
    + 基于注解的方式，配置简单,需要在使用事务管理的业务层类或方法添加`@Transactional`注解（经常使用）


### 6. 基于AspectJ的xml方式的声明式事务管理
``` xml
<!-- 配置事务管理器 -->
<bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="jdbc连接池对象id"/>
</bean>
<!-- 配置事务的通知（事务的增强） -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>    
        <!-- propagation:事务传播行为
            isolation:事务的隔离级别
            read-only:只读
            rollback-for:发生哪些异常回滚
            no-rollback-for:发生哪些异常不回滚
            timeout:过期信息    --> 
        <tx:method name="transfer" propagation="REQUIRED" isolation="DEFAULT" read-only="false" rollback-for="" timeout="" no-rollback-for=""/>
    </tx:attributes>
</tx:advice>
<!-- 配置切面 -->
<aop:config>
    <!-- 配置切入点 -->
    <aop:pointcut id="pointcut1" expression="execution(*cn.muke.spring.demo3.AccountService+.*(.))"/>
    <!-- 配置切面 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut1"/>
</aop:config>
```


### 7. 基于注解的声明式事务管理
1. 配置事务管理器
``` xml
<!-- 1.创建一个事务管理器对象 -->
<bean id="事务管理器id" 
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="jdbc连接池对象id"/>
</bean>
<!-- 2.开启声明式事务 -->
<tx:annotation-driven transaction-manager="事务管理器id" proxy-target-class="true|false" />
```

- **transaction-manager**：指定事务管理器(由框架提供类，在容器中创建这个对象并依赖于dataSource)
- **proxy-target-class**：决定是基于接口的还是基于类的代理被创建；为true则是基于类的代理将起作用(需要cglib库)，为false(默认)则标准的JDK 基于接口的代理将起作用。


2. 使用，在类上或者方法上标注`@Transactional`
``` java
@Transactional(
        rollbackFor={Exception.class}, 
        readOnly=false, 
        isolation=Isolation.DEFAULT,
        propagation=Propagation.REQUIRED)
public void transfer(){..}
```

- @Transactional的属性
    + **rollbackFor**：设置检查异常也回滚
    + **noRollbackFor**：指定运行时异常不回滚
    + **readOnly**： 只读属性，当事务方法都是select语句时，可以将readOnly设置成true优化方法，提高方法执行效率。当有DML操作时这个属性必须时false。
    + **isolation**：事务的隔离级别(枚举:DEFAULT,READ_UNCOMMITTED,READ_COMMITTED,REPEATABLE_READ,SERIALIZABLE)
    + **propagation**：事务的传播特性(枚举:REQUIRED,SUPPORTS,MANDATORY,REQUIRES_NEW,NOT_SUPPORTED,NEVER)
- Spring中事务管理器默认值针对**运行时异常**回滚，对**检查异常**不回滚。


