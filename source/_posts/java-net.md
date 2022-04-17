---
title: 「Java教程」网络编程基础
date: 2017-05-22 11:59:46
tags: [Java, 后端开发]
categories: Java教程
---


网络编程是指编写运行在多个设备（计算机）的程序，这些设备都通过网络连接起来。
java.net 包中的类和接口，它们提供低层次的通信细节。你可以直接使用这些类和接口，来专注于解决问题，而不用关注通信细节。
<!-- more -->


### 1. 网络编程常识
#### 1.1 七层网络模型
为了保证数据传输的可靠和安全，ISO(国际标准委员会组织)将数据的传递从逻辑上划分为以下7层：
<br>应用层、表示层、会话层、传输层、网络层、数据链路层、物理层
- 当发送数据时，需要按照上述七层模型从上到下层层加包再发送出去；
- 当接收数据时，需要按照上述七层模型从下到上层层拆包再显示出来；

#### 1.2 IP地址 
- IP地址：是互联网中的唯一地址标识，也就是根据IP地址可以定位到具体某一台设备，IP地址本质上是32位二进制组成的整数叫做IPv4，当然也有128位二进制组成的整数叫做IPv6，目前主流的还是IPv4。
- 日常生活中采用**点分十进制**表示法进行IP地址的描述，也就是将每个字节的二进制转换为一个十进制整数，不同的十进制整数之间采用小数点隔开。如：192.168.1.1

#### 1.3 端口号
- 根据IP地址可以定位到具体某一台设备，而该设备中启动的进程可能很多，此时可以使用端口号来定位该设备中的具体某一个进程。
- 网络编程需要提供：IP地址 和 端口号
- 端口号是16位二进制组成的整数，表示范围是：0 ~ 65535，其中0 ~ 1024之间通常被系统占用，因此网络编程需要从1025开始使用。

#### 1.4 tcp协议与udp协议
- TCP（Transmission Control Protocol，传输控制协议） 是面向连接的协议，也就是说，在收发数据前，必须和对方建立可靠的连接。一个TCP连接必须要经过三次“握手”才能建立起来。
- UDP（User Data Protocol，用户数据报协议） 是一个非连接的协议，传输数据之前源端和终端不建立连接，当它想传送时就简单地去抓取来自应用程序的数据，并尽可能快地把它扔到网络上。在发送端，UDP传送数据的速度仅仅是受应用程序生成数据的速度、计算机的能力和传输带宽的限制；在接收端，UDP把每个消息段放在队列中，应用程序每次从队列中读一个消息段。
- tcp协议与udp协议比较：

|tcp协议                         | udp协议|
|--------|-------|
|传输控制协议，面向连接           |用户数据报协议，非面向连接|
|通信过程全程保持连接             |通信过程不需要全程连接|
|保证了数据传输的可靠性和有序性    |不保证数据传输的可靠性和有序性|
|全双工的字节流的通信方式         |全双工的数据报的通信方式|
|服务器的资源消耗多，压力大，效率低|服务器资源消耗少，压力小，效率高|


### 2. 基于tcp协议的编程模型
#### 2.1 编程模型
```
服务器端                         客户端

创建监听服务
等待连接    <----建立连接------  连接服务器           
进行通讯    <----进行通讯----->  进行通讯
关闭连接                        关闭连接

```
- 服务器：
    1. 创建ServerSocket类型的对象并提供端口号；
    2. 等待客户端的连接请求，调用accept方法；
    3. 使用输入输出流进行通信；
    4. 关闭Socket；
- 客户端：
    1. 创建Socket类型的对象并提供服务器的通信地址和端口号；
    2. 使用输入输出流进行通信；
    3. 关闭Socket；

#### 2.2 ServerSocket类和Socket类
- java.net.ServerSocket类主要用于描述服务器套接字信息。

|常用方法||
|----|----|
|ServerSocket(int port)|根据参数指定的端口号来构造对象|
|Socket accept()|监听并接收到此套接字的连接请求|
|void close()|用于关闭套接字|

- java.net.Socket类主要用于描述客户端套接字，是两台机器间通信的端点。

|常用方法||
|----|----|
|Socket(String host, int port)|根据指定主机名和端口号来构造对象|
|InputStream getInputStream()|用于获取当前套接字的输入流|
|OutputStream getOutputStream()|用于获取当前套接字的输出流|
|void close()|用于关闭套接字|


### 3.客户端与服务端通信演示：

``` java
//服务端线程
public class ServerThread extends Thread {
  private Socket s;
  public ServerThread(Socket s) {
    this.s = s;
  }
  @Override
  public void run() {
    try {
      // 3.使用输入输出流进行通信
      BufferedReader br = new BufferedReader(
          new InputStreamReader(s.getInputStream()));
      PrintStream ps = new PrintStream(s.getOutputStream());
      while(true) {
        // 实现服务器接收到字符串内容后打印出来
        // 当客户端没有发送数据时，服务器会在这里阻塞
        String str = br.readLine();
        //System.out.println("服务器接收到的数据是：" + str);
        // 当服务器接收到"bye"后，则聊天结束
        if("bye".equalsIgnoreCase(str)) {
          System.out.println("客户端" + s.getInetAddress() + "已下线！");
          break;
        }
        System.out.println("客户端" + s.getInetAddress() 
          + "发来的消息是：" + str);  
        // 当服务器接收到客户端发来的消息后，向客户端回发消息"I received!"
        ps.println("I received!");
        //System.out.println("服务器发送数据成功！");
      } 
      // 4.关闭Socket
      ps.close();
      br.close();
      s.close();
    } catch(Exception e) {
      e.printStackTrace();
    }
  }
}
//服务端测试
public class ServerStringTest {
  public static void main(String[] args) {
    try {
      // 1.创建ServerSocket类型的对象并提供端口号
      ServerSocket ss = new ServerSocket(8888);
      // 2.等待客户端的连接请求，调用accept方法
      while(true) {
        System.out.println("等待客户端的连接请求...");
        // 当没有客户端连接时，阻塞在accept方法的调用这里
        Socket s = ss.accept();
        // 获取连接成功的客户端通信地址
        System.out.println("客户端" + s.getInetAddress() + "连接成功！");
        // 当有客户端连接成功后，则启动一个新的线程为之服务
        new ServerThread(s).start();
      }
      //ss.close();
    } catch(Exception e) {
      e.printStackTrace();
    }
  }
}
//客户端测试
public class ClientStringTest {
  public static void main(String[] args) {
    try {
      // 1.创建Socket类型的对象并提供服务器的通信地址和端口号
      Socket s = new Socket("XDL-20170621QCO", 8888);
      System.out.println("连接服务器成功！");
      // 2.使用输入输出流进行通信
      Scanner sc = new Scanner(System.in);
      PrintStream ps = new PrintStream(s.getOutputStream());
      BufferedReader br = new BufferedReader(
          new InputStreamReader(s.getInputStream()));
      while(true) {
        // 希望客户端连接服务器成功后睡眠10秒再发送数据，测试服务器是否阻塞
        //Thread.sleep(10000);
        // 练习：实现客户端向服务器发送的内容由用户从键盘输入
        System.out.println("请输入要发送的内容：");
        //String msg = sc.next(); // 读取字符串内容时，遇到空格停止
        String msg = sc.nextLine();
        // 实现客户端向服务器发送字符串内容"hello"
        //ps.println("hello");
        ps.println(msg);
        System.out.println("客户端发送数据成功！");
        // 判断客户端发送的内容是否为"bye"，若是则聊天结束
        if("bye".equalsIgnoreCase(msg)) {
          System.out.println("聊天结束！");
          break;
        }
        // 实现服务器回发消息的接收
        // 当客户端没有发送数据时，服务器会在这里阻塞
        String str = br.readLine();
        System.out.println("客户端接收到的数据是：" + str);
      }
      // 3.关闭Socket
      br.close();
      sc.close();
      ps.close();
      s.close();
    } catch(Exception e) {
      e.printStackTrace();
    }
  }
}
```


### 4. 基于udp协议的编程模型
#### 4.1 编程模型
- 主机A(接收方):
    1. 创建DatagramSocket类型的对象，并提供端口号；
    2. 创建DatagramPacket类型的对象，用于接收发来的数据；
    3. 从Socket中接收数据，调用**receive()**方法；
    4. 关闭Socket并释放有关的资源；
- 主机B(发送方)
    1. 创建DatagramSocket类型的对象；
    2. 创建DatagramPacket类型的对象，并提供接收方的IP地址和端口号；
    3. 通过Socket发送数据，调用**send()**方法；
    4. 关闭Socket并释放有关的资源；

#### 4.2 DatagramSocket类
- java.net.DatagramSocket类用于描述发送或接受数据报的套接字(邮局点);

|常用方法||
|----|----|
|DatagramSocket()              |无参的方式构造对象。|
|DatagramSocket(int port)      |根据参数指定的端口号来构造对象。|
|void receive(DatagramPacket p)|用于接收数据并存放到参数指定的变量中。|
|void send(DatagramPacket p)   |用于将参数指定的数据发送出去。|
|void close()                  | |


#### 4.3 DatagramPacket类
- java.net.DatagramPacket类用于描述数据报信息(信件)；

|常用方法||
|----|----|
|DatagramPacket(byte[] buf, int length) |用于接收数据包并记录到参数变量中； |
|DatagramPacket(byte[] buf, int length, InetAddress address, int port) |用于将参数指定的数据发送到参数指定的位置|
|InetAddress getAddress()               |用于获取发送方或接收方的通信地址信息。|
|int getPort()                          |用于获取发送方或接收方的端口信息。|
|int getLength()                        |用于获取发送或接收数据的长度。|

#### 4.4 InetAddress类
- java.net.InetAddress类用于描述互联网协议地址。

|常用方法||
|----|----|
|static InetAddress getLocalHost()         |用于获取本地主机的通信地址信息。|
|static InetAddress getByName(String host) |根据参数指定的主机名来获取通信地址。|
|String getHostName()                      |用于获取通信地址中的主机名信息。|
|String getHostAddress()                   |用于获取通信地址中的IP地址信息。|

