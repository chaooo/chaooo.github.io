---
title: 「Spring Reactive Stack」响应式 HTTP 请求客户端 WebClient
date: 2021-03-20 12:01:42
tags: [后端开发, Spring, SpringReactive, WebClient]
categories: SpringReactive
---

**WebClient**是`Spring WebFlux`模块提供的一个非阻塞的基于响应式编程的进行`HTTP`请求的客户端工具。<!-- more -->

引入`WebFlux`依赖则可使用`WebClient`：
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```


### 1. 创建WebClient实例
`WebClient`接口提供了三个不同的静态方法（`create()`，`create(String baseUrl)`，`builder()`）和一个内部类（`WebClient.Bulider`）来创建`WebClient`实例：
``` java
/**
 * 自动注入WebClient的内部类WebClient.Builder
 */
@Autowired
private WebClient.Builder clientBuilder;

void test() {
    // WebClient.create()创建
    WebClient webClient1 = WebClient.create();
    // WebClient.create(String baseUrl)
    WebClient webClient2 = WebClient.create("http://www.test.com");
    // WebClient.builder()创建
    WebClient webClient3 = WebClient.builder().build();
    // 内部类WebClient.Builder创建
    WebClient webClient4 = clientBuilder.build();
    WebClient webClient5 = clientBuilder.baseUrl("http://www.test.com").build();
}
```


### 2. GET 请求
#### 2.1 发起GET请求
``` java
@Slf4j
@RestController
public class TestController {
  
    @Autowired
    private WebClient.Builder clientBuilder;

    @GetMapping("/test")
    public Mono<String> test(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()                      // GET 请求
                .uri("/test/string")        // 请求路径
                .retrieve()                 // 获取响应体
                .bodyToMono(String.class);  // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }
}
```

#### 2.2 GET请求参数传递
``` java
@Slf4j
@RestController
public class TestController {
  
    @Autowired
    private WebClient.Builder clientBuilder;
    
    /**
     * 1. 请求路径里带参数(?id=5&name=abc)
     */
    @GetMapping("/test/get")
    public Mono<String> test(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()                             // GET 请求
                .uri("/test/param?id=5&name=abc")  // 请求路径(带参数)
                .retrieve()                        // 获取响应体
                .bodyToMono(String.class);         // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }
    
    /**
     * 2. 占位符的形式传递参数
     */
    @GetMapping("/test2")
    public Mono<String> test2(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()                             // GET 请求
                .uri("/test/{1}/{2}", name, id)    // 请求路径(占位符参数,若id=5,name=abc，则为"/test/abc/5")
                .retrieve()                        // 获取响应体
                .bodyToMono(String.class);         // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }
    
    /**
     * 3. 另一种占位符的形式传递参数(类似@PathVariable)
     */
    @GetMapping("/test/{name}/{id}")
    public Mono<String> test3(@PathVariable("id") Integer id, @PathVariable("name") String name){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()                              // GET 请求
                .uri("/test/{name}/{id}", name, id) // 请求路径(另一种占位符参数)
                .retrieve()                         // 获取响应体
                .bodyToMono(String.class);          // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    } 

    /**
     * 4. 使用 map 装载参数
     */
    @GetMapping("/test5")
    public Mono<String> test5(){
        Map<String,Object> map = new HashMap<>();
        map.put("name", "abc");
        map.put("id", 5);
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()                              // GET 请求
                .uri("/test/{name}/{id}", map)      // 请求路径(使用map装载参数)
                .retrieve()                         // 获取响应体
                .bodyToMono(String.class);          // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    } 
}
```


### 3. POST请求
#### 3.1 发起POST请求
``` java
@Slf4j
@RestController
public class TestController {
  
    @Autowired
    private WebClient.Builder clientBuilder;

    @PostMapping("/test/post")
    public Mono<String> test(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .post()                     // POST 请求
                .uri("/test/post")          // 请求路径
                .retrieve()                 // 获取响应体
                .bodyToMono(String.class);  // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }
}
```

#### 3.2 POST请求参数传递
``` java
@PostMapping("/test6")
public Mono<String> test6(){
    //提交参数设置
    MultiValueMap<String, String> mulMap = new LinkedMultiValueMap<>();
    mulMap.add("name", "abc");
    mulMap.add("id", "5");
    WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
    Mono<String> stringMono = webClient
            .post()                                             // POST 请求
            .uri("/test/post")                                  // 请求路径
            .contentType(MediaType.APPLICATION_FORM_URLENCODED) // Content-Type: application/x-www-form-urlencoded
            .body(BodyInserters.fromFormData(mulMap))           // 请求参数
            .retrieve()                                         // 获取响应体
            .bodyToMono(String.class);                          // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
    // 打印响应数据
    stringMono.subscribe(log::info);
    return stringMono;
}
```


### 4. 请求异常处理
使用`WebClient`发送请求时， 如果接口返回的不是`200`状态（而是`4xx`、`5xx`这样的异常状态），则会抛出`WebClientResponseException`异常。
``` java
@Slf4j
@RestController
public class TestController {
  
    @Autowired
    private WebClient.Builder clientBuilder;
    
    /**
     * 1. 【doOnError】方法适配所有异常
     */
    @GetMapping("/test/error1")
    public Mono<String> test7(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()
                .uri("/test/error")
                .retrieve()
                .bodyToMono(String.class)
                .doOnError(WebClientResponseException.class, err -> {
                    log.error("发生错误：" + err.getRawStatusCode() + " " + err.getResponseBodyAsString());
                });
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }

    /**
     * 2. 【onStatus】方法根据状态码来适配指定异常
     */
    @GetMapping("/test/error2")
    public Mono<String> test8(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()
                .uri("/test/error")
                .retrieve()
                .onStatus(HttpStatus::is4xxClientError, resp -> {
                    log.error("发生错误：" + resp.statusCode().value() + " " + resp.statusCode().getReasonPhrase());
                    return Mono.error(new RuntimeException("请求失败"));
                })
                .onStatus(HttpStatus::is5xxServerError, resp -> {
                    log.error("发生错误：" + resp.statusCode().value() + " " + resp.statusCode().getReasonPhrase());
                    return Mono.error(new RuntimeException("服务器异常"));
                })
                .bodyToMono(String.class);
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }

    /**
     * 3. 【onErrorReturn】方法来设置在发生异常时返回默认值，
     *    当请求发生异常是会使用该默认值作为响应结果
     */
    @GetMapping("/test/error3")
    public Mono<String> test9(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()
                .uri("/test/error")
                .retrieve()
                .bodyToMono(String.class)
                .onErrorReturn("请求失败");          // 失败时返回默认值“请求失败”
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }
    
    /**
     * 4. 设置【超时属性】
     * 使用timeout()方法设置一个超时时长。如果HTTP请求超时，便会发生TimeoutException异常。
     */
    @GetMapping("/test/setting")
    public Mono<String> test10(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()                          // GET 请求
                .uri("/test/setting")         // 请求路径
                .retrieve()                      // 获取响应体
                .bodyToMono(String.class)        // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
                .timeout(Duration.ofSeconds(3)); // 3秒超时
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }

    /**
     * 5. 设置【异常自动重试】
     * 使用retry()方法可以设置当请求异常时的最大重试次数，如果不带参数则表示无限重试，直至成功。
     */
    @GetMapping("/test/setting2")
    public Mono<String> test11(){
        WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
        Mono<String> stringMono = webClient
                .get()                          // GET 请求
                .uri("/test/setting")         // 请求路径
                .retrieve()                      // 获取响应体
                .bodyToMono(String.class)        // 响应数据类型转换(这里是String，也可以是自定义对象或集合)
                .timeout(Duration.ofSeconds(3))  // 3秒超时
                .retry(2);                       // 重试2次
        // 打印响应数据
        stringMono.subscribe(log::info);
        return stringMono;
    }
}
```


### 5. Exchange获取完整的请求响应结果
前面我们都是使用`retrieve()`方法是直接获取响应体的内容。

使用`exchangeToMono()`和`exchangeToFlux()`方法获取完整的代表响应结果的对象，通过该对象我们可以获取响应码、`contentType`、`contentLength`、响应消息体等。
``` java

@PostMapping("/test/res")
public Mono<String> test12() {
    //提交参数设置
    MultiValueMap<String, String> mulMap = new LinkedMultiValueMap<>();
    mulMap.add("name", "abc");
    mulMap.add("id", "5");
    WebClient webClient = webClientBuilder.baseUrl("https://www.test.com").build();
    Mono<String> stringMono = webClient
            .post()                                             // POST 请求
            .uri("/test/post")                               // 请求路径
            .contentType(MediaType.APPLICATION_FORM_URLENCODED) // Content-Type: application/x-www-form-urlencoded
            .body(BodyInserters.fromFormData(mulMap))           // 请求参数
            .exchangeToMono(response -> {                       // 响应结果response
                if (response.statusCode().equals(HttpStatus.OK)) {  //
                    return response.bodyToMono(String.class);
                } else {
                    return response.createException().flatMap(Mono::error);
                }
            });
    // 打印响应数据
    stringMono.subscribe(log::info);
    return stringMono;
}
```


### 6. WebClient在Spring Cloud中的使用
引入依赖：
``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

编写配置，创建`WebClient.Bulider`类型的`Bean`，加上`@LoadBalaced`为`WebClient`增加负载均衡的支持。
``` java
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class LoadbalanceConfiguration {
    @Bean
    @LoadBalanced
    public WebClient.Builder builder() {
        return WebClient.builder();
    }
}
```

编写Controller，客户端实现访问服务端资源，并对外提供访问接口：
``` java
@RestController
@RequestMapping("/test")
public class ReactiveClient {

    @Autowired
    private WebClient.Builder clientBuilder;

    @GetMapping("/get/string")
    public Mono<String> getServerString() {
        // 通过 WebClient 访问 CLOUD-SERVER 服务的资源 （这里的CLOUD-SERVER是服务注册中心注册的微服务名）
        return clientBuilder.baseUrl("http://CLOUD-SERVER").build().get().uri("/test/string").retrieve().bodyToMono(String.class);
    }
}
```
