# <font color="orange">Java 代码配置</font>

从 Spring 3.0 开始，Spring 官方就推荐大家使用 java 代码配置来代替以前的 xml 文件配置。而到了 SpringBoot，Java 代码配置更是成了标配。

Java 代码配置主要靠 Java 类和一些注解，比较常用的注解有：

| 常用注解 | 说明 |
| :- | :- |
| @Configuration | 声明一个类作为配置类，代替 *`.xml`* 文件 |
| @Bean | 声明在方法上，将方法的返回值加入Bean容器，代替 *`<bean>`* 标签 |
| @Value | 属性注入 |
| @PropertySource | 指定外部属性文件 |


## 1. 配置 Druid 数据库连接池的简单方式

Spring Boot 默认的数据库连接池是 HikariCP 。spring-boot-starter-parent 中设置了 HIkariCP 的版本信息。因此，如果使用 HikariCP 只用声明你要用它就行，而声明要使用 Druid，你需要自己指定所使用的版本。

引入依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.6</version>
</dependency>
```

编写 Java 配置类：

```java
@Configuration
public class JdbcConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://192.168.119.130:3306/scott?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        return dataSource;
    }
}
```

以上代码配置等同于如下的 *`.xml`* 配置：

```xml
<beans ...>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource.DruidDataSource">
        <propertie name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <propertie name="url" value="jdbc:mysql://192.168.119.130:3306/scott?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"/>
        <propertie name="username" value="root"/>
        <propertie name="password" value="123456"/>
    </bean>
</beans>
```

## 2. 代码配置中引用关系的表示

上面的 DruidDataSource ，我们为其属性赋值时，它的四个属性都是简单类型属性，因此十分容易处理。但是，在 Spring 的容器中，Java Bean 可能会存在引用。

在 *`.xml`* 配置文件中，我们是通过 *`ref`* 属性为引用类型的属性赋值的。

例如：

```xml
<beans>

  <bean id="teacher" class="xxx.xxx.xxx.Teacher">
    <property name="name" value="tom"/>
    <property name="age" value="40"/>
  </bean>

  <bean id="student" class="xxx.xxx.xxx.Student">
    <property name="name" value="jerry"/>
    <property name="age" value="20"/>
    <property name="teacher" ref="teacher"/>
  </bean>

</beans>
```

而在 Java 代码配置中，通过 @Bean 方法的入参来表达 Bean 之间的引用关系。

无论两个 Java Bean 是在不同的配置类中，还是在同一个配置类中，都可使用这种方式：

```java
@Bean
public Employee employee(Department department) {
    Employee employee = new Employee();
    employee.setId(10);
    employee.setName("tom");
    employee.setSalary(2000.0);
    employee.setDepartment(department);

    return employee;
}
```

当 Spring 在调用你的 `employee()` 方法时，发现需要传入一个 Department 对象，它就会在 Spring IoC 容器中去『找』一个 Department 对象<small>（单例对象）</small>，并传入到你的 `employee()` 方法中。

这种写法还表达了 Department 对象要『先于』Employee 对象创建的含义。


## 3. Druid 数据库连接池配置 v1

第一个版本的配置十分直观，容易理解。使用 **`@Value`** 注解为配置类的<small>（简单类型）</small>属性赋值。随后，你就可以在 @Bean 方法中使用这些属性。

```java
@Configuration
public class JdbcConfig {

    @Value("com.mysql.cj.jdbc.Driver")
    String driverClassName;

    @Value("jdbc:mysql://127.0.0.1:3306/scott?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false")
    String url;

    @Value("root")  
    String username;

    @Value("123456")
    String password;

    @Bean(initMethod = "init", destroyMethod = "close")
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(this.driverClassName);
        dataSource.setUrl(this.url);
        dataSource.setUsername(this.username);
        dataSource.setPassword(this.password);
        return dataSource;
    }
}
```

## 4. Druid 数据库连接池配置 v2

在上面的 Druid 的配置中，数据库连接的四大属性值是在代码中『写死』的。我们可以把它们『抽取』出来放在配置文件中。

### 4.1 配置文件 jdbc.properties

```properties
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/scott?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
jdbc.username=root
jdbc.password=123456
```

在配置类中可以结合 **`@PropertySource`** 和 `@Value` 注解，让 Spring 从指定配置文件中读取数据，再注入到配置类中。

这里的 `@PropertySource("classpath:jdbc.properties")` 起到的作用等效与我们在 `.xml` 配置文件中所用的：

```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
```

### 4.2 配置类

```java
@Configuration
@PropertySource("classpath:jdbc.properties")    // 这里
public class JdbcConfig {

    @Value("${jdbc.url}")                       // 这里
    String url;

    @Value("${jdbc.driverClassName}")           // 这里
    String driverClassName;

    @Value("${jdbc.username}")                  // 这里
    String username;

    @Value("${jdbc.password}")                  // 这里
    String password;

    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```


## 5. Druid 数据库连接池配置 v3

由于 Spring Boot 项目启动时，一定会加载项目的配置文件 **`application.properties`**，所以，我们可以直接将自定义的配置项写在 **`application.properties`** 中，而不必再单独的创建一个配置文件。

这种写法本质上，和上述的 **`jdbc.properties`** 的写法是一样的。

截至到这里，日常编程中的配置工作大体也就是如此。理论上，还有再进一步改进优化的余地，但是总体来说，必要性已经不大了。<small>（可能，还把简单问题复杂化了）</small>。

