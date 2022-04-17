---
title: 「Spring Security」基于Redis的Token自动续签优化
date: 2021-12-10 11:35:00
tags: [后端开发, SpringSecurity, 安全认证, Token]
categories: 安全认证
---

本文基于上一篇文章：《Spring Security（三）整合 JWT 实现无状态登录示例》。

在 `SpringSecurity` 整合 `JWT` 实现无状态登录示例中，我们在 `JwtAuthenticationFilter` (自定义`JWT`认证过滤器) 解析 `Token` 成功后，提供了续签逻辑：<!-- more -->
```java
/**
 * 刷新Token的时机：
 * 1. 当前时间 < token过期时间
 * 2. 当前时间 > (签发时间 + (token过期时间 - token签发时间)/2)
 */
private void refreshToken(HttpServletResponse response, Claims claims) {
    // 当前时间
    long current = System.currentTimeMillis();
    // token签发时间
    long issuedAt = claims.getIssuedAt().getTime();
    // token过期时间
    long expiration = claims.getExpiration().getTime();
    // (当前时间 < token过期时间) && (当前时间 > (签发时间 + (token过期时间 - token签发时间)/2))
    if ((current < expiration) && (current > (issuedAt + ((expiration - issuedAt) / 2)))) {
        /*
         * 重新生成token
         */
        Calendar calendar = Calendar.getInstance();
        // 设置签发时间
        calendar.setTime(new Date());
        Date now = calendar.getTime();
        // 设置过期时间: 5分钟
        calendar.add(Calendar.MINUTE, 5);
        Date time = calendar.getTime();
        String refreshToken = Jwts.builder()
                .setSubject(claims.getSubject())
                // 签发时间
                .setIssuedAt(now)
                // 过期时间
                .setExpiration(time)
                // 算法与签名(同生成token)：这里算法采用HS512，常量中定义签名key
                .signWith(SignatureAlgorithm.HS512, ConstantKey.SIGNING_KEY)
                .compact();
        // 主动刷新token，并返回给前端
        response.addHeader("refreshToken", refreshToken);
        log.info("刷新token执行时间: {}", (System.currentTimeMillis() - current) + " 毫秒");
    }
}
```
这里的逻辑是：`Token` 未过期并且当前时间已经超过 `Token` 有效时间的一半，重新生成一个 `refreshToken`，并返回给前端，前端需要用 `refreshToken` 替换之前旧的 `Token`。

### Token续签优化方案
预期效果：前端不需要手动替换 `Token`，每次用 `Token` 请求资源时自动续期。

实现方案：引入 `Redis`，实现逻辑：
1. 登录成功后将 `Token` 存储到 `Redis` 里面(k,v都为 `Token` 的值)，并设置 `Redis` 过期时间为： `Token` 过期时间。
2. 用户发起请求时，每次都根据k为`Token`的键去换取 `Redis` 的值，这里命名为 `cacheToken`：
    - 当 `cacheToken` 在有效期内，重设 `Redis` 过期时间为：当前时间 + (`cacheToken`过期时间 - `cacheToken`签发时间)。
    - 当 `cacheToken` 已过期（`Redis` 在有效期内），则 `JWT` 重新生成 `Token` 并覆盖v值(这时候k、v值不一样了)，然后设置 `Redis` 过期时间为： `cacheToken` 过期时间。
    - 若 `Redis` 也过期，取不到 `cacheToken`，则拒绝访问或返回错误信息，需要重新登录。

#### 具体实现
##### 1. 在 `pom.xml` 中引入 `Redis` 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

##### 2. 在 `application.yml` 配置文件中配置 `Redis`：

```ymal
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password: 123456
    # Redis数据库索引（默认为0）
    database: 0
    # 连接超时时间（毫秒）
    timeout: 5000
```

##### 3. 简单的 `RedisService` 封装

```java
@Service
public class RedisService {
    @Resource
    private RedisTemplate<Serializable, Object> redisTemplate;
    /**
     * 读取缓存
     */
    public Object get(String key) {
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        return operations.get(key);
    }
    /**
     * 判断缓存中是否存在
     */
    public boolean exists(String key) {
        return StringUtils.hasLength(key) && Boolean.TRUE.equals(redisTemplate.hasKey(key));
    }
    /**
     * 删除缓存
     */
    public void remove(String key) {
        if (exists(key)) {
            redisTemplate.delete(key);
        }
    }
    /**
     * 写入缓存
     */
    public boolean set(String key, Object value) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
    /**
     * 写入缓存 并 加上过期时间
     */
    public boolean set(String key, Object value, Date date) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            redisTemplate.expireAt(key, date);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
    /**
     * 写入过期时间（毫秒）
     */
    public boolean expire(String key, Long expireTimeMillis) {
        boolean result = false;
        try {
            redisTemplate.expire(key, expireTimeMillis, TimeUnit.MILLISECONDS);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

##### 4. 修改JWT登录过滤器 `JwtLoginFilter`，构造方法中加入 `RedisService`，并生成 `Token` 后存入 `Redis`:

``` java
@Slf4j
public class JwtLoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final RedisService redisService;
    public JwtLoginFilter(AuthenticationManager authenticationManager, RedisService redisService) {
        this.authenticationManager = authenticationManager;
        this.redisService = redisService;
    }

    /**
     * 尝试身份认证(接收并解析用户凭证)
     */
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        return authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(username, password, new ArrayList<>())
        );
    }

    /**
     * 认证成功(用户成功登录后，这个方法会被调用，我们在这个方法里生成token)
     */
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication auth) {
        try {
            Collection<? extends GrantedAuthority> authorities = auth.getAuthorities();
            // 定义存放角色集合的对象
            List<String> roleList = new ArrayList<>();
            for (GrantedAuthority grantedAuthority : authorities) {
                roleList.add(grantedAuthority.getAuthority());
            }
            /*
             * 生成token
             */
            Calendar calendar = Calendar.getInstance();
            // 设置签发时间
            calendar.setTime(new Date());
            Date now = calendar.getTime();
            // 设置过期时间: 5分钟
            calendar.add(Calendar.MINUTE, 5);
            Date time = calendar.getTime();
            String token = Jwts.builder()
                    .setSubject(auth.getName() + "-" + roleList)
                    // 签发时间
                    .setIssuedAt(now)
                    // 过期时间
                    .setExpiration(time)
                    // 自定义算法与签名：这里算法采用HS512，常量中定义签名key
                    .signWith(SignatureAlgorithm.HS512, ConstantKey.SIGNING_KEY)
                    .compact();
            // 将token存入redis,并设置超时时间为token过期时间
            redisService.set(token, token, time);
            /*
             * 返回token
             */
            log.info("用户登录成功，生成token={}", token);
            // 登录成功后，返回token到header里面
            response.addHeader("Authorization", token);
            // 登录成功后，返回token到body里面
            ResponseJson<String> result = ResponseJson.success("登录成功", token);
            response.setCharacterEncoding("UTF-8");
            response.getWriter().write(JSON.toJSONString(result));
        } catch (IOException e) {
            log.error("IOException:", e);
        }
    }

    /**
     * 认证失败调用
     */
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException {
        log.warn("登录失败[{}]，AuthenticationException={}", request.getRequestURI(), exception.getMessage());
        // 登录失败，返回错误信息
        ResponseJson<Void> result = ResponseJson.error(exception.getMessage(), null);
        response.setCharacterEncoding("UTF-8");
        response.getWriter().write(JSON.toJSONString(result));
    }
}
```

##### 5. 修改JWT认证过滤器 `JwtAuthenticationFilter`，构造方法中加入 `RedisService`，并添加 `Token` 续签逻辑:

``` java
@Slf4j
public class JwtAuthenticationFilter extends BasicAuthenticationFilter {

    private final RedisService redisService;
    public JwtAuthenticationFilter(AuthenticationManager authenticationManager, RedisService redisService) {
        super(authenticationManager);
        this.redisService = redisService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        UsernamePasswordAuthenticationToken authentication = getAuthentication(request, response);
        SecurityContextHolder.getContext().setAuthentication(authentication);
        chain.doFilter(request, response);
    }

    private UsernamePasswordAuthenticationToken getAuthentication(HttpServletRequest request, HttpServletResponse response) {
        /*
         * 解析token
         */
        String token = request.getHeader("Authorization");
        if (StringUtils.hasLength(token)) {
            String cacheToken = String.valueOf(redisService.get(token));
            if (StringUtils.hasLength(token) && !"null".equals(cacheToken)) {
                String user = null;
                try {
                    Claims claims = Jwts.parser()
                            // 设置生成token的签名key
                            .setSigningKey(ConstantKey.SIGNING_KEY)
                            // 解析token
                            .parseClaimsJws(cacheToken).getBody();
                    // 取出用户信息
                    user = claims.getSubject();
                    // 重设Redis超时时间
                    resetRedisExpire(token, claims);
                } catch (ExpiredJwtException e) {
                    log.info("Token过期续签，ExpiredJwtException={}", e.getMessage());
                    Claims claims = e.getClaims();
                    // 取出用户信息
                    user = claims.getSubject();
                    // 刷新Token
                    refreshToken(token, claims);
                } catch (UnsupportedJwtException e) {
                    log.warn("访问[{}]失败，UnsupportedJwtException={}", request.getRequestURI(), e.getMessage());
                } catch (MalformedJwtException e) {
                    log.warn("访问[{}]失败，MalformedJwtException={}", request.getRequestURI(), e.getMessage());
                } catch (SignatureException e) {
                    log.warn("访问[{}]失败，SignatureException={}", request.getRequestURI(), e.getMessage());
                } catch (IllegalArgumentException e) {
                    log.warn("访问[{}]失败，IllegalArgumentException={}", request.getRequestURI(), e.getMessage());
                }
                if (user != null) {
                    // 获取用户权限和角色
                    String[] split = user.split("-")[1].split(",");
                    ArrayList<GrantedAuthority> authorities = new ArrayList<>();
                    for (String s : split) {
                        authorities.add(new GrantedAuthorityImpl(s));
                    }
                    // 返回Authentication
                    return new UsernamePasswordAuthenticationToken(user, null, authorities);
                }
            }
        }
        log.warn("访问[{}]失败，需要身份认证", request.getRequestURI());
        return null;
    }

    /**
     * 重设Redis超时时间
     * 当前时间 + (`cacheToken`过期时间 - `cacheToken`签发时间)
     */
    private void resetRedisExpire(String token, Claims claims) {
        // 当前时间
        long current = System.currentTimeMillis();
        // token签发时间
        long issuedAt = claims.getIssuedAt().getTime();
        // token过期时间
        long expiration = claims.getExpiration().getTime();
        // 当前时间 + (`cacheToken`过期时间 - `cacheToken`签发时间)
        long expireAt = current + (expiration - issuedAt);
        // 重设Redis超时时间
        redisService.expire(token, expireAt);
    }

    /**
     * 刷新Token
     * 刷新Token的时机： 当cacheToken已过期 并且Redis在有效期内
     * 重新生成Token并覆盖Redis的v值(这时候k、v值不一样了)，然后设置Redis过期时间为：新Token过期时间
     */
    private void refreshToken(String token, Claims claims) {
        // 当前时间
        long current = System.currentTimeMillis();
        /*
         * 重新生成token
         */
        Calendar calendar = Calendar.getInstance();
        // 设置签发时间
        calendar.setTime(new Date());
        Date now = calendar.getTime();
        // 设置过期时间: 5分钟
        calendar.add(Calendar.MINUTE, 5);
        Date time = calendar.getTime();
        String refreshToken = Jwts.builder()
                .setSubject(claims.getSubject())
                // 签发时间
                .setIssuedAt(now)
                // 过期时间
                .setExpiration(time)
                // 算法与签名(同生成token)：这里算法采用HS512，常量中定义签名key
                .signWith(SignatureAlgorithm.HS512, ConstantKey.SIGNING_KEY)
                .compact();
        // 将refreshToken覆盖Redis的v值,并设置超时时间为refreshToken过期时间
        redisService.set(token, refreshToken, time);
        // 打印日志
        log.info("刷新token执行时间: {}", (System.currentTimeMillis() - current) + " 毫秒");
    }
}
```

##### 6. 修改 `SpringSecurity` 配置类，注入 `RedisService`：

``` java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private RedisService redisService;

    @Resource
    private UserDetailsService userDetailsService;

    @Resource
    private BCryptPasswordEncoder bCryptPasswordEncoder;

    /**
     * 全局请求忽略规则配置
     */
    @Override
    public void configure(WebSecurity web) {
        // 需要放行的URL
        web.ignoring().antMatchers("/register", "/hello");
    }

    /**
     * 自定义认证策略：登录的时候会进入
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth) {
        // 2. 通过实现 AuthenticationProvider 自定义身份认证验证组件
        auth.authenticationProvider(new AuthenticationProviderImpl(userDetailsService, bCryptPasswordEncoder));
    }

    /**
     * 自定义 HTTP 验证规则
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // 关闭Session
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            // 所有请求需要身份认证
            .and().authorizeRequests().anyRequest().authenticated()
            .and()
            // 自定义JWT登录过滤器
            .addFilter(new JwtLoginFilter(authenticationManager(), redisService))
            // 自定义JWT认证过滤器
            .addFilter(new JwtAuthenticationFilter(authenticationManager(), redisService))
            // 自定义认证拦截器，也可以直接使用内置实现类Http403ForbiddenEntryPoint
            .exceptionHandling().authenticationEntryPoint(new AuthenticationEntryPointImpl())
            // 允许跨域
            .and().cors()
            // 禁用跨站伪造
            .and().csrf().disable();
    }
}
```

> 源码地址：[https://github.com/chaooo/spring-security-jwt.git](https://github.com/chaooo/spring-security-jwt.git),
> 这里我将本文的基于Redis的Token自动续签优化放在github源码tag的V2.0中，防止后续修改后代码对不上。
