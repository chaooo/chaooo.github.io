---
title: 「Spring」Spring MVC框架
date: 2018-04-27 22:50:22
tags: [后端开发, Spring]
categories: Spring
---

Spring MVC是Spring提供的一个强大而灵活的web框架。借助于注解，Spring MVC提供了几乎是POJO的开发模式，使得控制器的开发和测试更加简单。这些控制器一般不直接处理请求，而是将其委托给Spring上下文中的其他bean，通过Spring的依赖注入功能，这些bean被注入到控制器中。
<!-- more -->


### 1. Spring MVC基本概念
#### 1.1 Spring MVC 五大核心组件
Spring MVC主要由DispatcherServlet、处理器映射、处理器(控制器)、视图解析器、视图组成。
1. DispatcherServlet：控制器，请求入口
2. HandlerMapping：控制器，分发请求，让请求和控制器建立一一对应关系
3. Controller：控制器，处理请求
4. ModelAndView：封装了 数据信息和视图信息
5. ViewResolver：视图处理器

他的两个核心是两个核心：
- 处理器映射：选择使用哪个控制器来处理请求 
- 视图解析器：选择结果应该如何渲染
> 通过以上两点，Spring MVC保证了如何选择控制处理请求和如何选择视图展现输出之间的松耦合。

#### 1.2 SpringMVC运行原理
1. Http请求：客户端请求提交到DispatcherServlet。 
2. 寻找处理器：由DispatcherServlet控制器查询一个或多个HandlerMapping，找到处理请求的Controller。 
3. 调用处理器：DispatcherServlet将请求提交到Controller。 
4. 调用业务处理和返回结果：Controller调用业务逻辑处理后，返回ModelAndView。 
5. 处理视图映射并返回模型： DispatcherServlet查询一个或多个ViewResoler视图解析器，找到ModelAndView指定的视图。 
6. Http响应：视图负责将结果显示到客户端。

#### 1.3 SpringMVC接口解释
1. **DispatcherServlet接口**：Spring提供的前端控制器，所有的请求都有经过它来统一分发。在DispatcherServlet将请求分发给Spring Controller之前，需要借助于Spring提供的HandlerMapping定位到具体的Controller。它是整个Spring MVC的核心。它负责接收HTTP请求组织协调Spring MVC的各个组成部分。其主要工作有以下三项： 
    1. 截获符合特定格式的URL请求。 
    2. 初始化DispatcherServlet上下文对应WebApplicationContext，并将其与业务层、持久化层的WebApplicationContext建立关联。 
    3. 初始化Spring MVC的各个组成组件，并装配到DispatcherServlet中。
2. **HandlerMapping接口**：能够完成客户请求到Controller映射。 
3. **Controller接口**： 需要为并发用户处理上述请求，因此实现Controller接口时，必须保证线程安全并且可重用。 
    - Controller将处理用户请求，这和Struts Action扮演的角色是一致的。一旦Controller处理完用户请求，则返回ModelAndView对象给DispatcherServlet前端控制器，ModelAndView中包含了模型（Model）和视图（View）。 
    - 从宏观角度考虑，DispatcherServlet是整个Web应用的控制器；从微观考虑，Controller是单个Http请求处理过程中的控制器，而ModelAndView是Http请求过程中返回的模型（Model）和视图（View）。 
4. **ViewResolver接口**：Spring提供的视图解析器（ViewResolver）在Web应用中查找View对象，从而将相应结果渲染给客户。

#### 1.4 SpringMVC配置
1. 在web.xml文件中进行配置applicationContext.xml路径
``` xml
<!-- 配置DispatcherServlet -->
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>
```

2. 配置applicationContext.xml，开启注解功能、配置试图解析器
``` xml
	<!-- 配置HandlerMapping -->
	<bean id="handlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		<property name="mappings">
			<props>
				<prop key="/toHello.do">helloController</prop>
			</props>
		</property>
	</bean>
	<!-- 控制器对象 -->
	<bean id="helloController" class="com.controller.MyHelleController"></bean>
	<!-- 配置视图处理器 -->
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
```


### 2. Spring MVC的编写步骤
1. 建立一个项目，导入jar包(ioc mvc) 拷贝spring配置文件到src下，同时在WEB-INF下建立jsp文件。
2. 在web.xml中配置DisappearServlet，并通过contextConfigLocation这个初始化参数关联Spring容器对应的配置文件。
3. 在 Spring配置文件中配置HandlerMapping的实现类SimpleUrlHandlerMapping需要通过mappings属性指定请求和控制器对应的关系。
4. 编写一个类实现Controller接口，实现接口方法，返回ModelAndView，并且在容器创建Controller对象
5. 在Spring配置文件中配置ViewResolver的实现类InternalResourceViewResolver，需要配置前缀prefix和后缀suffix。


### 3. 标注(注解)形式的MVC
1. 建立项目，导入jar(ioc aop mvc)，拷贝spring配置文件到src下，同时在WEB-INF下建立jsp文件。
2. 在web.xml中配置DispatcherServlet，并通过contextConfigLocation关联配置文件。
3. 开启组件扫描 和 标注形式mvc (容器帮你创建了一个HandlerMapping对象，类型时RequestMappingHandlerMapping)。
``` xml
<context:component-scan base-package="包名" />
<mvc:annotation-driven />
```

4. 编写一个Java类，不用实现Controller接口，方法返回值类型可以时String也可以是ModelAndView（方法名与参数都自由了）
    + 使用`@Controller` 可以把普通Java类转换成控制器，同时在容器中创建对象
    + 使用`@RequestMapping("/路径")` 设置方法上
5. 在Spring配置文件中配置ViewResolver的实现类InternalResourceViewResolver，需要配置前缀prefix和后缀suffix。


### 4. mvc控制器接收页面参数
1. 使用HttpServletRequest类型的参数来接收
``` java
@RequestMapping("/login.do")
public String login(HttpServletRequest request) {
    String acc_no = request.getParameter("acc_no");
    String acc_pwd = request.getParameter("acc_password");
    return "main";
}
```

2. 直接定义和页面请求参数同名的控制器参数
``` java
@RequestMapping("/login2.do")
public ModelAndView login2(String acc_no,String acc_password, ModelAndView mav) {
    mav.setViewName("main");
    return mav;
}
```

3. 当页面参数和控制器参数名字不一致，@RequestParam("acc_no") 让请求参数和控制器参数对应
``` java
@RequestMapping("/login3.do")
public ModelAndView login3(@RequestParam("acc_no") String a,String acc_password, ModelAndView mav) {
    mav.setViewName("main");
    return mav;
}
```

4. 控制器中 直接定义对象类型的参数
``` java
@RequestMapping("/login4.do")
public ModelAndView login4(Account acc, ModelAndView mav) {
    mav.setViewName("main");
    return mav;
}
```


### 5. mvc控制器把数据传递给页面 
使用EL表达式在jsp页面接收数据`<h1>欢迎 ${acc_no} </h1>`
1. 使用域对象 进行传输 (request session ServletContext )
``` java
@RequestMapping("/login6.do")
public String login6(String acc_no, HttpServletRequest req) {
    req.setAttribute("acc_no", acc_no);
    return "main";
}
```

2. 使用ModelAndView进行数据传输 
    + `mav.getModel().put("acc_no", acc_no);`
    + `mav.getModelMap().put(key, value);`
    + `mav.getModelMap().addAttribute("acc_no", acc_no);`

``` java
@RequestMapping("/login7.do")
public ModelAndView login7(String acc_no, ModelAndView mav) {
    mav.setViewName("main");
    //mav.getModel().put("acc_no", acc_no);
    //mav.getModelMap().put(key, value)
    mav.getModelMap().addAttribute("acc_no", acc_no);
    return mav;
}
```

3. 使用Model进行数据传输
``` java
@RequestMapping("/login8.do")
public String login8(String acc_no, Model m) {
    m.addAttribute("acc_no", acc_no);
    return "main";
}
```

4. 使用ModelMap进行数据传输
``` java
@RequestMapping("/login9.do")
public String login9(String acc_no, ModelMap m) {
    //m.addAttribute("acc_no", acc_no);
    m.put("acc_no", acc_no);
    return "main";
}
```

5. 使用自定义的对象类型默认传输（默认名类型首字母小写，可以通过@ModelAttribute("新名")修改）
    + 默认名：`<h1>欢迎 ${ account.acc_no } </h1>`
    + @ModelAttribute("acc")：`<h1>欢迎 ${ acc.acc_no } </h1>`

``` java
@RequestMapping("/login10.do")
public String login10(@ModelAttribute("acc") Account acc) {
    return "main";
}
```


### 6. Spring MVC实现重定向
1. 控制器方法返回String 
    + redirect:请求路径

``` java
@RequestMapping("/login11.do")
public String login11(@ModelAttribute("acc") Account acc) {
    //return "forward:toMain.do";
    return "redirect:toMain.do";
}
@RequestMapping("/toMain.do")
public String toMain() {
    // 干其它的事情
    return "main";
}
```

2. 控制器方法返回ModelAndView 
    + 使用RedirectView  完成

``` java
@RequestMapping("/login12.do")
public ModelAndView login12(@ModelAttribute("acc") Account acc) {
    ModelAndView mav = new ModelAndView();
    //重定向
    RedirectView rv = new RedirectView("toMain.do");
    mav.setView(rv);
    return mav;
}
```


### 7. Spring MVC 中文参数的乱码问题 
tomcat8中 get 没有乱码问题，post 请求有乱码问题 
1. 参数为页面(HttpServletRequest request)与(HttpServletResponse response)时
``` java
request.setCharacterEncoding("UTF-8");
response.setContentType("application/json;charset=UTF-8");
```

2. 传入参数为`@RequestParam`时，可以通过字符串重新编码来解决
``` java
new String(string.getBytes("ISO-8859-1"),"UTF-8");
```

3. 方法名前出现`@RequestMapping(value="XXX")`时可以在value属性后再加一个属性`produces="text/html;charset=UTF-8"`来解决

4. 在web.xml或者dispatcher-servlet.xml或者其他配置servlet的配置文件中添加编码过滤器
``` java
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>*.do</url-pattern>
</filter-mapping>
```


### 8. Spring MVC 拦截器
1. 拦截器和fiter的作用几乎一样，它是Spring提供的一个组件，可以用在HandlerMapping组件之后（用于身份认证，登录检查，编码设置）
2. HandlerMapping接口
    * preHandle：在HandlerMapping之后控制器之前调用，返回boolean(true:继续其他拦截器和处理器，false:终止后续调用)。
    * postHandle：处理器执行后、视图处理前调用。
    * afterCompletion：整个请求处理完毕后调用。


### 9. Spring MVC 拦截器的使用步骤
1. 搭建一个基于标注的mvc
2. 编写一个类实现HandlerInterceptor接口
3. 在Spring配置文件中配置拦截器
``` xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/login.do"/>
        <bean class="com.xdl.interceptor.SomeInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```


### 10. Spring MVC异常处理
1. 配置spring系统提供的简单异常处理器 SimpleMappingExceptionResolver 处理所有Controller异常
``` xml
<bean id="simpleExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
            <prop key="java.lang.RuntimeException">error</prop>
            <prop key="java.lang.Exception">error2</prop>
        </props>
    </property>
</bean>
```

2. 自定义异常处理器，实现HandlerExceptionResolver接口，处理所有Controller异常
``` java
@Controller
public class MyExceptionResolver implements HandlerExceptionResolver {
	@Override
	public ModelAndView resolveException(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception e) {
		ModelAndView mav = new ModelAndView();
		if(e instanceof RuntimeException) {
			mav.setViewName("error");
		}else if(e instanceof Exception) {
			mav.setViewName("error2");
		}
		return mav;
	}
}
```

3. 使用@ExceptionHandler注解实现异常处理，处理某一个Controller异常public String execute(HttpServletRequest request, Exception ex)
``` java
//@Controller
//public class MyController {
@ExceptionHandler
public String processException(Exception e) {
    System.out.println(e.getMessage());
    return "error3";
}
```


### 11. Spring MVC文件上传
1. jsp页面（method="POST" enctype="multipart/form-data type="file"）
``` html
<form action="upload.do" method="post" enctype="multipart/form-data">
    头像：<input type="file" name="head_img"><br>
    <input type="submit" value="上传"><br>
</form>
```
2. 控制器（MultipartFile类型来接收文件数据，需要配置文件解析器-需要依赖文件上传jar包-commons包）
``` xml
<bean id="multipartResolver" 
    class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
</bean>
```


### 12. 文件上传与异常处理的结合
``` java
@Controller
public class fileController {
	@RequestMapping("/toFile.do")
	public String tofile() {
		return "file";
	}
	@RequestMapping("/upload.do")
	public String upload(String acc_no, MultipartFile head_img) {
		System.out.println("acc_no:" + acc_no );
		if(head_img.getSize()>1024*10) {
			throw new RuntimeException("文件过大！");
		}
		// 把文件写入磁盘
		String uniqueStr = UUID.randomUUID().toString();
		String oriFilename = head_img.getOriginalFilename();
		String suffix = oriFilename.substring(oriFilename.lastIndexOf("."));
		File file = new File("F:/Eclipse/datas/"+uniqueStr+suffix);
		try {
			head_img.transferTo(file);
		} catch (IllegalStateException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		System.out.println(oriFilename);
		System.out.println(file);
		return "file";
	}
	/** 局部异常 */
	@ExceptionHandler
	public String processError(Exception e) {
		return "error4";
	}
}
```


### 13. Spring MVC响应JSON
1. 搭建基于标注的mvc
2. 在控制器中，设计控制方法，控制方法返回值数据类型对应的对象转换为JSON
4. 给方法加@RequestMapping("/请求路径")、@ResponseBody，它能把Java对象转换为JSON直接返回，依赖json转换包


### 14. REST
REST即表述性状态传递（Representational State Transfer），使用这种软件架构风格，可以降低开发的复杂性，提高系统的可伸缩性，便于分布式应用的开发。
1. REST两个核心规范
    * url请求路径的格式，由原来的基于操作的设计改变了基于资源的设计（如:http://test/source/1234）
    * 对http请求的方式做了规范，GET代表查询，POST增加，DELETE删除，PUT更新
2. restful
    * 符合REST设计规范和风格的应用程序或设计 就是RESTful
3. Spring MVC对REST的支持
    * @RequestMapping支持URI的模板，以及http请求方式设定的支持
        + `@RequestMapping(value="/account/{id}",method=RequestMethod.POST)`
    * 对URI上路径变量的处理的支持，@PathVariable
        + `@PathVariable("id") int id`
    * rest请求路径是没有后缀的，需要把url-parttern修改成`/`
    * `<servlet-mapping><servlet-name>DispatcherServlet</servlet-name><url-pattern>/</url-pattern></servlet-mapping>`
    * 需要对静态资源进行放行`<mvc:default-servlet-handler/>`



### 15. REST实例
1. 配置web.xml与applicationContext.xml(部分配置)

``` xml
<!-- 修改rest请求路径 -->
<!-- web.xml -->
    <servlet-mapping>
      <servlet-name>DispatcherServlet</servlet-name>
      <url-pattern>/</url-pattern>
    </servlet-mapping>

<!-- 对静态资源进行放行 -->
<!-- applicationContext.xml -->
    <mvc:default-servlet-handler/>
```

2. 编写控制类

``` java
@Controller
public class AccountController {
    @RequestMapping("/toLogin.do")
    public String toLogin() {
        return "login";
    }
/** 根据id查询账户 GET */
    @RequestMapping(value="/account/{id}", method=RequestMethod.GET)
    @ResponseBody
    public Account getAccountById(@PathVariable("id") int id) {
        Random rm = new Random();
        Account acc = new Account(id, "test"+rm.nextInt(100),"123", rm.nextInt(999)+1000);
        return acc;
    }
/** 新增账户 POST */
    @RequestMapping(value="/account/{id}",method=RequestMethod.POST)
    @ResponseBody
    public boolean addAccount(Account acc) {
        System.out.println("add:"+acc);
        if(acc.getId()>100) return true;
        return false;
    }
/** 根据id删除帐户对象 DELETE */
    @RequestMapping(value="/account/{id}",method=RequestMethod.DELETE)
    @ResponseBody
    public boolean deleteAccountById(@PathVariable("id") int id) {
        System.out.println("delete:"+id);
        if(id>100) return true;
        return false;
    }
/** 根据id更新帐户 PUT */
    @RequestMapping(value="/account/{id}",method=RequestMethod.PUT)
    @ResponseBody
    public boolean putAccount(@RequestBody Account acc) {
        //@RequestBody将接收的ajax请求的json字符串写入Account对象中
        System.out.println("update:"+acc);
        if(acc.getId()>100) return true;
        return false;
    }
}
```

3. 编写jsp页面

``` html
<form>
    <p>ID：<input id="accountId"></p>
    <p>姓名：<input id="accountNo"></p>
    <p>密码：<input id="accountPassword"></p>
    <p>金额：<input id="accountMoney"></p>
    <button id="findBtn" type="button">查询</button>
    <button id="addBtn" type="button">添加</button>
    <button id="updateBtn" type="button">更新</button>
    <button id="delBtn" type="button">删除</button>
</form>

<script src="js/jquery.min.js"></script>
<script>
$("#findBtn").on("click", function(){
    findAccount();
});
$("#addBtn").on("click", function(){
    addAccount();
});
$("#updateBtn").on("click", function(){
    updateAccount();
});
$("#delBtn").on("click", function(){
    delAccount();
});
function getDatas(){
	var accountId = $("#accountId").val();
	var accountNo = $("#accountNo").val();
	var accountPassword = $("#accountPassword").val();
	var accountMoney = $("#accountMoney").val();
	return {
	    id: accountId,
	    acc_no: accountNo,
	    acc_password: accountPassword,
	    acc_money: accountMoney
	};
}
function findAccount(){
	var datas = getDatas();
    $.ajax({
        url: "account/" + datas.id,
        type: "get",
        success: function(res){
            $("#accountNo").val(res.acc_no);
            $("#accountPassword").val(res.acc_password);
            $("#accountMoney").val(res.acc_money);
        },
    });
}
function addAccount(){
	var datas = getDatas();
    $.ajax({
        url: "account/" + datas.id,
        type: "post",
        data: datas,
        success: function(res){
            alert(res);
        },
    });
}
function delAccount(){
	var datas = getDatas();
    $.ajax({
        url: "account/" + datas.id,
        type: "delete",
        success: function(res){
            alert(res);
        },
    });
}
function updateAccount(){
	var datas = getDatas();
    $.ajax({
        url:"account/"+ datas.id,
        type:"put",
        data:JSON.stringify(datas),
        contentType:"application/json",//以json字符串提交数据
        success: function(res){
            alert(res);
        },
    });
}
</script>
```

注意：
> - PUT需要以json字符串提交数据`contentType:"application/json"`
> - @RequestBody将接收的ajax请求的json字符串写入Account对象中
> - JSON.stringify()：将json对象转换为json字符串
> - JSON.parse()：将json字符串转换为json对象

