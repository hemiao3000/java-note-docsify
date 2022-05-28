# <font color="">Java Date and Time API 用在持久层</font>

## LocalDateTime in Mybatis

MyBatis 从 3.4.5 版本开始就完全支持这种类型了，不在需要自己去写类型转换 。


## LocalDateTime in spring-data-jpa

```xml
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-java8</artifactId>
</dependency>
```


## LocalDateTime in spring-data-redis

```xml
<dependency>
  <groupId>com.fasterxml.jackson.datatype</groupId>
  <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>
```