---
title: 「Spring」SpringBoot使用RocketMQ实战样例
date: 2021-06-11 22:11:23
tags: [后端开发, Spring, RocketMQ, SpringBoot]
categories: Spring
---

通过`rocketmq-spring-boot-starte`r可以快速的搭建`RocketMQ`生产者和消费者服务。<!-- more -->

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

3. 发送消息与消费消息

使用`RocketMQTemplate`实现消息的发送;
使用实现`RocketMQListener`接口，并添加`@RocketMQMessageListener`注解，声明消费主题，消费者分组等，且默认消费模式是集群消费。

### 1. 普通消息
发送消息测试接口：`http://localhost:8080/send/common`
``` java
@RestController
@RequiredArgsConstructor
public class RocketMqController {

    private final RocketMQTemplate rocketMQTemplate;

    @GetMapping("send/common")
    public Object sendCommon() {
        return rocketMQTemplate.syncSend("common_topic", "普通消息");
    }
}
```

普通消息监听消费
``` java
/**
 * `@RocketMQMessageListener`默认的消费模式是集群消费
 * 在集群消费模式中，在同一个topic下，相同的ConsumerGroup只会有一个Consumer收到消息。
 * RocketMQListener<T> 泛型必须和接收的消息类型相同，这里是String
 */
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "common_topic",
        consumerGroup = "test_group")
public class CommonListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```


### 2. 带Tag的消息
发送消息测试接口：`http://localhost:8080/send/tag`
``` java
    @GetMapping("send/tag")
    public Object sendWithTag() {
        return rocketMQTemplate.syncSend("tag_topic"+ ":" + "tag", "tag消息,tag:tag");
    }
```

监听消费
``` java
/**
 * 如果我们的消费者指定了消费的Tag后，发送的消息如果不带tag，将会消费不到；
 * 如果我们的生产者指定了Tag，但是消费者的selectorExpression没有设置，即用默认的“*”,那么这个消费者也会消费到。
 */
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "tag_topic",
        selectorType = SelectorType.TAG,
        selectorExpression = "tag",//指定了tag后，发送的消息如果不带tag，将会消费不到
        consumerGroup = "tag_group")
public class TagMsgListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```


### 3. 消费模式为广播消费
发送消息测试接口：`http://localhost:8080/send/broadcast`
``` java
    @GetMapping("send/broadcast")
    public Object sendWithManyTag() {
        return rocketMQTemplate.syncSend("broadcast_topic", "广播消息");
    }
```

监听消费
``` java
/**
 * 通过messageModel = MessageModel.BROADCASTING 指定消费模式为广播消费。(默认集群模式)
 */
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "broadcast_topic",
        messageModel = MessageModel.BROADCASTING,//指定为广播消费
        consumerGroup = "broadcast_group")
public class BroadcastListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```


### 4. 顺序发送的消息，随机消费
发送消息测试接口：`http://localhost:8080/send/random`
``` java
    @GetMapping("send/random")
    public Object sendRandom() {
        List<SendResult> results = new ArrayList<>();
        for (int i = 0; i <= 3; i++) {
            SendResult sendResult = rocketMQTemplate.syncSend("random_topic", "无序消息" + i);
            results.add(sendResult);
        }
        return results;
    }
```

监听消费
``` java
/**
 * 顺序发送的消息，消费顺序不一定是按照我们发送的顺序来消费的。
 */
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "random_topic",
        consumerGroup = "random_group")
public class RandomListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        //发送消息是顺序发送的0,1,2,3,消费的顺序不一定是顺序的
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```


### 5. 顺序消费
发送消息测试接口：`http://localhost:8080/send/order`
``` java
    @GetMapping("send/order")
    public Object sendOrder() {
        List<SendResult> results = new ArrayList<>();
        for (int i = 0; i <= 3; i++) {
            SendResult sendResult = rocketMQTemplate.syncSendOrderly("order_topic", "有序消息" + i, "hashkey");
            results.add(sendResult);
        }
        return results;
    }
```

监听消费
``` java
/**
 * 通过设置属性consumeMode = ConsumeMode.ORDERLY，指定消费模式为顺序消费，消费的顺序也和发送顺序一致
 */
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "order_topic",
        consumeMode = ConsumeMode.ORDERLY,//指定为顺序消费
        consumerGroup = "order_group")
public class OrderListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        //发送消息是顺序发送的0,1,2,3,消费的顺序也是顺序的
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```


### 6. 异步消息
`producer`向`broker`发送消息时指定消息发送成功及发送异常的回调方法，调用`API`后立即返回，`producer`发送消息线程不阻塞 ，消息发送成功或失败的回调任务在一个新的线程中执行。

发送消息测试接口：`http://localhost:8080/send/async`
``` java
    @GetMapping("send/async")
    public Object sendAsync() {
        rocketMQTemplate.asyncSend("async_topic", "异步消息", new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
                log.info("发送成功:{}", JSON.toJSONString(sendResult));
                //可以处理相应的业务
            }
            @Override
            public void onException(Throwable throwable) {
                //可以处理相应的业务
            }
        });
        return null;
    }
```

监听消费
``` java
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "async_topic",
        consumerGroup = "async_group")
public class AsyncListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```


### 7. 单向发送消息
单向发送消息这种方式主要用在不特别关心发送结果的场景，例如日志发送。

发送消息测试接口：`http://localhost:8080/send/oneway`
``` java
    @GetMapping("send/oneway")
    public void sendOneway() {
        rocketMQTemplate.sendOneWay("oneway_topic", "单向消息");
    }
```

监听消费
``` java
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "oneway_topic",
        consumerGroup = "oneway_group")
public class OnewayListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```


### 8. 延时消息
发送消息测试接口：`http://localhost:8080/send/delay`
``` java
    @GetMapping("send/delay")
    public Object sendDelay() {
        // 延时消息的使用限制messageDelayLevel:"1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h"
        // 开源版RocketMQ并不支持任意时间的延时，需要设置几个固定的延时等级，从1s到2h分别对应着等级1到18 消息消费失败会进入延时消息队列
        String txt = "延时消息:" + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Message<String> message = MessageBuilder.withPayload(txt).build();
        // 延时等级取4，延时30s
        return rocketMQTemplate.syncSend("delay_topic", message, 2000, 4);
    }
```

监听消费
``` java
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "delay_topic",
        consumerGroup = "delay_group")
public class DelayListener implements RocketMQListener<String> {
    @Override
    public void onMessage(String message) {
        log.info("{}于{}收到消息：{}", this.getClass().getSimpleName(), LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")), message);
    }
}
```


### 9. 事务消息（半消息）
- 事务消息共有三种状态，提交状态、回滚状态、中间状态：
    + TransactionStatus.CommitTransaction: 提交事务，它允许消费者消费此消息。
    + TransactionStatus.RollbackTransaction: 回滚事务，它代表该消息将被删除，不允许被消费。
    + TransactionStatus.Unknown: 中间状态，它代表需要检查消息队列来确定状态。
- 事务消息仅仅只是保证本地事务和MQ消息发送形成整体的 原子性，而投递到MQ服务器后，并无法保证消费者一定能消费成功。

发送消息测试接口：`http://localhost:8080/send/tx`
``` java
    @GetMapping("send/tx")
    public Object sendTransaction() {
        int i = new Random().nextInt(1000);
        Map<String, String> txtMap = new HashMap<>(2);
        txtMap.put("key", "key" + i);
        txtMap.put("name", "事务消息");
        txtMap.put("desc", "事务消息" + i);
        Message<Map<String, String>> message = MessageBuilder.withPayload(txtMap).setHeader("key", txtMap.get("key")).build();
        return rocketMQTemplate.sendMessageInTransaction("tx_topic", message , i);
    }
```

生产者端需要实现`RocketMQLocalTransactionListener`接口，重写执行本地事务的方法和检查本地事务方法；
`@RocketMQTransactionListener`注解表明这个一个生产端的消息监听器，需要配置监听的事务消息生产者组。
``` java
@Slf4j
@Service
@RocketMQTransactionListener
public class TxProducerListener implements RocketMQLocalTransactionListener {
    /**
     * 每次推送消息会执行executeLocalTransaction方法，首先会发送半消息，到这里的时候是执行具体本地业务，
     * 执行成功后手动返回RocketMQLocalTransactionState.COMMIT状态，
     * 这里是保证本地事务执行成功，如果本地事务执行失败则可以返回ROLLBACK进行消息回滚。 
     * 此时消息只是被保存到broker，并没有发送到topic中，broker会根据本地返回的状态来决定消息的处理方式。
     */
    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message message, Object arg) {
        log.info("开始执行本地事务");
        RocketMQLocalTransactionState state;
        try{
            Integer i = (Integer) arg;
            if (i % 2 == 0) {
                // 偶数抛出异常
                int a = i / 0;
            }
            // COMMIT:即生产者通知Rocket该消息可以消费
            state = RocketMQLocalTransactionState.COMMIT;
            log.info("本地事务已提交。{}",message.getHeaders().get("key").toString());
        }catch (Exception e){
            log.info("执行本地事务失败。{}",e);
            // ROLLBACK:即生产者通知Rocket将该消息删除
            state = RocketMQLocalTransactionState.ROLLBACK;
        }
        return state;
    }
    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
        log.info("开始执行回查");
        // 判断具体业务逻辑，来决定是否回滚还是提交
        boolean flag = false;
        if ( flag ) {
            log.info("回滚半消息");
            return RocketMQLocalTransactionState.ROLLBACK;
        }
        log.info("提交半消息");
        return RocketMQLocalTransactionState.COMMIT;
    }
}
```

监听消费
``` java
/**
 * 事务消息只是保证了本地事务和消息发送的原子性，
 * 如果 消费端消费失败 后的处理方式，建议是记录异常信息然后 人工处理 ，
 * 并不建议回滚上游服务的数据(因为两者是 解耦 的，而且 复杂度 太高)
 */
@Slf4j
@Component
@RocketMQMessageListener(
        topic = "tx_topic",
        consumerGroup = "tx_group")
public class TxConsumerListener implements RocketMQListener<Map<String, String>> {
    @Override
    public void onMessage(Map<String, String> message) {
        log.info("{}收到消息：{}", this.getClass().getSimpleName(), message);
    }
}
```

### 10. 部分测试日志打印
``` shell
2021-06-11 17:25:02.861  INFO 13904 --- [MessageThread_3] com.demo.CommonListener      : CommonListener收到消息：普通消息
2021-06-11 17:25:10.296  INFO 13904 --- [MessageThread_2] com.demo.TagMsgListener      : TagMsgListener收到消息：tag消息,tag:tag
2021-06-11 17:26:05.070  INFO 13904 --- [MessageThread_1] com.demo.BroadcastListener   : BroadcastListener收到消息：广播消息
2021-06-11 17:27:19.969  INFO 13904 --- [MessageThread_2] com.demo.RandomListener      : RandomListener收到消息：无序消息1
2021-06-11 17:27:19.969  INFO 13904 --- [MessageThread_1] com.demo.RandomListener      : RandomListener收到消息：无序消息0
2021-06-11 17:27:19.970  INFO 13904 --- [MessageThread_4] com.demo.RandomListener      : RandomListener收到消息：无序消息3
2021-06-11 17:27:19.970  INFO 13904 --- [MessageThread_3] com.demo.RandomListener      : RandomListener收到消息：无序消息2
2021-06-11 17:28:15.530  INFO 13904 --- [MessageThread_2] com.demo.OrderListener       : OrderListener收到消息：有序消息0
2021-06-11 17:28:15.531  INFO 13904 --- [MessageThread_3] com.demo.OrderListener       : OrderListener收到消息：有序消息1
2021-06-11 17:28:15.533  INFO 13904 --- [MessageThread_4] com.demo.OrderListener       : OrderListener收到消息：有序消息2
2021-06-11 17:28:15.540  INFO 13904 --- [MessageThread_5] com.demo.OrderListener       : OrderListener收到消息：有序消息3
2021-06-11 17:29:24.630  INFO 13904 --- [MessageThread_1] com.demo.AsyncListener       : AsyncListener收到消息：异步消息
2021-06-11 17:29:24.644  INFO 13904 --- [ublicExecutor_1] o.example.controller.RocketMqController  : 发送成功:{"messageQueue":{"brokerName":"localhost.localdomain","queueId":0,"topic":"async_topic"},"msgId":"7F000001365018B4AAC237405B920066","offsetMsgId":"C0A8026400002A9F000000000004FF02","queueOffset":0,"regionId":"DefaultRegion","sendStatus":"SEND_OK","traceOn":true}
2021-06-11 17:30:12.790  INFO 13904 --- [MessageThread_1] com.demo.OnewayListener      : OnewayListener收到消息：单向消息
2021-06-11 17:31:06.185  INFO 13904 --- [MessageThread_1] com.demo.DelayListener       : DelayListener于2021-06-11 17:31:06收到消息：延时消息:2021-06-11 17:30:36
2021-06-11 17:31:22.216  INFO 13904 --- [nio-8080-exec-5] com.demo.TxProducerListener  : 开始执行本地事务
2021-06-11 17:31:22.217  INFO 13904 --- [nio-8080-exec-5] com.demo.TxProducerListener  : 本地事务已提交。key5
2021-06-11 17:31:22.233  INFO 13904 --- [MessageThread_3] com.demo.TxConsumerListener  : TxConsumerListener收到消息：{name=事务消息, key=key5, desc=事务消息5}
2021-06-11 17:31:27.872  INFO 13904 --- [nio-8080-exec-4] com.demo.TxProducerListener  : 开始执行本地事务
2021-06-11 17:31:27.880  INFO 13904 --- [nio-8080-exec-4] com.demo.TxProducerListener  : 执行本地事务失败。{}
java.lang.ArithmeticException: / by zero
	at com.demo.TxProducerListener.executeLocalTransaction(TxProducerListener.java:42) ~[classes/:na]
...
```

> 源码地址：[https://gitee.com/chaoo/mq-test.git](https://gitee.com/chaoo/mq-test.git)
