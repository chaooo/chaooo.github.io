---
title: 「Redis」深入学习Redis及集群
date: 2019-03-20 16:19:26
tags: [后端开发, Redis]
categories: Redis
---


Redis本质上是一个Key-Value类型的**内存数据库**，整个数据库统统加载在内存当中进行操作，定期通过异步操作把数据库数据flush到硬盘上进行保存。因为是纯内存操作，Redis的性能非常出色，每秒可以处理超过10万次读写操作，是已知性能最快的Key-Value DB。
<!-- more -->
Redis的出色之处不仅仅是性能，Redis最大的魅力是支持保存多种数据结构，此外单个value的最大限制是1GB。另外Redis也可以对存入的Key-Value设置expire时间。 Redis的主要缺点是数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。

### 1. Redis数据结构及命令操作
#### 1.1 基本概念及操作
+ 默认16个数据库，类似数组下表从零开始，初始默认使用零号库；
+ 统一密码管理，16个库都是同样密码，要么都OK要么一个也连接不上，redis默认端口是6379；
+ select命令切换数据库：select 0-15；
+ dbsize：查看当前数据库的key的数量；
+ flushdb：清空当前库；
+ flushall；通杀全部库；

#### 1.2 Redis数据结构
redis存储的是：key-value格式的数据，其中key都是字符串，value有5种不同的数据结构:String、Hash、List、Set、Zset(Sorted Set)
1. String：set, get, del, append, strlen
2. Hash：hset, hget, hdel, hmset(批量设值), hmget, hgetall
3. List：lpush, rpush, lrange, lpop(删除), rpop, lindex
4. Set：sadd, smembers, srem(根据可以移除member), sismember(判断是否为key的成员)
5. ZSet：zadd, zrange, zrem


#### 1.3 Redis键(key)--常用命令介绍
+ keys *：查看所有 key ；
+ exists key的名字：判断某个 key 是否存在；
+ move key dbID（0-15）： 当前库就没有了，被移除了；
+ expire key 秒钟： 为给定的 key 设置过期时间；
+ ttl key： 查看还有多少秒过期，-1表示永不过期，-2表示已过期；
+ type key： 查看你的 key 是什么类型；

### 2. Redis持久化
Redis作为一个键值对内存数据库(NoSQL)，数据都存储在内存当中，在处理客户端请求时，所有操作都在内存当中进行，为了避免内存中数据丢失，Redis提供了RDB和AOF两种不同的数据持久化方式。

#### 2.1 RDB（Redis DataBase）
RDB是一种快照存储持久化方式，具体就是将Redis某一时刻的内存数据保存到硬盘的文件当中，默认保存的文件名为dump.rdb，而在Redis服务器启动时，会重新加载dump.rdb文件的数据到内存当中恢复数据。
- 开启RDB持久化方式一：save命令，或bgsave(异步)
- 开启方式二：在Redis配置文件redis.conf配置，配置完后启动时加载：`redis-server redis.conf`

``` json
save 900 1     # 900s内至少达到一条写命令
save 300 10    # 300s内至少达至10条写命令
save 60 10000  # 60s内至少达到10000条写命令
```

+ RDB的几个优点
    + 与AOF方式相比，通过rdb文件恢复数据比较快。
    + rdb文件非常紧凑，适合于数据备份。
    + 通过RDB进行数据备，由于使用子进程生成，所以对Redis服务器性能影响较小。

+ RDB的几个缺点
    + 如果服务器宕机的话，采用RDB的方式会造成某个时段内数据的丢失，比如我们设置10分钟同步一次或5分钟达到1000次写入就同步一次，那么如果还没达到触发条件服务器就死机了，那么这个时间段的数据会丢失。
    + 使用save命令会造成服务器阻塞，直接数据同步完成才能接收后续请求。
    + 使用bgsave命令在forks子进程时，如果数据量太大，forks的过程也会发生阻塞，另外，forks子进程会耗费内存。

#### 2.2 AOF(Append-only file)
与RDB存储某个时刻的快照不同，AOF持久化方式会记录客户端对服务器的每一次写操作命令（以日志的形式），并将这些写操作以Redis协议追加保存到以后缀为aof文件末尾，在Redis服务器重启时，会加载并运行aof文件的命令，以达到恢复数据的目的。
- 开启方式：在Redis配置文件redis.conf配置

``` json
appendonly yes                  # 开启aof机制
appendfilename "appendonly.aof" # aof文件名
# 写入策略,always表示每个写操作都保存到aof文件中,也可以是everysec(每秒写入一次)或no(操作系统处理)
appendfsync always
no-appendfsync-on-rewrite no    # 默认不重写aof文件
dir ~/redis/                    # 保存目录
```

- aof文件太大，加载aof文件恢复数据时，就会非常慢，为了解决，Redis通过重写aof，可以生成一个恢复当前数据的最少命令集，两种方式：配置no-appendfsync-on-rewrite(默认no)，或者客户端向服务器发送bgrewriteaof命令

* AOF的优点：AOF只是追加日志文件，因此对服务器性能影响较小，速度比RDB要快，消耗的内存较少。
* AOF的缺点：AOF方式生成的日志文件太大，即使通过AFO重写，文件体积仍然很大。恢复数据的速度比RDB慢。
* 当RDB与AOF两种方式都开启时，Redis会优先使用AOF日志来恢复数据，因为AOF保存的文件比RDB文件更完整。

##### 2.2.1 AOF文件修复
1. 备份被写坏的AOF文件
2. 运行redis-check-aof –fix进行修复
3. 用diff -u来看下两个文件的差异，确认问题点
4. 重启redis，加载修复后的AOF文件


### 3. Redis的高并发和快速原因
1. redis是基于内存的，内存的读写速度非常快；
2. redis是单线程的，省去了很多上下文切换线程的时间；
3. redis使用多路复用技术，可以处理并发的连接。非阻塞IO 内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间。
4. 另外，数据结构也帮了不少忙，Redis全程使用hash结构，读取速度快，还有一些特殊的数据结构，对数据存储进行了优化，如压缩表，对短数据进行压缩存储，再如，跳表，使用有序的数据结构加快读取的速度。
5. 还有一点，Redis采用自己实现的事件分离器，效率比较高，内部采用非阻塞的执行方式，吞吐能力比较大。


### 4. Redis利用哨兵(Sentinel)，复制(Replication)这两个功能来保证高可用
1. 哨兵(Sentinel)：可以管理多个Redis服务器，它提供了监控，提醒以及自动的故障转移的功能。
    1. 集群监控：负责监控Redis master和slave进程是否正常工作
    2. 消息通知：如果某个Redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员
    3. 故障转移：如果master node挂掉了，会自动转移到slave node上
    4. 配置中心：如果故障转移发生了，通知client客户端新的master地址
2. 复制(Replication)：则是负责让一个Redis服务器可以配备多个备份的服务器。
    1. 从数据库向主数据库发送sync(数据同步)命令。
    2. 主数据库接收同步命令后，会保存快照，创建一个RDB文件。
    3. 当主数据库执行完保持快照后，会向从数据库发送RDB文件，而从数据库会接收并载入该文件。
    4. 主数据库将缓冲区的所有写命令发给从服务器执行。
    5. 以上处理完之后，之后主数据库每执行一个写命令，都会将被执行的写命令发送给从数据库。


### 5. Redis 主从复制、哨兵和集群这三个有什么区别
主从复制是为了数据备份，哨兵是为了高可用，Redis主服务器挂了哨兵可以切换，集群则是因为单实例能力有限，搞多个分散压力。
1. 主从模式：读写分离，备份，一个Master可以有多个Slaves。
2. 哨兵entinel：监控，自动转移，哨兵发现主服务器挂了后，就会从slave中重新选举一个主服务器。
3. 集群Cluster：为了解决单机Redis容量有限的问题，将数据按一定的规则分配到多台机器，内存/QPS不受限于单机，可受益于分布式集群高扩展性。


### 6. Redis Cluster集群
Redis Cluster，是Redis 3.0开始引入的分布式存储方案。
集群由多个节点(Node)组成，Redis的数据分布在这些节点中。集群中的节点分为主节点和从节点：只有主节点负责读写请求和集群信息的维护；从节点只进行主节点数据和状态信息的复制。
* 集群的作用：
    1. 数据分区：数据分区(或称数据分片)是集群最核心的功能。
    2. 高可用：集群支持主从复制和主节点的自动故障转移（与哨兵类似）；当任一节点发生故障时，集群仍然可以对外提供服务。

#### 6.1 Redis Cluster集群的搭建可以分为四步：
1. **启动节点**：将节点以集群模式启动，此时节点是独立的，并没有建立联系；
2. **节点握手**：让独立的节点连成一个网络；
3. **分配槽**：将16384个槽分配给主节点；
4. **指定主从关系**：为从节点指定主节点。

#### 6.2 Redis Cluster工作原理
+ 客户端与Redis节点直连,不需要中间Proxy层，直接连接任意一个Master节点
+ 根据公式`HASH_SLOT=CRC16(key) mod 16384`，计算出映射到哪个分片上，然后Redis会去相应的节点进行操作

```
         CRC16(key)    |  0~5460   | <--Slot--|Redis(M)|<---|Redis(S可多个从)
         mode 16384    |
Client --------------> | 5461~10922| <--Slot--|Redis(M)|<---|Redis(S可多个从)
                       |
                       |10923~10383| <--Slot--|Redis(M)|<---|Redis(S可多个从)

```


#### 6.3 Redis Cluster优点:
1. 无需Sentinel哨兵监控，如果Master挂了，Redis Cluster内部自动将Slave切换Master
2. 可以进行水平扩容
3. 支持自动化迁移，当出现某个Slave宕机了，那么就只有Master了，这时候的高可用性就无法很好的保证了，万一master也宕机了，咋办呢？ 针对这种情况，如果说其他Master有多余的Slave ，集群自动把多余的Slave迁移到没有Slave的Master 中。

#### 6.4 Redis Cluster缺点:
1. 批量操作是个坑（不同的key会划分到不同的slot中，因此直接使用mset或者mget等操作是行不通）
2. 资源隔离性较差，容易出现相互影响的情况。


#### 6.5 Redis Cluster总结：
1. Redis Cluster集群架构，不同的key是有可能分配在不同的Redis节点上的，在这种情况下Redis的事务机制是不生效。
2. 单机下的redis可以支持16个数据库（db0 ~ db15），在Redis Cluster集群架构下只有一个数据库空间，即db0。
3. 不同的key会划分到不同的slot中，因此直接使用mset或者mget等操作是行不通。
4. 如果Hash对象非常大，是不支持映射到不同节点的！只能映射到集群中的一个节点上。
5. Redis集群模式下进行批量操作：如果执行的key数量比较少，就用串行get操作； 如果需要执行的key很多，就使用Hashtag保证这些key映射到同一台redis节点上。
6. Redis Cluster的架构，是属于分片集群的架构，不做读写分离，因为redis本身在内存上操作，不会涉及IO吞吐，即使读写分离也不会提升太多性能，Redis在生产上的主要问题是考虑容量，单机最多10-20G，key太多降低redis性能.因此采用分片集群结构，已经能保证了我们的性能。其次，用上了读写分离后，还要考虑主从一致性，主从延迟等问题，徒增业务复杂度。

