---
title: 「安全认证」MD5算法加盐实现用户密码加密
date: 2019-11-15 21:49:28
tags: [后端开发, 安全认证]
categories: 安全认证
---


### 1. MD5加密算法介绍
MD5的全称是Message-Digest Algorithm 5（信息-摘要算法 第五版），经MD2、MD3和MD4发展而来的一种加密算法，是典型的消息摘要算法，属Hash算法一类。作用是让大容量信息在用数字签名软件签署私人密匙前被"压缩"成一种保密的格式（就是把一个任意长度的字节串变换成一定长的大整数）。通过MD5算法进行加密获得一个随机长度的信息并产生一个128位的信息摘要。如果将这个128位的二进制摘要信息换算成十六进制，可以得到一个32位的字符串，因此我们加密完成后的16进制的字符串长度为32位。
<!-- more --> 

### 2. MD5加密算法特点：
1. 压缩性：任意长度的数据，算出的MD5值长度都是固定的。
2. 容易计算：从原数据计算出MD5值很容易。
3. 抗修改性：对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。
4. 强抗碰撞：已知原数据和其MD5值，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。

### 3. 盐（Salt）
在密码学中，是指通过在密码任意固定位置插入特定的字符串，让散列后的结果和使用原始密码的散列结果不相符，这种过程称之为“加盐”。

### 4. java.security.MessageDigest类
JDK中的java.security.MessageDigest用于为应用程序提供信息摘要算法的功能，如 MD5 或 SHA 算法。
- MessageDigest 通过其getInstance系列静态函数来进行实例化和初始化。
- MessageDigest 对象通过使用 update 方法处理数据。任何时候都可以调用 reset 方法重置摘要。一旦所有需要更新的数据都已经被更新了，应该调用 digest 方法之一完成哈希计算并返回结果。
- 对于给定数量的更新数据，digest 方法只能被调用一次。digest 方法被调用后，MessageDigest  对象被重新设置成其初始状态。

### 5. 封装一个MD5加密工具类
``` java
import java.security.MessageDigest;
import java.util.UUID;
public class MD5Util {
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
     * @return
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


### 6. 使用封装的MD5工具类完成用户注册(主要代码)
``` java
public Object register(String name, String password) {
    //添加用户信息
    user = new User();
    //设置用户名
    user.setName(name);
    //密码加密后再保存
    String salt = MD5Util.salt();
    String md5Password = MD5Util.md5(password+salt);
    //存入MD5加密后的密码
    user.setPassword(md5Password);
    //随机盐存入数据库，用于登录校验
    user.setSalt(salt);
    //最后将用户数据数据存入数据库
    int row = userDao.insert(user);
    return ...
}
```


### 7. 使用封装的MD5工具类完成用户登录(主要代码)
``` java
public Object login(String name, String password) {
    //根据用户名在数据库查找用户
    User user = userDao.selectByName(name);
    //取出用户信息比对
    String dbPassword = user.getPassword();
    String  salt = user.getSalt();
    //通过密码+盐 重新生成 MD5密码
    String md5Password = MD5Util.md5(password+salt);
    if(md5Password.equals(dbPassword)) {
        //登录成功
    }
}
```


### 8. 扩展：MessageDigest类常用方法
#### 8.1 构造方法摘要
`MessageDigest(String algorithm)` --创建具有指定算法名称的MessageDigest 实例对象。
- MessageDigest类是一个工厂类，其构造器是受保护的，不允许直接使用new MessageDigist( )来创建对象，而必须通过其静态方法getInstance( )生成MessageDigest对象。其中传入的参数指定计算消息摘要所使用的算法，常用的有"MD5"，"SHA"等。

#### 8.2 成员方法摘要：
| 返回值 |   方法名  | 描述          |
|---------|--------------|---------|
| Object  | `clone()`    | 如果实现是可复制的，则返回一个副本。|
|byte[]   | `digest()`   | 通过执行诸如填充之类的最终操作完成哈希计算。|
|byte[]   |`digest(byte[] input)`| 使用指定的字节数组对摘要进行最后更新，然后完成摘要计算。|
| int     |`digest(byte[] buf, int offset, int len)`|通过执行诸如填充之类的最终操作完成哈希计算。|
| String  |`getAlgorithm()`|返回标识算法的独立于实现细节的字符串。|
| int     |`getDigestLength()`|返回以字节为单位的摘要长度，如果提供程序不支持此操作并且实现是不可复制的，则返回 0。|
|static MessageDigest|`getInstance(String algorithm)`|生成实现指定摘要算法的 MessageDigest 对象。|
|static MessageDigest|`getInstance(String algorithm, Provider provider)`|生成实现指定提供程序提供的指定算法的 MessageDigest 对象，如果该算法可从指定的提供程序得到的话。|
|static MessageDigest|`getInstance(String algorithm, String provider)`|生成实现指定提供程序提供的指定算法的 MessageDigest 对象，如果该算法可从指定的提供程序得到的话。|
| Provider|`getProvider()`|返回此信息摘要对象的提供程序。|
|static boolean|`isEqual(byte[] digesta, byte[] digestb)`|比较两个摘要的相等性。|
| void   |`reset()`|重置摘要以供再次使用。|
| String |`toString()`|返回此信息摘要对象的字符串表示形式。|
| void   |`update(byte input)`|使用指定的字节更新摘要。|
| void   |`update(byte[] input)`|使用指定的字节数组更新摘要。|
| void   |`update(byte[] input, int offset, int len)`|使用指定的字节数组，从指定的偏移量开始更新摘要。|
| void   |`update(ByteBuffer input)`|使用指定的 ByteBuffer 更新摘要。|


> ★ 编程思路：java.security包中的MessageDigest类提供了计算消息摘要（即生成散列码）的方法，首先生成对象，执行其update( )方法可以将原始数据传递给该对象，然后执行其digest( )方法即可得到消息摘要。
