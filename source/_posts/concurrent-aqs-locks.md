---
title: 「并发编程」AQS框架 与 锁框架（JUC.locks）
date: 2019-10-10 23:00:24
tags: [java, 后端开发, 并发编程]
categories: 并发编程
---


### 1. AQS（队列同步器）
`AbstractQueuedSynchronizer`：队列同步器，简称`AQS`。
- `AQS`维护了一个`volatile int `**`state`**(代表资源共享变量) 和一个**`FIFO`线程等待队列**(多线程争用资源被阻塞时会进入此队列)。
- `AQS`定义了两种资源共享方式：`Exclusive`(独占)，`Share`(共享)<!-- more -->
- `isHeldExclusively`方法：该线程是否正在独占资源
- `tryAcquire`/`tryRelease`：独占的方式尝试获取和释放资源
- `tryAcquireShared`/`tryReleaseShared`：共享的方式尝试获取和释放资源

整个框架的核心就是**如何管理线程阻塞队列**，该队列是严格的`FIFO`队列，因此不支持线程优先级的同步。
- `AQS`只有一个同步队列，可以有多个条件队列。
    + 同步队列的最佳选择是自身没有使用底层锁来构造的非阻塞数据结构，同步队列选择了**`CLH`**作为实现的基础。
    + 条件队列：`AQS`框架提供了一个`ConditionObject`类，给维护独占同步的类以及实现`Lock`接口的类使用。
- 使用`Node`实现**`FIFO`双向队列**，可以用于构建锁 或 其他同步装置的基础框架
- 内部有一个int变量表示的**`同步状态`**(同步状态通过**`getState`**、**`setState`**、**`compareAndSetState`**来维护，同时这三个方法能够保证线程安全)
- `AQS`是个**抽象类**（但没有抽象方法），同步组件一般通过维护`AQS`的**继承子类来实现**。
- `AQS`**既**支持独占地获取同步状态(**排它锁**)，**又**支持共享地获取同步状态(**共享锁**)，从而实现不同类型的组件。
- `AQS`是**基于模板方法**，同步组件需要继承同步器并重写指定的方法，随后将同步器组合在自定义同步组件的实现中，并调用同步器提供的模板方法，而这些模板方法将会调用使用者重写的方法。

> `Synchronizer`(同步器)：是一个对象，它根据本身的状态调节线程的控制流。常见类型的`Synchronizer`包括信号量、关卡和闭锁。


### 2. CountDownLatch（倒计时闭锁）
- 闭锁(`latch`)是一种`Synchronizer`，它可以延迟线程的进度直到线程达到**终止状态**。
- **`CountDownLatch`**(倒计时闭锁)是一个灵活的闭锁实现。
- `CountDownLatch`是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程执行完后再执行。
- `CountDownLatch`**原理**：是通过一个计数器来实现的，计数器的初始化值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就相应得`减1`。当计数器到达`0`时，表示所有的线程都已完成任务，然后在闭锁上等待的线程就可以恢复执行任务。
    + `await()`，阻塞程序继续执行
    + `countDown()`，计数器的值`减1`，当计数器值减至`零`时，所有因调用`await()`方法而处于等待状态的线程就会继续往下执行。
- 计数器不能被重置，如果业务上需要一个可以重置计数次数的版本，可以考虑使用`CycliBarrier`

> `CountDownLatch`使用场景：应用初始化


### 3. Semaphore（信号量）
- **`Semaphore`**(信号量)：用来**控制同时访问**特定资源的线程**数量**，它通过协调各个线程，以保证合理的使用公共资源。
- `Semaphore`**原理**：线程需要通过`acquire()`方法获取许可，而`release()`释放许可。如果许可数达到最大活动数，那么调用`acquire()`之后，便进入等待队列，等待已获得许可的线程释放许可，从而使得多线程能够合理的运行。
    - `acquire()`：获取权限，其底层实现与`CountDownLatch.countdown()`类似;
    - `release()`：释放权限，其底层实现与`acquire()`是一个互逆的过程。

> Semaphore可以用于做流量控制，特别公用资源有限的应用场景，比如数据库连接。


### 4. CyclicBarrier（同步屏障）
- **`CyclicBarrier`**(同步屏障)：可以让一组线程达到一个屏障时被阻塞，直到最后一个线程达到屏障时，所有被阻塞的线程才能继续执行。
- `CyclicBarrier`类似于``CountDownLatch``，它也是通过计数器来实现的。但是相比于`CountDownLatch`功能更加强大。
- `CyclicBarrier`**原理**：当某个线程调用`await`方法时，该线程进入等待状态，且计数器加1，当计数器的值达到设置的初始值时，所有因调用`await`进入等待状态的线程被唤醒，继续执行后续操作。因为`CycliBarrier`在释放等待线程后可以重用，所以称为循环`barrier`。


#### 4.1 CountDownLatch 和 CyclicBarrier 对比
1. `CountDownLatch`描述的是线程(1个或多个)等待其他线程的关系；`CyclicBarrier`描述的是多个线程相互等待的关系。
2. `CountDownLatch`的计数器只能使用一次。而`CyclicBarrier`的计数器可以使用`reset()`方法重置并复用。
3. `CountDownLatch`方法比较少，操作比较简单，而`CyclicBarrier`提供的方法更多，比如：
    + `getNumberWaiting()`：获取阻塞的线程数量。
    + `isBroken()`：获取阻塞线程的状态，被中断返回`true`，否则返回`false`。
    + `CyclicBarrier`的构造方法可以传入`barrierAction`，指定当所有线程都到达时执行的业务功能；

> `CyclicBarrier`可以用于多线程计算数据，最后合并计算结果的应用场景



### 5. JUC.locks 锁框架
```
java.util.concurrent.locks
    |———— Lock接口
        |———— ReentrantLock类
            |———— ReentrantReadWriteLock.ReadLock内部类
            |———— ReentrantReadWriteLock.WriteLock内部类
    |———— Condition接口
    |———— ReadWriteLock接口
        |———— ReentrantReadWriteLock类
    |———— LockSupport类
```

- **`Lock`接口**核心方法：`lock()`，`unlock()`，`lockInterruptibly()`，`newCondition()`，`tryClock()`
    + `lock()`方法类似于使用`synchronized`关键字加锁，如果锁不可用，出于线程调度目的，将禁用当前线程，并且在获得锁之前，该线程将一直处于休眠状态。
    + `lockInterruptibly()`方法顾名思义，就是如果锁不可用，那么当前正在等待的线程是可以被中断的，这比`synchronized`关键字更加灵活。
- **`Condition`接口**核心方法：`awit()`，`signal()`，`signalAll()`
    + 可以看做是Obejct类的wait()、notify()、notifyAll()方法的替代品，与Lock配合使用
- **`ReadWriteLock`接口**核心方法：`readLock()`，`writeLock()`
    + 获取读锁和写锁，注意除非使用`Java8`新锁，否则读读不互斥，读写是互斥的


### 6. ReentrantLock（可重入锁）
**`ReentrantLock`重入锁**使用**`AQS`同步状态**来保存锁重复持有的次数
- 底层代码分析：
    - **`state`**初始化为0，表示未锁定状态
    - A线程`lock()`时，会调用`tryAcquire(`)独占该锁并将**`state+1`**
    - 此后，其他线程再`tryAcquire()`时就会失败，直到A线程`unlock()`到`state=0`(即释放锁)为止，其他线程才有机会获取该锁
    - 当然，锁释放之前，A线程自己是可以重复获取此锁的(`state`会累加)，这就是可重入的概念

`synchronized`实现的锁的重入依赖于`JVM`，是一种重量级锁。
`ReentrantLock`实现了在内存语义上的`synchronized`，使用**`AQS`同步状态**来保存锁重复持有的次数。当锁被一个线程获取时，`ReentrantLock`也会记录下当前获得锁的线程标识，以便检查是否是重复获取，以及当错误的线程试图进行解锁操作时检测是否存在非法状态异常。
- 公平锁和非公平锁
    + 公平锁还是非公平锁取决于`ReentrantLock`的构造方法，**默认**无参为**非公平锁**(`NonfairSync`)；含参构造方法，入参`true`为`FairSync`，入参`false`为`NonfairSync`。
- 非公平锁中，抢到`AQS`的同步状态的未必是同步队列的首节点，只要线程通过`CAS`抢到了同步状态或者在`acquire`中抢到同步状态，就优先占有锁（插队），而相对同步队列这个严格的`FIFO`队列来说，所以会被认为是非公平锁。
- 公平锁的实现直接调用`AQS`的`acquire`方法，`acquire`中调用`tryAcquire`。和非公平锁相比，这里不会执行一次`CAS`，接下来在`tryAcquire`去抢占锁的时候，也会先调用`hasQueuedPredecessors`看看前面是否有节点已经在等待获取锁了，如果存在则同步队列的前驱节点优先（排队`FIFO`）。

> 虽然公平锁看起来在公平性上比非公平锁好，但是公平锁为此付出了大量线程切换的代价，而非公平锁在锁的获取上不能保证公平，就有可能出现锁饥饿，即有的线程多次获取锁而有的线程获取不到锁，没有大量的线程切换保证了非公平锁的吞吐量。


### 7. 读写锁RRW（ReentrantReadWriteLock）
`ReentrantLock`是独占锁，`ReentrantReadWriteLock`是读写锁。
- 独占锁通过`state`变量的`0`和`1`两个状态来控制是否有线程占有锁，共享锁通过`state`变量`0`或者`非0`来控制多个线程访问。
- 读写锁定义为：一个资源能够被多个读线程访问，或者被一个写线程访问，但是不能同时存在读写线程。
- `ReentrantReadWriteLock`的特殊之处其实就是用一个`int`值表示两种不同的状态（`低16`位表示写锁的重入次数，`高16`位表示读锁的使用次数），并通过两个内部类同时实现了`AQS`的两套`API`，核心部分与共享/独占锁并无什么区别。

> `ReentrantReadWriteLock`也会发生**写请求饥饿**的情况，因为写请求一样会排队，不管是公平锁还是非公平锁，在有读锁的情况下，都**不能保证写锁**一定能获取到，这样只要读锁一直占用，就会发生写饥饿的情况。`JDK8`中新增的改进读写锁`StampedLock`可解决饥饿问题


### 8. LockSupport工具类
归根结底，`LockSupport`调用的`Unsafe`中的`native`代码：`park()`，`unpark()`；
- `park`函数是将当前`Thread`阻塞，而`unpark`函数则是将另一个`Thread`唤醒。
- 与`Object`类的`wait/notify`机制相比，`park/unpark`有两个优点：
    1. 以`thread`为操作对象更符合阻塞线程的直观定义；
    2. 操作更精准，可以准确地唤醒某一个线程（`Object`类的`notify`随机唤醒一个线程，`notifyAll`唤醒所有等待的线程），增加了灵活性

> `park`方法的调用一般要在方法一个循环判断体里面。之所以这样做，是为了防止线程被唤醒后，不进行判断而意外继续向下执行，这其实是一种的多线程设计模式-Guarded Suspension。


### 9. StampedLock（Java8新型锁）
`ReentrantReadWriteLock`锁具有读写锁，问题在于`ReentrantReadWriteLock`使得多个读线程同时持有读锁（只要写锁未被占用），而写锁是独占的 ，很容易造成写锁获取不到资源(写请求饥饿)。
- `Java8`引入了一个新的读写锁叫`StampedLock`. 不仅这个锁更快，而且它提供强大的乐观锁API。这种乐观策略的锁非常类似于无锁的操作，使得乐观锁完全不会阻塞写线程。
- `StampedLock`的主要特点：
    1. 所有**获取锁**的方法，都返回一个邮戳（`Stamp`），`Stamp`为0表示获取失败，其余都表示成功；
    2. 所有**释放锁**的方法，都需要一个邮戳（`Stamp`），这个`Stamp`必须是和成功获取锁时得到的`Stamp`一致；
    3. `StampedLock`是**不可重入**的；（如果一个线程已经持有了写锁，再去获取写锁的话就会造成死锁）
    4. `StampedLock`有**三种访问模式**：
        + `Reading`（读模式）：功能和ReentrantReadWriteLock的读锁类似
        + `Writing`（写模式）：功能和ReentrantReadWriteLock的写锁类似
        + `Optimistic reading`（乐观读模式）：这是一种优化的读模式。
    5. `StampedLock`支持读锁和写锁的相互转换
    6. `RRW`(ReentrantReadWriteLock)中，当线程获取到写锁后，可以降级为读锁，但是读锁是不能直接升级为写锁的；`StampedLock`提供了读锁和写锁相互转换的功能，使得该类支持更多的应用场景。
    7. 无论写锁还是读锁，都不支持`Conditon`等待

