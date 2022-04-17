---
title: 「安全认证」JSON Web Token 入门
date: 2019-11-18 15:50:53
tags: [后端开发, 安全认证]
categories: 安全认证
---


## JSON Web Token
JSON Web Token（缩写 JWT）基于JSON格式信息一种Token令牌，是目前最流行的跨域认证解决方案。
<!-- more -->
- JWT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户。
- 此后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。
- 服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

### 1. JWT数据结构
它是一个很长的字符串，中间用点（.）分隔成三个部分。
- 例如：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`.`eyJhdWQiOiJjaGFvIiwidWlkIjoyOSwiZXhwIjoxNTY3OTM2NzgwfQ`.`6zvimBNs_MCiov4MOkkUodgKmRFBS2dVhmhIb1MV6m4。

JWT 的三个部分(`Header.Payload.Signature`)依次如下:
1. Header（头部）
2. Payload（负载）
3. Signature（签名）

#### 1.1 Header（头部）
Header 部分是一个 JSON 对象，描述 JWT 的元数据。
``` json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg`：签名的算法（algorithm），默认是 HMAC SHA256（写成`HS256`）
- `typ`：表示这个令牌（token）的类型（type），JWT令牌统一写为`JWT`。

最后，将上面的 JSON 对象使用 `Base64URL算法`转成字符串。

#### 1.2 Payload（负载）
Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段(Reserved claims)，供选用。标准中建议使用这些字段，但不强制。
+ iss (issuer)：签发人
+ exp (expiration time)：过期时间
+ sub (subject)：主题
+ aud (audience)：受众
+ nbf (Not Before)：生效时间
+ iat (Issued At)：签发时间
+ jti (JWT ID)：编号，JWT唯一标识，能用于防止JWT重复使用

除了官方字段，还有公共声明的字段（见：[http://www.iana.org/assignments/jwt/jwt.xhtml](http://www.iana.org/assignments/jwt/jwt.xhtml)）也可以定义私有字段，如：
``` json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
> 注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。

这个 JSON 对象也要使用 `Base64URL算法`转成字符串。


#### 1.3 Signature（签名）
Signature 部分是对前两部分的签名，防止数据篡改。该签名信息是通过header和payload，加上secret，通过算法加密生成。
- 首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。
    + `HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)`
- 算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就可以返回给用户。


### 2. Base64URL算法
前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略、`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。


### 3. JWT 的使用方式及特点
1. 认证原理：
    + 客户端向服务器申请授权，服务器认证以后，生成一个token字符串并返回给客户端，此后客户端在请求受保护的资源时携带这个token，服务端进行验证再从这个token中解析出用户的身份信息。
2. JWT的使用方式：
    - 客户端收到服务器返回的JWT，存储在浏览器（Cookie或localStorage）
    - 此后，客户端每次与服务器通信，都要带上这个JWT。
        1. 一种做法是放在HTTP请求的头信息Authorization字段里面，格式如下：
            + `Authorization: <token>`
            + 需要将服务器设置为接受来自所有域的请求，用Access-Control-Allow-Origin: *
        2. 另一种做法是，跨域的时候，JWT就放在POST请求的数据体里面。

3. 对JWT实现token续签的做法：
    1. 额外生成一个refreshToken用于获取新token，refreshToken需存储于服务端，其过期时间比JWT的过期时间要稍长。
    2. 用户携带refreshToken参数请求token刷新接口，服务端在判断refreshToken未过期后，取出关联的用户信息和当前token。
    3. 使用当前用户信息重新生成token，并将旧的token置于黑名单中，返回新的token。

4. JWT 的几个特点
    1. JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
    2. JWT 不加密的情况下，不能将秘密数据写入JWT。
    3. JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
    4. JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
    5. JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
    6. 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。


### 4. Java中JWT的使用
java-jwt工具包提供了**JWT算法的封装**
1. 导入java-jwt，选择一种算法（HMAC256为例）
    + **`Algorithm algorithm = Algorithm.HMAC256("secret");`**
    + 算法定义了一个令牌是如何被签名和验证的。
2. 创建一个签名的`JWT token`（通过调用jwt.create()创建一个JWTCreator实例）
    + **`String token = JWT.create().withIssuer("auth0").sign(algorithm);`**
    + *如果Claim不能转换为JSON，或者在签名过程中使用的密钥无效，那么将会抛出**JWTCreationException**异常*
3. 验证令牌（调用jwt.require()和传递算法实例来创建一个JWTVerifier实例。方法build()返回的实例是可重用的，因此可以定义一次，并使用它来验证不同的标记。最后调用verifier.verify()来验证token）
    + **`JWTVerifier verifier = JWT.require(algorithm).withIssuer("auth0").build();`**
    + **`verifier.verify(token);`**
    + *如果令牌有一个无效的签名，或者没有满足Claim要求，那么将会抛出**JWTVerificationException**异常*
4. jwt时间的验证（当验证一个令牌时，时间验证会自动发生；JWT令牌可能包括可用于验证的DateNumber字段）
    + `"iat" < TODAY`：这个令牌发布了一个过期的时间
    + `"exp" > TODAY`：这个令牌还没过期
    + `"nbf" > TODAY`：这个令牌已经被使用了
5. 解码一个jwt令牌
    + `DecodedJWT jwt = JWT.decode(token);`
    + `jwt.getAlgorithm();`:返回jwt的算法值,如果没有定义则返回null
    + `jwt.getType();`:返回jwt的类型值，如果没有定义则返回null（多数情况类型值为jwt）
    + *如果令牌有无效的语法，或者消息头或有效负载不是JSONs，那么将会抛出**JWTDecodeException**异常*


### 5. Java中JWT的使用实例
封装一个JWT工具类：
``` java
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import com.auth0.jwt.JWT; //导入java-jwt
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.DecodedJWT;
import com.auth0.jwt.interfaces.JWTVerifier;

import com.entity.User; //引入User实体类

public class JwtUtil {
    //设置过期时间，这里设置15分钟
    private static final long EXPIRE_TIME = 15 * 60 * 1000;
    //服务端的私钥secret,在任何场景都不应该流露出去
    private static final String TOKEN_SECRET = "zhengchao";
    /**
     * 生成签名 
     * @param **User**
     * @param **password**
     * @return
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
                    .withClaim("aud", user.getName())
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
     * @param **token**
     * @return
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
     * @param token
     * @param key
     * @return
     */
    public static int parseTokenUid(String token) {
        DecodedJWT jwt = JWT.decode(token);
        return jwt.getClaim("uid").asInt();
    }
    /**
     *从token解析出aud信息,用户名
     * @param token
     * @param key
     * @return
     */
    public static String parseTokenAud(String token) {
        DecodedJWT jwt = JWT.decode(token);
        return jwt.getClaim("aud").asString();
    }
}
```

登录成功后，生成token给浏览器，存储在浏览器（Cookie或localStorage）
``` java
String token = JwtUtil.createToken(user);
```

此后，客户端每次与服务器通信（需权限的资源），都要带上这个JWT。
1. 一种做法是放在HTTP请求的头信息Authorization字段里面，格式如下：
    + `Authorization: <token>`
    + 需要将服务器设置为接受来自所有域的请求，用Access-Control-Allow-Origin: *
2. 另一种做法是，跨域的时候，JWT就放在POST请求的数据体里面。


> jwt 适合做简单的 restful api 认证，颁发一个固定有效期的 jwt，降低 jwt 暴露的风险，尽量不要对 jwt 做服务端的状态管理，这样才能体现出 jwt 无状态的优势。


### 附：java-jwt已经实现的算法
|JWS	|算法		|介绍								|
|-------|-----------|-----------------------------------|
|HS256	|HMAC256	|HMAC with SHA-256					|
|HS384	|HMAC384	|HMAC with SHA-384					|
|HS512	|HMAC512	|HMAC with SHA-512					|
|RS256	|RSA256		|RSASSA-PKCS1-v1_5 with SHA-256		|
|RS384	|RSA384		|RSASSA-PKCS1-v1_5 with SHA-384		|
|RS512	|RSA512		|RSASSA-PKCS1-v1_5 with SHA-512		|
|ES256	|ECDSA256	|ECDSA with curve P-256 and SHA-256	|
|ES384	|ECDSA384	|ECDSA with curve P-384 and SHA-384	|
|ES512	|ECDSA512	|ECDSA with curve P-521 and SHA-512	|

