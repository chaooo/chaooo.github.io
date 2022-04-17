---
title: 「Redis」基于Redis的分布式锁实现
date: 2019-04-08 16:04:23
tags: [后端开发, Redis]
categories: Redis
---


### SETNX命令简介
- `SETNX key value`返回(`1:key`的值被设置，`0:key`的值没被设置)，将`key`的值设为`value`，并且仅当`key`不存在。
- 锁的`key`为目标数据的唯一键，`value`为锁的期望超时时间点；
- 基于`Redis`实现的分布式锁，主要基于`redis`的`setnx（set if not exist）`命令；
<!-- more -->

### 1. jedis实现分布式锁
``` xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.0.1</version>
</dependency>
```

#### 1.1 实现示例:
``` java
public static boolean correctGetLock(String lockKey, String requestId, int expireTime) {
    String result = jedis.set(lockKey, requestId, "NX", "PX", expireTime);
    if ("OK".equals(result)) {
        return true;
    }
    return false;
}
```
`jedis.set(String key, String value, String nxxx, String expx, int time)`
    - **`key`**：保证唯一，用来当锁（`redis`记录的`key`）
    - **`value`**：`redis`记录的`value`，目的是为了标志锁的所有者（竞争锁的客户端），保证解锁时只能解自己加的锁。`requestId`可以使用`UUID.randomUUID().toString()`方法生成
    - **`nxxx`**：`"NX"`意思是`SET IF NOT EXIST`，即当`key`不存在时，我们进行`set`操作，若`key`已经存在，则不做任何操作
    - **`expx`**：`"PX"`意思是要给这个`key`加一个过期的设置（单位毫秒），过期时间由第五个参数决定
    - **`time`**：`expx`设置为`"PX"`时，`redis key`的过期时间

#### 1.2 解锁示例:
``` java
public boolean correctReleaseLock(String lockKey, String requestId) {
    String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
    Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
    if (RELEASE_SUCCESS.equals(result)) {
        return true;
    }
    return false;
}
```
`eval`命令执行`Lua`代码的时候，`Lua`代码将被当成一个命令去执行，并且直到`eval`命令执行完成，`Redis`才会执行其他命令，所以保证了检查和删除操作都是原子的。

#### 1.3 这类琐最大的缺点
加锁时只作用在一个`Redis`节点上，即使`Redis`通过`sentinel`保证高可用，如果这个`master`节点由于某些原因发生了主从切换，那么就会出现锁丢失的情况：
1. 在`Redis`的`master`节点上拿到了锁；
2. 但是这个加锁的`key`还没有同步到`slave`节点；
3. `master`故障，发生故障转移，`slave`节点升级为`master`节点；
4. 导致锁丢失。

> 因此，`Redis`作者antirez基于分布式环境下提出了一种更高级的分布式锁的实现方式：`Redlock`。基于`Redis`的`Redisson`实现了`Redlock`。


### 2. Redisson实现普通分布式锁
普通分布式实现非常简单，无论是那种架构，向`Redis`通过`EVAL`命令执行`LUA脚本`即可。
``` xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.3.2</version>
</dependency>
```

**单机模式:**
``` java
// 构造redisson实现分布式锁必要的Config
Config config = new Config();
config.useSingleServer().setAddress("redis://172.29.1.180:5379")
                        .setPassword("a123456").setDatabase(0);
// 构造RedissonClient
RedissonClient redissonClient = Redisson.create(config);
// 设置锁定资源名称, 还可以getFairLock(), getReadWriteLock()
RLock lock = redissonClient.getLock("DISLOCK");
boolean isLock;
try {
    // 尝试获取分布式锁
    // 500ms拿不到锁, 就认为获取锁失败。10000ms即10s是锁失效时间。
    isLock = lock.tryLock(500, 10000, TimeUnit.MILLISECONDS);
    if (isLock) {
        //TODO if get lock success, do something;
    }
} catch (Exception e) {
} finally {
    // 无论如何, 最后都要解锁
    lock.unlock();
}
```

**哨兵模式:**
即`Sentinel`模式，实现代码和单机模式几乎一样，唯一的不同就是`Config`的构造：
``` java
Config config = new Config();
config.useSentinelServers().addSentinelAddress(
        "redis://172.29.3.245:26378","redis://172.29.3.245:26379", "redis://172.29.3.245:26380")
      .setMasterName("mymaster").setPassword("a123456").setDatabase(0);
```

**集群模式:**
即`Cluster`模式，集群模式构造`Config`如下：
``` java
Config config = new Config();
config.useClusterServers().addNodeAddress(
        "redis://172.29.3.245:6375","redis://172.29.3.245:6376", "redis://172.29.3.245:6377",
        "redis://172.29.3.245:6378","redis://172.29.3.245:6379", "redis://172.29.3.245:6380")
      .setPassword("a123456").setScanInterval(5000);
```


### 3. Redisson实现Redlock分布式锁
#### 3.1 Redlock算法大概原理：
- 在`Redis`的分布式环境中，我们假设有`N`个`Redis master`。这些节点**完全互相独立，不存在主从复制或者其他集群协调机制**。我们确保将在`N`个实例上使用与在`Redis`单实例下相同方法获取和释放锁。
- 为了取到锁，客户端应该执行以下操作:
    - 获取当前`Unix`时间，以毫秒为单位。
    - 依次尝试从`N`个实例，使用相同的`key`和具有唯一性的`value`（例如UUID）获取锁。
    - 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。
    - **当且仅当(N/2+1)的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功**，例如3个节点至少需要`3/2+1=2`2个。
    - 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
    - 若获取锁失败，客户端应该在**所有的Redis实例上进行解锁**（即便某些Redis实例根本就没有加锁成功）。

#### 3.2 使用`Redlock`
单机模式`Redis`为例:
``` java
Config config = new Config();
config.useClusterServers().addNodeAddress(
        "redis://127.0.0.1:6379","redis://127.0.0.1:6369", "redis://127.0.0.1:6359",
        "redis://127.0.0.1:6349","redis://127.0.0.1:6339")
        .setPassword("******");
// 节点1
Config config1 = new Config();
config1.useSingleServer().setAddress("redis://127.0.0.1:6379");
RedissonClient redissonClient1 = Redisson.create(config1);
// 节点2
Config config2 = new Config();
config2.useSingleServer().setAddress("redis://127.0.0.1:6378");
RedissonClient redissonClient2 = Redisson.create(config2);
// 节点3
Config config3 = new Config();
config3.useSingleServer().setAddress("redis://127.0.0.1:6377");
RedissonClient redissonClient3 = Redisson.create(config3);
// 设置锁定资源名称
String resourceName = "REDLOCK";
RLock lock1 = redissonClient1.getLock(resourceName);
RLock lock2 = redissonClient2.getLock(resourceName);
RLock lock3 = redissonClient3.getLock(resourceName);
// 实例化RedissonRedLock
RedissonRedLock redLock = new RedissonRedLock(lock1, lock2, lock3);
try {
    boolean isLock = redLock.tryLock(500, 30000, TimeUnit.MILLISECONDS);
    if (isLock) {
        //TODO if get lock success, do something;
        Thread.sleep(30000);
    }
} catch (Exception e) {
} finally {
    //解锁
    redLock.unlock();
}
```
最核心的变化就是 `RedissonRedLock redLock`=**`new RedissonRedLock(lock1,lock2,lock3)`;**，因为我这里是以三个节点为例。
+ 如果是主从`Redis`架构、哨兵`Redis`架构、集群`Redis`架构实现`Redlock`，只需要改变上述`config1`、`config2`、`config3`为主从模式、哨兵模式、集群模式配置即可，但相应需要`3`个独立的`Redis`主从集群、`3`个`Redis`独立的哨兵集群、`3`个独立的`Cluster`集群。
+ 以`sentinel`模式架构为例，`3`个`sentinel`模式集群，如果要获取分布式锁，那么需要向这`3`个`sentinel`集群通过`EVAL`命令执行`LUA`脚本，需要`3/2+1=2`，即至少2个`sentinel`集群响应成功，才算成功的以`Redlock`算法获取到分布式锁。


### 4. Redlock问题合集
#### 4.1 N个节点的理解
假设我们用`N(>=3)`个节点实现`Redlock`算法的分布式锁。**不是**一个有`N`个主节点的cluster集群；而是**要么是`N`个redis单实例，要么是`N`个sentinel集群，要么是`N`个cluster集群**。

#### 4.2 失效时间如何设置
这个问题的场景是，假设设置失效时间10秒，如果由于某些原因导致10秒还没执行完任务，这时候锁自动失效，导致其他线程也会拿到分布式锁。
这确实是Redis分布式最大的问题，不管是普通分布式锁，还是Redlock算法分布式锁，都没有解决这个问题。也有一些文章提出了对失效时间续租，即延长失效时间，很明显这又提升了分布式锁的复杂度（没有现成的框架有实现）。

#### 4.3 redis分布式锁的高可用
关于Redis分布式锁的安全性问题，在分布式系统专家Martin Kleppmann和Redis的作者Antirez之间已经发生过一场争论。有兴趣的同学，搜索"基于Redis的分布式锁到底安全吗"就能得到你想要的答案，需要注意的是，有上下两篇（这应该就是传说中的神仙打架吧）。

#### 4.4 使用Zookeeper还是Redis实现分布式锁
没有绝对的好坏，只有更适合自己的业务。
就**性能**而言，`Redis`很明显优于`Zookeeper`；就分布式锁实现的健壮性(**高可用**)而言，`Zookeeper`很明显优于`Redis`。至于如何选择，还要看具体业务场景。

> 参考：[https://mp.weixin.qq.com/s/8uhYult2h_YUHT7q7YCKYQ](https://mp.weixin.qq.com/s/8uhYult2h_YUHT7q7YCKYQ)
