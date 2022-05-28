# <font color="orange">服务网关（Zuul）</font>

    为什么需要使用微服务网关？

微服务网关最重要的功能就是可以实现路由转发和过滤。在微服务架构体系中，一个项目会包含多个微服务，并且每个微服务会独立部署，提供不同的网络地址，客户端可以通过调用多个微服务接口完成一个用户请求，这样会带来以下几个明显问题：

- 客户端多次请求不同的微服务，增加了客户端的复杂性。
- 每个服务都需要独立认证，过于繁琐、复杂。
- 存在跨域请求，在特殊场景下处理会比较复杂。
- 难以重构。

通过网关就可以解决以上问题。

微服务网关时存在于客户端和服务端的中间层，客户端发来的所有请求都会先通过微服务网关再转发到真正的微服务进行处理。


Zuul 是 Netflix 的一个开源组件，是通过 Servlet 实现的。Zuul 作为 Spring Cloud 中的服务网关组件，能够通过与 Eureka 进行整合，将自身注册到 Ereka Server 中，与 Eureka、Ribbon、Hystrix 等进行组合，同时从 Eureka 中获得其它微服务实例的信息。

通过这样的设计，能够把网关和服务管理整合到一起，让 Zuul 可以获取到服务注册信息，结合 Ribbon、Hystrix 等更好地实现路由转发、负载均衡等功能。

## 1. Zuul 快速入门

### 1.1 创建服务提供者

提前准备两个被调服务：`eureka-client-department` 和 `eureka-client-employee` 。简单起见，他们不需要是完整的项目，只有 **@Controller** 对外返回简单的字符串数据即可。

将被调用服务注册到 Eureka Server 注册中心。

> 在引入『网关』的概念之后，对于服务的访问就分为：『外部访问』和『内部访问』两种。外部访问都要经过网关，而内部访问则不需要经过网关。

它们两个分别对外暴露如下 URI：

- eureka-client-department
  - http://localhost:8080
  - http://localhost:8080/xxx
- eureka-client-employee
  - http://localhost:9090
  - http://localhost:9090/xxx

<font color="red">**注意**</font> 为了为后续的一个知识点做铺垫，这里我们的 URI 中没有出现 `employee` 和 `department` 字样。

### 1.2 创建 gateway 工程

新建 zuul 工程，命名为 **zuul-gateway** 。在 Spring Initializer 中引入三项依赖：*Spring Web*、*Eureka Discovery Client* 和 **Zuul** 。

> 网关项目<small>（Zuul）</small>和注册中心项目<small>（Eureka Server）</small>一样，都是独立的项目，是独立运行的。而且，网关项目还是一个 Eureka Client 项目，它需要连接、注册到 Eureka Server 。

创建 EurekaZuulApplication 启动类，并增加 **@EnableZuulProxy** 注解:

```java
@SpringBootApplication
@EnableEurekaClient
@EnableZuulProxy  // 看这里，看这里，看这里。
@EnableCircuitBreaker // 开启熔断器功能
public class EurekaZuulApplication {
  public static void main(String[] args) {
    SpringApplication.run(EurekaZuulApplication.class, args);
  }
}
```

- 添加 bootstrap.yml 配置文件：

  ```yml
  eureka:
    instance:
      instance-id: ${eureka.instance.hostname}:${server.port}
      prefer-ip-address: true
      lease-renewal-interval-in-seconds: 5
      lease-expiration-duration-in-seconds: 10
    client:
      service-url:
        defaultZone: http://127.0.0.1:8761/eureka/
  ```

- 添加 **application.yml** 配置文件:

  ```yaml
  server:
    port: 7600

  spring:
    application:
      name: zuul-gateway
  eureka:
    instance:
      hostname: ${spring.cloud.client.ip-address}
  zuul:
    routes:
      eureka-client-employee: /employee/**
      eureka-client-department: /department/**
  ```

这里的核心配置就是 `zuul.routes` 这段：

- 当 zuul 收到以 `/employee` 开头的请求时，将转发至 eureka-client-employee 微服务，至于 eureka-client-employee 微服务的具体地址再哪，zuul 会自己去查从 Eureka Server 那里下载的注册表；

- 当 zuul 收到以 `/department` 开头的请求时，将转发至 eureka-client-department 微服务，至于 eureka-client-department 微服务的具体地址在哪，zuul 会自己去查从 Eureka Server 那里下载的注册表。

<font color="red">**注意**</font> eureka-client-employee 和 eureka-client-department 对外暴露的路径中，不强求有 `/employee`、`/department` 。

依次启动启动服务注册中心、服务提供者、服务网关，访问地址: [http://localhost:7600/employee/xxx](http://localhost:7600/employee/xxx) 和 [http://localhost:7600/department/xxx](http://localhost:7600/department/xxx) 测试。


另外，当一个服务启动多个实例时，Zuul 服务网关会依次请求不同端口，以达到负载均衡的目的。


## 2. Zuul 路由的映射规则配置

### 2.1 服务路由配置 

Zuul 通过与 Eureka 的整合，实现了对服务实例的自动化维护，即服务路由功能。

只需要通过 `zuul.routes.<路由名>.path` 和 `zuul.routes.<路由名>.serviceId` 的方式成对配置即可。例如：

```yml
zuul:
  routes:
    # eureka-client-employee: /employee/**
    # eureka-client-department: /department/**
    xxx:
      path: /employee/**
      service-id: eureka-client-employee
    yyy:
      path: /department/**
      service-id: eureka-client-department
```

有 2 点需要指出：

- `路由名` 是程序员自定义的名字，是一个任意内容。
- 上一章的写法，本质上是上述写法的简写。即，`zuul.routes.<serviceId>=<path>` 。

### 2.2 服务路由的默认规则

如果你有意<small>（或无意）</small>忘记了在网关项目的配置文件中配置 zuul 的路由规则，那么 zuul 也是可用的！

在这种情况下，zuul 使用的就是默认的路由规则：以服务的『**服务名**』作为前缀路径。即：

```yml
zuul:
  routes:
    xxx:
      path: /eureka-client-employee/**
      service-id: eureka-client-employee
    yyy:
      path: /eureka-client-department/**
      service-id: eureka-client-department
```

即简写形式为：

```yml
zuul:
  routes:
    eureka-client-employee: /employee-client-employee/**
    eureka-client-department: /employee-client-department/**
```

如果不想使用默认的路由规则，就可以在配置文件中加入下列内容，即可关闭所有默认的路由规则：

```yaml
zuul:
  ignored-services: '*'
```

在关闭默认的路由配置之后，此时需要在配置文件中逐个为需要路由的服务添加映射规则。

## 3. 路由截取

默认情况下，zuul 会截取、删除掉你访问它<small>（zuul）</small>的 URI 的第一部分，而后再路由到目标服务。例如，

你访问的 zuul 的 URI 是 `/department/xxx`，根据路由配置，请求路由到部门服务后，触发的是 `/xxx` URI 。

如果你不需要这种默认的路由截取的功能，你可以通过 `strip-prefix` 配置项进行配置：

```yaml
zuul:
  routes:
    xxx:
      path: /employee/**
      service-id: eureka-client-employee
      strip-prefix: true
    yyy:
      path: /department/**
      service-id: eureka-client-department
      strip-prefix: false
```

在 Zuul 中，路由表达式采用了 Ant 风格定义。

Ant 风格的路由表达式共 有 3 种通配符：

| 通配符 | 说明 | 举例 |
| :- | :- | :- |
| ? | 匹配任意单个字符 | /xxx/? |
| * | 匹配任意数量的字符 | /xxx/* |
| ** | 匹配任意数量的字符，<br>包括多级目录 | /xxx/** |

---

为了让用户更灵活地使用路由配置规则，zuul 还提供了一个忽略表达式参数 `zuul.ignored-patterns`，该参数用来设置不被网关进行路由的 RUL 表达式。

```yaml
zuul:
  ignored-patterns: /**/xxx/**
  routes:
    eureka-client-employee: /employee/**
    eureka-client-department: /department/**
```

## 4. Zuul 与 Hystrix 结合实现熔断 

Zuul 和 Hystrix 结合使用实现熔断功能时，需要完成 FallbackProvider 接口。该接口提供了 2 个方法：

- **.getRoute** 方法：用于指定为哪个服务提供 fallback 功能。
- **.fallbackResponse** 方法：用于执行回退操作的具体逻辑。

例如，我们为 service-id 为 `eureka-client-department` 的微服务<small>（在 zuul 网关处）</small>提供熔断 fallback 功能。

- 实现 FallbackProvider 接口，并托管给 Spring IoC 容器

  ```java
  @Component
  public class DepartmentFallback implements FallbackProvider {

      public String getRoute() {
          // 服务名 application name，而非路径
          return "eureka-client-department";

          // return "*"; 对所有的路由服务生效
      }

      public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
          // 这里需要自定义 fallback 时的响应信息
          return new DepartmentClientHttpResponse();
      }
  ```

- 实现 ClientHttpResponse 接口<small>（被 DepartmentFallback 使用）</small>

  ```java
  public class DepartmentClientHttpResponse implements ClientHttpResponse {

      public HttpStatus getStatusCode() throws IOException {
          return HttpStatus.OK;
      }

      public int getRawStatusCode() throws IOException {
          return getStatusCode().value();
      }

      public String getStatusText() throws IOException {
          return getStatusCode().getReasonPhrase();
      }

      public void close() {
      }

      public InputStream getBody() throws IOException {
          // json-string
          String body = "连接异常，请稍后重试";
          return new ByteArrayInputStream(body.getBytes());
      }

      public HttpHeaders getHeaders() {
          HttpHeaders headers = new HttpHeaders();
          MediaType type = new MediaType("application", "json", StandardCharsets.UTF_8);
          headers.setContentType(type);
          return headers;
      }
    }
  ```

## 5. Zuul 中的 Eager Load 配置 

zuul 的路由转发也是由通过 Ribbon 实现负载均衡的。默认情况下，客户端相关的 Bean 会延迟加载，在第一次调用微服务时，才会初始化这些对象。所以 zuul 无法在第一时间加载到 Ribbon 的负载均衡。

如果想提前加载 Ribbon 客户端，就可以在配置文件中开启饥饿加载<small>（即，立即加载）</small>：

```yaml
zuul:
  ribbon:
    eager-load:
      enabled: true
```

<font color="red">注意</font> **eager-load** 配置对于默认路由不起作用。因此，通常它都是结合 `忽略所有的默认路由` 一起使用的：`zuul.ignored-services=*` ，以达到 zuul 启动时就默认已经初始化各个路由所要转发的负载均衡对象。


## 6. 服务拦截 

前面我们提到，服务网关还有个作用就是接口的安全性校验，这个时候我们就需要通过 zuul 进行统一拦截，zuul 通过继承过滤器 **ZuulFilter** 进行处理，下面请看具体用法。 

新建一个类 ***ApiFilter*** 并继承 ***ZuulFilter***：

```java
@Component
public class ApiFilter extends ZuulFilter {

  @Override
  public String filterType() {
    return "pre";
  }

  @Override
  public int filterOrder() {
    return 0;
  }

  @Override
  public boolean shouldFilter() {
    return true;
  }

  @Override
  public Object run() throws ZuulException {
    // 这里写校验代码。例如 JWT 的校验。
    RequestContext context = RequestContext.getCurrentContext();
    HttpServletRequest request = context.getRequest();
    HttpServletResponse response = context.getResponse();
    response.setCharacterEncoding("UTF-8");
    response.setHeader("Content-type","text/html;charset=UTF-8");

    log.info("请求进入过滤器，访问的 url：{}，访问的方法：{}", 
                request.getRequestURL(), 
                request.getMethod());

    // 根据需要获取请求中的参数
    String token = request.getParameter("token");

    int random = (int) (Math.random() * 100);

    if (random % 2 == 0) {
      log.info("请求不通过过滤器");
      try {
        response.getWriter().write("token is invalid.");
      } catch (Exception e) {
        e.printStackTrace();
      }

      // 根据业务逻辑，本次请求就【到此为止】，不要再向下传递了。
      context.setSendZuulResponse(false);
      context.setResponseStatusCode(200);
    }
    else {
      log.info("请求通过过滤器");
    }

    return null;
  }
}
```

其中:

- **filterType** 为过滤类型，可选值有 

    - pre：在 Zuul 按照规则路由到下级服务之前执行。如果需要对请求进行预处理，比如鉴权、限流等，都应该考虑在此类 Filter 中实现。
    - route：这类 Filter 是 Zuul 路由动作的执行者。<small>我们通常不会实现这类路由。</small>
    - post：这类 Filter 是在源服务返回结果或者异常信息发生后执行的，如果需要对返回信息做一些处理，则在此类 Filter 进行处理。
    - error：在整个路由环节中如果发生异常，则会进入 error Filter，可做全局异常处理。

- **filterOrdery** 为过滤的顺序，如果有多个过滤器，则数字越小越先执行

- **shouldFilter** 表示是否过滤。它的返回值决定了该 Filter 是否执行，可以作为开关来使用。<small>不过，通常它都是返回 true，『开关』的问题有专门的配置项进行配置。</small>

- **run** 为过滤器执行的具体逻辑，在这里可以做很多事情，比如:权限判断、合法性校验等。

启动 gateway，在浏览器输入地址: [http://localhost:8080/api/hello](http://localhost:8080/api/hello)，可以看到以下界面：

```
token is invalid
```

再通过浏览器输入地址: [http://localhost:8080/api/index?token=12345](http://localhost:8080/api/index?token=12345)，可以看到以下界面:

```
端口: 8762
```

## 7. 禁用 zuul 过滤器 

Spring Cloud 默认为 zuul 编写并启动了一些过滤器，这些过滤器都放在 ` org.springframework.cloud.netflix.zuul.filters` 包下。

如果需要禁用某个过滤器，只需要设置 `zuul.<SimpleClassName>.<filterType>.disabled=true`，就能禁用名为 `<SimpleClassName>` 的过滤器。例如:

```yaml
zuul:
  JwtFilter:
    pre:
      disable: true
```

上述配置就禁用掉了我们自定义的 JwtFilter 。