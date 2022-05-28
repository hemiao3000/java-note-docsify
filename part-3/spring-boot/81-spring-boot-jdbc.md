<span class="title">Spring Boot 中使用 spring-jdbc</span>

```xml
<!-- SQL > MySQL Driver -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- SQL > Spring Data JDBC -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

# 数据源配置

从 Spring Boot 2.0 开始，默认使用 HikariCP 数据库连接池，<small>（也是传说中最快的数据库连接池）</small>。

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/scott\
    ?serverTimezone=UTC\
    &useUnicode=true\
    &characterEncoding=utf-8\
    &useSSL=true
spring.datasource.username=root
spring.datasource.password=123456

logging.level.root=INFO
logging.level.hemiao3000.gitee.io=DEBUG
logging.pattern.console=${CONSOLE_LOG_PATTERN:\
  %clr(${LOG_LEVEL_PATTERN:%5p}) \
  %clr(|){faint} \
  %clr(%-40.40logger{39}){cyan} \
  %clr(:){faint} %m%n\
  ${LOG_EXCEPTION_CONVERSION_WORD:%wEx}}
```

在 Spring Boot 2.0+ 中，由于引用了高版本（6.0+）的 MySQL 驱动包，因此，会显示 *`com.mysql.jdbc.Driver`* 已经过期，推荐使用 *`com.mysql.cj.jdbc.Driver`* 。于此同时，*`serverTimezone=TUC`* 成为了 *`url`* 的必要参数。

# 其他

详见：[spring-jdbc 章节](/Part-III/spring-b/05-spring-jdbc.md)


