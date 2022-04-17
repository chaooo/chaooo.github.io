---
title: 「Spring」SpringBoot MVC应用
date: 2018-06-20 17:31:30
tags: [后端开发, Spring, SpringBoot]
categories: Spring
---

对Spring Web MVC封装，简化MVC结构web应用开发。
<!-- more -->

### 1. SpringBoot MVC开发Restful服务（前后分离）*
按rest规则发送HTTP请求-->Spring MVC-->返回JSON结果

主要步骤：
1. 导入spring-boot-starter-web（springmvc、rest、jackson、tomcat）
2. 在application.properties修改tomcat端口
3. 定义启动类RunBoot，追加@SpringBootApplication
4. 定义Controller、Service、Dao组件


### 2. SpringBoot MVC开发JSP应用（PC浏览器）
HTTP请求-->Spring MVC-->JSP-->HTML响应输出结果

主要步骤：
1. 导入spring-boot-starter-web、jasper解析器、jstl
2. 在application.properties修改tomcat端口、viewResolver
3. 定义启动类RunBoot，追加@SpringBootApplication
4. 定义Controller组件，返回ModelAndView
5. 在src/main/webapp下定义JSP组件


### 3. SpringBoot MVC开发Thymeleaf应用（PC浏览器）*
HTTP请求-->Spring MVC-->Thymeleaf模板-->HTML响应输出结果

主要步骤：
1. 导入spring-boot-starter-web、spring-boot-starter-thymeleaf
2. 在application.properties修改tomcat端口
3. 定义启动类RunBoot，追加@SpringBootApplication
4. 定义Controller组件，返回ModelAndView
5. 在src/main/resources/templates下定义模板文件
``` html
<html xmlns:th="https://www.thymeleaf.org/">
	<h1>Hello</h1>
	<h2 th:text="${data}"></h2>
</html>
```
> - th:text表达式作用：将模型中的数据以只读文本显示到元素中
> - th:text表达式作用：将模型中的数据以只读文本显示到元素中。
> - th:if 表达式作用：if判断逻辑
> - th:each 表达式作用：循环逻辑
> - th:href 表达式作用：动态生成href链接

Thymeleaf模板和JSP区别
1. 运行机制不同
    - JSP-->Servlet-->HTML
    - 模板+数据-->HTML输出
2. 模板简单易用;JSP相对复杂些
    - JSP:9大内置对象、EL、JSTL、嵌入Java代码、框架标签
    - 模板：模板表达式
3. 模板效率高,比JSP性能好
    - 模板：缓存



### 4. SpringBoot MVC静态资源处理
静态资源包含图片、js、css等，动态资源servlet、jsp等。

SpringBoot中src/main/resources目录下有几个约定的静态资源存放位置
- META-INF/resources（优先级最高）
- resources
- static
- public（优先级最低）

自定义静态资源访问路径，编写一个配置文件
``` java
//@Configuration
@Component
public class MyStaticConfiguration implements WebMvcConfigurer{
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**")
            .addResourceLocations(
                "classpath:/images/",
                "classpath:/resources/",
                "classpath:/static/",
                "classpath:/public/");
    }
}
```


### 5. SpringBoot MVC异常处理
1. 异常处理机制
    - SpringBoot底层提供了异常处理机制。SpringBoot提供了一个ErrorMvcAutoConfiguration自动配置组件，创建了一个BasicErrorController对象，提供两个/error请求处理，一个返回html，另一个返回json。当MVC底层遇到异常会用转发方式发出/error请求。

2. 可以自定义ErrorController替代底层BasicErrorController，将错误提示转发到自定义提示界面(全局)
``` java
@Controller
//@RequestMapping("/error")
public class MyErrorController implements ErrorController{
    @RequestMapping(value="/error",produces= MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml() {
        ModelAndView mav = new ModelAndView();
        mav.setViewName("myerror");
        return mav;
    }
    @RequestMapping(value="/error")
    @ResponseBody
    public Object error(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("msg", "程序发生了异常");
        return map;
    }
    @Override
    public String getErrorPath() {
        return "/error";
    }
}
```

3. @ExceptionHandler异常处理（局部）
    - ErrorController管理全局异常，@ExceptionHandler管理所在Controller组件的异常。
``` java
@ExceptionHandler
@ResponseBody
public Object error(Exception ex) {
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("msg", "发生异常");
    map.put("type", ex.getClass());
    return map;
}
```

可以将上述方法封装成一个BasicController，通过@ControllerAdvice作用到所有Controller组件上。
``` java
@ControllerAdvice//等价于所有Controller都继承它
public class BasicController {
    @ExceptionHandler
    @ResponseBody
    public Object error(Exception ex) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("msg", "发生异常");
        map.put("type", ex.getClass());
        return map;
    }
}
```


### 6. SpringBoot AOP
1. 引入spring-boot-starter-aop
2. 定义一个切面组件
``` java
@Component//将Bean组件纳入Spring容器
@Aspect//将Bean组件定义为Aspect切面
public class MyAspectBean {
    @Before("within(cn.xdl.controller.*)")//前置通知
    public void before() {
        System.out.println("----开始处理----");
    }
    @After("within(cn.xdl.controller.*)")//最终通知
    public void after() {
        System.out.println("----处理完毕----");
    }
    @Around("within(cn.xdl.controller.*)")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        StopWatch watch = new StopWatch();
        watch.start();
        Object obj = pjp.proceed();//调用目标组件方法
        watch.stop();
        System.out.println("处理时间:"+watch.getTotalTimeMillis()+" 毫秒");
        return obj;
    }
}
```

3. 配置切面组件
    - @Aspect、@Before、@After、@Around、@AfterReturning、@AfterThrowing等



### 7. SpringBoot MVC拦截器
1. 编写一个拦截器组件,实现HandlerInterceptor接口
``` java
@Component
public class MyInterceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler)
            throws Exception {
        System.out.println("执行了MyInterceptor拦截器");
        String user = (String)request.getSession().getAttribute("user");
        if(user == null) {
            response.sendRedirect("/tologin");
            return false;//阻止后续流程执行
        }
        return true;//继续执行后续处理
    }
}
```

2. 配置拦截器组件
``` java
@Configuration
public class MyInterceptorConfiguration implements WebMvcConfigurer{
    @Autowired
    private MyInterceptor my;
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(my).addPathPatterns("/direction/list");
    }
}
```


### 8. SpringBoot整合Servlet/Filter
#### 8.1 整合Servlet
首先导入spring-boot-starter-web

##### 8.1.1 整合Servlet方式一：
1. 编写一个Servlet组件，继承HttpServlet
2. 在Servlet类定义前使用@WebServlet
``` java
@WebServlet(name="helloservlet",urlPatterns= {"/hello.do"},loadOnStartup=1)
public class HelloServlet extends HttpServlet{
    public void service(
        HttpServletRequest request,
        HttpServletResponse response
    ) throws IOException {
        response.getWriter().println("Hello SpringBoot Servlet");
    }
}
```

3. 启动类前需要使用@ServletComponentScan扫描@WebServlet配置
``` java
@SpringBootApplication
@ServletComponentScan //扫描@WebServlet、@WebFilter、@WebListener组件
public class RunBoot {
    public static void main(String[] args) {
        SpringApplication.run(RunBoot.class, args);
    }
}
```


##### 8.1.2 整合Servlet方式二：
1. 编写一个Servlet组件，继承HttpServlet
``` java
public class SomeServlet extends HttpServlet {
    public void service(
        HttpServletRequest request, 
        HttpServletResponse response
    ) throws IOException {
        response.getWriter().println("Hello Spring Some Servlet");
    }
}
```

2. 使用ServletRegistrationBean+@Bean
``` java
@SpringBootApplication
public class RunBoot {
    public static void main(String[] args) {
        SpringApplication.run(RunBoot.class, args);
    }

    @Bean
    public ServletRegistrationBean<Servlet> someservlet(){
        ServletRegistrationBean<Servlet> bean = new ServletRegistrationBean<Servlet>();
        bean.setServlet(new SomeServlet());
        bean.setLoadOnStartup(1);
        List<String> urls = new ArrayList<>();
        urls.add("/some.do");
        bean.setUrlMappings(urls);
        return bean;
    }
}
```


#### 8.2 整合Filter
在SpringBoot整合Servlet的基础上整合Filter

##### 8.2.1 整合Filter方式一：
1. 编写一个Filter组件，继承Filter
2. 在Filter类定义前使用@WebFilter
``` java
@WebFilter(urlPatterns="/hello.do")
public class HelloFilter implements Filter{
    @Override
    public void doFilter(
        ServletRequest request, ServletResponse response, FilterChain chain
    ) throws IOException, ServletException {
        System.out.println("-----hello filter------servlet执行之前");
        chain.doFilter(request, response);
        System.out.println("-----hello filter------servlet执行之后");
    }
}
```

3. 启动类前需要使用@ServletComponentScan扫描@WebServlet配置
``` java
@SpringBootApplication
@ServletComponentScan //扫描@WebServlet、@WebFilter、@WebListener组件
public class RunBoot {
    public static void main(String[] args) {
        SpringApplication.run(RunBoot.class, args);
    }
}
```

##### 8.2.2 整合Filter方式二：
1. 编写一个Filter组件，继承Filter
``` java
public class SomeFilter implements Filter{
    @Override
    public void doFilter(
        ServletRequest request, ServletResponse response, FilterChain chain
    ) throws IOException, ServletException {
        System.out.println("-----som filter------servlet执行之前");
        chain.doFilter(request, response);
        System.out.println("-----som filter------servlet执行之后");
    }
}
```

2. 使用FilterRegistrationBean+@Bean 注册过滤器并设置拦截的请求地址
``` java
@SpringBootApplication
public class RunBoot {
    public static void main(String[] args) {
        SpringApplication.run(RunBoot.class, args);
    }
    ...
    @Bean
    public FilterRegistrationBean<Filter> somefilter(){
        FilterRegistrationBean<Filter> bean = new FilterRegistrationBean<Filter>();
        bean.setFilter(new SomeFilter());
        // 配置要拦截的请求
        bean.addUrlPatterns("/some.do");
        return bean;
    }
}
```


### 9. SpringBoot 任务调度
#### 9.1 服务器启动后自动调用
tomcat服务器启动后自动调用任务，可以使用ApplicationRunner或CommandLineRunner接口。
``` java
@Component
@Order(2)
public class SomeTask1 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("----服务器启动后自动执行SomeTask1任务---" + new Date());
    }
}

@Component
@Order(1)
public class SomeTask2 implements CommandLineRunner{
    @Override
    public void run(String... args) throws Exception {
        System.out.println("----服务器启动后自动执行SomeTask2任务-----"+new Date());
        Thread.sleep(5000);
    }
}
```
> 多个Task任务，可以通过@Order指定先后顺序，多个任务是线程同步调用。



#### 9.2 程序运行后定时调用任务
Spring提供了一个Spring Schedule模块，封装了任务调用，之前都是采用Quartz组件调用。
``` java
@Component
@EnableScheduling//开启Schedule模块
public class SomeTask3 {
    //在服务器启动1秒后调用任务，每隔3秒调用一次
    @Scheduled(initialDelay=1000,fixedRate=3000)
    public void execute() {
        System.out.println("-----周期性调用SomeTask3-----"+new Date());
    }
    
    //在服务器启动0秒后调用任务，每隔5秒调用一次
    @Scheduled(cron="0/5 * * * * ?")//秒 分 时 日 月 星期
    public void execute2() {
        System.out.println("-----周期性调用SomeTask4-----"+new Date());
    }
}
```

使用Spring Schedule还需要指定cron表达式，表达式具体规则：
``` 
秒   分    时    日   月   星期   年（可省略）
0    0     10    1   10    ？
秒： 0-59
分： 0-59
时： 0-23
日： 1-31
月： 1-12
星期：1-7，1表示星期日，7表示星期六
* ： 表示每一分、每一秒、每一天，任何一个可能值
? ： 只用在日和星期部分，如果指定日，星期用？;如果指定星期，日用?，避免日和星期冲突 
/ ： 表示增量，0/1表示0\1\2\3\4递增加1；0/5表示0\5\10\15；1/5表示1\6\11\16\21
L ： 只用在日和星期部分，表示最后一天、周六
```

cron表达式案例：
```
"30 * * * * ?" 每半分钟触发任务
"30 10 * * * ?" 每小时的10分30秒触发任务
"30 10 1 * * ?" 每天1点10分30秒触发任务
"30 10 1 20 * ?" 每月20号1点10分30秒触发任务
"30 10 1 20 10 ? *" 每年10月20号1点10分30秒触发任务
"30 10 1 20 10 ? 2011" 2011年10月20号1点10分30秒触发任务
"30 10 1 ? 10 * 2011" 2011年10月每天1点10分30秒触发任务
"30 10 1 ? 10 SUN 2011" 2011年10月每周日1点10分30秒触发任务
"15,30,45 * * * * ?" 每15秒，30秒，45秒时触发任务
"15-45 * * * * ?" 15到45秒内，每秒都触发任务
"15/5 * * * * ?" 每分钟的每15秒开始触发，每隔5秒触发一次
"15-30/5 * * * * ?" 每分钟的15秒到30秒之间开始触发，每隔5秒触发一次
"0 0/3 * * * ?" 每小时的第0分0秒开始，每三分钟触发一次
"0 15 10 ? * MON-FRI" 星期一到星期五的10点15分0秒触发任务
"0 15 10 L * ?" 每个月最后一天的10点15分0秒触发任务
"0 15 10 LW * ?" 每个月最后一个工作日的10点15分0秒触发任务
"0 15 10 ? * 5L" 每个月最后一个星期四的10点15分0秒触发任务
"0 15 10 ? * 5#3" 每个月第三周的星期四的10点15分0秒触发任务
```



#### 9.3 SpringBoot+Quartz
导入spring-boot-starter-quartz, 编写Job任务组件，继承QuartzJobBean
``` java
public class MyTask5 extends QuartzJobBean{
    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        System.out.println("通过Quartz工具调用定时任务"+new Date());
    }
}
```

配置Job组件（JobDetail、Tigger）
``` java
@Configuration
public class QuartzConfiguration {
    @Bean//将MyTask5任务组件封装成JobDetail
    public JobDetail task5() {
        return JobBuilder.newJob(MyTask5.class)
            .withIdentity("task5").storeDurably().build();
    }
    @Bean//为JobDetail指定触发时间cron表达式
    public Trigger task5Trigger() {
        CronScheduleBuilder cronScheduleBuilder = 
            CronScheduleBuilder.cronSchedule("0/5 46 10 * * ?");
        return TriggerBuilder.newTrigger()
                .forJob(task5())
                .withIdentity("task5")
                .withSchedule(cronScheduleBuilder)
                .build();
    }
}
```
