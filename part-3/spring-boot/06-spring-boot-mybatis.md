# <font color="orange">SpringBoot 整合 Mybatis</font>

## 1. 什么是 MyBatis-Spring-Boot-Starter

`mybatis-spring-boot-starter` 是 MyBatis 帮助我们快速集成 SpringBoot 提供的一个组件包，使用这个组件可以做到以下几点：

- 几乎可以零配置
- 需要很少的 XML 配置

mybatis-spring-boot-starter 依赖于 ***MyBatis-Spring*** 和 ***SpringBoot***，最新版 `1.3.2` 需要 MyBatis-Spring `1.3` 以上，Spring Boot 版本 `1.5` 以上。

> 注意 mybatis-spring-boot-starter 是 MyBatis 官方开发的 Starter，而不是 Spring Boot 官方开发的启动包。mybatis-spring-boot-starter 支持 XML 配置和注解配置两种。

## 2. 关键依赖包

`mybatis-spring-boot-starter` 的 pom 文件，现在最新版本是 `2.1.0` 。

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```

## 3. application 配置

application.properties：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/scott\
    ?useUnicode=true\
    &characterEncoding=utf-8\
    &useSSL=true\
    &serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=123456
## jdbc-starter 中自带了一个连接池：HikariCP
spring.datasource.hikari.idle-timeout=60000
spring.datasource.hikari.maximum-pool-size=30
spring.datasource.hikari.minimum-idle=10

mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml

logging.level.root=INFO
logging.level.hemiao3000.gitee.io=DEBUG
logging.pattern.console=${CONSOLE_LOG_PATTERN:\
  %clr(${LOG_LEVEL_PATTERN:%5p}) \
  %clr(|){faint} \
  %clr(%-40.40logger{39}){cyan} \
  %clr(:){faint} \
  %m%n${LOG_EXCEPTION_CONVERSION_WORD:%wEx}}
```

其中：

| 配置 | 说明 |
| :- | :- |
| `mybatis.config-locations` | 配置 *mybatis-config.xml* 路径，<br>*mybatis-config.xml* 中配置 MyBatis 基础属性；|
| `mybatis.mapper-locations` | 配置 Mapper 对应的 XML 文件路径； |
| `mybatis.type-aliases-package` | 配置项目中实体类包路径； |
| `spring.datasource.*` | 数据源配置。 |

如果你需要使用 Druid 连接池，也可以使用 Druid 官方提供的启动器：

```xml
<!-- Druid连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

而连接信息的配置与上面类似，四大连接属性不变，只不过在连接池特有属性上，方式略有不同：

```yml
## 初始化连接数
spring.datasource.druid.initial-size=1
## 最小空闲连接
spring.datasource.druid.min-idle=1
## 最大活动连接
spring.datasource.druid.max-active=20
```

Spring Boot 启动时数据源会自动注入到 SqlSessionFactory 中，使用 SqlSessionFactory 构建 SqlSessionFactory，再自动注入到 Mapper 中，最后我们直接使用 Mapper 即可。

## 4. 启动类

在启动类中添加对 Mapper 包扫描 ***@MapperScan***，SpringBoot 启动的时候会自动加载包径下的 Mapper 。

```java
@SpringBootApplication
@MapperScan("com.woniu.dao")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

或者直接在 Mapper/DAO 类上面添加注解 `@Mapper`，建议使用上面那种，不然每个 mapper 加个注解也挺麻烦的。

> 如果使用的是 Idea，注入 DAO 时经常会报 `could not autowire`，Eclipse 却没有问题，其实代码是正确的，这是 Idea 的误报。可以选择降低 Autowired 检测的级别，不要提示就好。 在 File | Settings | Editor | Inspections 选项中使用搜索功能找到 `Autowiring for Bean Class`，将 Severity 的级别由之前的 error 改成 warning 即可。

