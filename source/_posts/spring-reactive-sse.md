---
title: 「Spring Reactive Stack」服务端事件推送Server-Sent Events
date: 2021-03-12 17:01:42
tags: [后端开发, Spring, SpringReactive, SSE]
categories: SpringReactive
---

**SSE**：`Server-Sent Events`服务器推送事件，是一种仅发送文本消息的技术。`SSE`基于`HTTP`协议中的持久连接。`SSE`是`HTML5`标准协议中的一部分。

客户端接收服务端**异步更新**的消息可以分为两类：客户端拉取和服务端推送。<!-- more -->

客户端拉取：通过短轮询或者长轮询定期请求服务器进行更新。

服务端推送：`SSE`和`WebSocket`，`SSE`是单向，`WebSocket`是双向；`SSE`基于`HTTP`协议，`WebSocket`基于`WebSocket`协议(`HTTP`以外的协议);


### SSE网络协议
* 基于纯文本的简单协议。服务器端的响应内容类型必须是`text/event-stream`。响应文本的内容是一个事件流，事件流是一个简单的文本流，仅支持`UTF-8`格式的编码。
* 事件流由不同的事件组成。不同事件间通过仅包含回车符和换行符的空行（`\r\n`）来分隔。
* 每个事件可以由多行构成，每行由类型和数据两部分组成。类型与数据通过冒号（`:`）进行分隔，冒号前的为类型，冒号后的为其对应的值。每个事件可以包含如下类型的行：
    - 类型为空白，表示该行是注释，会在处理时被忽略。
    - 类型为`data`，表示该行是事件所包含的数据。以`data`开头的行可以出现多次。所有这些行都是该事件的数据。
    - 类型为`event`，表示该行用来声明事件的类型，即事件名称。浏览器在收到数据时，会产生对应名称的事件。
    - 类型为`id`，表示该行用来声明事件的标识符。
    - 类型为`retry`，表示该行用来声明浏览器在连接断开之后进行重连的等待时间。

``` bash
data: china // 该事件仅包含数据

data: Beijing // 该事件包含数据与事件标识
id: 100

event: myevent // 该事件指定了名称
data:shanghai
id: 101

: this is a comment // 该事件具有注释、名称，且包含两行数据
event:city
data: guangzhou
data: shenzhen
```

* 事件标识`id`作用: 如果服务端发送的事件中包含事件标识`id`，那么浏览器会将最近一次接收到的事件标识`id`记录到`HTTP`头的`Last-Event-ID`属性中。如果浏览器与服务端的连接中断，当浏览器再次连接时，会将`Last-Event-ID`记录的事件标识`id`发送给服务端。服务器端通过浏览器端发送的事件标识`id`来确定将继续连接哪个事件。


订阅一个服务端推送事件(`GET`请求)，需要设置 包含如下请求头的`Request`：
``` bash
Accept: text/event-stream # 指明MediaType是事件流
Cache-Control: no-cache   # 不要对事件进行缓存
Connection: keep-alive    # 长连接
```
服务端需要提供 包含以下响应头的Response：
``` bash
Content-Type: text/event-stream;charset=UTF-8  # 告诉客户端响应是一个事件流
Transfer-Encoding: chunked                     # 告诉客户端内容大小未知，为流传输
```

### 在WebFlux中实现发送事件
首先，在`pom.xml`文件中，引入`webflux`；
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```
创建一个`controller`类并用`@RestController`注解标记；
创建一个接受`Http GET`请求的方法，该方法返回一个`Flux`对象，并配置`produces=text/event-stream`；
``` java
@GetMapping(value = "/test/sse", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
private Flux<String> flux() {
    return Flux.interval(Duration.ofMillis(1000)).map(i -> " -> " +i);
}
```
浏览器访问`/test/sse`就会看到每一秒推送一个数据：
``` bash
data: -> 0
data: -> 1
data: -> 2
data: -> 3
...
```

* **一次性事件**：短暂的去发送事件是比较简单的，只需要使用`Flux.just()`将消息列表里的消息一条条发送出去即可。
* **周期性事件**：长期的发送事件在发送本身上是没有区别，主要是需要一个周期性线程定期处理发送事务，这里直接使用`Flux.inteval()`来轮询。
* **非周期性事件**：可以通过`Spring`的事件监听接口来实现，关键点在于要把监听消息的处理器和Flux的构造结合起来。


### SSE的注意事项
1. `SSE`只适合发送文本消息；尽管可以使用`Base64`编码和`gzip`压缩来发送二进制消息，但效率可能很低。
2. 早期的一些浏览器，如`Internet Explorer`不支持。
3. `Internet Explorer/Edge`和许多移动浏览器不支持`SSE`；尽管可以使用`polyfills`，但它们可能效率低下
4. 在系统设计时，同一个页面最好只维持1个`SSE`连接，通过事件来区分。因为浏览器对同时并发的连接数有限制，一般最大是6个。


### 前端接收Server-Sent事件通知
`JavaScript`里用**EventSource**对象来接收服务器发送事件通知：
``` javascript
if (typeof(EventSource)!=="undefined") {
    var source=new EventSource("http://localhost:8080/test/sse");
    source.onmessage=function(event) {
        alert(event.data);
    };
} else {
    alert("抱歉，你的浏览器不支持 server-sent 事件...");
}
```
`EventSource`是服务器推送的一个网络事件接口。一个`EventSource`实例会对`HTTP`服务开启一个持久化的连接，以`text/event-stream`格式发送事件, 会一直保持开启直到被要求关闭。
* `EventSource`属性
    + `EventSource.onerror`：EventHandler，当发生错误时被调用，并且在此对象上派发`error`事件。
    + `EventSource.onmessage`：EventHandler，当收到一个`message`事件，当接收到消息时被调用。
    + `EventSource.onopen`：EventHandler，当收到一个`open`事件，当连接刚打开时被调用。
    + `EventSource.readyState`(只读)：unsigned short值，代表连接状态。可能值是CONNECTING(0), OPEN(1), 或者CLOSED(2)。
    + `EventSource.url`(只读)：一个DOMString，代表事件源的URL。

