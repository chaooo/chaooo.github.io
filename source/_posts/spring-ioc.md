---
title: 「Spring」IoC控制反转
date: 2018-03-27 22:38:23
tags: [后端开发, Spring]
categories: Spring
---

Spring是一个开源的轻量级控制反转(IOC)和面向切面(AOP)的容器框架，它主要是为了解决企业应用开发的复杂性而诞生的，但现在已不止应用于企业服务。
- IOC：Inversion Of Control（控制反转），构成Spring框架的核心基础
<!-- more -->
- DAO：Data Access Object（数据 访问对象），Spring对JDBC访问数据库的简化和封装
- WebMVC：Spring对Web部分(jsp,servlet,ajax)以及MVC设计模式的支持
- AOP：是在面向对象的基础上发展来的更高级的技术
- ORM：Object Relation Mapping（对象关系映射），以面向对象的思想来访问数据库
- JEE：Java的消息服务，远程调用，邮件服务等


### 1. IoC（控制反转）
**IoC**：(Inversion of Control),控制反转：控制权的转移，应用程序本身不负责依赖对象的创建和维护，而是由外部容器负责创建和维护。
1. 控制：控制对象的创建及销毁（生命周期）
2. 反转：将对象的控制权交给IoC容器

**DI**：(Dependence Injection),依赖注入(注射)是IoC控制反转的一种具体实现方法，通过参数的方式从外部传入依赖，将依赖的创建由主动变为被动。
- 简单来说， 当 组件A 依赖 组件B 时，IoC容器通过设置A的属性，把B传入的过程叫依赖注入

> IoC的好处：降低了组件的依赖程度，让组件之间变成低耦合设计。


### 2. Spring容器初始化
任何Java类都可以在Spring容器中创建对象 并交由容器来进行管理和使用，Spring容器 实现了 IOC 和 AOP 机制，Spring容器的类型是 BeanFactory 或者 ApplicationContext
- BeanFactory提供配置结构和基本功能，加载并初始化Bean
- ApplicationContext保存了Bean对象并在Spring中被广泛使用

#### 2.1 初始化ApplicationContext的几种方式：
1. 本地文件
``` java
FileSystemXmlApplicationContext app = 
    new FileSystemXmlApplicationContext("F:/workspace/appcontext.xml");
```
2. Classpath
``` java
ClassPathXmlApplicationContext app = 
    new ClassPathXmlApplicationContext("classath:applicationContext.xml");
```
3. Web应用中依赖Servlet或Listener
``` xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```


#### 2.2 Spring容器完成IOC的步骤
1. 建立一个动态的Web项目，导入jar包(ioc) 拷贝Spring容器配置文件到src(Source classpath)下
2. 在spring容器配置文件中配置文件中配置一个对象的创建
  + `<baen id="对象引用名" class="包名.类名"></baen>`
3. 写一个测试类 创建Spring容器对象，然后从容去中获取创建的组件
  + `applicationContext.getBean("对象引用名", 类名.class)`


### 3. spring容器创建对象(实例化)
#### 3.1 构造器方式实例化
+ 配置文件：`<baen id="对象引用名" class="包名.类名"></baen>`
+ `applicationContext.getBean("对象引用名", 类名.class)`默认调用类型对应的无参构造方法
``` xml
<bean id="date" class="java.util.Date"></bean>
```
``` java
ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
Date date = app.getBean("date", Date.class);
```

#### 3.2 静态工厂方法实例化
+ 使用一个类型对应的静态方法来获取这个类型的对象
+ `<bean id="对象引用名" class="包名.工厂类名" factory-method="静态方法名"></bean>`
``` xml
<bean id="cal" class="java.util.Calendar" factory-method="getInstance"></bean>
```
``` java
ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
Calendar cal = app.getBean("cal", Calendar.class);
```

#### 3.3 实例工厂方法实例化
+ 使用一个已经存在的对象，来调用对应的成员方法来获取另一个类型的对象
+ `<bean id="对象的引用名" factory-bean="工厂方法的id" factory-method="成员方法名"></bean>`
``` xml
<bean id="cal" class="java.util.Calendar" factory-method="getInstance"></bean>
<bean id="time" factory-bean="cal" factory-method="getTime"></bean>
```
``` java
ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
Date time = app.getBean("time", Date.class);
```


### 4. Spring DI注入的实现
Spring注入是指在启动Spring容器加载bean配置的时候，完成对变量的赋值行为。
Bean属性值：基本数据类型用value，复杂数据类型用ref(传入组件id)。
DI的实现方法：设值注入(setter注入)、构造注入、自动化注入(自动装配)

* 实例：准备两个实体类Card，Player：Card有suit(花色)和point(点数)，Player有name(名字)和card(牌)。

#### 4.1 设值注入
property(属性)的name参考对象set方法
``` xml
<bean id="card" class="bean.Card">
    <property name="suit" value="黑桃"></property>
    <property name="point" value="A"></property>
</bean>
<!-- Player参考其setCard方法 -->
<bean id="player" class="bean.Player">
    <property name="name" value="玩家1"></property>
    <property name="card" ref="card"></property>
</bean>
```

#### 4.2 构造注入（Constructor arguments）
构建对象时赋值，参考对应构造方法（name为构造方法参数名，也可以用index:0开始）
``` xml
<bean id="card2" class="bean.Card">
    <constructor-arg name="suit" value="红桃"></constructor-arg>
    <constructor-arg name="point" value="K"></constructor-arg>
</bean>
<!-- Player参考其构造方法Player(name,card) -->
<bean id="player2" class="bean.Player">
    <constructor-arg name="name" value="玩家2"></constructor-arg>
    <constructor-arg name="card" ref="card2"></constructor-arg>
</bean>
```

#### 4.3 自动化注入（Autowiring mode）
一般用来解决复杂值的注入，可以通过bean标记的autowrie属性(autowire="byName/byType/constructor")指定对应的自动化的注入方式
``` xml
<bean id="bean1" class="example.exampleBean" autowire="" />
```

**自动装配autowire**属性 有五种自动装配的方式：
+ No：默认，需要通过`ref`属性来连接bean。
+ **byName**： 与当前组件属性名 和 容器中其他组件的id 一致的bean，自动装配。
``` xml
<bean id="card3" class="bean.Card">
    <property name="suit" value="方片"></property>
    <property name="point" value="J"></property>
</bean>
<!-- Player中必须要有setCard3 方法(setter方法名要与注入组件id对应)
    否则Spring会将id为card的bean通过setter方法进行自动装配(若有setCard方法)-->
<bean id="player3" class="bean.Player" autowire="byName"></bean>
```

+ **byType**：与当前组件属性类型 和 容器中其他组件的class 一致的bean，自动装配，如果存在多个则抛出异常。
``` xml
<bean class="bean.Card">
    <property name="suit" value="方片"></property>
    <property name="point" value="J"></property>
</bean>
<!-- Spring会将类型为Card的bean通过setter方法进行自动装配(setter参数类型与注入组件类型对应) -->
<bean id="player4" class="bean.Player" autowire="byType"></bean>
```

+ **constructor**：与当前组件 构造方法的参数 容器中其他组件的id 一致的bean，自动装配，不匹配再和 容器中其他组件的class 一致的bean，自动装配（如果存在多个则不装配），如果构造方法中第一个参数不匹配，则终止后续赋值。
``` xml
<bean id="card5" class="bean.Card">
    <property name="suit" value="方片"></property>
    <property name="point" value="J"></property>
</bean>
<!-- Player添加构造方法Player(Card card5)，构造方法参数名与注入组件id对应，不匹配再用构造方法参数类型和注入组件class匹配，如果存在多个则不装配 -->
<bean id="player5" class="bean.Player" autowire="constructor"></bean>
```

+ **autodetect**：如果有默认的构造器，则通过constructor方式进行自动装配，否则使用byType方式进行自动装配。


### 5. DI的参数的注入
Bean对象 注入类型 可以是 字符串、集合、bean对象。

#### 5.1 注入字符串
``` xml
<bean id="msg" class="com.xdl.bean.OracleDataSource">
    <property name="username" value="scott"/>
    <property name="password"><value>tiger</value></property>
    <property name="msg"><null/></property>
</bean>
```

#### 5.2 注入集合
``` xml
<!-- 1. 定义list集合 -->
<property name="friends">
    <list>
        <value>值1</value>
        <value>值2</value>
    </list>
</property>
<!-- 2. 定义set集合 -->
<property name="friends2">
    <set>
        <value>值1</value>
        <value>值2</value>
    </set>
</property>
<!-- 3. 定义map集合 -->
<property name="phones">
    <map>
        <entry key="1594546454" value="值1"></entry>
        <entry key="1594546464" value="值2"></entry>
    </map>
</property>
<!-- 4. props集合 -->
<property name="phones2">
    <props>
        <prop key="164545564">值1</prop>
        <prop key="164546756">值2</prop>
    </props>
</property>
```


#### 5.3 集合参数的单独定义
注入集合--引入：List、Set、Map、Properties集合也可以先独立定义，再注入的方式使用，这样便于重复利用。
``` xml
<!-- 1. 定义list集合 -->
<util:list id="ref_friends">
    <value>值1</value>
    <value>值2</value>
</util:list>
<!-- 2. 定义set集合 -->
<util:set id="ref_buddys">
    <value>值</value>
    <value>值2</value>
</util:set>
<!-- 3. 定义map集合 -->
<util:map id="ref_phones">
    <entry key="159454644" value="值1"></entry>
    <entry key="1594546454" value="值2"></entry>
</util:map>
<!-- 4. props集合 -->
<util:properties id="ref_phonePro">
    <prop key="164545564">值1</prop>
    <prop key="16454675665564">值2</prop>
</util:properties>
<util:properties id="ref_db" location="classpath:db.properties"></util:properties>
<!-- 使用 -->
<property name="phones" ref="ref_phones"></property>
<property name="phones2" ref="ref_phonePro"></property>
```


#### 5.3 Spring的'EL'表达式
它和EL在语法上很 相似，可以读取一个bean对象/集合中的数据。
Spring EL 采用 #{Sp Expression Language} 即 `#{spring表达式}`，可在xml配置和注解中使用。

+ Spring EL配置连接池对象
``` xml
<!-- 引入数据库配置文件 -->
<util:properties id="db" location="classpath:db.properties"/>
<!-- 配置连接池 -->
<bean id="dataSource" class="com.xdl.bean.OracleDataSource">
    <property name="username" value="#{db.name}"/>
    <property name="password" value="#{db.password}"/>
    <property name="url" value="#{db.url}"/>
</bean>
```


### 6. Bean的常用配置项(作用域,生命周期,懒加载等)
Bean的常用配置项：Id、Class、Scope、Constructor arguments、Propertties、Autowiring mode、Lazy-initialization mode、Initialization/destruction method

#### 6.1 Bean作用域（Scope）
1. Singleton作用域
    + 单例，指一个Bean容器只存在一份
2. prototype作用域
    + 每次请求(使用)创建新的实例，destroy方式不生效
3. Web环境作用域：
    + request作用域：每个request请求都会创建一个单独的实例。
    + session作用域：每个session都会创建一个单独的实例。
    + application作用域：每个servletContext都会创建一个单独的实例。
    + websocket作用域：每个websocket连接都会创建一个单独的实例。
4. 自定义作用域
    + SimpleThreadScope作用域：每个线程都会创建一个单独的实例。

#### 6.2 Bean的生命周期（Initialization/destruction method）
Bean的生命周期：定义 --> 初始化 --> 使用 --> 销毁

##### 6.2.1 Bean初始化
如果需要在Bean实例化之后执行一些逻辑，有两种方法：
+ 实现InitializingBean接口(org.springframework.beans.factory.InitializingBean)，覆盖afterPropertiesSet方法，在afterPropertiesSet中执行一些初始化后的工作。
+ **配置init-method**
    - 配置**`beans`**的`default-init-method`属性 来指定一个初始化方法，这个指定针对容器中所有的对象，由于这样影响的范围比较广，所以当对象没有对应的初始化方法程序也不会报错。
    - 配置**`bean`**的`init-method`来指定初始化方法，这样只影响包含init-method属性所在的bean标记创建的对象，这样控制的对象比较精准，所以当类型中没有这个初始化方法则程序崩溃。
``` xml
<bean id="exampleId" class="example.exampleBean" init-method="init"></bean>
```
``` java
public class ExampleBean{
    public void init(){
        //执行一些初始化后的工作
    }
}
```


##### 6.2.2 Bean销毁
如果需要在Bean销毁之前执行一些逻辑，有两种方法：
+ 实现DisposableBean接口(org.springframework.beans.factory.DisposableBean)覆盖destroy方法，，在destroy中执行一些销毁前的工作。
+ **配置destroy-method**
    - 配置**`beans`**的`default-destroy-method`属性 来指定一个销毁方法，这个指定针对容器中所有的对象，由于这样影响的范围比较广，所以当对象没有对应的销毁方法程序也不会报错。
    - 配置**`bean`**的`destroy-method`来指定销毁方法，这样只影响包含destroy-method属性所在的bean标记创建的对象，这样控制的对象比较精准，所以当类型中没有这个销毁方法则程序崩溃。
``` xml
<bean id="exampleId" class="example.exampleBean" destroy-method="cleanup"></bean>
```
``` java
public class ExampleBean{
    public void cleanup(){
        //执行一些销毁前的工作
    }
}
```

> 注意：销毁方法只针对单例模式的对象


#### 6.3 Bean的懒加载（Lazy-initialization mode）
Spring容器会在创建容器时提前初始化`Singleton作用域`的bean，可以通过bean标记`lazy-init="true"`延迟实例化(对象被使用时才创建)。
+ **配置lazy-init**
    - 配置**`beans`**的`default-lazy-init="true"`为所有Bean设定懒加载。
    - 配置**`bean`**的`lazy-init="true"`为单独的某个Bean设定懒加载。
``` xml
<bean id="bean1" class="example.exampleBean" lazy-init="true"/>
```

+ 适用场景：如果某个Bean在程序整个运行周期都可能不会被使用，可以考虑设定该Bean为懒加载
    - 优点：尽可能的节约了资源
    - 缺点：可能导致某个操作响应时间增加


#### 6.4 Bean装配的Aware接口 
实现了Aware接口的bean在初始化后可以获取相应资源并进行相应的操作。
1. ApplicationContextAware
    + 接口方法：setApplicationContext
    + 作用：通常用来获取上下文对象，声明全局变量后在方法中对变量进行初始化并供其他方法调用
    + 实现过程：创建一个类并实现ApplicationContextAware接口，重写setApplicationContext方法；在xml文件中配置该类；当spring加载该配置文件时即调用接口方法。
2. BeanNameAware
    + 接口方法：setBeanName
    + 作用：获取声明的类名，声明全局变量后在方法中对变量进行初始化并供其他方法调用
    + 实现过程：创建一个类并实现BeanNameAware接口，重写setBeanName方法；在xml文件中配置该类；当spring加载该配置文件时即调用接口方法。


#### 6.4 Bean装配之Resource
**Resources**（针对于资源文件的统一接口）
1. UrlResource：URL 对应的资源，根据一个 URL 地址即可获取
2. ClassPathResource：获取类路径下的资源
3. FileSystemResource：获取文件系统里面的资源
4. ServletContextResource：ServletContext 封装的资源，用于访问 ServletContext 环境下的资源
5. InputStreamResource：获取输入流封装的资源
6. ByteArrayResource：获取字节数组封装的资源

ResourceLoader: 所有的 application contexts 都实现了 ResourceLoader 接口，因此所有的 application contexts 都能通过getResource()获取Resource实例。
- getResource()参数：
    + classPath方式："classPath:class路径下文件"
    + file方式： "file:本地磁盘文件绝对地址"
    + url方式： "url:URL地址下文件"
    + 没有前缀时依赖applicationContext的配置文件路径: "文件全名"
- eg:`applicationContext.getResource("classpath:config.txt")`

