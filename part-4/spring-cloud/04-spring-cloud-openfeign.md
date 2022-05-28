# <font color="orange">声明式 RESTful 客户端：OpenFeign</font>

OpenFeign 实对 RestTemplate 发起 HTTP 请求的进一步包装和简化，并且还囊括、整合了 Ribbon 。

## 1. 什么是 OpenFeign 

即便是使用了 RestTemplate 简化了代码，但是我们仍面对一个明显的缺点：

我们<small>（程序员）</small>仍能感觉到，甚至参与到了一个 HTTP 请求与响应的收发过程中。这种『感觉』和『**调用**』相去甚远。

对此，Netflix 提供了 **Feign**，并在 2016.7 月的最后一个版本 `8.18.0` 之后，将其捐赠给 spring cloud 社区，并更名为 **OpenFeign** 。OpenFeign 的第一个版本就是 `9.0.0` 。

OpenFeign 会完全代理 HTTP 的请求，在使用过程中我们只需要依赖注入 Bean，然后调用对应的方法传递参数即可。这对程序员而言屏蔽了 HTTP 的请求响应过程，让代码更趋近于『**调用**』的形式。

> 当然，OpenFeign 的作用并不单这一点。当前，我们仅涉及到这点，更多内容后续继续讲解。


## 2. Feign 的入门案例 

在创建一个 Spring Boot Mave 项目，命名为 eureka-client-consumer-feign<small>（或其他）</small>，在 Spring Initializer 中引入依赖：

- 在 Initializer 的搜索框内搜索 **Eureka Client** 和 **OpenFeign** 。 或

- 在 **Spring Cloud Discovery** 下选择 **Eureka Discovery Client** ；在 **Spring Cloud Routing** 下选择 **OpenFeign** 。

注意事项

- pom 中实际引入的核心依赖是

  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  ```

为项目添加配置：

- **application.yaml** ：

  ```yaml
  server:
    port: 9527
  spring:
    application:
      name: eureka-client-consumer-feign
  eureka:
    client:
      service-url:
        defaultZone: http://127.0.0.1:8761/eureka
  ```

最后创建一个启动类 FeignApplication：

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients(basePackages = "...") // 看这里，看这里，看这里
public class EurekaFeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaFeignApplication.class, args);
    }
}
```

我们可以看到启动类增加了一个新的注解: **@EnableFeignClients** ，如果我们要使用 OpenFeign ，必须要在启动类加入这个注解，以开启 OpenFeign 。

这样，我们的 Feign 就已经集成完成了，那么如何通过 Feign 去调用之前我们写的 HTTP 接口呢？

和 MyBatis 类似：

首先创建一个接口 ApiService<small>（名字任意）</small>，并且通过注解配置要调用的服务地址:

```java
@FeignClient(value = "eureka-client-producer")  // 看这里，看这里，看这里
public interface ApiService {
  @GetMapping(value = "/hello")
  String hello(@RequestParam("name") String name);
}
```

**@FeignClient** 注解的 **name** 属性的值是服务提供者在 Eureka Server 上所注册的名字<small>（也就是在 Eureka Server 上看到的那一串全大写的字符串）</small>，通常也就是服务提供者的 **spring.application.name** 。

一个服务只能被一个类绑定。

一个服务只能被一个类绑定。

一个服务只能被一个类绑定，不能让多个类绑定同一个远程服务，否则，会在启动项目是出现『**已绑定**』异常。

然后在 OpenFeign 里面通过单元测试来查看效果。

### 添加测试代码 {docsify-ignore}

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ApiServiceTest {

  @Autowired
  private ApiService apiService;

  @Test
  public void test() {
    try {
      log.debug("{}", apiService.hello("tom"));
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```

> 最后在强调一次，OpenFeign 的能力包括但不仅包括这个。


## 3. OpenFeign 的配置 

### 3.1 超时和超时重试

OpenFeign 本身也具备重试能力，在早期的 Spring Cloud 中，OpenFeign 使用的是 feign.Retryer.Default#Default() ，重试 5 次。但 OpenFeign 整合了 Ribbon ，Ribbon 也有重试的能力，此时，就可能会导致行为的混乱。<small>（总重试次数 = OpenFeign 重试次数 x Ribbon 的重试次数，这是一个笛卡尔积。）</small>

后来 Spring Cloud 意识到了此问题，因此做了改进<small>（[issues 467](https://github.com/spring-cloud/spring-cloud-netflix/issues/467)）</small>，将 OpenFeign 的重试改为 feign.Retryer#NEVER_RETRY ，即，**默认关闭** 。

所以，OpenFeign 对外表现出的超时和重试的行为，实际上是 Ribbon 的超时和超时重试行为。我们在项目中进行的配置，也都是配置 Ribbon 的超时和超时重试。

```yml
# 全局配置
ribbon:
  readTimeout: 1000     # 请求处理的超时时间
  MaxAutoRetries: 5     # 最大重试次数
  MaxAutoRetriesNextServer: 1   # 切换实例的重试次数
  # 对所有请求开启重试，而非 get 请求。一般不会开启这个功能。
  # okToRetryOnAllOperations: true
```

整个 OpenFeign<small>（实际上是 Ribbon）</small>的最大重试次数为：

    (1 + MaxAutoRetries) x (1 + MaxAutoRetriesNextServer)

这里需要注意的是『重试』次数是不包含『本身那一次』的。

故意加大被调服务的返回响应时长，你会看到主调服务中打印类似如下消息：

```
feign.RetryableException: Read timed out executing GET http://SERVICE-PRODUCER/demo?username=tom&password=123

	at feign.FeignException.errorExecuting(FeignException.java:249)
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:129)
	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:89)
  ...
```

另外，在被调服务方，你会发现上述配置会导致被调服务收到 12 次请求：

    请求次数 = (1 + 5) x (1 + 1)

你也可以指定对某个特定服务的超时和超时重试：

```yml
# 针对 SERVICE-PRODUCER 的设置
SERVICE-PRODUCER:
  ribbon:
    readTimeout: 3000
    MaxAutoRetries: 2
    MaxAutoRetriesNextServer: 0
```

### 3.2 如何替换底层 HTTP 实现

> 本质上是 OpenFeign 所使用的 RestTemplate 替换底层 HTTP 实现。

- 替换成 HTTPClient

  将 OpenFeign 的底层 HTTP 客户端替换成 HTTPClient 需要 2 步:

  1. 引入依赖：

     ```xml
     <dependency>
       <groupId>io.github.openfeign</groupId>
       <artifactId>feign-httpclient</artifactId>
      </dependency>
     ```

  2. 在配置文件中启用它：

     ```yaml
     feign:
       httpclient:
         enabled: true # 激活 httpclient 的使用
     ```


- 替换成 OkHttp

  将 OpenFeign 的底层 HTTP 客户端替换成 OkHttp 需要 2 步:

  1. 引入依赖：

     ```xml
     <dependency>
       <groupId>io.github.openfeign</groupId>
       <artifactId>feign-okhttp</artifactId>
     </dependency>
     ```

  2. 在配置文件中启用它：

     ```yaml
     feign:
       httpclient:
         enabled: false  # 关闭 httpclient 的使用
       okhttp:
         enabled: true   # 激活 okhttp 的使用
     ```


### 3.3 日志配置（了解）

SpringCloudFeign 为每一个 FeignClient 都提供了一个 feign.Logger 实例。可以根据 **logging.level.\<FeignClient>** 参数配置格式来开启 Feign 客户端的 DEBUG 日志，其中 **\<FeignClient>** 部分为 Feign 客户端定义接口的完整路径。如：

```yaml
logging:
  level:
    com.woniu.service.feign: DEBUG
```

然后再在配置类<small>（比如主程序入口类）</small>中加入 Looger.Level 的 Bean：

```java
@Bean
public Logger.Level feignLoggerLevel() {
    return  Logger.Level.FULL;
}
```

| 级别	| 说明 |
| :- | :- |
| NONE	  | 不输出任何日志 |
| BASIC	  | 只输出 Http 方法名称、请求 URL、返回状态码和执行时间 |
| HEADERS	| 输出 Http 方法名称、请求 URL、返回状态码和执行时间 和 Header 信息 |
| FULL	  | 记录 Request 和 Response 的 Header，Body 和一些请求元数据 |
