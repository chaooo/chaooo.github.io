---
title: 「环境配置」RocketMQ安装并整合SpringBoot
date: 2021-06-10 18:11:23
tags: [环境配置, CentOS, RocketMQ]
categories: 环境配置
---

**RocketMQ** 是阿里巴巴团队使用`Java`语言开发的一个分布式、队列模型的消息中间件，后开源给`Apache`基金会成为了`Apache`的顶级开源项目，具有高性能、高可靠、高实时、分布式特点。

**RocketMQ** 主要由`Producer`、`Broker`、`Consumer`、`NameServer`组成；其中`Producer`负责生产消息；`Consumer`负责消费消息；`Broker`是`MQ`服务,负责接收、分发消息；`NameServer`是路由中心，负责`MQ`服务之间的协调。<!-- more -->

### 1.  Centos安装RocketMQ
1. [官网](https://rocketmq.apache.org/dowloading/releases/)下载`RocketMQ`安装包

``` shell
# 进入自定义软件安装目录
cd /mnt/newdatadrive/apps
# wget下载RocketMQ安装包
wget -c "https://mirrors.bfsu.edu.cn/apache/rocketmq/4.8.0/rocketmq-all-4.8.0-bin-release.zip"
```

2. 解压安装（环境基于`JDK1.8`或以上）

``` shell
# 解压
unzip rocketmq-all-4.8.0-bin-release.zip
# 重命名为rocketmq
mv rocketmq-all-4.8.0-bin-release rocketmq
# 进入安装目录
cd rocketmq
```

3. 修改配置

``` shell
# RocketMQ的默认内存占用非常高，是4×4g的，将4g调整为512m
# 编辑runserver.sh
vim bin/runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
# 编辑runbroker.sh
vim bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m"
```

4. 配置`RocketMQ`的环境变量

``` shell
# 编辑/etc/profile
vim /etc/profile
# 添加：
export ROCKETMQ_HOME=/mnt/newdatadrive/apps/rocketmq
export PATH=$JAVA_HOME/bin:$ROCKETMQ_HOME/bin:$PATH
# 使rocketmq的配置生效
source /etc/profile
```

5. 启动`RockerMQ`顺序

``` shell
# 先启动 NameServer，然后启动 Broker
nohup sh bin/mqnamesrv &
nohup sh bin/mqbroker -n localhost:9876 &
```

6. 关闭`RockerMQ`顺序

``` shell
# 先关闭Broker，再关闭NameServer
sh bin/mqshutdown broker
sh bin/mqshutdown namesrv
```

7. 启动日志

``` shell
# 查看 Name Server 启动日志
tail -f ~/logs/rocketmqlogs/namesrv.log
# 查看 Broker Server 启动日志
tail -f ~/logs/rocketmqlogs/broker.log

# 若出现如下报错
file doesn't exist on this path: /root/store/commitlog
file doesn't exist on this path: /root/store/consumequeue
# 对应创建即可：
cd ~/store
mkdir commitlog consumequeue
```

8. 防火墙
    + 若外网`IP`调试，关闭防火墙 或 开放防火墙端口`9876,10911`

``` shell
# NameServer默认端口：9876
firewall-cmd --add-port=9876/tcp --permanent
# Broker对外服务的监听端口
firewall-cmd --add-port=10911/tcp --permanent
# 更新防火墙规则
firewall-cmd --reload
```

### 2. SpringBoot 整合 RocketMQ
1. `pom.xml`引入组件`rocketmq-spring-boot-starter`依赖

``` xml
<!-- https://mvnrepository.com/artifact/org.apache.rocketmq/rocketmq-spring-boot-starter -->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
```

2. 修改`application.yml`，添加`RocketMQ`相关配置

``` yml
# 多个name-server(集群)使用英文;分割
rocketmq:
  name-server: 192.168.2.100:9876
  producer:
    group: test-group
```

3. 消息生产者

``` java
@Component
public class MessageProducer {

    @Resource
    private RocketMQTemplate rocketMQTemplate;

    /**
     * 生产者发送消息
     * @param topic   主题
     * @param message 消息体
     */
    public void sendMessage(String topic, String message){
        this.rocketMQTemplate.convertAndSend(topic, message);
    }
}
```

4. 消费者

``` java
/**
 * 消费者监听消息
 */
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "test-topic",          // 指定topic
        consumerGroup = "test-group",  // 指定消费组
        selectorExpression = "*"
)
public class MessageConsumer implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        log.info("收到的消息是: {}", message);
    }
}
```

5. 测试类

``` java
@Slf4j
@SpringBootTest
class DemoApplicationTests {

    @Autowired
    private MessageProducer messageProducer;

    @Test
    void testMQ() {
        String message = "Hello RocketMQ!";
        messageProducer.sendMessage("test-topic",message);
        log.info("生产一条消息：" + message);
    }
}
```

6. 运行结果
``` shell
2021-06-10 14:56:25.180  INFO 17720 --- [           main] a.r.s.s.DefaultRocketMQListenerContainer : running container: DefaultRocketMQListenerContainer{consumerGroup='test-group', nameServer='192.168.2.100:9876', topic='test-topic', consumeMode=CONCURRENTLY, selectorType=TAG, selectorExpression='*', messageModel=CLUSTERING}
2021-06-10 14:56:25.188  INFO 17720 --- [           main] o.a.r.s.a.ListenerContainerConfiguration : Register the listener to container, listenerBeanName:messageConsumer, containerBeanName:org.apache.rocketmq.spring.support.DefaultRocketMQListenerContainer_1
2021-06-10 14:56:25.230  INFO 17720 --- [           main] c.e.fastdfsdemo.DemoApplicationTests     : Started DemoApplicationTests in 10.371 seconds (JVM running for 13.888)
2021-06-10 14:56:26.410  INFO 17720 --- [           main] c.e.fastdfsdemo.DemoApplicationTests     : 生产一条消息：Hello RocketMQ!
2021-06-10 14:56:26.426  INFO 17720 --- [MessageThread_1] c.e.f.rocketmq.MessageConsumer           : 收到的消息是: Hello RocketMQ!
2021-06-10 14:56:26.496  INFO 17720 --- [lientSelector_1] RocketmqRemoting                         : closeChannel: close the connection to remote address[192.168.2.100:10911] result: true
2021-06-10 14:56:26.496  INFO 17720 --- [lientSelector_1] RocketmqRemoting                         : closeChannel: close the connection to remote address[192.168.2.100:9876] result: true
```
