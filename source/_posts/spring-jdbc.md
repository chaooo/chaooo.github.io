---
title: 「Spring」JDBC详解
date: 2018-04-09 22:46:07
tags: [后端开发, Spring]
categories: Spring
---

Spring对JDBC做了简化和封装；简化了DAO实现类编写；提供了基于AOP的**声明式**事务管理；对JDBC中异常做了封装，把原来检查异常封装成了继承自RuntimeException的异常（DataAcessException）。
<!-- more -->


### 1. 数据源配置
``` java
@Configuration
@ComponentScan("com.jdbc")
public class MyConfiguration {
    @Bean
    public DataSource mysqlDataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        return dataSource;
    }
    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(mysqlDataSource());
    }
}
```

也可以使用XML配置来实现配置效果：
``` xml
<!-- 配置连接池对象 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/test"></property>
    <property name="username" value="root"></property>
    <property name="password" value="123456"></property>
</bean>
<!-- 定义jdbcTemplate对象 -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg index="0" ref="dataSource"></constructor-arg>
</bean>
<!-- 开启组件扫描 -->
<context:component-scan base-package="com.jdbc"></context:component-scan>
```


### 2. JdbcTemplate的使用
JdbcTemplate模板是Spring JDBC模块中主要的API，它提供了常见的数据库访问功能。
JdbcTemplate类执行SQL查询、更新语句和存储过程调用，执行迭代结果集和提取返回参数值。

基本的查询：
``` java
//DAO实现类
@Repository("empDao")
public class EmpDaoImpl implements EmpDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Override
    public int getCount() {
        String sql = "select count(*) from emp32";
        return jdbcTemplate.queryForObject(sql, Integer.class);
    }
//...
```


### 3. 通过实现RowMapper接口把查询结果映射到Java对象
``` java
public class EmpRowMapper implements RowMapper<Emp> {
    @Override
    public Emp mapRow(ResultSet rs, int n) throws SQLException {
        return new Emp(
            rs.getInt("id"),
            rs.getString("name"),
            rs.getDouble("salary")
        );
    }
}
```

``` java
//DAO实现类
@Repository("empDao")
public class EmpDaoImpl implements EmpDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Override
    public Emp getEmpById(int id) {
        String sql = "select * from emp32 where id=?";
        return jdbcTemplate.queryForObject(sql, new EmpRowMapper(), id);
    }
//...
```


### 4. JdbcTemplate对象的主要方法 
1. queryForInt()：
``` java
//查询一个整数类型
int count = jdbcTemplateObject.queryForInt("select count(*) from emp32");

//一个使用绑定变量的简单查询
int age = jdbcTemplateObject.queryForInt("select age from emp32 where id = ?", new Object[]{10});
```

2. queryForLong()：
``` java
//查询一个 long类型
long count = jdbcTemplateObject.queryForLong("select count(*) from emp32");
```

3. queryForObject()：
``` java
//查询字符串
String SQL = "select name from emp32 where id = ?";
String name = jdbcTemplateObject.queryForObject(SQL, new Object[]{10}, String.class);

//查询并返回一个对象：
String SQL = "select * from emp32 where id = ?";
emp32 student = jdbcTemplateObject.queryForObject(SQL, new Object[]{10}, new EmpRowMapper());

//查询并返回多个对象：
String SQL = "select * from emp32";
List<emp32> students = jdbcTemplateObject.query(SQL, new EmpRowMapper());
```


4. update()：
``` java
//在表中插入一行：
String SQL = "insert into emp32 (name, age) values (?, ?)";
jdbcTemplateObject.update( SQL, new Object[]{"Zara", 11} );

//更新表中的一行：
String SQL = "update emp32 set name = ? where id = ?";
jdbcTemplateObject.update( SQL, new Object[]{"Zara", 10} );

//从表中删除一行：
String SQL = "delete emp32 where id = ?";
jdbcTemplateObject.update( SQL, new Object[]{20} );
```

5. execute()：执行DDL语句
``` java
String SQL = "CREATE TABLE emp32(
	id		INT AUTO_INCREMENT,
	NAME	VARCHAR(30),
	salary	DOUBLE DEFAULT 5000,
	CONSTRAINT student_id_pk PRIMARY KEY(id),
	CONSTRAINT student_name_uk UNIQUE(NAME)
)";
jdbcTemplateObject.execute( SQL );
```


### 5. 异常转换
- Spring提供了自己的开箱即用的数据异常分层——DataAccessException作为根异常，它负责转换所有的原始异常。
- 所以开发者无需处理底层的持久化异常，因为Spring JDBC模块已经在DataAccessException类及其子类中封装了底层的异常。
- 这样可以使异常处理机制独立于当前使用的具体数据库。
- 除了默认的SQLErrorCodeSQLExceptionTranslator类，开发者也可以提供自己的SQLExceptionTranslator实现。

例如：自定义SQLExceptionTranslator实现的简单例子，当出现完整性约束错误时自定义错误消息：
``` java
public class CustomSQLErrorCodeTranslator extends SQLErrorCodeSQLExceptionTranslator {
    @Override
    protected DataAccessException customTranslate
      (String task, String sql, SQLException sqlException) {
        if (sqlException.getErrorCode() == -104) {
            return new DuplicateKeyException("完整性约束冲突", sqlException);
        }
        return null;
    }
}
```
