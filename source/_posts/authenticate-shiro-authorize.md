---
title: 「安全认证」基于Shiro前后端分离的认证与授权(二.授权篇)
date: 2020-01-21 18:10:55
tags: [后端开发, 安全认证]
categories: 安全认证
---

前面我们整合了`SpringBoot+Shiro+JWT`实现了登录认证，但还没有实现权限控制，这是接下来的工作。<!-- more -->

### 1. JWT的Token续签
#### 1.1 续签思路
1. 业务逻辑：
    + 登录成功后，用户在未过期时间内继续操作，续签token。
    + 登录成功后，空闲超过过期时间，返回token已失效，重新登录。
2. 实现逻辑：
    1. 登录成功后将token存储到redis里面(这时候k、v值一样都为token)，并设置过期时间为token过期时间
    2. 当用户请求时token值还未过期，则重新设置redis里token的过期时间。
    3. 当用户请求时token值已过期，但redis中还在，则JWT重新生成token并覆盖v值(这时候k、v值不一样了)，然后设置redis过期时间。
    4. 当用户请求时token值已过期，并且redis中也不存在，则用户空闲超时，返回token已失效，重新登录。

#### 1.2 编码实现
1. `pom.xml`引入`Redis`

``` xml
<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.8.0</version>
</dependency>
```

2. 编写`Redis`工具类

``` java
@Component
public class RedisUtil {
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    /**
     * 指定缓存失效时间
     * @param key  键
     * @param time 时间(秒)
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }
    /**
     * 普通缓存放入并设置时间
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
}
```

3. JwtUtil中增加返回过期秒数的方法

``` java
public class JwtUtil {
    /** 设置过期时间: 30分钟 */
    private static final long EXPIRE_TIME = 30 * 60 * 1000;
    //... 其他代码省略
    /**
     * 返回设置的过期秒数
     * @return long 秒数
     */
    public static long getExpireTime(){
        return  EXPIRE_TIME/1000;
    }
}
```

4. 改写登录逻辑，生成`token`后存入`Redis`

``` java
@Service
public class SysServiceImpl implements SysService {
    private String getToken(User user){
        // 生成token
        String token = JwtUtil.createToken(user);
        // 为了过期续签，将token存入redis，并设置超时时间
        redisUtil.set(token, token, JwtUtil.getExpireTime());
        return token;
    }
    /**
     * 用户登录(用户名，密码)
     * @param account 用户名
     * @param password 密码
     * @return token
     */
    @Override
    public ResponseVo<String> login(String account, String password) {
        //处理比对密码
        User user = sysDao.selectByAccount(account);
        if(user!=null) {
            String  salt = user.getSalt();
            String md5Password = Md5Util.md5(password+salt);
            String dbPassword = user.getPassword();
            if(md5Password.equals(dbPassword)) {
                //生成token给用户，并存入redis
                String token = getToken(user);
                return new ResponseVo<>(0,"登录成功", token);
            }
        }
        return new ResponseVo<>( -1, "登录失败");
    }
}
```

5. 改写`MyRealm`，加入`token`续签逻辑

``` java
@Slf4j
@Component("MyRealm")
public class MyRealm extends AuthorizingRealm {
    /**
     * JWT Token续签：
     * 业务逻辑：登录成功后，用户在未过期时间内继续操作，续签token。
     *         登录成功后，空闲超过过期时间，返回token已失效，重新登录。
     * 实现逻辑：
     *    1.登录成功后将token存储到redis里面(这时候k、v值一样都为token)，并设置过期时间为token过期时间
     *    2.当用户请求时token值还未过期，则重新设置redis里token的过期时间。
     *    3.当用户请求时token值已过期，但redis中还在，则JWT重新生成token并覆盖v值(这时候k、v值不一样了)，然后设置redis过期时间。
     *    4.当用户请求时token值已过期，并且redis中也不存在，则用户空闲超时，返回token已失效，重新登录。
     */
    public boolean tokenRefresh(String token, User user) {
        String cacheToken = String.valueOf(redisUtil.get(token));
        // 过期后会得到"null"值，所以需判断字符串"null"
        if (cacheToken != null && cacheToken.length() != 0 && !"null".equals(cacheToken)) {
            // 校验token有效性
            if (!JwtUtil.isVerify(cacheToken)) {
                // 生成token
                String newToken = JwtUtil.createToken(user);
                // 将token存入redis,并设置超时时间
                redisUtil.set(token, newToken, JwtUtil.getExpireTime());
            } else {
                // 重新设置超时时间
                redisUtil.expire(token, JwtUtil.getExpireTime());
            }
            log.info("打印存入redis的过期时间："+redisUtil.getExpire(token));
            return true;
        }
        return false;
    }
    /**
     * 重写认证逻辑
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) throws AuthenticationException {
        log.info("————————身份认证——————————");
        String token = (String) auth.getCredentials();
        if (null == token) {
            throw new AuthenticationException("token为空!");
        }
        // 解密获得username，用于和数据库进行对比
        String account = JwtUtil.parseTokenAud(token);
        User user = sysService.selectByAccount(account);
        if (null == user) {
            throw new AuthenticationException("用户不存在!");
        }
        // 校验token是否过期
        if (!tokenRefresh(token, user)) {
            throw new AuthenticationException("Token已过期!");
        }
        return new SimpleAuthenticationInfo(user, token,"MyRealm");
    }
}
```

到此，JWT的Token续签的功能已经全部实现了。


### 2. 权限管理
#### 2.1 首先增加三张数据表
``` sql
/** 角色表 */
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role` (
  `id` INT(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `role_name` VARCHAR(100) DEFAULT NULL COMMENT '角色名称',
  `description` VARCHAR(100) DEFAULT NULL COMMENT '描述',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=INNODB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT COMMENT='角色表';
INSERT  INTO `sys_role`(`id`,`role_name`,`description`) VALUES (1,'admin','管理角色'),(2,'user','用户角色');
/** 权限表 */
DROP TABLE IF EXISTS `sys_permission`;
CREATE TABLE `sys_permission` (
  `id` VARCHAR(32) NOT NULL COMMENT '主键id',
  `name` VARCHAR(100) DEFAULT NULL COMMENT '菜单标题',
  `url` VARCHAR(255) DEFAULT NULL COMMENT '路径',
  `menu_type` INT(11) DEFAULT NULL COMMENT '菜单类型(0:一级菜单; 1:子菜单:2:按钮权限)',
  `perms` VARCHAR(255) DEFAULT NULL COMMENT '菜单权限编码',
  `sort_no` INT(10) DEFAULT NULL COMMENT '菜单排序',
  `del_flag` INT(1) DEFAULT '0' COMMENT '删除状态 0正常 1已删除',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `index_prem_sort_no` (`sort_no`) USING BTREE,
  KEY `index_prem_del_flag` (`del_flag`) USING BTREE
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT COMMENT='菜单权限表';
INSERT  INTO `sys_permission`(`id`,`name`,`url`,`menu_type`,`perms`,`sort_no`,`del_flag`) VALUES ('1','新增用户','/user/add',2,'user:add',1,0),('2','删除用户','/user/delete',2,'user:delete',2,0),('3','修改用户','/user/update',2,'user:update',3,0),('4','查询用户','/user/list',2,'user:list',4,0);
/** 角色与权限关联表 */
DROP TABLE IF EXISTS `sys_role_permission`;
CREATE TABLE `sys_role_permission` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `role_id` INT(11) DEFAULT NULL COMMENT '角色id',
  `permission_id` INT(11) DEFAULT NULL COMMENT '权限id',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `index_group_role_per_id` (`role_id`,`permission_id`) USING BTREE,
  KEY `index_group_role_id` (`role_id`) USING BTREE,
  KEY `index_group_per_id` (`permission_id`) USING BTREE
) ENGINE=INNODB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT COMMENT='角色权限表';
INSERT  INTO `sys_role_permission`(`id`,`role_id`,`permission_id`) VALUES (1,1,1),(2,1,2),(3,1,3),(4,1,4),(5,2,4);
```


#### 2.2 编码实现
1. 补全`MyRealm`中授权验证逻辑

``` java
@Slf4j
@Component("MyRealm")
public class MyRealm extends AuthorizingRealm {
    //...其他代码省略
/**
     * 获取用户权限信息，包括角色以及权限。
     * 只有当触发检测用户权限时才会调用此方法，例如checkRole,checkPermissionJwtToken
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        log.info("————权限认证 [ roles、permissions]————");
        User user = null;
        if (principals != null) {
            user = (User) principals.getPrimaryPrincipal();
        }
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        if (user != null) {
            // 用户拥有的角色，比如“admin/user”
            String role = sysService.getRoleByRoleid(user.getRoleid());
            simpleAuthorizationInfo.addRole(role);
            log.info("角色为："+role);
            // 用户拥有的权限集合，比如“role:add,user:add”
            Set<String> permissions = sysService.getPermissionsByRoleid(user.getRoleid());
            simpleAuthorizationInfo.addStringPermissions(permissions);
            log.info("权限有："+permissions.toString());
        }
        return simpleAuthorizationInfo;
    }
}
```

2. `Service`中添加获取角色与权限的方法，DAO与Mapper请移步源码。

``` java
public interface SysService {
	/**
	 * 根据roleid查找用户角色名，自定义Realm中调用
	 * @param roleid
	 * @return roles
	 */
	String getRoleByRoleid(Integer roleid);

	/**
	 * 根据roleid查找用户权限，自定义Realm中调用
	 * @param roleid
	 * @return  Set<permissions>
	 */
	Set<String> getPermissionsByRoleid(Integer roleid);
}
/**
 * 实现类
 */
@Service
public class SysServiceImpl implements SysService {
    @Override
    public String getRoleByRoleid(Integer roleid) {
        return sysDao.getRoleByRoleid(roleid);
    }
    @Override
    public Set<String> getPermissionsByRoleid(Integer roleid) {
        return sysDao.getPermissionsByRoleid(roleid);
    }
}
```

3. `Controller`中使用`@RequiresPermissions`来控制权限

``` java
@RestController
public class UserApi {
	/**
	 * 获取所有用户信息
	 * @return
	 */
	@RequiresPermissions("user:list")
	@GetMapping("/user/list")
	public ResponseVo list() {
		return userService.loadUser();
	}
	/**
	 * 用户更新资料
	 * @param user
	 * @return
	 */
	@RequiresPermissions("user:update")
	@PostMapping("/user/update")
	public ResponseVo update(User user, HttpServletRequest request) {
		String token = request.getHeader("X-Token");
		return userService.modifyUser(token, user);
	}
}
```


> 注：这里的登录认证+授权控制 在`github`源码`tag`的`V2.0`中，后续版本再加入前端动态路由控制等。
> 源码地址: [https://github.com/chaooo/springboot-vue-shiro.git](https://github.com/chaooo/springboot-vue-shiro.git)
> 仅下载后端认证+授权控制源码:
> `git clone --branch V2.0 https://github.com/chaooo/springboot-vue-shiro.git`

