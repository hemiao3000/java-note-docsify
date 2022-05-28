# <font color="orange">SpringBoot 的多配置文件</font>



## 1. spring.profiles.active 配置

默认情况下，当你启动 SpringBoot 项目时，会在日志中看到如下一条 INFO 信息：

    No active profile set, falling back to default profiles: default

这条消息是在告诉你，由于你没有激活、启用某个配置文件，SpringBoot 使用了默认的配置文件，也就是 `application.properties` 或 `application.yml` 。

> 当然，这并不是什么错误。

SpringBoot 允许我们的项目提供多配置文件，并『激活、启用』其中的某一个。这些配置文件的命名规则为：`application-xxx.properties` 或 `application-xxx.yml` 。

提供多个配置文件之后，你在 SpringBoot 默认加载的配置文件 **application.properties** 或 **application.yml** 中只用写一个配置项，用以激活、启用某个 *.properties* 或 *.yml* 即可。例如：

```yml
spring:
  profiles:
    active: dev
```

上例中的 `dev` 就是 `application-xxx.properties` 或 `application-xx.yml` 中的那个 xxx 。

现在你再启动 SpringBoot，你会看到如下的 INFO 信息：

    The following profiles are active: dev

这表示 SpringBoot 本次启动使用的就是这个配置文件。

另外，有一种『**简写**』：可以将 application.yml 和各个 application-xxx.yml 凑在一起，就写在 application.yml 配置文件中：

```yml
spring:
  profiles:
    active: dev

---
spring:
  profiles: dev
...

---
spring:
  profiles: test
...

---
spring:
  profiles: prod
...
```



## 2. @Profile 和 @ActiveProfiles 注解

**@Profile** 注解配合 `spring.profiles.active` 参数，也可以实现不同环境下<small>（开发、测试、生产）</small>配置参数的切换。

另外，**@ActiveProfiles** 注解<small>（在测试环境中）</small>可以起到 `spring.profiles.active` 参数的作用。

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Profile("xxx")
    public Human tommy() {
        return new Human("tom", 20);
    }

    @Bean
    @Profile("yyy")
    public Human jerry() {
        return new Human("jerry", 19);
    }

}
```

在上面的配置中：

- 存在 2 套配置：`xxx` 和 `yyy` ；
- name 为 `tommy` 的 Human Bean 仅存在于 `xxx` 的配置套餐中；
- name 为 `jerry` 的 Human Bean 仅存在于 `yyy` 的配置套餐中；

在 application.yml 配置文件通过 active 配置激活启动一个：

```yml
spring:
  profiles:
    active: yyy
```

我们可以在 JUnit 中验证结果：

```java
@SpringBootTest
class AppTest {

    @Autowired
    private Human human;

    @Test
    public void demo() {
        System.out.println(human);  // 这里输出的是 jerry Human Bean
    }
}
```

在测试类的使用中，你也可以将 application.yml 中的 active 配置项去掉，转而在测试类的头上使用 **@ActiveProfiles** 注解，也能起到同样效果：

```java
@SpringBootTest
ActiveProfiles(profiles = "xxx")
class AppTest {
    ...
}
```