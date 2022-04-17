---
title: 「Spring Reactive Stack」Reactor异常处理
date: 2021-03-02 20:12:42
tags: [后端开发, Spring, SpringReactive, Reactor]
categories: SpringReactive
---

不管是在响应式编程还是普通的程序设计中，异常处理都是一个非常重要的方面。

对于`Flux`或者`Mono`来说，所有的异常都是一个终止的操作，即使你使用了异常处理，原生成序列也不会继续。
但是如果你对异常进行了处理，那么它会将`oneError`信号转换成为新的序列的开始，并将替换掉之前上游产生的序列。<!-- more -->

先看一个`Flux`产生异常的例子
``` java
@Test
void test1() {
    Flux.just(10, 5, 0)
        .map(i -> "100 / " + i + " = " + (10 / i))
        .subscribe(System.out::println);
}
```
会得到一个异常`ErrorCallbackNotImplemented`：
``` bash
100 / 10 = 1
100 / 5 = 2
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.ArithmeticException: / by zero
Caused by: java.lang.ArithmeticException: / by zero
```


### 1. onError方法
`Reactor`中`subscribe`的`onError`方法(subscribe第二个参数)，就是`try catch`的一个具体应用：
``` java
@Test
void test2() {
    Flux.just(10, 5, 0)
        .map(i -> "100 / " + i + " = " + (10 / i))
        .subscribe(System.out::println,
            error -> System.err.println("Error: " + error));
}
```
运行结果：
``` bash
100 / 10 = 1
100 / 5 = 2
Error: java.lang.ArithmeticException: / by zero
```


### 2. onErrorReturn方法
`onErrorReturn`可以在遇到异常的时候`fallback`到一个静态的默认值：
``` java
@Test
void test3() {
    Flux.just(10, 5, 0)
        .map(i -> "100 / " + i + " = " + (10 / i))
        .onErrorReturn("Divided by zero :(")
        .subscribe(System.out::println);
}
```
运行结果：
``` bash
100 / 10 = 1
100 / 5 = 2
Divided by zero :(
```

`onErrorReturn`还支持一个`Predicate`参数，用来判断要`falback`的异常是否满足条件。
``` java
public final Flux<T> onErrorReturn(Predicate<? super Throwable> predicate, T fallbackValue) 
```


### 3. onErrorResume方法
`onErrorResume`可以在捕获异常之后调用其他的方法。
``` java
@Test
void test4() {
    Flux.just(10, 5, 0)
        .map(i -> "100 / " + i + " = " + (10 / i))
        .onErrorResume(e -> System.out::println)
        .subscribe(System.out::println);
}
```
运行结果：
``` bash
100 / 10 = 1
100 / 5 = 2
reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber@23469199
```

### 4. retry方法
`retry`的作用就是当遇到异常的时候，重启一个新的序列
``` java
@Test
void test8() {
    Flux.just(10, 5, 0)
        .map(i -> "100 / " + i + " = " + (10 / i))
        .retry(1)
        .elapsed()
        .subscribe(System.out::println);
}
```
运行结果：
``` bash
[4,100 / 10 = 1]
[0,100 / 5 = 2]
[9,100 / 10 = 1]
[0,100 / 5 = 2]
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.ArithmeticException: / by zero
Caused by: java.lang.ArithmeticException: / by zero
```

> `elapsed`是用来展示产生的value时间之间的duration。从结果我们可以看到，`retry`之前是不会产生异常信息的。


### 5. doOnError方法
`doOnError`只记录异常信息，不破坏原来的`React`结构。
``` java
@Test
void test5() {
    Flux.just(10, 5, 0)
        .map(i -> "100 / " + i + " = " + (10 / i))
        .doOnError(error -> System.out.println("we got the error: "+ error))
        .subscribe(System.out::println);
}
```
运行结果：
``` bash
100 / 10 = 1
100 / 5 = 2
we got the error: java.lang.ArithmeticException: / by zero
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.ArithmeticException: / by zero
Caused by: java.lang.ArithmeticException: / by zero
```

> `doOn`系列方法是`publisher`的同步钩子方法，在`subscriber`触发一系列事件的时候触发


### 6. doFinally方法
`doFinally`可以像传统的同步代码那样使用`finally`去做一些事情，比如关闭`http`连接，清理资源等。
``` java
@Test
void test6() {
    Flux.just(10, 5, 0)
        .map(i -> "100 / " + i + " = " + (10 / i))
        .doFinally(error -> System.out.println("Finally，I will make sure to do something:"+error))
        .subscribe(System.out::println);
}
```
运行结果：
``` bash
100 / 10 = 1
100 / 5 = 2
reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.ArithmeticException: / by zero
Caused by: java.lang.ArithmeticException: / by zero
	at ...
Finally，我会确保做一些事情:onError
```

第二种收尾操作的方法是`using`，我们先看一个`using`的定义：
``` java
public static <T, D> Flux<T> using(Callable<? extends D> resourceSupplier, Function<? super D, ? extends
    Publisher<? extends T>> sourceSupplier, Consumer<? super D> resourceCleanup)
```
可以看到`using`支持三个参数，`resourceSupplier`是一个生成器，用来在`subscribe`的时候生成要发送的`resource`对象。
`sourceSupplier`是一个生成`Publisher`的工厂，接收`resourceSupplier`传过来的`resource`，然后生成`Publisher`对象。
`resourceCleanup`用来对`resource`进行收尾操作。
``` java
@Test
void test7() {
    AtomicBoolean isDisposed = new AtomicBoolean();
    Disposable disposableInstance = new Disposable() {
        @Override
        public void dispose() {
            isDisposed.set(true);
        }
        @Override
        public String toString() {
            return "DISPOSABLE";
        }
    };
    Flux.using(
        () -> disposableInstance,
        disposable -> Flux.just(disposable.toString()),
        Disposable::dispose)
        .subscribe(System.out::println);
}
```
