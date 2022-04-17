---
title: 「Spring」持久层框架Mybatis
date: 2018-05-06 22:51:34
tags: [后端开发, Spring]
categories: Spring
---

Mybatis支持普通sql操作，存储过程的调用，它是一个高级的ORM框架(Object Relation Mapping对象关系映射--以面向对象思想访问数据库)，是一个基于Java的持久层框架。
<!-- more -->
MyBatis封装了几乎所有的JDBC操作和参数的手工设置，它会对结果集自动封装成对象，以及直接把对象存入数据库，甚至可以做到对象与对象的关系维护；诸如：建立连接、操作 Statment、ResultSet，处理 JDBC 相关异常等等都可以交给 MyBatis 去处理，我们的关注点于是可以就此集中在 SQL 语句上，关注在增删改查这些操作层面上。


### 1. Mybatis框架的构成
- 实体类 ： 封装记录信息（JavaBean）
- SQL定义文件 ：定义sql语句（编写SQL语句的XML）
- 主配置文件 ：定义连接信息、加载SQL文件 以及其他设置的XML
- 框架API ：用于实现数据库增删改查操作（主要通过SqlSession）


### 2. 使用Mybatis访问数据库
以员工表`Emp(id,name,salary)`为例
1. 准备数据库及创建项目（需要mybatis的jar包和数据库驱动包）
2. 根据表建立对应的实体类：`Emp(id,name,salary)`
3. 在「src」目录下创建 MyBaits 的主配置文件 mybatis-config.xml ，其主要作用是提供连接数据库用的驱动，数据名称，编码方式，账号密码等
``` xml
<configuration>
    <environments default="environment">
        <environment id="environment">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" 
                    value="com.mysql.cj.jdbc.Driver" />
                <property name="url"
                    value="jdbc:mysql://localhost:3306/test"/>
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/mapper/EmpMapper.xml" />
    </mappers>
</configuration> 
```

4. 在「src」包路径下创建配置文件（com/mapper/EmpMapper.xml）,然后根据需求定义sql
``` xml
<mapper namespace="com.mapper.EmpMapper">
    <!-- 定义SQL语句 -->	
    <select id="findById" parameterType="int" 
      resultType="com.mapper.Emp">
         select * from emp32 where id = #{id}
    </select>
    <select id="findByName" parameterType="String" 
      resultType="com.mapper.Emp">
         select * from emp32 where name = #{name}
    </select>
</mapper>
```
> - parameterType：要求输入参数的类型
> - resultType：输出的类型

5. 封装工具类获取SQLSession
``` java
public class SqlSessionUtil {
    public static SqlSessionFactory ssf;
    static {
        // 先构建SQLSession工厂构建器
        SqlSessionFactoryBuilder ssfb = new SqlSessionFactoryBuilder();
        // 构建SqlSessionFactory关联主配置文件
        InputStream inputStream = SqlSessionUtil.class.getClassLoader().getResourceAsStream("mybatis-config.xml");
        ssf = ssfb.build(inputStream);
    }
     // 获取SQLSession
    public static SqlSession getSqlSession() {
        // 通过SqlSession 工厂对象 来获取SqlSession
        return ssf.openSession();
    }
}
```

6. 编写测试类
``` java
public class EmpTest {
    public static void main(String[] args) {
        SqlSession ss =SqlSessionUtil.getSqlSession();
        Emp emp = ss.selectOne("findById", 6);
        System.out.println(emp);
    }
}
```

> 基本原理
- 应用程序找 MyBatis 要数据
- MyBatis 从数据库中找来数据
- 通过 mybatis-config.xml 定位哪个数据库
- 通过 EmpMapper.xml 执行对应的 sql 语句
- 基于 EmpMapper.xml 把返回的数据库封装在 Emp 对象中
- 返回一个 Emp 对象


### 3. Mybatis的CRUD操作
以员工表`Emp(id,name,salary)`为例
1. 第一步：配置EmpMapper.xml

``` xml
<insert id="insertEmp" parameterType="com.mapper.Emp">
    insert into emp32(name, salary) values(#{name}, #{salary})
</insert>
<delete id="deleteEmpById" parameterType="int">
    delete from emp32 where id=#{id}
</delete>
<update id="updateEmpById" parameterType="com.mapper.Emp">
    update emp32 set name=#{name} where id=#{id}
</update>
<select id="findById" parameterType="int" resultType="com.mapper.Emp">
     select * from emp32 where id = #{id}
</select>
<select id="findAll" resultType="com.mapper.Emp">
    select * from emp32
</select>
```
> - parameterType：要求输入参数的类型
> - resultType：输出的类型


2. 第二步：SQLSession实现增删改查

``` java
// 先构建SQLSession工厂构建器
SqlSessionFactoryBuilder ssfb = new SqlSessionFactoryBuilder();
// 构建SqlSessionFactory关联主配置文件
InputStream inputStream = EmpTest.class.getClassLoader().getResourceAsStream("sqlmap-config.xml");
SqlSessionFactory ssf = ssfb.build(inputStream);
// 通过SqlSession 工厂对象 来获取SqlSession
SqlSession ss = ssf.openSession();

//增加
Emp emp = new Emp(0,"ef2",50000);
int addRows = ss.insert("insertEmp", emp);
//删除
int delRows = ss.delete("deleteEmpById", 12);
//更新
Emp emp2 = new Emp(1,"hello",0);
int updateRows = ss.update("updateEmpById", emp2);
//查找
Emp emp3 = ss.selectOne("findById", 6);
List<Emp> empList = ss.selectList("findAll");

ss.commit();
```
> SqlSession对象的操作方法如下：
- insert(..) 插入操作
- update(..) 更新操作
- delete(..) 删除操作
- selectOne(..) 单行查询操作
- selectList(..) 多行查询操作
- 通过 session.commit() 来提交事务，也可以简单理解为更新到数据库


### 4. Mapper映射器
使用规则：
1. 接口的方法名和SQL定义文件中的id保持一致
2. 接口方法的返回值类型 要和resultType 保持一致
    - 单行：`resultType`
    - 多行：`List<resultType>`
    - 增删改返回值，推荐int，也可以是void
3. 接口方法参数和parameterType保持 一致，如果没有parameterType则参数任意
4. SQL定义文件中的namespace必须包名.接口名


### 5. 向mapper传多个参数
#### 5.1 第一种方案：#{0}，#{1} / #{param1} 和 #{param2}
DAO层的函数方法 
``` java
public Emp findByIdAndName(int id, String name);
```

对应的Mapper.xml
``` xml  
<select id="findByIdAndName" resultType="com.bean.Emp">
    select * from emp32 where id = #{0} and name = #{1}
</select>
```
其中，#{0}代表接收的是dao层中的第一个参数，#{1}代表dao层中第二参数，更多参数一致往后加即可。
也可以用#{param1} 和 #{param2}实现同意效果。


#### 5.2 第二种方案@param
Dao层的函数方法
``` java
public Emp findByIdAndName(@param("id")int id, @param("name")String name);
```

对应的Mapper.xml
``` xml
<select id="findByIdAndName" resultType="com.bean.Emp">
    select * from emp32 where id = #{id} and name = #{name}
</select>
```


#### 5.3 第三种方案：采用对象或Map传多参数
Dao层的函数方法
``` java
public Emp findByIdAndName(Emp emp);
public Emp findByIdAndName2(Map<String, Object> params);
```

对应的Mapper.xml
``` xml
<select id="findByIdAndName" parameterType="com.bean.Emp" resultType="com.bean.Emp">
    select * from emp32 where id = #{id} and name = #{name}
</select>

<select id="findByIdAndName2" parameterType="map" resultType="com.bean.Emp">
    select * from emp32 where id = #{id} and name = #{name}
</select>
```


### 6. 结果集列名和属性名不一致的解决方法
在SQL定义中，resultType属性用于指定查询数据采用哪种类型封装，规则为结果集列名和属性名一致，如果不一致将不能接收查询结果。
解决方法：
1. 使用别名，select语句使用与属性一致的别名
2. 使用resultMap替换resultType，用resultMap指定结果集列名和属性名的对应关系

``` xml
<!-- 定义resultMap将sql 结果集列名(数据库中的字段)和Emp类中的属性做一个映射关系
    type:resultMap最终所映射的Java对象类型，可以使用别名
    id:对resultMap的唯一标识 
-->
<resultMap type="com.bean.Emp" id="empMap">
    <!-- id表示查询结果集中唯一标识 
        column:查询出的列名
        property:type所指定的类中的属性名 
    -->
    <id column="e_id" property="id"/>
    <!-- 对普通列的映射定义 -->
    <result  column="salary"  property="sal"/>
</resultMap>

<!-- 使用resultMap -->
<select id="findEmpById" parameterType="int" resultMap="empMap">
         select * from  emp32 where id = #{id}
</select>
```


### 7. 类型的别名和日志输出
在mybatis-config.xml中自定义类型的别名
``` xml
<typeAliases>
    <typeAlias alias="emp" type="com.bean.Emp"/>
</typeAliases>
```

在EmpMapper.xml中使用别名 resultType="emp"
``` xml
<select id="findById" parameterType="int" resultType="emp">
    select id,name,salary sal from emp32 where id = #{id}
</select>
```


设置MyBatis的日志输出到控制台
``` xml
<settings>
    <!--设置是否允许缓存-->
    <setting name="cacheEnabled" value="true"/>
    <!--设置日志输出的目标-->
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```


### 8. JdbcType
在执行SQL时MyBatis会自动通过对象中的属性给SQL中参数赋值，它会自动将Java类型转换成数据库的类型。而一旦传入的是null它就无法准确判断这个类型应该是什么，就有可能将类型转换错误，从而报错。
- 所以 MyBatis 插入空值时，需要指定JdbcType，这样相对来说是比较安全的。
- 一般情况下，我们没有必要按个字段去识别/判断它是否可以为空，而是将所有的字段都当做可以为空，全部手动设置转换类型。
- MyBatis包含的JdbcType类型，主要有下面这些：
    + BIT、FLOAT、CHAR 、TIMESTAMP 、 OTHER 、UNDEFINEDTINYINT 、REAL 、VARCHAR 、BINARY 、BLOB NVARCHAR、SMALLINT 、DOUBLE 、LONGVARCHAR 、VARBINARY 、CLOB、NCHAR、INTEGER、 NUMERIC、DATE 、LONGVARBINARY 、BOOLEAN 、NCLOB、BIGINT 、DECIMAL 、TIME 、NULL、CURSOR

``` xml
<select id="findByName" parameterType="String" resultType="com.bean.Emp">
    select * from emp32 where name = #{name, jdbcType=VARCHAR}
</select>
```


### 9. Mabatis中#{}和${}的区别
1. `${}`是字符串替换，底层使用的Statement（sql注入问题，效率低，编写sql复杂）
    - 支持${param1}或${变量名},不支持${0}，Dao层必须使用@Param(),用到字符串时需要手动加单引号
2. `#{}`是预编译处理命令，底层使用PreparedStatement（可以有效防止sql注入）
    - 不支持表名、排序方式等的占位，默认会将其当成字符串



### 10. 分页
1. 在主配置文件中配置 分页拦截器（依赖于pageHelper、sqlparse相关jar）

``` xml
<!-- 配置分页拦截器 -->
<plugins>
    <plugin interceptor="com.github.pagehelper.PageHelper"></plugin>
</plugins>
```

2. 查询前使用分页API

``` xml
PageHelper.startPage(2, 2);
List<Emp> emps = dao.orderBySalary();
for(Emp emp: emps) {
    System.out.println(emp);
}
```



### 11. Spring+MyBatis整合
Spring与MyBatis整合需要引入一个mybatis-spring.jar文件包，该包提供了下面几个与整合相关的API:
1. SqlSessionFactoryBean
    + 创建SqlSessionFactory对象，为整合应用提供SqlSession对象资源
    + 依赖于dataSource 和加载SQL定义文件
2. MapperFactoryBean
    + 根据指定的某一个Mapper接口生成Bean实例
    + 依赖于SqlSessionFactory 和 MApper接口
3. MapperScannerConfigurer
    + 根据指定包批量扫描Mapper接口并生成实例
4. SqlSessionTemplate
    + 类似于JdbcTemplate，便于程序员自己编写Mapper实现类


### 12. Spring+MyBatis完成sql操作
**第一步**：使用Mybatis（同上）
+ 导jar包(mybatis包/数据库驱动包)，建立实体类，定义SQL文件，编写Mapper映射接口

**第二步**：配置SqlSessionFactoryBean
+ 导入jar包（mabatis-spring/ioc/aop/dao/连接池）
+ 配置SqlSessionFactoryBean注入dataSource和指定sql定义文件

``` xml
<!-- 配置SqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="mapperLocations" value="classpath:com/mapper/*.xml"></property>
</bean>
<!-- 配置连接池对象 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/test"></property>
    <property name="username" value="root"></property>
    <property name="password" value="123456"></property>
</bean>
```

**第三步**：
1. 方式一： 使用SqlSessionFactoryBean结合接口和SqlSessionFactory
    + 最终产生Mapper接口的 实现类，注意这是实现类

``` xml
<!--  配置SqlSessionFactoryBean 产生Mapper接口的 实现类  -->
<bean id="empDao" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
    <property name="mapperInterface" value="com.dao.EmpDao"></property>
</bean>
<bean id="empDao2" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
    <property name="mapperInterface" value="com.dao.EmpDao2"></property>
</bean> 
```

2. 方式二： MapperScannerConfigurer
    + MapperFactoryBean一次只能生产一个DAO的实现类，可以通过MapperScannerConfigurer批量生产DAO接口实现类

``` xml
<!-- 批量生产DAO接口实现类  ,实现类id为类名首字母小写 -->
<bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- <property name="sqlSessionFactory" ref="sqlSessionFactory"></property> -->
    <property name="basePackage" value="com.dao"></property>
    <!-- 自定义注解可以让只让有注解的接口产生实现类，另一部分一部分不产生 -->
    <property name="annotationClass" value="com.annotation.MyAnnotation"></property>
</bean>
```


### 13. 使用SqlSessionTemplate模板来完成DAO接口的实现类
1. 使用Mybatis（同上）
2. 配置SqlSessionFactoryBean（同上）
3. 编写DAO接口的实现类
    + 开启组件扫描，注入SqlSessionTemplate,依赖于SqlSessionFactory
    + 使用SqlSessionTemplate对应API完成增删改查

``` xml
<!-- 开启组件扫描 -->
<context:component-scan base-package="com.mapper"></context:component-scan>
<!-- 创建SqlSessionTemplate -->
<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
</bean>
```

``` java
@Repository("empDao")
public class EmpDaoImpl implements EmpDao {
    @Autowired
    private SqlSessionTemplate sqlSessionTemplate;
    @Override
    public Emp findById(int id) {
        return sqlSessionTemplate.selectOne("findById", id);
    }
}
```
