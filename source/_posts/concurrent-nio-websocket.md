---
title: 「并发编程」NIO、Netty及websocket实现
date: 2019-10-20 23:34:31
tags: [java, 后端开发, 并发编程]
categories: 并发编程
---


### 1. BIO/NIO/AIO演变
Java IO 方式有很多种，基于不同的IO抽象模型和交互方式，可以进行简单区分。

| IO类型 | 模型     |客户端:线程数|API使用难度|调试难度|可靠性|吞吐量|
|-------|----------|------------|----------|-------|-----|-----|
|BIO    |流，同步阻塞|1:1        |简单       |简单   |很差  |非常低|
|伪异步IO|同步阻塞   |M:N        |简单       |简单   |较差  |中等  |
|NIO    |同步非阻塞  |M:1        |复杂       |复杂   |较高  |高    |
|AIO    |异步非阻塞  |M:0,被动回调|复杂       |复杂   |高    |高    |

<!-- more -->

1. 区分同步(synchronous)或异步(asynchronous)
    + 同步是一种可靠的有序运行机制，当我们进行同步操作时，后续的任务是等待当前调用返回，才会进行下一步；
    + 异步则相反，其他任务不需要等待当前调用返回，通常依靠事件、回调等机制来实现任务间次序关系。
2. 区分阻塞(blocking)与非阻塞(non-blocking)
    + 在进行阻塞操作时，当前线程会处于阻塞状态，无法从事其他任务，只有当条件就绪才能继续，比如 ServerSocket 新连接建立完毕，或数据读取、写入操作完成；
    + 非阻塞则是不管 IO 操作是否结束，直接返回，相应操作在后台继续处理

+ 传统的java.io包，它基于流模型实现，**同步阻塞**的交互方式，如File抽象、输入输出流等。好处是代码简单、直观，缺点是IO效率和扩展性局限性
+ 很多时候，也把java.net下面提供的部分网络API，比如Socket、ServerSocket、HttpURLConnection也归类到同步阻塞IO类库，因为网络通信同样是IO行为。
+ 伪异步IO：后端通过维护一个消息队列和N个活跃线程, 通过一个**线程池**来处理多个客户端的请求接入，通过线程池，可以灵活地调配线程资源，设置线程的最大值，防止由于海量并发接入而导致的线程耗尽和宕机。
+ JDK4引入了NIO框架(java.nio)，提供了Channel、Selector、Buffer等新的抽象，可以构建**多路复用**的、**同步非阻塞**IO程序，同时提供了更接近操作系统底层的高性能数据操作方式。
+ JDK7中，NIO有了进一步的改进，引入了异步非阻塞IO方式，也叫AIO(Asynchronous IO)。异步IO操作基于事件和**回调机制**，可以简单理解为，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

#### 1.1 NIO的主要组成部分：
1. Buffer(缓冲区)，高效的数据容器，除了布尔类型，所有原始数据类型都有相应的Buffer实现。
    + Buffer最常见的类型是ByteBuffer，另外还有CharBuffer，ShortBuffer，IntBuffer，LongBuffer，FloatBuffer，DoubleBuffer。
2. Channel(通道)，是NIO中被用来支持批量式IO操作的一种抽象。
    + 和流不同，通道是双向的。数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。
3. Selector(多路复用器)，是NIO实现多路复用的基础，它允许单线程处理多个Channel。
    + Selector是基于底层操作系统机制，不同模式、不同版本都存在区别。
    + 要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。

#### 1.2 NIO多路复用的过程
1. 通过Selector.open()创建一个Selector，作为类似调度员的角色。
2. 创建一个ServerSocketChannel，并绑定监听端口，设置为非阻塞模式
3. 将Channel向Selector注册，通过指定SelectionKey.OP_ACCEPT，告诉调度员，它关注的是新的连接请求。
3. Selector循环阻塞在select操作，当有Channel发生接入请求，就会被唤醒。
4. 调用selectedKeys方法获取就绪channel集合
5. 通过SocketChannel和Buffer进行数据操作。

#### 1.3 AIO
- AIO也叫NIO2.0 是一种非阻塞异步的通信模式。在NIO的基础上引入了新的**异步通道**的概念，并提供了异步文件通道和异步套接字通道的实现。
    + 没有采用NIO的多路复用器，而是使用异步通道的概念。
    + 其read，write方法的返回类型都是Future对象。而Future模型是异步的，其核心思想是：去主函数等待时间。
    + AIO模型中通过AsynchronousSocketChannel和AsynchronousServerSocketChannel完成套接字通道的实现。非阻塞，异步。



### 2. Netty框架
Netty是一个高性能事件驱动，异步非阻塞的IO开源框架，由Jboss提供，用于建立Tcp等底层的链接，基于Netty可以建立高性能的Http服务器，快速开发高性能、高可靠的网络服务器和客户端程序。支持Http、websocket，tcp，udp等协议。
- Netty使用场景：高性能领域（游戏，大数据分布式计算等）、多线程并发领域（多路复用模型，多线程模型，主从多线程模型）、异步通信领域
- Netty 是一个吸收了多种协议（包括FTP、SMTP、HTTP等各种二进制文本协议）的实现经验，在保证易于开发的同时还保证了其应用的性能，稳定性和伸缩性。

#### 2.1 Netty的核心概念
1. ServerBootstrap，服务器端程序的入口，这是 Netty 为简化网络程序配置和关闭等生命周期管理，所引入的 Bootstrapping 机制。我们通常要做的创建 Channel、绑定端口、注册 Handler 等，都可以通过这个统一的入口，以**Fluent API**等形式完成，相对简化了 API 使用。与之相对应， Bootstrap则是 Client 端的通常入口。
2. Channel，作为一个基于 NIO 的扩展框架，Channel 和 Selector 等概念仍然是 Netty 的基础组件，但是针对应用开发具体需求，提供了相对易用的抽象。
3. EventLoop，这是 Netty 处理事件的核心机制。例子中使用了 EventLoopGroup。我们在 NIO 中通常要做的几件事情，如注册感兴趣的事件、调度相应的 Handler 等，都是 EventLoop 负责。
4. ChannelFuture，这是 Netty 实现异步 IO 的基础之一，保证了同一个 Channel 操作的调用顺序。Netty 扩展了 Java 标准的 Future，提供了针对自己场景的特有Future定义。
5. ChannelHandler，这是应用开发者**放置业务逻辑的主要地方**，也是我上面提到的“Separation Of Concerns”原则的体现。
6. ChannelPipeline，它是 ChannelHandler 链条的容器，每个 Channel 在创建后，自动被分配一个 ChannelPipeline。在上面的示例中，我们通过 ServerBootstrap 注册了 ChannelInitializer，并且实现了 initChannel 方法，而在该方法中则承担了向 ChannelPipleline 安装其他 Handler 的任务。

#### 2.2 对比 Java 标准 NIO 类库，Netty是如何实现更高性能的？
单独从性能角度，Netty 在基础的 NIO 等类库之上进行了很多改进，例如：
1. 更加优雅的 Reactor 模式实现、灵活的线程模型、利用 EventLoop 等创新性的机制，可以非常高效地管理成百上千的 Channel。
2. 充分利用了 Java 的 Zero-Copy 机制，并且从多种角度，“斤斤计较”般的降低内存分配和回收的开销。例如，使用池化的 Direct Buffer 等技术，在提高 IO 性能的同时，减少了对象的创建和销毁；利用反射等技术直接操纵 SelectionKey，使用数组而不是 Java 容器等。
3. 使用更多本地代码。例如，直接利用 JNI 调用 Open SSL 等方式，获得比 Java 内建 SSL 引擎更好的性能。
4. 在通信协议、序列化等其他角度的优化。

Netty 的设计强调了 “Separation Of Concerns”，通过精巧设计的事件机制，将业务逻辑和无关技术逻辑进行隔离，并通过各种方便的抽象，一定程度上填补了了基础平台和业务开发之间的鸿沟，更有利于在应用开发中普及业界的最佳实践。另外，Netty > java.nio + java. net！

除了核心的事件机制等，Netty 还额外提供了很多功能，例如：
1. 从网络协议的角度，Netty 除了支持传输层的 UDP、TCP、SCTP协议，也支持 HTTP(s)、WebSocket 等多种应用层协议，它并不是单一协议的 API。
2. 在应用中，需要将数据从 Java 对象转换成为各种应用协议的数据格式，或者进行反向的转换，Netty 为此提供了一系列扩展的编解码框架，与应用开发场景无缝衔接，并且性能良好。
3. 它扩展了 Java NIO Buffer，提供了自己的 ByteBuf 实现，并且深度支持 Direct Buffer 等技术，甚至 hack 了 Java 内部对 Direct Buffer 的分配和销毁等。同时，Netty 也提供了更加完善的 Scatter/Gather 机制实现。



### 3. 基于Netty搭建简单的Http服务
1. 环境准备：`jdk1.8`、`Netty4.1.43.Final`
2. 代码编写：`MyChannelInitializer.java`、`MyClientHandler.java`、`NettyServer.java`

> MyChannelInitializer.java：添加了Http的处理协议
``` java
public class MyChannelInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel channel) {
        // 数据解码操作
        channel.pipeline().addLast(new HttpResponseEncoder());
        // 数据编码操作
        channel.pipeline().addLast(new HttpRequestDecoder());
        // 在管道中添加我们自己的接收数据实现方法
        channel.pipeline().addLast(new MyServerHandler());
    }
}
```

> MyServerHandler.java
``` java
public class MyServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        if (msg instanceof HttpRequest) {
            DefaultHttpRequest request = (DefaultHttpRequest) msg;
            System.out.println("URI:" + request.getUri());
            System.err.println(msg);
        }
        if (msg instanceof HttpContent) {
            LastHttpContent httpContent = (LastHttpContent) msg;
            ByteBuf byteData = httpContent.content();
            if (!(byteData instanceof EmptyByteBuf)) {
                //接收msg消息
                byte[] msgByte = new byte[byteData.readableBytes()];
                byteData.readBytes(msgByte);
                System.out.println(new String(msgByte, StandardCharsets.UTF_8));
            }
        }
        String sendMsg = "不平凡的岁月终究来自你每日不停歇的刻苦拼搏，每一次真正成长都因看清脚下路而抉择出的生活。";
        FullHttpResponse response = new DefaultFullHttpResponse(
                HttpVersion.HTTP_1_1,
                HttpResponseStatus.OK,
                Unpooled.wrappedBuffer(sendMsg.getBytes(StandardCharsets.UTF_8)));
        response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain;charset=UTF-8");
        response.headers().set(HttpHeaderNames.CONTENT_LENGTH, response.content().readableBytes());
        response.headers().set(HttpHeaderNames.CONNECTION, HttpHeaderValues.KEEP_ALIVE);
        ctx.write(response);
        ctx.flush();
    }
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
        cause.printStackTrace();
    }
}
```

> NettyServer.java
``` java
public class NettyServer {
    public static void main(String[] args) {
        new NettyServer().bing(7397);
    }
    private void bing(int port) {
        //配置服务端NIO线程组
        EventLoopGroup parentGroup = new NioEventLoopGroup();
        EventLoopGroup childGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(parentGroup, childGroup)
                    .channel(NioServerSocketChannel.class)//非阻塞模式
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new MyChannelInitializer());
            ChannelFuture f = b.bind(port).sync();
            System.out.println("http-netty server start done. ");
            f.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            childGroup.shutdownGracefully();
            parentGroup.shutdownGracefully();
        }
    }
}
```

> 启动`NettyServer`，`Postman`访问`http://localhost:7397`并设置参数



### 4. WebSocket
WebSocket是一种H5协议规范，通过握手机制客户端与服务器之间就能够建立一个类似Tcp的连接，从而方便客户端与服务器之间的通信。
- 它是一种解决客户端与服务端实时通信而产生的技术：WebSocket本质是一种基于TCP协议，先通过Http/Https发一个特殊的Http请求进行握手，握手后会创建一个用于交换数据的TCP链接，之后客户端和服务端使用该TCP链接进行实时通信。当WebSocket的客户端和服务端握手后 建立通信后，就不再需要之前的http请求参与。

#### 4.1 WebSocket的优点：
- 节省通信开销，之前WebServer实现通信，都使用轮询，需要不停的向服务器发送请求，而HttpRequest的handler很长，请求包含真正的数据可能很小，会占用很多额外的带宽和服务器资源。
- 建立连接后，服务器可主动传数据给客户端，客户端也可以随意向服务端传数据。交换数据时所携带的头信息很小。浏览器（客户端）和服务器只需要做一个握手的动作。
- 实时通信：WebSocket不仅限于Ajax方式通信。ajax方式需要浏览器发起请求。而WebSocket技术 服务端和客户端可以彼此相互推送信息，从而实现实时通信。

#### 4.2 WebSocket建立连接过程：
`客户端发起握手请求 ---> 服务端响应请求 ---> 建立连接`
- 详细流程：建立一个WebSocket连接，客户端或浏览器首先向服务器发送一个特殊的Http请求(携带一些附加头信息)Upgrade:websocket，服务端解析附加头信息，产生应答消息，然后响应给客户端，之后客户端就与服务端建立响应的链接。

#### 4.3 WebSocket生命周期：
1. 打开事件：端点上建立新链接时，该事件是先于其他任何事件发生之前。该事件发生会产生三部分信息。
    1. 创建WebSocket Session对象：用于表示已经建立好的链接
    2. 配置对象：包含配置端点的信息。
    3. 一组路径参数，用于打开节点握手时，WebSocket端入栈匹配的URI
2. 消息事件：主要是接收WebSocket对话中，另一端发送的消息。链接上的消息将会有三种形式抵达客户端。
    1. 文本消息 用String处理
    2. 二进制消息 用byteBuffer或者byte[]处理
    3. pong消息 用Java WebSocket API中的pong.message接口的实例来处理
3. 错误事件：WebSocket链接或者端点发生错误时产生。可以处理入栈消息时发生的各种异常。入栈消息可能产生的三种异常。
    1. WebSocket建立链接时发生错误：SessionException类型
    2. WebSocket试图将入栈消息解码成开发人员使用的对象时 EncodeException类型
    3. WebSocket端点的其他方法运行时产生的错误，WebSocket实现将记录端点操作过程中产生的任何异常
4. 关闭事件：WebSocket链接端点关闭，做一些清理工作，可以由参与连接的任意一个端点发出。

#### 4.4 WebSocket如何关闭链接：
流程：当服务器被指示关闭WebSocket链接时，服务端会发起一个TCP Close操作， 客户端应该等待服务器的TCP Close
- 关闭WebSocket连接，端点需关闭底层TCP连接。
- 底层TCP连接，在大多数正常情况下，应该首先被服务器关闭，服务器持有TIME_WAIT状态（因为这会防止它在2个报文最大生存时间（2MLS）内重新打开连接，然而当一个新的带有更高的seq number的SYN时没有对应的服务器影响TIME_WAIT连接被立即重新打开）。
- 在异常情况下（例如在一个合理的时间量后没有接收到服务器的TCP Close）,客户端可以发起`TCP Close`。



### 5. 基于Netty搭建WebSocket多人聊天室
1. 使用SpringBoot+Netty+WebSocket搭建功能。
2. 使用Netty提供的HttpServerCodec、HttpObjectAggregator、ChunkedWriteHandler进行编码解码处理。
3. 环境准备：`jdk1.8`、`Netty4.1.43.Final`、`spring-boot-starter-web`

> 目录结构
```
└── src.main
       ├── java
       │   └── top.chaooo.hellonetty
       │       ├── domain
       │       │    ├── ClientMsgProtocol.java
       │       │    └── ServerMsgProtocol.java
       │       ├── server
       │       │    ├── MyChannelInitializer.java
       │       │    ├── MyServerHandler.java
       │       │    └── NettyServer.java
       │       ├── util
       │       │    ├── ChannelHandler.java
       │       │    └── MsgUtil.java
       │       ├── controller
       │       │    └── NettyController.java    
       │       └── NettyApplication.java
       └── resources
            ├── static(js,img)
            ├── templates
            │    └── index.html
            └── application.yml

```

> resources/application.yml：基础配置信息，包括了；应用端口、netty服务端端口等
``` yml
server:
  port: 8080

netty:
  host: 127.0.0.1
  port: 7397

spring:
  thymeleaf:
    mode: HTML5
    encoding: UTF-8
    content-type: text/html
    cache: false
```

> server/MyChannelInitializer.java：websocket处理协议
``` java
public class MyChannelInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel channel) {
        channel.pipeline().addLast("http-codec", new HttpServerCodec());
        channel.pipeline().addLast("aggregator", new HttpObjectAggregator(65536));
        channel.pipeline().addLast("http-chunked", new ChunkedWriteHandler());
        // 在管道中添加我们自己的接收数据实现方法
        channel.pipeline().addLast(new MyServerHandler());
    }
}
```

> server/MyServerHandler.java：处理websocket消息信息
``` java
public class MyServerHandler extends ChannelInboundHandlerAdapter {
    private Logger logger = LoggerFactory.getLogger(MyServerHandler.class);
    private WebSocketServerHandshaker handshaker;
    /**
     * 当客户端主动链接服务端的链接后，这个通道就是活跃的了。
     * 也就是客户端与服务端建立了通信通道并且可以传输数据
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        SocketChannel channel = (SocketChannel) ctx.channel();
        logger.info("链接报告开始");
        logger.info("链接报告信息：有一客户端链接到本服务端");
        logger.info("链接报告IP:{}", channel.localAddress().getHostString());
        logger.info("链接报告Port:{}", channel.localAddress().getPort());
        logger.info("链接报告完毕");
        ChannelUtil.channelGroup.add(ctx.channel());
    }
    /**
     * 当客户端主动断开服务端的链接后，这个通道就是不活跃的。
     * 也就是说客户端与服务端的关闭了通信通道并且不可以传输数据
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        logger.info("客户端断开链接{}", ctx.channel().localAddress().toString());
        ChannelUtil.channelGroup.remove(ctx.channel());
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //http
        if (msg instanceof FullHttpRequest) {
            FullHttpRequest httpRequest = (FullHttpRequest) msg;
            if (!httpRequest.decoderResult().isSuccess()) {
                DefaultFullHttpResponse httpResponse = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.BAD_REQUEST);
                // 返回应答给客户端
                if (httpResponse.status().code() != 200) {
                    ByteBuf buf = Unpooled.copiedBuffer(httpResponse.status().toString(), CharsetUtil.UTF_8);
                    httpResponse.content().writeBytes(buf);
                    buf.release();
                }
                // 如果是非Keep-Alive，关闭连接
                ChannelFuture f = ctx.channel().writeAndFlush(httpResponse);
                if (httpResponse.status().code() != 200) {
                    f.addListener(ChannelFutureListener.CLOSE);
                }
                return;
            }
            WebSocketServerHandshakerFactory wsFactory = new WebSocketServerHandshakerFactory("ws:/" + ctx.channel() + "/websocket", null, false);
            handshaker = wsFactory.newHandshaker(httpRequest);
            if (null == handshaker) {
                WebSocketServerHandshakerFactory.sendUnsupportedVersionResponse(ctx.channel());
            } else {
                handshaker.handshake(ctx.channel(), httpRequest);
            }
            return;
        }
        //ws
        if (msg instanceof WebSocketFrame) {
            WebSocketFrame webSocketFrame = (WebSocketFrame) msg;
            //关闭请求
            if (webSocketFrame instanceof CloseWebSocketFrame) {
                handshaker.close(ctx.channel(), (CloseWebSocketFrame) webSocketFrame.retain());
                return;
            }
            //ping请求
            if (webSocketFrame instanceof PingWebSocketFrame) {
                ctx.channel().write(new PongWebSocketFrame(webSocketFrame.content().retain()));
                return;
            }
            //只支持文本格式，不支持二进制消息
            if (!(webSocketFrame instanceof TextWebSocketFrame)) {
                throw new Exception("仅支持文本格式");
            }
            String request = ((TextWebSocketFrame) webSocketFrame).text();
            System.out.println("服务端收到：" + request);
            ClientMsgProtocol clientMsgProtocol = JSON.parseObject(request, ClientMsgProtocol.class);
            //1请求个人信息
            if (1 == clientMsgProtocol.getType()) {
                ctx.channel().writeAndFlush(MsgUtil.buildMsgOwner(ctx.channel().id().toString()));
                return;
            }
            //群发消息
            if (2 == clientMsgProtocol.getType()) {
                TextWebSocketFrame textWebSocketFrame = MsgUtil.buildMsgAll(ctx.channel().id().toString(), clientMsgProtocol.getMsgInfo());
                ChannelUtil.channelGroup.writeAndFlush(textWebSocketFrame);
            }
        }
    }
    /**
     * 抓住异常，当发生异常的时候，可以做一些相应的处理，比如打印日志、关闭链接
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
        logger.info("异常信息：\r\n" + cause.getMessage());
    }
}
```

> server/NettyServer.java：主服务
``` java
@Component("nettyServer")
public class NettyServer {
    private Logger logger = LoggerFactory.getLogger(NettyServer.class);
    //配置服务端NIO线程组
    private final EventLoopGroup parentGroup = new NioEventLoopGroup();
    private final EventLoopGroup childGroup = new NioEventLoopGroup();
    private Channel channel;
    public ChannelFuture bing(InetSocketAddress address) {
        ChannelFuture channelFuture = null;
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(parentGroup, childGroup)
                    .channel(NioServerSocketChannel.class)    //非阻塞模式
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childHandler(new MyChannelInitializer());
            channelFuture = b.bind(address).syncUninterruptibly();
            channel = channelFuture.channel();
        } catch (Exception e) {
            logger.error(e.getMessage());
        } finally {
            if (null != channelFuture && channelFuture.isSuccess()) {
                logger.info("demo-netty server start done");
            } else {
                logger.error("demo-netty server start error");
            }
        }
        return channelFuture;
    }
    public void destroy() {
        if (null == channel) return;
        channel.close();
        parentGroup.shutdownGracefully();
        childGroup.shutdownGracefully();
    }
    public Channel getChannel() {
        return channel;
    }
}
```

> util/MsgUtil.java：消息构建工具类
``` java
public class MsgUtil {
    public static TextWebSocketFrame buildMsgAll(String channelId, String msgInfo) {
        //模拟头像
        int i = Math.abs(channelId.hashCode()) % 10;

        ServerMsgProtocol msg = new ServerMsgProtocol();
        msg.setType(2); //链接信息;1自发信息、2群发消息
        msg.setChannelId(channelId);
        msg.setUserHeadImg("head" + i + ".jpg");
        msg.setMsgInfo(msgInfo);

        return new TextWebSocketFrame(JSON.toJSONString(msg));
    }

    public static TextWebSocketFrame buildMsgOwner(String channelId) {
        ServerMsgProtocol msg = new ServerMsgProtocol();
        msg.setType(1); //链接信息;1链接信息、2消息信息
        msg.setChannelId(channelId);
        return new TextWebSocketFrame(JSON.toJSONString(msg));
    }
}
```

> util/ChannelUtil.java：存储每一个客户端接入进来时的channel对象
``` java
public class ChannelUtil {
    //用于存放用户Channel信息，也可以建立map结构模拟不同的消息群
    public static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
}
```

> domain/*MsgProtocol.java：省略get/set
``` java
public class ServerMsgProtocol {
    private int type;             //链接信息;1:自发信息、2:群发消息
    private String channelId;     //通信管道ID，实际使用中会映射成用户名
    private String userHeadImg;   //用户头像[模拟分配]
    private String msgInfo;       //通信消息
    // ...
}
public class ClientMsgProtocol {
    private int type;       //1:请求个人信息，2:发送聊天信息
    private String msgInfo; //消息
    // ...
}
```

> controller/NettyController.java：路由控制层
``` java
@Controller
public class NettyController {
    @RequestMapping(value = "/index")
    public String index(Model model) {
        model.addAttribute("name", "Dear");
        return "index";
    }
}
```

> js逻辑：依赖jquery.min.js、jquery.serialize-object.min.js
``` javaScript
// JavaScript Document
var socket;
$(function(){
    if(!window.WebSocket){
        window.WebSocket = window.MozWebSocket;
    }
    if(!window.WebSocket){
        alert("您的浏览器不支持WebSocket协议！推荐使用谷歌浏览器进行测试。");
        return;
    }
    socket = new WebSocket("ws://localhost:7397/websocket");
    socket.onmessage = function(event){
        var msg = JSON.parse(event.data);
        //链接信息;1自发信息、2群发消息
        if(1 == msg.type){
            jQuery.data(document.body, 'channelId', msg.channelId);
            return;
        }
        //链接信息;1自发信息、2群发消息
        if(2 == msg.type){
            var channelId =    msg.channelId;
            //自己
            if(channelId == jQuery.data(document.body, 'channelId')){
                var module = $(".msgBlockOwnerClone").clone();
                module.removeClass("msgBlockOwnerClone").addClass("msgBlockOwner").css({display: "block"});
                module.find(".headPoint").attr("src", "res/img/"+msg.userHeadImg);
                module.find(".msgBlock_msgInfo .msgPoint").text(msg.msgInfo);
                $("#msgPoint").before(module);
                util.divScroll();
            }
            //好友
            else{
                var module = $(".msgBlockFriendClone").clone();
                module.removeClass("msgBlockFriendClone").addClass("msgBlockFriend").css({display: "block"});
                module.find(".headPoint").attr("src", "res/img/"+msg.userHeadImg);
                module.find(".msgBlock_channelId").text("ID："+msg.channelId);
                module.find(".msgBlock_msgInfo .msgPoint").text(msg.msgInfo);
                $("#msgPoint").before(module);
                util.divScroll();
            }
        }
    };
    socket.onopen = function(event){
        console.info("打开WebSoket 服务正常，浏览器支持WebSoket!");
        var clientMsgProtocol = {};
        clientMsgProtocol.type = 1;
        clientMsgProtocol.msgInfo = "请求个人信息";
        socket.send(JSON.stringify(clientMsgProtocol));
     };
    socket.onclose = function(event){
        console.info("WebSocket 关闭");
    };
    document.onkeydown = function(e) {
        if (13 == e.keyCode && e.ctrlKey){
            util.send();
        }
    }
});
util = {
    send: function(){
        if(!window.WebSocket){return;}
        if(socket.readyState == WebSocket.OPEN){
            var clientMsgProtocol = {};
            clientMsgProtocol.type = 2;
            clientMsgProtocol.msgInfo = $("#sendBox").val();
            socket.send(JSON.stringify(clientMsgProtocol));
            $("#sendBox").val("");
        }else{
            alert("WebSocket 连接没有建立成功！");
        }
    },
    divScroll: function(){
         var div = document.getElementById('show'); 
        div.scrollTop = div.scrollHeight; 
    }    
};
```

> 主要Html
``` html
<html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <script type="text/javascript" src="/js/jquery.min.js"></script>
    <script type="text/javascript" src="/js/jquery.serialize-object.min.js"></script>
    <script type="text/javascript" src="/js/index.js"></script>
</head>
<body>
<div id="chatDiv">
    <div id="chat" style="width:529px; height:667px; background-color:#F5F5F5; float:right;">
        <!-- 会话区域 begin -->
        <div id="show" style="width:529px; height:450px; float:left;overflow-y:scroll;">
            <!-- 消息块；好友 -->
            <div class="msgBlockFriendClone" style=" display:none; margin-left:30px; margin-top:15px; width:340px; height:auto; margin-bottom:15px; float:left;">
                <div class="msgBlock_userHeadImg" style="float:left; width:35px; height:35px;border-radius:3px;-moz-border-radius:3px; background-color:#FFFFFF;">
                    <img class="headPoint" src="/img/head5.jpg" width="35px" height="35px" style="border-radius:3px;-moz-border-radius:3px;"/>
                </div>
                <div class="msgBlock_channelId" style="float:left; width:100px; margin-top:-5px; margin-left:10px; padding-bottom:2px; font-size:10px;">
                    <!-- 名称 -->
                </div>
                <div class="msgBlock_msgInfo" style="height:auto;width:280px;float:left;margin-left:12px; margin-top:4px;border-radius:3px;-moz-border-radius:3px; ">
                    <div style="width:4px; height:20px; background-color:#CC0000; float:left;border-radius:3px;-moz-border-radius:3px;"></div>
                    <div class="msgPoint" style="float:left;width:260px; padding:7px; background-color:#FFFFFF; border-radius:3px;-moz-border-radius:3px; height:auto; font-size:12px;display:block;word-break: break-all;word-wrap: break-word;">
                        <!-- 信息 -->
                    </div>
                </div>
            </div>
            <!-- 消息块；自己 -->
            <div class="msgBlockOwnerClone" style=" display:none; margin-right:30px; margin-top:15px; width:340px; height:auto; margin-bottom:15px; float:right;">
                <div style="float:right; width:35px; height:35px;border-radius:3px;-moz-border-radius:3px; background-color:#FFFFFF;">
                    <img class="headPoint" src="/img/head3.jpg" width="35px" height="35px" style="border-radius:3px;-moz-border-radius:3px;"/>
                </div>
                <div class="msgBlock_msgInfo" style="height:auto;width:280px;float:left;margin-left:12px; margin-top:4px;border-radius:3px;-moz-border-radius:3px; ">
                    <div class="msgPoint" style="float:left;width:260px; padding:7px; background-color:#FFFFFF; border-radius:3px;-moz-border-radius:3px; height:auto; font-size:12px;display:block;word-break: break-all;word-wrap: break-word;">
                        <!-- 信息 -->
                    </div>
                    <div style="width:4px; height:20px; background-color:#CC0000; float:right;border-radius:3px;-moz-border-radius:3px;"></div>
                </div>
            </div>
            <span id="msgPoint"></span>
        </div>
        <!-- 会话区域 end -->
        <div style="width:100%; height:2px; float:left; background-color:#CCCCCC;"></div>
        <div style="margin:0 auto; width:100%; height:149px; margin-top:5px;  background-color:#FFFFFF; float:left;">
            <textarea id="sendBox" style="font-size:14px; border:0; width:499px; height:80px; outline:none; padding:15px;font-family:”微软雅黑”;resize: none;"></textarea>
            <div style="margin-top:20px; float:right; margin-right:35px; padding:5px; padding-left:15px; padding-right:15px; font-size:12px; background-color:#F5F5F5;border-radius:3px;-moz-border-radius:3px; cursor:pointer;" onclick="javascript:util.send();">发送(S)</div>
        </div>
    </div>
</div>
</body>
</html>
```

+ 启动SpringBoot，Netty会随着启动；
+ 用不同浏览器访问 `http://localhost:8080/index` 测试多人实时聊天。

