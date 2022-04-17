---
title: 「Spring」IoC注解实现
date: 2018-04-03 22:44:07
tags: [后端开发, Spring]
categories: Spring
---


### 1. 回顾xml方式管理Java Bean
1. 将一个Bean交由Spring创建并管理
    - `<baen id="bean" class="包名.Bean"></baen>`
2. 获取Spring上下文
    - `ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");`
3. 获取Bean
    - `Bean bean = app.getBean("bean", Bean.class);`

<!-- more -->

### 2. 注解方式管理Java Bean
一、创建一个class配置文件
``` java
@Configuration
public class MyConfiguration{
    //将一个Bean交由Spring创建并管理
    @Bean(name="bean1")
    public Bean bean(){
        return Bean = new Bean();
    }
}
```

二、获取Spring上下文
``` java
ApplicationContext context = 
    new AnnotationConfigApplicationContext(MyConfiguration.class);
```

三、获取Bean
``` java
Bean1 bean1 = context.getBean("bean1", Bean1.class);
```

#### 2.1 简化注解方式的步骤1
一、 开启组件扫描（去掉上述步骤1中MyConfiguration实例化Bean的方法）
``` java
@Configuration //该注解可理解当前class等同于一个xml文件
@ComponentScan("包路径") //开启组件扫描
public class MyConfiguration{}
```
> 在applicationContext.xml中开启组件扫描方式`<context:component-scan base-package="包路径"/>`。

二、 将交由Spring管理的类加上`@Component`注解，或（`@Repository`，`@Controller`，`@Service`）
``` java
@Component("bean1")//通过构造方法实例化Bean1
public class Bean1{
    //...
}
```
> @Component是通用注解，其他三个注解是这个注解的拓展，并且具有了特定的功能 
> - @Repository注解在持久层中，具有将数据库操作抛出的原生异常翻译转化为spring的持久层异常的功能。 
> - @Controller层是spring-mvc的注解，具有将请求进行转发，重定向的功能。 
> - @Service层是业务逻辑层注解，这个注解只是标注该类处于业务逻辑层。 

#### 2.2 Bean别名
一、 xml形式：通过name属性或alias标签
``` xml
<bean id="bean1" name="bean2,bean3" class="com...Bean"/>
<alias name="bean1" alias="bean4"/>
```

二、 注解形式
``` java
@Configuration
public class MyConfiguration{
    @Bean(name={"bean1","bean2","bean3"})
    public Bean1 bean1(){
        return Bean1 = new Bean1();
    }
}
```
> 注意：@Component只能指定一个名字，@Component默认值为**类名首字母小写**，也可以自定义，如:`@Component("bean1")`； 默认@scope为singleton单例，也可以进行指定


### 3. 注解方式Bean的注入
一、 **`@Value("值")`**：常用于基本数据类型值注入，`值`可用EL表达式。
``` java
@Component
public class Player{
    @Value("张三")
    private String name;
    //...
}
```

二、 **`@Autowired`**：常用于复杂类型值的注入
    + `@Autowired`：可以用在**成员变量**，**setter方法**，**构造方法**上；优先按照类型进行匹配，匹配不上启用名字进行匹配。
    + `@Qualifier("名字")` 根据名字匹配，配合@Autowired，**不能用在构造方法上**；@Qualifier指定对象必须存在，否则程序报错，可以使用@Autowired的required属性来解除这种强依赖，`@Autowired(required=false)`:尽量去找，组件不存在也不报错。
    + @Autowired的**原理**：在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性
``` java
@Component
public class Player{
    @Value("张三")
    private String name;

    /** 用于成员变量 */
    //@Autowired
    //@Qualifier("card1")
    private Card card;

    /** 用于构造方法 */
    //@Autowired
    public Player(Card card) {
        super();
        this.card = card;
    }

    /** 用于setter方法 */
    @Autowired(required=false)
    public void setCard(Card card) {
        this.card = card;
    }
}
```

三、 **`@Resource`**：常用于复杂类型值的注入
    + @Resource：用在**成员变量**和**setter方法**上，是**JDK**1.6支持的注解，优先按照名字匹配，可以通过`@Resource(name="名")`指定；如果没有指定name属性，用在成员变量上默认取字段名，用在setter方法上默认取属性名进行装配。名字匹配不上，会动用类型匹配。但注意：如果name属性一旦指定，就只会按照名称进行装配。
``` java
@Component
public class Player{
    @Resource(name="card")
    private Card card;
    //...
}
```

集合类型值注入实例
``` java
@Configuration
@ComponentScan("包路径")
public class MyConfiguration{
    @Bean
    public List<String> list(){
        List<String> list = new ArrayList<String>();
        list.add("aaa");
        list.add("bbb");
        return list;
    }
}

@Component
public class Player{
    @Autowired
    private List<String> list;
    //...
}
```


### 4. 注解方式Bean的常用配置项(作用域,生命周期,懒加载等)
#### 4.1 注解方式Bean的作用域
``` java
@Configuration
@ComponentScan("包路径")
public class MyConfiguration{
    @Bean(name="bean1")
    @Scope("singleton")
    public Bean1 bean1(){
        return Bean1 = new Bean1();
    }
}

@Component
@Scope("singleton")
public class Bean{}
```


#### 4.2 注解方式Bean的懒加载
``` java
@Configuration
@ComponentScan("包路径")
@Lazy //相当于xml中default-lazy-init="true"
public class MyConfiguration{
    @Bean(name="bean1")
    @Lazy 
    public Bean1 bean1(){
        return Bean1 = new Bean1();
    }
}

@Component
@Lazy
public class Bean{}
```


#### 4.3 Bean初始化和销毁
一、实现InitializingBean和DisposableBean接口（xml和注解都支持）。
``` java
public class Bean implements InitializingBean{
    @Override
    public void afterPropertiesSet(){
        //执行一些初始化后的工作
    }
}

public class Bean implements DisposableBean{
    @Override
    public void destroy(){
        //执行一些销毁前的工作
    }
}
```

二、xml形式
``` java
public class Bean{
    public void init(){
        //执行一些初始化后的工作
    }
    public void cleanup(){
        //执行一些销毁前的工作
    }
}
```
``` xml
<bean id="bean" class="example.Bean" 
        init-method="init" 
        destroy-method="cleanup"></bean>
```

三、注解形式1，@Bean(initMethod="init", destroyMethod="cleanup")
``` java
public class Bean{
    public void init(){
        //执行一些初始化后的工作
    }
    public void cleanup(){
        //执行一些销毁前的工作
    }
}

@Configuration
public class MyConfiguration{
    @Bean(initMethod="init", destroyMethod="cleanup")
    public Bean bean(){
        return new Bean();
    }
}
```

四、注解形式2，添加@PostConstruct，@PreDestroy
``` java
@Component
public class Bean{
    @PostConstruct
    public void init(){
        //执行一些初始化后的工作
    }
    @PreDestroy
    public void cleanup(){
        //执行一些销毁前的工作
    }
}
```

