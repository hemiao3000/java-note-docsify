# <font color="orange">使⽤ Spring Boot Actuator 监控应⽤</font>

Spring Boot 是⼀个⾃带监控的开源框架，组件 Spring Boot Actuator 负责监控应⽤的各项静态和动态的变量。在项⽬中结合 Spring Boot Actuator 的使⽤，便可轻松对 Spring Boot 应⽤监控治理。

## 1. Actuator 监控

只需要在项⽬中添加 *`spring-boot-starter-actuator`*，就⾃动启⽤了监控功能。

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
</dependencies>
```

Spring Boot Actuator 是 Spring Boot 提供的对应⽤系统的检查和监控的集成功能，可以查看应⽤配置的详细信息，例如⾃动化配置信息、创建的 Spring beans 以及⼀些环境属性等。

Actuator 监控分成两类：<strong>原⽣端点</strong> 和 <strong>⽤户⾃定义端点</strong>。⾃定义端点主要是指扩展性，⽤户可以根据⾃⼰的实际应⽤，定义⼀些⽐较关⼼的指标，在运⾏期进⾏监控。


## 2. Actuator 的 REST 接⼝

原⽣端点是在应⽤程序⾥提供众多 Web 接⼝，通过它们了解应⽤程序运⾏时的内部状况，原⽣端点⼜可以分成三类：

- **应⽤配置类**：可以查看应⽤在运⾏期的静态信息，例如⾃动配置信息、加载的 springbean 信息、yml ⽂件配置信息、环境信息、请求映射信息；

- **度量指标类**：主要是运⾏期的动态信息，如堆栈、请求连、⼀些健康指标、metrics 信息等；

- **操作控制类**：主要是指 shutdown，⽤户可以发送⼀个请求将应⽤的监控功能关闭。

Actuator 提供了十多个接⼝。

## 3. 常见命令详解

在 Spring Boot 2.x 中为了安全期间，Actuator 只开放了两个端点 ***/actuator/health*** 和 ***/actuator/info***，可以在配置⽂件中设置打开其它。

可以打开所有的监控点：

```properties
management.endpoints.web.exposure.include=*
```

Actuator 默认所有的监控点路径都在 ***/actuator/\****，当然如果你对 ***actuator*** 这个名字不满意，你也可以自定义：

```properties
management.endpoints.web.base-path=/manage
```

设置完重启后，再次访问地址就会变成 ***/manage/\**** ，不再是以前的 */actuator/\**。<small>当然一般情况下不会动这个配置。</small>

Actuator ⼏乎监控了应⽤涉及的⽅⽅⾯⾯，我们重点讲述⼀些经常在项⽬中常⽤的命令。

### 3.1 health 

health 主要⽤来检查应⽤的运⾏状态，这是我们使⽤最⾼频的⼀个监控点，通常使⽤此接⼝提醒我们应⽤实例的运⾏状态，以及应⽤不“健康”的原因，如数据库连接、磁盘空间不够等。

默认情况下 ***/actuator/health*** 的状态是开放的，添加依赖后启动项⽬，访问：[*http://127.0.0.1:8080/actuator/health*](http://127.0.0.1:8080/actuator/health) 即可看到应⽤的状态。

```json
{
    "status" : "UP"
}
```

默认情况下只是展示简单的 ***UP*** 和 ***DOWN*** 状态，为了查询更详细的监控指标信息，可以在配置⽂件中添加以下信息：

```
management.endpoint.health.show-details=always
```

重启后再次访问⽹址 [*http://localhost:8080/actuator/health*](http://localhost:8080/actuator/health)，返回信息如下：

```json
{
  "status": "UP",
  "diskSpace": {
    "status": "UP",
    "total": 209715195904,
    "free": 183253909504,
    "threshold": 10485760
  }
}
```

可以看到 HealthEndPoint 给我们提供默认的监控结果，包含磁盘空间描述总磁盘空间，剩余的磁盘空间和最⼩阈值。


### 3.2 info 

info 是我们⾃⼰在配置⽂件中以 info 开头的配置信息，⽐如在示例项⽬中的配置是：

```properties
info.app.name=spring-boot-actuator
info.app.version=1.0.0
info.app.test=test
```

启动示例项⽬，访问 [*http://localhost:8080/actuator/info*](http://localhost:8080/actuator/info) 返回部分信息如下：

```json
info
{
    "app": {
        "name": "spring-boot-actuator",
        "version": "1.0.0",
        "test":"test"
    }
}
```

### 3.3 env 

展示了系统环境变量的配置信息，包括使⽤的环境变量、JVM 属性、命令⾏参数、项⽬使⽤的 jar 包等信息。

启动示例项⽬，访问⽹址 [*http://localhost:8080/actuator/env*](http://localhost:8080/actuator/env) 返回部分信息如下：

```json
{
  "profiles": [

  ],
  "server.ports": {
    "local.management.port": 8088,
    "local.server.port": 8080
  },
  "servletContextInitParams": {

  },
  "systemProperties": {
    ...
  }
```

为了避免敏感信息暴露到 ***/actuator/env*** ⾥，所有名为 password、secret、key<small>（或者名字中最后⼀段是这些）</small>的属性在 ***/actuator/env*** ⾥都会加上 ***\**** 。

举个例⼦，如果有⼀个属性名字是 ***database.password***，那么它在 ***/actuator/env*** 中的显示效果是这样的：

```json
"database.password": "******"
```

### 3.4 /env/{name} ⽤法

就是 env 的扩展可以获取指定配置信息，⽐如 `http://localhost:8080/actuator/env/java.vm.version`，返回 `{"java.vm.version":"25.101-b13"}`。


### 3.5 shutdown

开启接⼝优雅关闭 Spring Boot 应⽤，要使⽤这个功能⾸先需要在配置⽂件中开启：

```
management.endpoint.shutdown.enabled=true
```

配置完成之后，启动示例项⽬，使⽤ curl 模拟 post 请求访问 shutdown 接⼝。

<strong>shutdown 接⼝默认只⽀持 post 请求</strong> 。

```
curl -X POST "http://localhost:8080/actuator/shutdown"
{
 "message": "Shutting down, bye..."
}
```

此时会发现应⽤已经被关闭。

### 3.6 mappings

描述全部的 URI 路径，以及它们和控制器的映射关系。

启动示例项⽬，访问⽹址 *http://localhost:8080/actuator/mappings* 返回部分信息如下：

```json
{
  "/**/favicon.ico": {
    "bean": "faviconHandlerMapping"
  },
  "{[/hello]}": {
    "bean": "requestMappingHandlerMapping",
    "method": "public java.lang.String com.neo.controller.HelloController.index()"
  },
  "{[/error]}": {
    "bean": "requestMappingHandlerMapping",
    "method": "public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)"
  }
}
```