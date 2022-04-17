---
title: 「Spring Reactive Stack」Spring WebFlux响应式Web框架入门
date: 2021-02-25 15:50:42
tags: [后端开发, Spring, SpringReactive, WebFlux]
categories: SpringReactive
---

![](up-ee8803006ea5a4169467074917d6fdd0b12.webp)

`Spring WebFlux`是`Spring Framework 5.0`中引入的以`Reactor`为基础的响应式编程`Web`框架。

**WebFlux** 的异步处理是基于`Reactor`实现的，是将输入流适配成`Mono`或`Flux`进行统一处理。<!-- more -->

### 1. 响应式流(Reactive Streams)
+ **Reactor** 是一个响应式流，它有对应的发布者(`Publisher`)，用两个类来表示：
    * `Flux`(返回0-n个元素)
    * `Mono`(返回0或1个元素)
+ **Reactor** 的订阅者(`Subscriber`)则是由`Spring`框架去完成。

+ **响应式流(Reactive Streams)** 其实就是一个规范，其特点：
    * 无阻塞；
    * 一个数据流；
    * 可以异步执行；
    * 能够处理背压；

+ **背压(Backpressure)** 可以简单理解为 消费决定生产，生产者可以根据消费压力进行动态调节生产速率的机制。


### 2. 发布者(Publisher)
由于响应流的特点，我们不能再返回一个简单的`POJO`对象来表示结果了。必须返回一个类似`Java`中的`Future`的概念，在有结果可用时通知消费者进行消费响应。

`Reactive Stream`规范中这种被定义为`Publisher` ，`Publisher`是一个可以提供`0-N`个序列元素的提供者，并根据其订阅者`Subscriber`的需求推送元素。一个`Publisher`可以支持多个订阅者，并可以根据订阅者的逻辑进行推送序列元素。

可以通过下图Excel来理解，`1-9`行可以看作发布者`Publisher`提供的元素序列，`10-13`行的结果计算看作订阅者`Subscriber`。

![](publisher.gif)

响应式的一个重要特点：当没有订阅时发布者(`Publisher`)什么也不做。

而`Flux`和`Mono`都是`Publisher`在`Reactor3`实现。`Publisher`提供了`subscribe`方法，允许消费者在有结果可用时进行消费。如果没有消费者`Publisher`不会做任何事情，他根据消费情况进行响应。`Publisher`可能返回零或者多个，甚至可能是无限的，为了更加清晰表示期待的结果就引入了两个实现模型`Mono`和`Flux`。

在`WebFlux`中，你的方法只需返回`Mono或Flux`即可。你的代码基本也只和`Mono`或`Flux`打交道。而`WebFlux`则会实现`Subscriber` ，`onNext`时将业务开发人员编写的`Mono`或`Flux`转换为`HTTP Response`返回给客户端。

### 3. Mono和Flux的抽象模型
Mono和Flux都是Publisher(发布者)的实现模型。

#### 3.1 Flux
`Flux`是一个发出(`emit`)`0-N`个元素组成的异步序列的`Publisher<T>`,可以被`onComplete`信号或者`onError`信号所终止。
在响应流规范中存在三种给下游消费者调用的方法 `onNext`, `onComplete`, 和`onError`。
下面这张图表示了`Flux`的抽象模型：

![](flux.jpg)


#### 3.2 Mono
`Mono`是一个发出(`emit`)`0-1`个元素的`Publisher<T>`,可以被`onComplete`信号或者`onError`信号所终止。
下面这张图表示了`Mono`的抽象模型(整体和`Flux`差不多,只不过这里只会发出`0-1`个元素)：

![](mono.jpg)


### 4. Mono API
`Mono`和`Flux`都是实现`org.reactivestreams.Publisher`接口的抽象类。

`Mono`代表`0-1`个元素的发布者(`Publisher`)。

+ `Mono`里面有很多`API`：
    - **just()**：可以指定序列中包含的全部元素。创建出来的 `Mono`序列在发布这些元素之后会自动结束。
    - **empty()**：创建一个不包含任何元素，只发布结束消息的序列。
    - **justOrEmpty(Optional<? extends T> data)**：从一个`Optional`对象或可能为`null`的对象中创建`Mono`。只有`Optional`对象中包含值或对象不为`null`时，`Mono`序列才产生对应的元素。
    - **error(Throwable error)**：创建一个只包含错误消息的序列。
    - **never()**：创建一个不包含任何消息通知的序列。
    - **delay(Duration duration)**和**delayMillis(long duration)**：创建一个`Mono`序列，在指定的延迟时间之后，产生数字 `0` 作为唯一值。
    - **fromCallable()**、**fromCompletionStage()**、**fromFuture()**、**fromRunnable()**和**fromSupplier()**：分别从 `Callable`、`CompletionStage`、`CompletableFuture`、`Runnable`和`Supplier`中创建`Mono`。
    - **ignoreElements(Publisher source)**：创建一个`Mono`序列，忽略作为源的`Publisher`中的所有元素，只产生结束消息。
    - **create()**：通过`create()`方法来使用`MonoSink`来创建`Mono`。
+ `API`使用案例如下所示。

``` java
@Slf4j
@SpringBootTest
public class MonoTest {
    @Test
    public void mono() {
        // 通过just直接赋值
        Mono.just("my name is charles").subscribe(log::info);
        // empty 创建空mono
        Mono.empty().subscribe();
        // ustOrEmpty 只有 Optional 对象中包含值或对象不为 null 时，Mono 序列才产生对应的元素。
        Mono.justOrEmpty(null).subscribe(System.out::println);
        Mono.justOrEmpty("测试justOrEmpty").subscribe(System.out::println);
        Mono.justOrEmpty(Optional.of("测试justOrEmpty")).subscribe(System.out::println);
        // error 创建一个只包含错误消息的序列。
        Mono.error(new RuntimeException("error")).subscribe(System.out::println, System.err::println);
        // never 创建一个不包含任何消息通知的序列。
        Mono.never().subscribe(System.out::println);
        // 延迟生成0
        Mono.delay(Duration.ofMillis(2)).map(String::valueOf).subscribe(log::info);
        // 通过fromRunnable创建，并实现异常处理
        Mono.fromRunnable(() -> {
            System.out.println("thread run");
            throw new RuntimeException("thread run error");
        }).subscribe(System.out::println, System.err::println);
        // 通过Callable
        Mono.fromCallable(() -> "callback function").subscribe(log::info);
        // future
        Mono.fromFuture(CompletableFuture.completedFuture("from future")).subscribe(log::info);
        // 通过runnable
        Mono<Void> runnableMono = Mono.fromRunnable(() -> log.warn(Thread.currentThread().getName()));
        runnableMono.subscribe();
        // 通过使用 Supplier
        Mono.fromSupplier(() -> new Date().toString()).subscribe(log::info);
        // flux中
        Mono.from(Flux.just("from", "flux")).subscribe(log::info);  // 只返回flux第一个
        //通过 create()方法来使用 MonoSink 来创建 Mono。
        Mono.create(sink -> sink.success("测试create")).subscribe(System.out::println);
    }
}
```

运行结果：

![](monotest.jpg)


### 5. Flux API
`Mono`和`Flux`都是实现`org.reactivestreams.Publisher`接口的抽象类。

`Flux`表示连续序列，和`Mono`的创建方法有些不同，`Mono`是`Flux`的简化版，`Flux`可以用来表示流。

+ `Flux API`：
    - **just()**：可以指定序列中包含的全部元素。
    - **range()**：可以用来创建连续数值。
    - **empty()**：创建一个不包含任何元素。
    - **error(Throwable error)**：创建一个只包含错误消息的序列。
    - **fromIterable()**：通过迭代器创建如list，set
    - **fromStream()**：通过流创建
    - **fromArray(T[])**：通过列表创建 如 String[], Integer[]
    - **merge()**：通过将两个flux合并得到新的flux
    - **interval()**：每隔一段时间生成一个数字，从1开始递增
+ `API`使用案例如下所示。

``` java
@Slf4j
@SpringBootTest
public class FluxTest {
    @Test
    public void flux () throws InterruptedException {
        // 通过just赋值
        Flux<Integer> intFlux = Flux.just(1, 2, 3, 4, 5);
        // 以6开始，取4个值：6,7,8,9
        Flux<Integer> rangeFlux = Flux.range(6, 4);
        // 通过merge合并
        Flux<Integer> intMerge = Flux.merge(intFlux, rangeFlux);
        intMerge.subscribe(System.out::print);
        System.out.println();//换行
        // 通过fromArray构建
        Flux.fromArray(new Integer[]{1,3,5,7,9}).subscribe(System.out::print);
        System.out.println();//换行
        // 通过流和迭代器创建
        Flux<String> strFluxFromStream = Flux.fromStream(Stream.of(" just", " test", " reactor", " Flux", " and", " Mono"));
        Flux<String> strFluxFromList = Flux.fromIterable(Arrays.asList(" just", " test", " reactor", " Flux", " and", " Mono"));
        // 通过merge合并
        Flux<String> strMerge = Flux.merge(strFluxFromStream, strFluxFromList);
        strMerge.subscribe(System.out::print);
        System.out.println();
        // 通过interval创建流数据
        Flux.interval(Duration.ofMillis(100)).map(String::valueOf)
            .subscribe(System.out::print);
        Thread.sleep(2000);
    }
}
```

运行结果：
``` bash
123456789
13579
 just test reactor Flux and Mono just test reactor Flux and Mono
012345678910111213141516171819
```


### 6. subscribe方法
subscribe()方法表示对数据流的订阅动作，subscribe()方法有多个重载的方法，最多可以传入四个参数；
``` java
    @Test
    public void subscribe () throws InterruptedException {
        // 测试Mono
        Mono.just(1).subscribe(System.out::println);
        // 测试Flux
        Flux.just('a', 'b').subscribe(System.out::println);
        // 测试2个参数的subscribe方法
        Flux.just('i', 'j').map(chr -> {
                if ('j'== chr) throw new RuntimeException("test 2 parameters");
                else return String.valueOf(chr);
            })
            .subscribe(System.out::println,    // 参数1,接受内容
                       err -> log.error(err.getMessage())); // 参数2,对err处理的lambda函数
        // 测试3个参数的subscribe方法
        Flux.just("你", "我", "他", "它", "ta")
            .subscribe(System.out::print,   // 参数1,接受内容
                       System.err::println, // 参数2,对err处理的lambda函数
                       () -> System.out.println("complete for 3"));// 参数3,完成subscribe之后执行的lambda函数
        // 测试4个参数的subscribe方法
        Flux.interval(Duration.ofMillis(100))
            .map(i -> {
                if (i == 3) throw new RuntimeException("fake a mistake");
                else return String.valueOf(i);
            })
            .subscribe(info -> log.info("info: {}", info), // 参数1,接受内容
                       err -> log.error("error: {}", err.getMessage()),// 参数2,对err处理的lambda函数
                       () -> log.info("Done"),     // 参数3,完成subscribe之后执行的lambda函数
                       sub -> sub.request(10));  // 参数4,Subscription操作,设定从源头获取元素的个数
        Thread.sleep(2000);
    }
```

运行结果：

![](fluxtest.jpg)


### 7. 使用StepVerifier测试响应式异步代码
通过expectNext执行类似断言的功能，如果断言不符合实际情况，就会报错。
``` java
@Test
public void StepVerifier () {
    // 使用StepVerifier测试Flux，正常
    Flux flux = Flux.just(1, 2, 3, 4, 5, 6);
    StepVerifier.create(flux)
                // 测试下一个期望的数据元素
                .expectNext(1, 2, 3, 4, 5, 6)
                // 测试下一个元素是否为完成信号
                .expectComplete()
                .verify();
    // 使用StepVerifier测试Mono，报错
    Mono<String> mono = Mono.just("charles").log();
    StepVerifier.create(mono)
                .expectNext("char")
                .verifyComplete();
}
```

运行结果：
``` bash
java.lang.AssertionError: expectation "expectNext(char)" failed (expected value: char; actual value: charles)
...
```

> 参考连接：`https://mp.weixin.qq.com/s/O1VGS7d1TLQhgrCaQ-UQCw`
