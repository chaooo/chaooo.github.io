---
title: 「Redis」Redis故障处理-持久化时内存不足
date: 2021-05-06 16:01:23
tags: [后端开发, Redis]
categories: Redis
---

### 问题描述
``` bash
# Java错误日志:
redis.clients.jedis.exceptions.JedisDataException: MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error.

# Redis错误日志:
Can't save in background: fork: Resource temporaily unavailable
# 或
Can’t save in background: fork: Cannot allocate memory
```

<!-- more -->

`Redis`在个默认情况下，如果在`RDB snapshots`持久化过程中出现问题，`Redis`不允许用户进行任何更新操作；即：`stop-writes-on-bgsave-error yes`。

临时解决方案是通过命令：`config set stop-writes-on-bgsave-error no` 设置这个选项为`false`，让程序忽略了这个异常，使得程序能够继续往下运行，但写硬盘仍然是失败的！

### 问题分析
#### Redis数据回写机制
`Redis`在进行持久化的时候，有的时候可以在日志中看到fork进程失败的提示，一般是系统可用的内存空间不够导致，这需要我们对`fork`原理明白，才能更好的进行参数调整。

一般来说`Redis`在进行`RDB`的时候，会`fork`出一个子进程，子进程和父进程会共享一个地址空间，在`fork`子进程的时候，会检查当前机器可用的内存是否满足`fork`出一个子进程的要求，一般由操作系统`overcommit_memory`(系统内存分配策略)决定。

+ `Redis`的数据回写机制分同步和异步两种，
    * 同步回写即`SAVE`命令，主进程直接向磁盘回写数据。在数据大的情况下会导致系统假死很长时间，所以一般不是推荐的。
    * 异步回写即`BGSAVE`命令，主进程`fork`后，复制自身并通过这个新的进程回写磁盘，回写结束后新进程自行关闭。由于这样做不需要主进程阻塞，系统不会假死，一般默认会采用这个方法。

`Redis`默认采用异步回写，所以如果我们要将数据刷到硬盘上，这时`Redis`分配内存不能太大，否则很容易发生内存不够用无法`fork`的问题；
设置一个合理的写磁盘策略，否则写频繁的应用，也会导致频繁的`fork`操作，对于占用了大内存的`Redis`来说，`fork`消耗资源的代价是很大的；

#### 系统内存分配策略
`Linux`对大部分申请内存的请求都回复`yes`，以便能跑更多更大的程序。

因为申请内存后，并不会马上使用内存，将这些不会使用的空闲内存分配给其它程序使用，以提高内存利用率，这种技术叫做`Overcommit`。

一般情况下，当所有程序都不会用到自己申请的所有内存时，系统不会出问题，但是如果程序随着运行，需要的内存越来越大，在自己申请的大小范围内，不断占用更多内存，直到超出物理内存，当`Linux`发现内存不足时，会发生`OOM killer(OOM=out-of-memory)`。

`OOM killer`会选择杀死一些进程，以便释放内存。当发生`OOM killer`时，会记录在系统日志中`/var/log/messages`。

用户态进程，非内核线程，占用内存越多和运行时间越短的进程越有可能被杀掉。

+ 在`Linux`下有个vm内核参数：`CommitLimit`用于限制系统应用使用的内存资源；执行`grep -i commit  /proc/meminfo`，看到`CommitLimit`和`Committed_As`参数。
    * `CommitLimit`是一个内存分配上限，`CommitLimit = 物理内存 * overcommit_ratio(/proc/sys/vm/overcmmit_ratio，默认50，即50%) + swap大小`
    * `Committed_As`是已经分配的内存大小(应用程序要申请的内存 + 系统已经分配的内存)。

+ `vm.overcommit_memory`文件指定了内核针对内存分配的策略，其值可以是`0、1、2`。                          
    * `0`：启发策略(默认)；表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。系统在为应用进程分配虚拟地址空间时，会判断当前申请的虚拟地址空间大小是否超过剩余内存大小，如果超过，则虚拟地址空间分配失败。因此，也就是如果进程本身占用的虚拟地址空间比较大或者剩余内存比较小时，`fork`、`malloc`等调用可能会失败。 `0`即是启发式的`overcommitting handle`，会尽量减少`swap`交换分区的使用，`root`可以分配比一般用户略多的内存。
    * `1`：允许`overcommit`；表示内核允许分配所有的物理内存，而不管当前的内存状态如何，允许超过`CommitLimit`，这种情况下，避免了`fork`可能产生的失败，但由于`malloc`是先分配虚拟地址空间，而后通过异常陷入内核分配真正的物理内存，在内存不足的情况下，这相当于完全屏蔽了应用进程对系统内存状态的感知，即`malloc`总是能成功，一旦内存不足，会引起系统`OOM`杀进程，应用程序对于这种后果是无法预测的。 直至内存用完为止。在数据库服务器上不建议设置为1，从而尽量避免使用`swap`交换分区。
    * `2`：禁止`overcommit`；表示不允许超过`CommitLimit`值。由于很多情况下，进程的虚拟地址空间占用远大于其实际占用的物理内存，这样一旦内存使用量上去以后，对于一些动态产生的进程(需要复制父进程地址空间)则很容易创建失败，如果业务过程没有过多的这种动态申请内存或者创建子进程，则影响不大，否则会产生比较大的影响 。这种情况下系统所能分配的内存不会超过上面提到的`CommitLimit`大小，如果这么多资源已经用光，那么后面任何尝试申请内存的行为都会返回错误，这通常意味着此时没法运行任何新程序。


### 解决方案
#### 修改系统内存分配策略
我们可以通过设置`overcommit_memory=1`的优化，减少操作系统内存，提高`Redis`的`fork`成功率，因为`fork`后的进程和父进程共享一个数据空间，持久化要新增的内存空间都会小于父进程已经使用的空间，具体有三种方式修改内核参数，但要有`root`权限：
1. 编辑`/etc/sysctl.conf` ，改`vm.overcommit_memory=1`，然后`sysctl -p`使配置文件生效；
2. 命令：`sysctl vm.overcommit_memory=1` ；
3. 命令：`echo 1 > /proc/sys/vm/overcommit_memory`；

#### 关闭THP（Transparent Huge Pages）
当`Redis`持久化`fork`子进程后，占用内存大小和父进程等同，由于`Linux`在写时有`copy-on-write`机制，父子进程共享相同的物理内存页，当父进程处理写请求的时候会把要修改的页创建副本，而子进程在`fork`过程中共享整个父进程的内存快照。如果我们要减少创建的副本的大小，就涉及操作系统的另外一个概念`Huge Pages`(大页)。

在`Redhat Linux`中，内存都是以页的形式划分的，默认情况下每页是`4K`，这就意味着如果物理内存很大，则映射表的条目将会非常多，会影响`CPU`的检索效率。因为内存大小是固定的，为了减少映射表的条目，可采取的办法只有增加页的尺寸。`Linux Kernel`在`2.6.38`内核中增加了`THP`(Transparent Huge Pages)的特性，支持大内存页（`2MB`）分配，默认开启。当开启后可以加快`fork`子进程的速度，但`fork`操作之后，每个内存页从原来的`4KB`变成了`2MB`，会大幅增加重写期间父进程内存消耗，同时每次写命令引起的复制内存页单位放大了`512`倍，会拖慢写操作的执行时间，因此在使用`Redis`的时候`Redis`建议关闭`THP`，方法为：`echo never > /sys/kernel/mm/transparent_hugepage/enabled`。为了让机器重启该参数仍然生效，建议在`/etc/rc.local`中追加`echo never > /sys/kernel/mm/transparent_hugepage/enabled`，避免失效。当大页被关闭后，可以看到同等操作下，`RDB`备份时候的`copy-on-write`变化内存空间会减少。

综上分析，我们可以操作系统物理内存和`Redis`内存之间的一些关系，尤其`Redis`在持久化的时候`fork`进程会随操作系统的参数不同，需要的内存也有所不同，为了加快`fork`子进程的速度以及主备之间的文件传输同步，一般我们建议一个`Redis`节点的最大内存在`10G-15G`左右，操作系统的内存适当冗余，尽量控制同一台机器的多个`Redis`节点在同一个时间点进行`RDB`备份（可以通过缓存中心定时备份），导致内存同一时刻增加避免内存空间不足导致的`fork`失败，最安全保险的情况是内存为`Redis`的`2倍`，但是在**vm.overcommit_memory=1**和**大页关闭**的情况下，可以根据实际使用，降低操作系统的整个内存大小 。

+ 参考文章：
    * `https://www.jianshu.com/p/785ee3bea266`
    * `https://www.cnblogs.com/wjoyxt/p/3777042.html`
