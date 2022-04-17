---
title: 「并发编程」阻塞队列 与 线程池
date: 2019-10-14 23:04:54
tags: [java, 后端开发, 并发编程]
categories: 并发编程
---


- 池和队列的关系
    + 线程池或者数据库连接池，都有最大限制。如果超出了限制数量，则新进来的申请连接都要放入额外的**队列里**，等到池空出来时，从队列中取出连接放进池里。<!-- more -->


### 1. BlockingQueue（阻塞队列）
```
Queue接口
    |———— BlockingQueue接口
        |———— ArrayBlockingQueue类
        |———— DelayQueue类
        |———— LinkedBlockingQueue类
        |———— PriorityBlockingQueue类
        |———— SynchronousQueue类
```

- `BlockingQueue`继承了`Queue`接口，提供了一些阻塞方法，主要作用如下：
    + 当线程向队列中插入元素时，如果队列已满，则阻塞线程，直到队列有空闲位置（非满）；
    + 当线程从队列中取元素（删除队列元素）时，如果队列为空，则阻塞线程，直到队列有元素；
- `BlockingQueue`在`Queue`方法基础上增加了两类和阻塞相关的方法：`put(e)`、`take()`；`offer(e, time, unit)`、`poll(time, unit)`。

| 操作类型 | 抛出异常 | 返回特殊值 | 阻塞线程 | 超时 |
|---------|---------|-----------|---------|------|
| 插入    | add(e)   | offer(e) | put(e) | offer(e, time, unit) |
| 删除    | remove() | poll()   | take() | poll(time, unit) |
| 读取    | element()| peek()   | /      | /     |

- **`put(e)`**和**`take()`**方法会一直阻塞调用线程，直到线程被中断或队列状态可用；
- **`offer(e, time, unit)`**和**`poll(time, unit)`**方法会限时阻塞调用线程，直到超时或线程被中断或队列状态可用。
- 阻塞队列主要用在生产者/消费者的场景


#### 1.1 ArrayBlockingQueue
`ArrayBlockingQueue`是一个有边界的阻塞队列，它的内部实现是一个数组。
- 有边界的意思是它的容量是有限的，我们必须在其初始化的时候指定它的容量大小，容量大小一旦指定就不可改变。
- `ArrayBlockingQueue`是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。


#### 1.2 DelayQueue
`DelayQueue`阻塞的是其内部元素，`DelayQueue`中的元素必须实现`java.util.concurrent.Delayed`接口，`Delayed`接口继承了`Comparable`接口，这是因为`DelayedQueue`中的元素需要进行排序，一般情况，我们都是按元素过期时间的优先级进行排序。
- `DelayQueue`应用场景：定时关闭连接、缓存对象，超时处理等


#### 1.3 LinkedBlockingQueue
`LinkedBlockingQueue`阻塞队列大小的配置是可选的，如果我们初始化时指定一个大小，它就是有边界的，如果不指定，它就是无边界的。
- 说是无边界，其实是采用了默认大小为`Integer.MAX_VALUE`的容量 。它的内部实现是一个链表。
- 和`ArrayBlockingQueue`一样，`LinkedBlockingQueue` 也是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。


#### 1.4 PriorityBlockingQueue
`PriorityBlockingQueue`是一个没有边界的队列，它的排序规则和`java.util.PriorityQueue`一样。需要注意，`PriorityBlockingQueue`中允许插入null对象。
- 所有插入`PriorityBlockingQueue`的对象必须实现`java.lang.Comparable`接口，队列优先级的排序规则就是按照我们对这个接口的实现来定义的。
- 从`PriorityBlockingQueue`获得一个迭代器`Iterator`，但这个迭代器并不保证按照优先级顺序进行迭代。


#### 1.5 SynchronousQueue
`SynchronousQueue`队列内部仅允许容纳一个元素。
- 当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。



### 2. Callable & Future
`Callable`与`Runnable`的功能大致相似，`Callable`功能强大一些，就是被线程执行后，可以返回值，并且能抛出异常。
- `Runnable`接口只有一个`run()`方法，实现类重写`run`方法，把一些费时操作写在其中，然后使用某个线程去执行该`Runnable`实现类即可实现多线程。
- `Callable`是一个泛型接口只有一个`call()`方法，返回的类型就是创建`Callable`传进来的V类型。

``` java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

> `Callable`一般是和`ExecutorService`配合来使用的，在`ExecutorService`接口中声明了若干个`submit`方法的重载版本


#### 2.1 Future & FutureTask
`Future`就是对于具体的`Runnable`或者`Callable`任务的**执行结果**进行取消、查询是否完成、获取结果。必要时可以通过`get`方法获取执行结果，该方法会阻塞直到任务返回结果。
- 也就是说`Future`提供了三种功能：
    1. 判断任务是否完成；
    2. 能够中断任务；
    3. 能够获取任务执行结果。
- 在`Future`接口中声明了5个方法：**`cancel`**、**`isCancelled`**、**`isDone`**、**`get`**
    + `boolean` **`cancel(boolean mayInterruptIfRunning)`**;//用来取消任务，参数`mayInterruptIfRunning`表示是否允许取消正在执行却没有执行完毕的任务。
        * 如果取消已经完成的任务会返回`false`；如果任务还没有执行会返回`true`；
        * 如果任务正在执行，则返回`mayInterruptIfRunning`设置的值(`true/false`)；
    + `boolean` **`isCancelled()`**;//任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。
    + `boolean` **`isDone()`**;//任务是否已经完成，若任务完成，则返回true；
    + `V` **`get()`**;//获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
    + `V` **`get(long timeout, TimeUnit unit)`**;//获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

> `Future`可以得到别的线程任务方法的返回值。`Future`是一个接口,引用对象指向的实际是**FutureTask**。



### 3. FutureTask
**`FutureTask`**的父类是`RunnableFuture`，而`RunnableFuture`继承了`Runnbale`和`Futrue`这两个接口
- 从`FutureTask`构造方法可以了解到：
    1. `FutureTask`最终都是执行`Callable`类型的任务。
    2. 如果构造函数参数是`Runnable`，会被`Executors.callable`方法转换为`Callable`类型。
    3. `Executors.callable`方法直接返回一个`RunnableAdapter`实例。
    4. `RunnableAdapter`是`FutureTask`的一个静态内部类并且实现了`Callable`，也就是说`RunnableAdapter`是`Callable`子类。
    5. `RunnableAdapter`的`call`方法实现代码是，执行`Runnable`的`run`方法，并返回构造`FutureTask`传入`result`参数。
- `FutureTask`总结：
    + `FutureTask`实现了两个接口，`Runnable`和`Future`，所以它既可以作为`Runnable`被线程执行，又可以作为`Future`得到`Callable`的返回值，这个组合的好处：假设有一个很费时逻辑需要计算并且返回这个值，同时这个值不是马上需要，那么就可以使用这个组合，用另一个线程去计算返回值，而当前线程在使用这个返回值之前可以做其它的操作，等到需要这个返回值时，再通过`Future`得到！

> 注意：
- 通过`Executor`执行线程任务都是以`Callable`形式，如果传入`Runnable`都会转化为`Callable`。
- 通过`new Thread(runnable)`，只能是`Runnable`子类形式。


### 4. Fork/Join
从`JDK1.7`开始，Java提供`Fork/Join`框架用于并行执行任务，它的思想就是讲一个大任务分割成若干小任务，最终汇总每个小任务的结果得到这个大任务的结果。
- 主要有两步：任务切分 -> 结果合并
    1. 第一步**`分割任务`**。首先我们需要有一个 `fork` 类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。
    2. 第二步执行任务并**`合并结果`**。分割的子任务分别放在**双端队列**里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。
- 工作窃取算法（`work-stealing`）是指某个线程从其他队列里窃取任务来执行。
- `Fork/Join` 使用两个类来完成以上两个步骤：
    1. **`ForkJoinTask`**：我们要使用 `ForkJoin` 框架，必须首先创建一个 `ForkJoin` 任务。它提供在任务中执行 `fork()` 和 `join()` 操作的机制，通常情况下我们不需要直接继承 `ForkJoinTask` 类，而只需要继承它的子类，`Fork/Join` 框架提供了以下两个子类：
        + `RecursiveAction`：用于没有返回结果的任务。
        + `RecursiveTask`：用于有返回结果的任务。
    2. **`ForkJoinPool`**：`ForkJoinTask` 需要通过 `ForkJoinPool` 来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。


### 5. 线程池
线程池可以看作是一个资源集，任何池的作用都大同小异，主要是用来减少资源创建、初始化的系统开销。
- 一个线程池包括以下四个基本组成部分：
    1. 线程池管理器（`ThreadPool`）：用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；
    2. 工作线程（`PoolWorker`）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；
    3. 任务接口（`Task`）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；
    4. 任务队列（`taskQueue`）：用于存放没有处理的任务。提供一种缓冲机制。

```
Executor接口
    |———— ExecutorService接口
        |———— AbstractExecutorService抽象类
            |———— ForkJoinPool类
            |———— ThreadPoolExecutor类
        |———— ScheduledExecutorService接口
            |———— ScheduledThreadPoolExecutor类
Executors类
```

#### 5.1 通过Executors工厂类中的六个静态方法创建线程池
六大静态方法创建的`ThreadPoolExecutor`对象，返回的父接口的引用，即返回的`ExecutorService`的引用。六大静态方法内部都是直接或间接调用`ThreadPoolExecutor`类的构造方法创建线程池对象。
1. `newCachedThreadPool(ThreadPoolExecutor)`：创建一个可缓存的线程池
    + 如果线程池的大小超过了处理任务所需要的线程,那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）,`maximumPoolSize`最大可以至(`Integer.MAX_VALUE`),若达到该上限,直接OOM。
2. `newFixedThreadPool(ThreadPoolExecutor)`：创建固定大小的线程池。
    + 每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
3. `newSingleThreadExecutor(ThreadPoolExecutor)`：创建一个单线程的线程池。
    + 这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务,保证按任务的提交顺序依次执行。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
4. `newScheduledThreadPool(ScheduledThreadPoolExecutor)`：创建一个支持定时及周期性任务执行的线程池。
    + 线程数最大至`Integer.MAX_ VALUE`,存在OOM风险,不回收工作线程.
5. `newSingleThreadScheduledExecutor(ScheduledThreadPoolExecutor)`：创建一个单线程用于定时以及周期性执行任务的需求。
6. `newWorkStealingPool(ForkJoinPool)`：创建一个工作窃取
    + JDK8 引入,创建持有足够线程的线程池支持给定的并行度;并通过使用多个队列减少竞争;

> Executors返回的线程池对象的弊端：
> 1. `FixedThreadPool`和`SingleThreadExecutor`：
>    + 允许的**请求队列长度**为`Integer.MAX_VALUE`，可能会**堆积大量的请求**，从而导致OOM。
> 2. `CachedThreadPool`：
>    + 允许的**创建线程数量**为`Integer.MAX_VALUE`，可能会**创建大量的线程**，从而导致OOM。


#### 5.2 通过`ThreadPoolExecutor`构造方法创建线程池
``` java
public ThreadPoolExecutor(int corePoolSize,   //核心线程数，包括空闲线程
                          int maximumPoolSize,//最大线程数
                          long keepAliveTime, //线程空闲时间
                          TimeUnit unit,      //时间单位
                          BlockingQueue<Runnable> workQueue,//缓存队列
                          ThreadFactory threadFactory,      //线程工厂
                          RejectedExecutionHandler handler  //拒绝策略
) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ? 
        null : AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

##### 5.2.1 corePoolSize(核心线程数量)
- `corePoolSize`的设置非常关键：
    * `=0`：则任务执行完之后,没有任何请求进入时销毁线程池的线程
    * `>0`：即使本地任务执行完毕,核心线程也不会被销毁
    * 设置过大会浪费资源; 设置过小会导致线程频繁地创建或销毁
- 若设置了`allowCoreThreadTimeOut`这个参数,当提交一个任务到线程池时,若`线程数量(包括空闲线程)小于corePoolSize`,线程池会创建一个新线程放入`works(一个HashSet)`中执行任务,等到需要执行的任务数大于线程池基本大小时就不再创建,会尝试放入等待队列`workQueue`；如果调用线程池的`prestartAllCoreThreads()`,线程池会提前创建并启动所有核心线程

##### 5.2.2 maximumPoolSize（线程池最大线程数）
- `maximumPoolSize`表示线程池能够容纳同时执行的最大线程数,必须>=1.
- 若队列满,并且已创建的线程数小于最大线程数,则线程池会再创建新的线程放入`works`中执行任务,`CashedThreadPool`的关键,固定线程数的线程池无效
- 如果`maximumPoolSize = corePoolSize`,即是固定大小线程池.
- 若使用了无界任务队列,这个参数就没什么效果

##### 5.2.3 keepAliveTime（线程池中的线程空闲时间）
- 线程没有任务执行时最多保持多久时间终止（线程池的工作线程空闲后，保持存活的时间)
- 如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率
- 当空闲时间达到`keepAliveTime`时,线程会被销毁,直到只剩下`corePoolSize`个线程;避免浪费内存和句柄资源.
- 在默认情况下,当线程池的线程数大于`corePoolSize`时,`keepAliveTime`才起作用.
- 但是当`ThreadPoolExecutor`的`allowCoreThreadTimeOut=true`时,核心线程超时后也会被回收.

##### 5.2.4 TimeUnit（时间单位）
- keepAliveTime的时间单位通常是`TimeUnit.SECONDS`
- 可选的单位：天(`DAYS`)、小时(`HOURS`)、分钟(`MINUTES`)、毫秒(`MILLISECONDS`)、微秒(`MICROSECONDS`，千分之一毫秒) 和 纳秒(`NANOSECONDS`，千分之一微秒)

##### 5.2.5 workQueue（缓存队列）
- 存储待执行任务的阻塞队列，这些任务必须是`Runnable`的对象（如果是`Callable`对象，会在`submit`内部转换为`Runnable`对象） 
- 当请求的线程数大于`maximumPoolSize`时,线程进入`BlockingQueue`.
- 可以选择以下几个阻塞队列:
    + `LinkedBlockingQueue`:一个基于链表结构的阻塞队列,此队列按`FIFO`排序元素,吞吐量通常要高于`ArrayBlockingQueue`.静态工厂方法`Executors.newFixedThreadPool()`使用了这个队列
    + `SynchronousQueue`:一个不存储元素的阻塞队列.每个插入操作必须等到另一个线程调用移除操作,否则插入操作一直处于阻塞状态,吞吐量通常要高于`LinkedBlockingQueue`,静态工厂方法`Executors.newCachedThreadPoo`l使用了这个队列

##### 5.2.6 threadFactory （线程工厂）
- 用于设置创建线程的工厂;
- 线程池的命名是通过增加组名前缀来实现的，可以通过线程工厂给每个创建出来的线程设置更有意义的名字
- 在虚拟机栈分析时,就可以知道线程任务是由哪个线程工厂产生的.

##### 5.2.7 RejectedExecutionHandler（拒绝策略）
- 当队列和线程池都满,说明线程池饱和,必须采取一种策略处理提交的新任务；策略默认**`AbortPolicy`**,表无法处理新任务时抛出异常
- 当超过参数`workQueue`的任务缓存区上限的时候,就可以通过该策略处理请求,这是一种简单的限流保护.
- 友好的拒绝策略可以是如下三种:
    1. 保存到数据库进行削峰填谷;在空闲时再提取出来执行
    2. 转向某个提示页面
    3. 打印日志
- `AbortPolicy`：丢弃任务，抛出`RejectedExecutionException`
- `CallerRunsPolicy`：只用调用者所在线程来运行任务,有反馈机制，使任务提交的速度变慢）。
- `DiscardOldestPolicy`：若没有发生shutdown,尝试丢弃队列里最近的一个任务,并执行当前任务, 丢弃任务缓存队列中最老的任务，并且尝试重新提交新的任务
- `DiscardPolicy`:不处理,丢弃掉, 拒绝执行，不抛异常 
- 当然,也可以根据应用场景需要来实现`RejectedExecutionHandler`接口自定义策略.如记录日志或持久化存储不能处理的任务


#### 5.3 自定义一个ThreadPoolExecutor线程池
``` java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    5, //核心线程数
    Runtime.getRuntime().availableProcessors() * 2,//最大线程数
    60,//线程空闲时间
    TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(200),
    new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setName("order-thread");//设置有意义的线程名字
            if(t.isDaemon()) {//若是守护线程将其释放
                t.setDaemon(false);
            }
            if(Thread.NORM_PRIORITY != t.getPriority()) {
                //恢复线程优先级
                t.setPriority(Thread.NORM_PRIORITY);
            }
            return t;
        }
    },
    new RejectedExecutionHandler() {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            System.err.println("拒绝策略:" + r);
        }
    }
 );
```

#### 5.3.1 线程池执行流程
- 要求 线程池有上限，使用有限队列
1. 当线程池核心线程数量用完，先扔进队列
2. 队列也用完后，看最大线程数量
3. 最大线程数量用完后，走拒绝策略
4. 拒绝策略可以打印一些日志，做一些补偿
5. 线程池用完一定要优雅的关闭

> 线程池要统一管理，不要用Executors工厂类，要用ThreadPoolExecutor自定义线程池

#### 5.3.2 线程池配置-核心线程数量
线程CPU时间所占比例越高，需要越少线程(CPU密集)。线程等待时间所占比例越高，需要越多线程(IO密集)。
1. **CPU密集型**：内存运算、不涉及IO操作等
    + 设置线程数为：`CPU核数+1` 
2. **IO密集型**：数据读取、存取、数据库操作、持久化操作等
    + 最佳线程数目：`CPU核数/(1-阻塞系数)` 这个阻塞系数一般为0.8~0.9之间，也可以取0.8或者0.9。

> java.lang.`Runtime.availableProcessors()` 方法返回到Java虚拟机的可用的处理器数量(CPU核数)。此值可能会改变在一个特定的虚拟机调用。应用程序可用处理器的数量是敏感的，因此偶尔查询该属性，并适当地调整自己的资源使用情况.