---
title: 「Spring」SpringBoot数据库访问
date: 2018-06-14 17:29:31
tags: [后端开发, Spring, SpringBoot]
categories: Spring
---


Springboot对于数据访问层，无论是SQL还是NOSQL，都默认采用整合Spring Data的方式进行统一处理，Springboot添加大量自动配置，屏蔽了很多设置。并引入各种*Template，*Repository来简化我们对数据访问层的操作。
<!-- more -->


### 1.SpringBoot数据库访问
#### 1.1 Spring DAO JdbcTemplate
引入spring-boot-starter-jdbc后（hikari、spring-jdbc包），就可以借助DataSourceAutoConfiguration、JdbcTemplateAutoConfiguration自动配置组件创建出HikariDataSource、JdbcTemplate对象。
1. 引入jdbc启动器、驱动包，创建连接池
2. 根据要操作表定义entity（pojo，属性名与字段名一致）

``` java
public class Direction {
    private int id;
    private String name;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

3. 定义Dao接口

``` java
public interface DirectionDao {
    public List<Direction> findAll();
}
```

4. 定义Dao实现类，扫描并注入JdbcTemplate使用

``` java
@Repository//通过组件扫描加载到Spring容器
public class JdbcDirectionDao implements DirectionDao {
    @Autowired
    private JdbcTemplate template;//通过自动配置加载到Spring容器
    @Override
    public List<Direction> findAll() {
        String sql = "select * from direction";
        RowMapper<Direction> rowMapper = 
            new BeanPropertyRowMapper<Direction>(Direction.class);
        return template.query(sql, rowMapper);
    }
}
```


#### 1.2 Spring MyBatis（XML SQL版本）
- 引入spring-boot-starter-jdbc、驱动包、mybatis-spring-boot-starter
- 引入application.properties（连接池参数）
- 实体类(同上)
- SQL定义

``` xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.xdl.dao.DirectionMapper">
    <select id="selectAll" resultType="cn.xdl.entity.Direction">
        select * from direction
    </select>
    <select id="selectById" parameterType="int" 
        resultType="cn.xdl.entity.Direction">
        select * from direction where id=#{id}
    </select>
</mapper>
```

- Mapper接口
``` java
public interface DirectionMapper {
    public List<Direction> selectAll();
    public Direction selectById(int id);
}
```

- 使用@MapperScan和mybatis.mapperLocations=classpath:sql/*.xml
- 在启动类前追加@MapperScan

``` java
@SpringBootApplication
@MapperScan(basePackages="cn.xdl.dao")//扫描Mapper接口创建对象加载到Spring容器
public class RunBoot {
    ... ...
}
```

- 在application.properties追加mybatis.mapperLocations

``` properties
mybatis.mapperLocations=classpath:sql/*.xml
```


#### 1.3 Spring MyBatis（注解 SQL版本）
- 引入spring-boot-starter-jdbc、驱动包、mybatis-spring-boot-starter
- 引入application.properties（连接池参数）
- 实体类(同上)
- 定义Mapper接口，使用@Select、@Update、@Insert、@Delete注解定义SQL

``` java
public interface DirectionMapper {
    @Select("select * from direction")
    public List<Direction> findAll();
    @Select("select * from direction where id=#{id}")
    public Direction findById(int id);
    @Update("update direction set name=#{name} where id=#{id}")
    public int updateName(@Param("id")int id,@Param("name")String name);
}
```

- 使用@MapperScan（同上）




### 2. Spring Data JPA
#### 2.1 Jpa
Jpa (Java Persistence API) 是 Sun 官方提出的 Java 持久化规范。中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

Sun引入新的JPA ORM规范主要是为了简化现有的持久化开发工作和整合 ORM 技术，结束现在 Hibernate，TopLink，JDO 等 ORM 框架各自为营的局面。

> 注意:Jpa 是一套规范，不是一套产品，那么像 Hibernate,TopLink,JDO 他们是一套产品，如果说这些产品实现了这个 Jpa 规范，那么我们就可以叫他们为 Jpa 的实现产品。


#### 2.2 Spring Boot Jpa
Spring Boot Jpa 是 Spring 基于 ORM 框架、Jpa 规范的基础上封装的一套 Jpa 应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。
> Spring Boot Jpa 让我们解脱了 DAO 层的操作，基本上所有 CRUD 都可以依赖于它来实现

在Spring中使用JPA访问数据库，需要使用Spring Data模块支持。
    - SpringData是对Spring框架一个扩展模块，包含对JPA、Redis、MongoDB等技术的访问支持。

Spring Boot Jpa的使用 
1. 引入spring-boot-starter-jdbc、spring-boot-starter-data-jpa、驱动包
2. 在application.properties定义db连接池参数（同上）
3. 定义RunBoot启动类，使用@SpringBootApplication标记（同上）
4. 根据要操作的表定义实体类，使用@Entity、@Table、@Id、@Column定义该对象和表结构之间的映射关系
``` java
@Entity
@Table(name="direction")
public class Direction {

    @Id
    @Column(name="id")
    private int id;

    @Column(name="name")
    private String name;

    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

5. 定义Dao接口，可以选择继承JpaRepository、PagingAndSortingRepository、CrudRepository等
``` java
public interface DirectionDao 
    extends JpaRepository<Direction, Integer>{
    //...
}
```

#### 2.3 Dao扩展操作
分页查询
``` java
Pageable pageable = PageRequest.of(1, 3);//of(页数从0开始,记录条数)
Page<Direction> page = dao.findAll(pageable);
List<Direction> list = page.getContent();
list.forEach(d->{System.out.println(d.getId()+" "+d.getName());});
System.out.println("总记录数:"+page.getTotalElements()
+" 页数:"+(page.getNumber()+1)+"/"+page.getTotalPages());
List<Direction> list1 = dao.findByIdGreaterThan2(5);
```

按方法名规则扩展
``` java
//where id>?
public List<Direction> findByIdGreaterThan(int id);
```

定义SQL语句扩展
``` java
@Query(nativeQuery=true,value="select * from direction where id>:id")
public List<Direction> findByIdGreaterThan1(@Param("id")int id);
```

定义JPQL面向查询语句扩展
``` java
@Query("from Direction where id>:id") //使用类型名和属性名替代表名和字段名
public List<Direction> findByIdGreaterThan2(@Param("id")int id);
```

按名称模糊查询，带分页支持
``` java
@Query(nativeQuery=true,value="select * from direction where name like :name"
    ,countQuery="select count(*) from direction where name like :name")
public Page<Direction> findByNameLike1(@Param("name")String name,Pageable pageable);
```
