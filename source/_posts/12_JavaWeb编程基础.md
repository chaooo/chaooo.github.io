---
title: 十二、JavaWeb编程基础
date: 2019-07-20 20:59:40
tags: [javaEE, 后端开发]
categories: JavaWeb编程
---


JavaWeb是用Java技术来解决相关web互联网领域的技术总和。Java提供了技术方案可以解决客户端和服务器端的实现，特别是服务器的应用，比如Servlet，JSP和第三方框架等等。
<!-- more -->

1. [http协议](#id1)
2. [Servlet](#id2)
3. [get和post区别](#id3)
4. [上下文对象ServletContext](#id4)
5. [会话跟踪（状态管理）](#id5)
6. [JSP](#id6)


<span id="id1"><span>
### 1. http协议
超文本传输协议，是一种应用层的网络传输协议

- http协议的特点：
  1. 简单，快速：支持多种不同的的数据提交方式，如get/post
  2. 数据传输灵活，支持任意类型数据的传输
  3. 无连接协议：每次连接，只处理一次请求，进行一次响应，响应完毕，立即断开。
  4. 无状态协议：处理请求与响应时没有记忆能力，如果需要处理之间的信息，只能重新传递。
- http协议的组成部分：
  1. 请求：浏览器连接服务器的过程
  2. 响应：服务器回复浏览器的过程
- http协议的请求：
  1. 请求头：描述客户端的信息
  2. 请求体：GET没有请求体，请求体用于存储POST请求发送的数据。
  3. 请求空行：请求头与请求体之间的一行空白
  4. 请求行：描述请求方式，服务器地址，协议版本等
- http协议的响应：
  1. 响应头：描述服务器的信息
  2. 响应体：响应的内容，文本，json数据等。
  3. 响应行：描述服务器协议版本，响应状态码，以及响应成功或失败的解释。

<span id="id2"><span>
### 2. Servlet
servlet是一个运行在tomcat上的Java类，用户通过浏览器输入地址，触发这个类，这个类执行完毕，准备一个响应体，发送给浏览器。

#### 2.1 Servlet编写步骤：
1. 编写一个Java类，继承HttpServlet类
2. 重新service方法
3. 在service方法中，对用户请求进行响应。

``` java
//注解：添加访问的网址
@WebServlet("/hello")
public class MyServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
	@Override
    public void service(ServletRequest req, ServletResponse res) throws IOException {
		//1.设置响应体的编码，以及内容类型
		res.setContentType("text/html;charset=utf-8");
		//2.得到响应体输出的打印流
		PrintWriter out = res.getWriter();
		//3.打印文字
		out.println("<h1>Hello Servlet!</h1>");
	}

}
```


#### 2.2 配置ervlet类的访问网址
- web3.0版本之后使用注解的方式配置ervlet类的访问网址
- web3.0版本之前配置Servlet访问网址的方式：
  * 将Servlet类，配置到web.xml中，告知tomcat，servlet的类名 
  * 配置Servlet类的别名，并给指定别名的Servlet添加映射网址。

``` xml
  <!-- 将servlet类，配置到web.xml中，告知tomcat，servlet的类名 -->
  <servlet>
  	<!-- Servlet类别名，用于后续添加映射网址 -->
  	<servlet-name>demo1</servlet-name>
  	<!-- Servlet类全名 -->
  	<servlet-class>day01_Servlet.demo1.MyServlet</servlet-class>
  </servlet>
  <servlet-mapping>
  	<!-- 给指定别名的Servlet添加映射网址 -->
  	<servlet-name>demo1</servlet-name>
  	<url-pattern>/hello</url-pattern>
  </servlet-mapping>
```


#### 2.3 Servlet生命周期
- 实例化 --> 初始化(init) --> 服务(service) --> 销毁(销毁之前调用destory) --> 不可用
- 创建时机：默认情况下，当用户第一次访问Servlet的映射网址是Servlet对象被创建，后续用户再次访问，是重复利用此对象。
- 销毁时机：当tomcat关闭时 或 应用从tomcat卸载时。
- tomcat为了便于我们进行资源的合理缓存，为生命周期事件提供了三个方法：
  * init(); 当Servlet对象被创建时，方法执行，通常在这里进行一些可重用资源的初始化工作。
  * service(); 服务方法，当用户每次发起请求时，此方法用于处理请求，并进行响应，此方法每次都执行在新的线程中。
  * destory(); 当Servlet即将被销毁时，方法执行，释放资源的代码可写在此方法中。


#### 2.4 get和post区别
- GET请求：
  * 没有请求体，请求时携带参数在url中，参数在url地址的?后，参数由=连接的键值对组成，&连接键值对。
  * 只能传输字符串类型参数
  * 浏览器url地址最大长度4kb
  * 数据传输时，参数在url中明文显示，不安全。
- POST请求：
  * 有请求体，是一个单独的数据包，用于存储请求中的多个参数
  * 可传输任意类型的数据，进行文件上传必须POST请求
  * 可以传递的数据大小，理论上没有上限
  * 数据传输时在单独的数据包，较为安全。


#### 2.5 接收请求中的参数
1. 根据参数的名称，接收参数的单个值
  - String value = request.getParameter(String name);
2. 根据参数的名称，接收一组参数的值
  - String[] values = request.getParameterValues(String name);

``` java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  request.setCharacterEncoding("UTF-8");
  response.setContentType("text/html;charset=utf-8");
  //1.接收
  String username = request.getParameter("username");
  String[] password = request.getParameterValues("password");
  //2.打印
  System.out.println("username:" + username);
  System.out.println("password:" + password[0]);
  System.out.println("password2:" + password[1]);
  //3.浏览器输出
  response.getWriter().append("<div>很遗憾注册失败，点击<a href=\"demo1.html\">重新注册</a></div>");

}
```


#### 2.6 乱码处理
#### 2.6.1 乱码情况：
* 浏览器提交表单时，会对中文参数值进行自动编码。Tomcat服务器接收到的浏览器请求后，默认使用iso-8859-1去解码，当编码与解码方式不一致时，就会乱码。
* tomcat8版本之前(不包含tomcat8版本), GET请求乱码
* 任何版本, POST请求乱码

#### 2.6.2 请求乱码处理：
* 适用于所有乱码问题：(Tomcat8之后get无乱码)
  1. 指定浏览器打开页面的编码`<meta charset="UTF-8">`;
  2. 将接收到的中文乱码重新编码：

``` java
String name = request.getParameter("userName");
String userName = new String( name.getByte("ISO-8859-1"),"utf-8");
```

* 仅适用于POST请求：
  1. 指定浏览器打开页面的编码`<meta charset="UTF-8">`;
  2. Servlet接收之前设置解码（需在调用request.getParameter("key")之前设置）`request.setCharacterEncoding("utf-8")`;

#### 2.6.3 响应乱码的处理：
* 方式一：设置响应的内容类型, 以及编码格式:`response.setContentType("text/html;charset=utf-8")`;
* 方式二：进设置编码格式, 不设置响应内容类型:`response.setCharacterEncoding("UTF-8")`(常用于客户端不是浏览器的情况, 如果在浏览器的环境下设置, 有部分浏览器无法识别, 依然会乱码);


#### 2.7 Servlet的创建时机
- 通过web.xml配置Servlet, 可以修改Servlet加载的时机。
- 可以给Servlet节点，添加`<load-on-startup>`节点来制定servlet启动顺序。
- 节点中的值为数字：
  * `-1`：默认-1，表示当用户第一次请求时，创建对象
  * `>=0`：大于等于0，当服务器启动时，创建对象，值越小创建越早，值相同按web.xml配置顺序创建

``` xml
<servlet>
    <servlet>
        <servlet-name>s1</servlet-name>
        <servlet-class>demo.ServletDemo</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>s1</servlet-name>
        <url-pattern>/s1</url-pattern>
    </servlet-mapping>
</servlet>
<servlet-mapping></servlet-mapping>
```


<span id="id3"><span>
### 3. 请求的转发与重定向
#### 3.1 请求对象request的常用操作
1. getMethod() : 得到请求的方式
2. getRequestURI() : 获取浏览器请求地址
3. getRemoteAddr() : 获取客户端ip地址
4. getRemoteHost() : 获取客户端名称
5. getServerName() : 获取服务器名称
6. getServerPort() : 获取服务器端口号
7. getQueryString() : 获取get请求参数字符串，其他请求返回null


#### 3.1 请求的转发与重定向注意事项
* 请求转发与重定向操作，必须要有出口。
* 当一个请求在servlet中进行了重定向，那么这个servlet就不要再进行响应了


#### 3.2 转发*
- 一个web组件，将未处理完毕的请求，通过tomcat转交给另一个web组件处理
- 步骤：
  1. 获取请求转发器：`RequestDispather rd = request.getRequestDispacher("转发地址");`
  2. 进行转发操作：`rd.forward(request, response);`
- 因为通常请求转发器获取后, 只会使用一次 , 一般不给对象起名, 简写:
  * `request.getRequestDispacher("转发地址").forward(request, response);`
- 特点：
  * 转发过程中，多个web组件之间共享一个请求对象request与响应对象response
  * 在转发过程中，无论转发多少次，浏览器只发起了一次请求，所以浏览器地址不会改变
  * 转发不能跨项目实现
  * 比重定向效率更高


#### 3.3 重定向*
- 一个web组件，处理完毕请求后，告知浏览器，将请求转向另一个地址
- 格式：`response.sendRedirect("重定向地址")`；
- 原理：当客户端请求服务器时，发起重定向流程：
  1. 给浏览器响应302的状态码 , 以及一个键值对, 键为: location , 值为重定向新地址.
  2. 当浏览器接收到302的状态码时, HTTP协议规定了浏览器会寻找location对象的新地址.
  3. 浏览器自动发起新的请求 , 跳转到新地址.
- 特点：
  1. 重定向会产生两个请求对象，多个请求对象中数据不互通
  2. 浏览器地址发生了改变
  3. 重定向可以跨域实现
  4. 比转发效率低


<span id="id4"><span>
### 4. 上下文对象ServletContext
- 用于关联多个servlet，是servlet之间通讯的桥梁，用于多个servlet之间的信息共享
- 每一个项目运行时，tomcat会为这个项目创建一个servletContext，项目关闭时销毁。

获取ServletContext对象：`ServletContext context = getServletContext();`

- 常用方法
  * context.setAttributes(String key, Objexct value); //设置替换数据
  * context.getAttributes(String key); //获取数据
  * context.removeAttributes(String key); //删除数据
  * context.getRealPath("/"); //获取项目运行时所在文件路径


<span id="id5"><span>
### 5. 会话跟踪（状态管理）
- 存在两种实现：
  1. cookie: 将浏览器产生的状态存储在浏览器中
  2. Session: 将浏览器产生的状态存储在服务器中

- cookie技术原理：
  * 服务器向客户端响应时，将数据以set-Cookie消息头（响应头）的方式发给浏览器，
  * 浏览器接收到cookie后，会将这些数据以文本文件的方式（.txt文件）保存起来
  * 当浏览器再次发起相同请求时，浏览器会将之前存储的cookie,添加到请求头，发给服务器
- Session技术原理：
  * 当浏览器访问服务器时，服务器可以选择为用户创建一个Session对象(类似于map集合)，
  * 该Session对象有一个id属性，称之为SessionId，服务器会将这个SessionId以cookie方式发送给浏览器
  * 浏览器再次访问服务器时，同时会传递SessionId的cookie给i服务器，服务器根据sessionId找到Session对象，供程序使用。

#### 5.1 Cookie
- 创建Cookie：Cookie在Java中是一个类，每个cookie的对象都表示一个键值对
  * `Cookie cookie = new Cookie(String key, String value);`
  * 注意：tomcat8.5版本之前，cookie无法出场中文
- 通过响应对象，将cookie添加到响应头,可添加多个
  * response.addCookie(Cookie cookie);
- 通过请求头得到cookie数组，没有则返回null
  * Cookie[] cookies = request.getCookies();
  * 取键：cookie.getName();
  * 取值：cookie.getValue()
- Cookie的存储时长：
  * cookie.setMaxAge(int 秒)；
    + 正数：倒计时秒数
    + 0：表示立即删除此cookie，常用于覆盖一个存活时长较长的cookie,用于删除它
    + 负数：默认-1，表示会话结束时自动删除（关闭浏览器）
- Cookie的存储路径问题
  * 存储的cookie发送到服务器时，判断是否发送的依据是：域名相同，路径相同
  * 为了避免路径问题，通常会将cookie设置统一路径为根路径：cookie.setPath("/");

#### 5.2 Cookie的优缺点
- 缺点：
  1. Cookie技术存储的数据类型，只能是字符串，且早期版本(8.5之前)不可存储中文。
  2. 数据存储在客户的计算机中，不安全，不建议存储安全敏感数据
  3. 保存数据量有限制，大约4kb左右
  3. 依赖于用户的浏览器设置，用户可以金庸cookie，可能被用户主动删除
- 优点：
  1. 分散服务器的压力

#### 5.3 Session
- 获取Session
  * 格式1：request.getSession();//等价参数传true
  * 格式2：request.getSession(boolean isNew);
    + true，根据浏览器的SessionId查找一个session，若没有就新创建一个对象并返回
    + false，根据浏览器的SessionId查找一个session，若没有就返回null
- Session常用方法
  * session.setAttribute(String key, object value);//设置/替换值
  * session.getAttribute(String key);//获取值
  * session.invalidate();//销毁
- 设置session存活时长
  * 默认会话时长30分钟，当浏览器最后一次访问服务器后30分钟后，若没有再次连接，则session被销毁。
  * 可以通过修改配置文件，修改所有的session时长
    + 修改`conf/web.xml`的`<session-config><session-tiomeout>数值分钟</session-tiomeout></session-config>`
  * 可以通过session对象，修改单个对象的session时长
    + void session.setMaxInactiveInterval(int seconds)

#### 5.4 Session的优缺点
- 缺点：
  * 数据存储在服务器端，当用户量大时，对服务器造成极大的压力，很容易耗尽服务器资源
- 优点：
  1. 数据存储在服务器中，安全
  2. 数据类型为Object，在Java中表示可以存储所有类型的数据
  3. session存储的数据大小，理论上无限的。

#### 5.5 Cookie和Session的使用
- Cookie和Session不是互斥的，是相辅相成的
- 在项目开发时：
  * 对安全敏感的数据，存储在session中
  * 对安全不敏感的字符串数据，可以选择存储在Cookie中
  * 对于大的数据，应该存在数据库和文件中

> 注意：cookie和session是为了管理状态而非存储数据。


<span id="id6"><span>
### 6.JSP
#### 6.1 JSP语法基础
- Java Server Pages：java动态网页技术
- JSP引擎原理：JSP引擎读取JSP文件，将文件转换为Servlet，由servlet给用户响应
- 注意：
  1. JSP文件的转换 发生在服务器启动时，当用户访问JSP时，其实访问的是JSP文件转换的Servlet
  2. 执行流程：浏览器请求-->tomcat-->JSP引擎转换为Servlet-->转换的Servlet-->准备响应体-->响应给浏览器-->浏览器解析html

- JSP语法结构
  1. html代码
  2. Java代码
  3. Jsp特有的语法结构

- Java代码声明区：指的是类的成员位置

``` jsp
<%!
  // Java代码声明区
%>
```

- Java代码执行区：指的是Servlet的service方法中，每次用户请求，执行区的代码都会执行起来

``` jsp
<%
  // Java代码执行区
%>
```

- JSP输出表达式
  * 用于快速的将Java中的数据，输出到网页中..
  * 语法格式：`<%=数据 %>`，编译后被转换成out.print(数据)
- JSP注释：
  * html中可以用`<!-- -->`
  * java中可以用`//，/**/，/** */`
  * jsp注释`<%-- --%>`
    + html和java注释会被编译，其中html注释会被编译到页面，jsp注释编译器会自动忽略

#### 6.2 JSP三大指令
* page指令
* include指令
* taglib指令

- 指令使用格式：<%@ 指令名称 属性1=值 属性2=值 属性n=值 %>
  *语法上，JSP允许在单个页面出现多个相同的JSP指令
  
##### 6.2.1 page指令
- 用于配置页面信息

``` jsp
<%@ page
  language="java"：语言
  contentType="text/html;charset=utf-8"：响应的内容类型，以及响应的编码格式
  pageEncoding="UTF-8"：文件存储的编码格式
  extends="继承的父类"
  buffer="数字/none"：是否允许缓存，默认值8kb
  autoFlush="true/false"：是否自动清除缓存，默认true
  session="true/false"：是否提前准备session对象，默认true
  isThreadSafe="true/false"：是否线程安全的
  import="java.util.List"：用于导包，多个包使用","隔开
  errorPage="网址"：当页面发生BUG后，显示哪个页面
  isErrorPage="true/false"：当前页面是否是一个错误处理页面，如果结果为true，当别的页面产生错误，跳转到此页面，会提前准备好一个对象exception，此对象封装了错误信息
%>
```

#### 6.3 项目发生错误时，统一的处理方式
  1. 打开项目的web.xml
  2. 加入子节点`<error-page><error-code>错误码</error-code><location>处理网址</location></error-page>`

``` xml
<error-page>
    <error-code>500</error-code>
    <location>/error.jsp</location>
</error-page>
<error-page>
    <error-code>404</error-code>
    <location>/404.jsp</location>
</error-page>
```

- include指令：用于将jsp或html引入到另一个jsp中
  * 语法格式：`<%@ include file="地址" %>`
- include动作：用于将jsp或html引入到另一个jsp中
  * 语法格式：`<jsp:include page="地址">`
  
>include指令 与 include动作区别：
<br>include指令：引入文件操作，是在JSP引擎的转换时发生，将多个jsp文件，生产为了一个Servlert（多个jsp => 一个Servlet）
<br>include动作：引入文件操作，是在浏览器请求时，将引用文件的响应体添加到了请求文件的响应体中（多个jsp => 多个Servlet）