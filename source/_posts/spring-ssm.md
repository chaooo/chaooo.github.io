---
title: 「Spring」SSM框架整合(Spring+SpringMVC+MyBatis)
date: 2018-05-11 22:52:55
tags: [后端开发, Spring]
categories: Spring
---

SSM框架是spring MVC ，spring和mybatis框架的整合，是标准的MVC模式，将整个系统划分为表现层，controller层，service层，DAO层。<!-- more -->
- 使用spring MVC负责请求的转发和视图管理
- spring实现业务对象管理
- mybatis作为数据对象的持久化引擎


### 1.搭建SSM架构步骤：
1. 设计数据库
2. 先写实体类entity，定义对象的属性，（参照数据库中表的字段来设置）。
3. 编写Mapper.xml（Mybatis），定义功能，对应要对数据库进行的那些操作，比如 insert、selectAll、selectByKey、delete、update等。
4. 编写Mapper.java(DAO接口)，将Mapper.xml中的操作按照id映射成Java函数。
5. 配置spring和mybatis框架的整合(applicationContext.xml)
6. 编写Service.java，为控制层提供服务，接受控制层的参数，完成相应的功能，并返回给控制层。
7. 配置SpringMVC(web.xml)
8. 编写Controller.java，连接页面请求和服务层，获取页面请求的参数，通过自动装配，映射不同的URL到相应的处理函数，并获取参数，对参数进行处理，之后传给服务层。
9. 编写JSP页面调用，请求哪些参数，需要获取什么数据。

>DataBase --> Entity --> Mapper.xml --> Mapper.Java(DAO) --> Service.java --> Controller.java --> Jsp


### 2.搭建SSM架构实例（管理员登录）

#### 2.1 设计数据库(以MySql为例)
建立web项目，在src下新建sql脚本(admin.sql)，并在数据库中执行
``` sql
CREATE DATABASE exam_sys;
/** 管理员表 */
DROP TABLE admin;
CREATE TABLE admin(
    id INT AUTO_INCREMENT COMMENT '管理员ID',
    name VARCHAR(30) NOT NULL COMMENT '管理员账号',
    password VARCHAR(30) COMMENT '管理员密码',
    CONSTRAINT et_admin_id_pk PRIMARY KEY(id),
    CONSTRAINT et_admin_name_uk UNIQUE(NAME)
);
/** 插入数据 */
INSERT INTO admin (name, password) VALUES('admin', '123456');
SELECT * FROM admin;
COMMIT;
```


#### 2.2 先写实体类entity，定义对象的属性
参照数据库中表的字段来设置
``` javaScript
package com.exam.entity;

public class Admin {
    private int id;
    private String name;
    private String password;
    /** 添加 getter/setter方法
     *  添加 无参，有参构造
     *  重写toString()以便于测试
     */
    // ...
}
```


#### 2.3 编写AdminMapper.xml（Mybatis），定义功能
对应要对数据库进行的那些操作，比如 insert、selectAll、selectByKey、delete、update等。
``` xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN" "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<!-- namespace指定和哪个Mapper映射器接口对应 -->
<mapper namespace="com.exam.mapper.AdminDao">
    <!-- 定义SQL语句 -->	
    <select id="findByNameAndPassword" resultType="com.exam.entity.Admin">
        select * from admin where name=#{name, jdbcType=VARCHAR} and password=#{password, jdbcType=VARCHAR}
    </select>
</mapper>
```


#### 2.4 编写AdminDao.java，将AdminMapper.xml中的操作按照id映射成Java函数。
导入Mybatis相关jar包：mybatis.jar、mysql-connector-java.jar(数据库驱动)、mybatis-spring.jar(SM整合)
``` javaScript
package com.exam.mapper;

import org.apache.ibatis.annotations.Param;
import com.exam.entity.Admin;

public interface AdminDao {
    public Admin findByNameAndPassword(@Param("name") String name, @Param("password") String password);
}
```


#### 2.5 配置spring和mybatis框架的整合
导入Spring相关jar包：ioc/aop/dao/连接池；添加Spring配置文件（applicationContext.xml）到src下。
``` xml
<!-- 配置连接池对象 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/exam_sys"></property>
    <property name="username" value="root"></property>
    <property name="password" value="123456"></property>
</bean>
<!-- 配置SqlSessionFactoryBean来创建SqlSessionFactory
        属性dataSource：注入连接池对象
        属性mapperLocations：指定MyBatis的映射器XML配置文件的位置
        属性typeAliasesPackage：对应我们的实体类所在的包，配置此项可在Mapper映射器直接使用类名，而非包名.类名
 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="mapperLocations" value="classpath:com/exam/mapper/*.xml"></property>
    <!-- <property name="typeAliasesPackage" value="com.exam.entity"></property> -->
</bean>

<!-- 批量生产DAO接口实现类  ,实现类id为类名首字母小写 -->
<bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- <property name="sqlSessionFactory" ref="sqlSessionFactory"></property> -->
    <property name="basePackage" value="com.exam.mapper"></property>
    <!-- 自定义注解可以让只让有注解的接口产生实现类，另一部分一部分不产生 -->
    <!-- <property name="annotationClass" value="com.annotation.MyAnnotation"></property> -->
</bean>
<!-- 开启服务层组件扫描 -->
<context:component-scan base-package="com.exam.service"/>
```


#### 2.6 编写Service.java，为控制层提供服务
接受控制层的参数，完成相应的功能，并返回给控制层。
``` javaScript
package com.exam.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.exam.mapper.AdminDao;

@Service("adminService")
public class AdminService {
    @Autowired
    private AdminDao dao;
    
    public boolean Login(String name, String password) {
        try {
            return dao.findByNameAndPassword(name, password)!=null?true:false;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }
}
```

#### 2.7 配置SpringMVC
导入jar包（spring-web.jar，spring-webmvc.jar）,生成web.xml并配置DispatcherServlet分发请求。
``` xml
<!-- 配置编码过滤器 -->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/</url-pattern>
</filter-mapping>
<!-- 配置DispatcherServlet分发请求 -->
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
    <url-pattern>/</url-pattern>
</servlet-mapping>
<!-- 在applicationContext.xml对静态资源进行放行 ：mvc:default-servlet-handler-->
```

在applicationContext.xml中开启组件扫描(com.controller)，开启标注形式mvc，配置视图处理器 并 对静态资源进行放行。
``` xml
<!-- 开启控制器组件扫描 -->
<context:component-scan base-package="com.exam.controller"/>
<!-- 开启标注形式mvc -->
<mvc:annotation-driven />
<!-- 配置视图处理器 -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
<!-- 对静态资源进行放行 -->
<mvc:default-servlet-handler/>
```


#### 2.8 编写Controller.java，连接页面请求和服务层
获取页面请求的参数，通过自动装配，映射不同的URL到相应的处理函数，并获取参数，对参数进行处理，之后传给服务层。（导入Json相关包：jackson-core.jar，jackson-databind.jar，jackson-annotations.jar）
``` javaScript
package com.exam.controller;

import javax.servlet.http.HttpServletRequest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import com.exam.entity.Admin;
import com.exam.service.AdminService;

@Controller
@RequestMapping("/admin")
public class AdminController {
    @Autowired
    private AdminService as;
    
    @RequestMapping("/tologin")
    public String toLogin() {
        return "admin/login";
    }
    @RequestMapping(value="/login",method=RequestMethod.POST)
    @ResponseBody
    public boolean addUser(Admin admin, HttpServletRequest request) {
        System.out.println("add:"+admin);
        System.out.println(admin.getName()+"---"+admin.getPassword());
        boolean bl = as.Login(admin.getName(), admin.getPassword());
        if(bl) {
            //登录成功的逻辑
            request.getSession().setAttribute("admin", admin);
            return true;
        }
        //登录失败的逻辑
        request.setAttribute("msg", "登录失败");
        return false;
    }
}
```


#### 2.9 编写JSP页面调用
``` html
<form>
    管理员: <input id="aName"  type="text"><br>
    密码:<input id="aPassword"  type="text"><br>
    <input id="loginBtn"  type="button"  value="登录">
</form>
<script src="js/jquery.min.js"></script>
<script>
$("#loginBtn").on("click", function(){
    $.ajax({
        url: "admin/login",
        type: "post",
        data: {
            name: $("#aName").val(),
            password: $("#aPassword").val()
        },
        success: function(res){
            alert(res);
        }
    });
});
</script>
```
