---
title: 「Spring Reactive Stack」使用R2DBC访问MySQL
date: 2021-03-03 21:01:42
tags: [后端开发, Spring, SpringReactive, MySQL]
categories: SpringReactive
---

`MySQL`等一系列的关系型数据库也在`R2DBC`等包的帮助下支持响应式。
`R2DBC`基于`Reactive Streams`反应流规范，它是一个开放的规范，为驱动程序供应商和使用方提供接口(`r2dbc-spi`)，与`JDBC`的阻塞特性不同，它提供了完全反应式的**非阻塞API**与**关系型数据库**交互。<!-- more -->

1. 创建`Maven`项目，并导入依赖`pom.xml`:
``` xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<!-- data-r2dbc同时也会将r2dbc-pool导入 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<!-- r2dbc mysql 库-->
<dependency>
  <groupId>dev.miku</groupId>
  <artifactId>r2dbc-mysql</artifactId>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
```

2. 配置文件`application.yml`
``` ymal
spring:
  r2dbc:
    url: r2dbcs:mysql://192.168.2.100:3306/test?characterEncoding=UTF8&serverTimezone=Asia/Shanghai
    username: developer
    password: 123456
    
#日志配置
logging:
  level:
    root: info
    # 调试环境查看执行的sql
    dev.miku.r2dbc.mysql.client.ReactorNettyClient: debug
```

3. `SQL`与`Model`类
``` sql
create table `base_user` (
  `userId` int not null auto_increment,
  `userName` varchar(100) default null comment '用户名',
  `userMobile` varchar(20) default null comment '手机号',
  primary key (`userId`),
  unique key `userMobile` (`userMobile`)
) engine=innodb comment='测试用户信息';
insert into `base_user` (`userName`, `userMobile`) values ('黑子', '15914061216');
insert into `base_user` (`userName`, `userMobile`) values ('大黄', '15914061217');
```

``` java
@Data
public class BaseUser {
    private Integer id;
    private String name;
    private String mobile;
}
```
> 实际开发中，由于历史原因数据库字段大多与`Model`类字段无法对应，这里也不对应，在`sql`中用别名对应。

4. 编写`DAO`层，这里继承的事响应式的`crud`类`ReactiveCrudRepository`
``` java
@Repository
public interface BaseUserDao extends ReactiveCrudRepository<BaseUser, Integer> {
    /**
     * 根据用户id查询用户
     * @param id userId
     * @return BaseUser
     */
    @Override
    @Query("select userId as id, userMobile as mobile, userName as name from base_user where userId= :id")
    Mono<BaseUser> findById(Integer id);

    /**
     * 更新用户名
     * @param id userId
     * @param name userName
     */
    @Modifying
    @Query("update base_user set userName= :name where userId= :id")
    Mono<Integer> updateNameById(Integer id, String name);

    /**
     * 新增用户
     * @param name userName
     * @param mobile userMobile
     */
    @Modifying
    @Query("insert into base_user(userName, userMobile) values (:name, :mobile)")
    Mono<Void> insertUser(String name, String mobile);

    /**
     * 根据用户id删除用户
     * @param id userId
     */
    @Override
    @Modifying
    @Query("delete from base_user where userId= :id")
    Mono<Void> deleteById(Integer id);
}
```

5. 编写`Service`层，这里返回的值为`Reactor`的对象`Flux`或`Mono`
``` java
@Service
@RequiredArgsConstructor
public class BaseUserService {

    private final BaseUserDao baseUserDao;

    public Mono<BaseUser> findById (Integer id) {
        return baseUserDao.findById(id);
    }

    public Mono<Integer> updateById(BaseUser user){
        return baseUserDao.updateNameById(user.getId(), user.getName());
    }

    public Mono<Void> insertUser(BaseUser user){
        return baseUserDao.insertUser(user.getName(), user.getMobile());
    }

    public Mono<Void> deleteById(Integer id) {
        return baseUserDao.deleteById(id);
    }

}
```

6. 编写`Controller`层
``` java
@RestController
@RequiredArgsConstructor
@RequestMapping("/user")
public class TestApi {

    private final BaseUserService baseUserService;

    @GetMapping("/get")
    public Mono<BaseUser> findById(Integer id) {
        return baseUserService.findById(id);
    }

    @PostMapping("/update")
    public Mono<Integer> updateById(BaseUser user) {
        return baseUserService.updateById(user);
    }

    @PostMapping("/insert")
    public Mono<Void> insertUser(BaseUser user) {
        return baseUserService.insertUser(user);
    }

    @PostMapping("/delete")
    public Mono<Void> deleteById(Integer id) {
        return baseUserService.deleteById(id);
    }
}
```

7. 测试:
    + 查询：`curl -X GET "http://localhost:8080/user/get?id=1"`
    + 更新：`curl -X POST "http://localhost:8080/user/update?id=1&name=小黑子"`
    + 新增：`curl -X POST "http://localhost:8080/user/insert?name=小蓝子&mobile=15815011618"`
    + 删除：`curl -X POST "http://localhost:8080/user/delete?id=1"`

> 本文使用Spring Boot版本：2.4.3

