---
title: 「安全认证」基于Shiro前后端分离的认证与授权(一.认证篇)
date: 2020-01-18 23:26:02
tags: [后端开发, 安全认证]
categories: 安全认证
---

### 1. 开始之前

#### 1.1 技术选型

选用`SpringBoot+Shiro+JWT`实现登录认证，结合`Redis`服务实现`token`的续签，前端选用`Vue`动态构造路由及更细粒度的操作权限控制。

- 前后端分离项目中，我们一般采用的是无状态登录：服务端不保存任何客户端请求者信息，客户端需要自己携带着信息去访问服务端，并且携带的信息可以被服务端辨认。
- 而`Shiro`默认的拦截跳转都是跳转`url`页面，拦截校验机制恰恰使用的`session`；而前后端分离后，后端并无权干涉页面跳转。
- 因此前后端分离项目中使用`Shiro`就需要对其进行改造，我们可以在整合`Shiro`的基础上自定义登录校验，继续整合`JWT`(或者 oauth2.0 等)，使其成为支持服务端无状态登录，即`token`登录。
- 在`Vue`项目中，只需要根据登录用户的权限信息动态的加载路由列表就可以动态的构造出访问菜单。<!-- more -->

#### 1.2 整体流程

- 首次通过`post`请求将用户名与密码到`login`进行登入，登录成功后返回`token`；
- 每次请求，客户端需通过`header`将`token`带回服务器做`JWT Token`的校验；
- 服务端负责`token`生命周期的刷新，用户权限的校验；

![](auth-global.png)

### 2. SpringBoot 整合 Shiro+JWT

这里贴出主要逻辑，源码请移步文章末尾获取。

1. 数据表

```sql
/** 系统用户表 */
DROP TABLE IF EXISTS sys_user;
CREATE TABLE sys_user(
    id INT AUTO_INCREMENT COMMENT '用户ID',
    account VARCHAR(30) NOT NULL COMMENT '用户名',
    PASSWORD VARCHAR(50) COMMENT '用户密码',
    salt VARCHAR(8) COMMENT '随机盐',
    nickname VARCHAR(30) COMMENT '用户昵称',
    roleId INT COMMENT '角色ID',
    createTime DATE COMMENT '创建时间',
    updateTime DATE COMMENT '更新时间',
    deleteStatus VARCHAR(2) DEFAULT '1' COMMENT '是否有效：1有效，2无效',
    CONSTRAINT sys_user_id_pk PRIMARY KEY(id),
    CONSTRAINT sys_user_account_uk UNIQUE(account)
);
COMMIT;
```

2. pom.xml

```xml
<!-- JWT -->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.8.3</version>
</dependency>
<!-- shiro -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.2</version>
</dependency>
```

3. `shiro`配置类：构建`securityManager`环境，及配置`shiroFilter`并将`jwtFilter`添加进`shiro`的拦截器链中，放行登录注册请求。

```java
@Configuration
public class ShiroConfig {
    @Bean("securityManager")
    public DefaultWebSecurityManager getManager(MyRealm myRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 使用自己的realm
        securityManager.setRealm(myRealm);
        /*
         * 关闭shiro自带的session，详情见文档
         * http://shiro.apache.org/session-management.html#SessionManagement-StatelessApplications%28Sessionless%29
         */
        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        securityManager.setSubjectDAO(subjectDAO);
        return securityManager;
    }

    @Bean("shiroFilter")
    public ShiroFilterFactoryBean factory(DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        factoryBean.setSecurityManager(securityManager);
        // 拦截器
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
        // 配置不会被拦截的链接 顺序判断，规则：http://shiro.apache.org/web.html#urls-
        filterChainDefinitionMap.put("/register", "anon");
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/unauthorized", "anon");

        // 添加自己的过滤器并且取名为jwt
        Map<String, Filter> filterMap = new HashMap<>(1);
        filterMap.put("jwt", new JwtFilter());
        factoryBean.setFilters(filterMap);

        // 过滤链定义，从上向下顺序执行，一般将/**放在最为下边
        filterChainDefinitionMap.put("/**", "jwt");
        // 未授权返回
        factoryBean.setUnauthorizedUrl("/unauthorized");

        factoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return factoryBean;
    }
    /**
     * 添加注解支持
     */
    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        // 强制使用cglib，防止重复代理和可能引起代理出错的问题
        // https://zhuanlan.zhihu.com/p/29161098
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(DefaultWebSecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }
}
```

4. 自定义`Realm`：继承`AuthorizingRealm`类，在其中实现登陆验证及权限获取的方法。

```java
@Slf4j
@Component("MyRealm")
public class MyRealm extends AuthorizingRealm {
    /** 注入SysService */
    private SysService sysService;
    @Autowired
    public void setSysService(SysService sysService) {
        this.sysService = sysService;
    }
    /**
     * 必须重写此方法，不然Shiro会报错
     */
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;
    }
    /**
     * 用来进行身份认证，也就是说验证用户输入的账号和密码是否正确，
     * 获取身份验证信息，错误抛出异常
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) throws AuthenticationException {
        log.info("————————身份认证——————————");
        String token = (String) auth.getCredentials();
        if (null == token || !JwtUtil.isVerify(token)) {
            throw new AuthenticationException("token无效!");
        }
        // 解密获得username，用于和数据库进行对比
        String account = JwtUtil.parseTokenAud(token);
        User user = sysService.selectByAccount(account);
        if (null == user) {
            throw new AuthenticationException("用户不存在!");
        }
        return new SimpleAuthenticationInfo(user, token,"MyRealm");
    }
    /**
     * 获取用户权限信息，包括角色以及权限。
     * 只有当触发检测用户权限时才会调用此方法，例如checkRole,checkPermissionJwtToken
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        log.info("————权限认证 [ roles、permissions]————");
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        /* 暂不编写，此处编写后，controller中可以使用@RequiresPermissions来对用户权限进行拦截 */
        return simpleAuthorizationInfo;
    }
}

```

5. 鉴权登录过滤器：继承`BasicHttpAuthenticationFilter`类,该拦截器需要拦截所有请求除(除登陆、注册等请求)，用于判断请求是否带有`token`，并获取`token`的值传递给`shiro`的登陆认证方法作为参数，用于获取`token`；

```java
@Slf4j
public class JwtFilter extends BasicHttpAuthenticationFilter {
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        try {
            executeLogin(request, response);
            return true;
        } catch (Exception e) {
            unauthorized(response);
            return false;
        }
    }
    /**
     * 认证
     */
    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        String authorization = httpServletRequest.getHeader("X-Token");
        JwtToken token = new JwtToken(authorization);
        // 提交给realm进行登入，如果错误他会抛出异常并被捕获
        getSubject(request, response).login(token);
        return true;
    }
    /**
     * 认证失败 跳转到 /unauthorized
     */
    private void unauthorized(ServletResponse resp) {
        try {
            HttpServletResponse httpServletResponse = (HttpServletResponse) resp;
            httpServletResponse.sendRedirect("/unauthorized");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
    /**
     * 对跨域提供支持
     */
    @Override
    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;
        httpServletResponse.setHeader("Access-control-Allow-Origin", httpServletRequest.getHeader("Origin"));
        httpServletResponse.setHeader("Access-Control-Allow-Methods", "GET,POST,OPTIONS,PUT,DELETE");
        httpServletResponse.setHeader("Access-Control-Allow-Headers", httpServletRequest.getHeader("Access-Control-Request-Headers"));
        // 跨域时会首先发送一个option请求，给option请求直接返回正常状态
        if (httpServletRequest.getMethod().equals(RequestMethod.OPTIONS.name())) {
            httpServletResponse.setStatus(HttpStatus.OK.value());
            return false;
        }
        return super.preHandle(request, response);
    }
}
```

6. `JwtToken`

```java
public class JwtToken implements AuthenticationToken {
    private String token;
    JwtToken(String token) {
        this.token = token;
    }
    @Override
    public Object getPrincipal() {
        return token;
    }
    @Override
    public Object getCredentials() {
        return token;
    }
}
```

7. `JWT`工具类：利用登陆信息生成`token`，根据`token`获取`username`，`token`验证等方法。

```java
public class JwtUtil {
    /** 设置过期时间: 30分钟 */
    private static final long EXPIRE_TIME = 30 * 60 * 1000;
    /** 服务端的私钥secret,在任何场景都不应该流露出去 */
    private static final String TOKEN_SECRET = "zhengchao";
    /**
     * 生成签名，30分钟过期
     */
    public static String createToken(User user) {
        try {
            // 设置过期时间
            Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
            // 私钥和加密算法
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
            // 设置头部信息
            Map<String, Object> header = new HashMap<>(2);
            header.put("typ", "JWT");
            header.put("alg", "HS256");
            // 返回token字符串
            return JWT.create()
                    .withHeader(header)
                    .withClaim("aud", user.getAccount())
                    .withClaim("uid", user.getId())
                    .withExpiresAt(date)
                    .sign(algorithm);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
    /**
     * 检验token是否正确
     */
    public static boolean isVerify(String token){
        try {
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
            JWTVerifier verifier = JWT.require(algorithm).build();
            verifier.verify(token);
            return true;
        } catch (Exception e){
            return false;
        }
    }
    /**
     *从token解析出uid信息,用户ID
     */
    public static int parseTokenUid(String token) {
        DecodedJWT jwt = JWT.decode(token);
        return jwt.getClaim("uid").asInt();
    }
    /**
     *从token解析出aud信息,用户名
     */
    public static String parseTokenAud(String token) {
        DecodedJWT jwt = JWT.decode(token);
        return jwt.getClaim("aud").asString();
    }
    /**
     *从token解析出过期时间
     */
    public static Date paraseExpiresTime(String token){
        DecodedJWT jwt = JWT.decode(token);
        return  jwt.getExpiresAt();
    }
}
```

8. MD5 加密工具类

```java
public class Md5Util {
    /**
     * md5加密
     * @param s：待加密字符串
     * @return 加密后16进制字符串
     */
    public static String md5(String s) {
        try {
            //实例化MessageDigest的MD5算法对象
            MessageDigest md = MessageDigest.getInstance("MD5");
            //通过digest方法返回哈希计算后的字节数组
            byte[] bytes = md.digest(s.getBytes("utf-8"));
            //将字节数组转换为16进制字符串并返回
            return toHex(bytes);
        }
        catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 获取随即盐
     */
    public static String salt(){
        //利用UUID生成随机盐
        UUID uuid = UUID.randomUUID();
        //返回a2c64597-232f-4782-ab2d-9dfeb9d76932
        String[] arr = uuid.toString().split("-");
        return arr[0];
    }
    /**
     * 字节数组转换为16进制字符串
     * @param bytes数组
     * @return 16进制字符串
     */
    private static String toHex(byte[] bytes) {
        final char[] HEX_DIGITS = "0123456789ABCDEF".toCharArray();
        StringBuilder ret = new StringBuilder(bytes.length * 2);
        for (int i=0; i<bytes.length; i++) {
            ret.append(HEX_DIGITS[(bytes[i] >> 4) & 0x0f]);
            ret.append(HEX_DIGITS[bytes[i] & 0x0f]);
        }
        return ret.toString();
    }
}
```

### 3. 注册与登录主要逻辑

这里只贴出主要逻辑，`DAO`和`Mapper`映射可查看源码，源码请移步文章末尾获取。

1. 登录`Controller`

```java
@RestController
public class SysApi {
    /**
     * 注入服务类
     */
    private SysService sysService;
    @Autowired
    public void setSysService(SysService sysService) {
        this.sysService = sysService;
    }
    /**
     * 注册(用户名，密码)
     * @param account
     * @param password
     * @return
     */
    @PostMapping("/register")
    public ResponseVo<String> register(String account, String password) {
        return sysService.register(account, password);
    }
    /**
     * 登录(用户名，密码)
     * @param account
     * @param password
     * @return
     */
    @PostMapping("/login")
    public ResponseVo<String> login(String account, String password) {
        return sysService.login(account, password);
    }
    /**
     * 处理非法请求
     */
    @GetMapping("/unauthorized")
    public ResponseVo unauthorized(HttpServletRequest request) {
        return new ResponseVo(-1, "Token失效请重新登录!");
    }
}
```

2. Service

```java
public interface SysService {
    /**
     * 注册(用户名，密码)
     */
    ResponseVo<String> register(String account, String password);
    /**
     * 登录(用户名，密码)
     */
    ResponseVo<String> login(String account, String password);
    /**
     * 根据account查找用户，自定义Realm中调用
     */
    User selectByAccount(String account);
}
/**
 * 实现类
 */
@Service
public class SysServiceImpl implements SysService {

    private SysDao sysDao;
    /**
     * 注入DAO
     */
    @Autowired
    public void setSysDao(SysDao sysDao) {
        this.sysDao = sysDao;
    }
    /**
     * 用户注册(用户名，密码)
     *
     * @param account 用户名
     * @param password 密码
     * @return token
     */
    @Override
    public ResponseVo<String> register(String account, String password) {
        //检查用户名是否被占用
        User user = sysDao.selectByAccount(account);
        if(user!=null) {
            return new ResponseVo<>( -1, "用户名被占用");
        }
        //添加用户信息
        user = new User();
        //设置用户名
        user.setAccount(account);
        //密码加密后再保存
        String salt = Md5Util.salt();
        String md5Password = Md5Util.md5(password+salt);
        user.setPassword(md5Password);
        user.setSalt(salt);
        //设置注册时间
        user.setCreatetime(new Date());
        //添加到数据库
        int row = sysDao.insertSelective(user);
        //返回信息
        if(row>0) {
            //生成token给用户
            String token = JwtUtil.createToken(user);
            return new ResponseVo<>(0,"注册成功", token);
        }else {
            return new ResponseVo<>( -1, "注册失败");
        }
    }
    /**
     * 用户登录(用户名，密码)
     *
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
                //生成token给用户
                String token = JwtUtil.createToken(user);
                return new ResponseVo<>(0,"登录成功", token);
            }
        }
        return new ResponseVo<>( -1, "登录失败");
    }
    /**
     * 根据account查找用户，自定义Realm中调用
     *
     * @param account
     * @return User
     */
    @Override
    public User selectByAccount(String account) {
        return sysDao.selectByAccount(account);
    }
}
```

3. 统一接口返回格式

```java
public class ResponseVo<T> {
    /** 状态码 */
    private int code;
    /** 提示信息 */
    private String msg;
    /** 返回的数据 */
    private T data;
    public ResponseVo() {}
    public ResponseVo(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }
    public ResponseVo(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
    public int getCode() {
        return code;
    }
    public void setCode(int code) {
        this.code = code;
    }
    public String getMsg() {
        return msg;
    }
    public void setMsg(String msg) {
        this.msg = msg;
    }
    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}
```

> 注：这里的登录认证逻辑在`github`源码`tag`的`V1.0`中，后续版本再加入`Token`续签和`shiro`前后端权限管理等。
> 源码地址: [https://github.com/chaooo/springboot-vue-shiro.git](https://github.com/chaooo/springboot-vue-shiro.git)
> 仅下载认证逻辑源码:
> `git clone --branch V1.0 https://github.com/chaooo/springboot-vue-shiro.git`
