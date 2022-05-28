# <font color="orange">SpringBoot 中使用 JUnit</font>

## 1. SpringBoot 所使用的 JUnit 版本 

不同版本的 Spring Boot 依赖/使用了不同版本的 Junit。

- Spring Boot `2.1.x.RELEASE` 使用的是 JUnit 4 ；

- Spring Boot `2.2.x.RELEASE` 使用的是 JUnit 5 。
 
Junit 4 和 Junit 5 的不同导致了 Spring Boot 的 `2.1.x.RELEASE` 和 `2.2.1.RELEASE` 版本中的相关配置又有所不同。

JUnit4 与 JUnit 5 常用部分注解对比：

| JUnit4        | JUnit5        | 
| :------------ | :------------ |
| @Test	        | @Test	        |
| @BeforeClass	| @BeforeAll    |
| @AfterClass	| @AfterAll	    |
| @Before	    | @BeforeEach   |
| @After	    | @AfterEach    |
| @Ignore	    | @Disabled     |
| @RunWith	    | @ExtendWith   |


-   Junit 5 的测试类头部

    ```java
    import org.junit.jupiter.api.Test;
    import org.junit.jupiter.api.extension.ExtendWith;

    // @ExtendWith(SpringExtension.class)
    @SpringBootTest
    class DemoApplicationTests {
        ...
    }
    ```

    实际上，如果你使用的是 JUnit5，那么在 SpringBootTest 上没有必要添加 **@ExtendWith**，因为 @SpringBootTest 已经添加了 @ExtendWith 。


-   Junit 4 的测试类头部

    ```java
    import org.junit.Test;
    import org.junit.runner.RunWith;

    @RunWith(SpringJUnit4ClassRunner.class)
    @SpringBootTest
    class DemoApplicationTests {
        ...
    }
    ```


## 2. 加载配置文件 

为了隔离开发和测试环境，我们通常会提供不同的配置文件，在其中配置不同的配置项。例如，开发和测试使用不同的数据库环境。

为此，我们通常会提供一个 `application-test.properties` 或 `application-test.yml` 配置文件用于测试。

在测试时，我们会要求 SpringBoot 去加载测试环境的相关配置。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("test") // 看这里，看这里，看这里
public class SpringBootTestExampleApplicationTests {

    @Value("${spring.datasource.url}")
    private String url; // 可通过这种方式来验证

    ...
}
```

如上例所示，在测试类的同上加上注解 `@ActiveProfiles("test")` 表示本测试类运行时启用 `application-test.properties` 或 `application-test.yml` 配置文件。

## 3. 使用事务锁定测试数据库中的数据 

如果涉及测试增删改的 DAO 方法，或者是测试涉及这些 DAO 的 Service 方法，每一个 Test 方法执行结束后都会对数据库造成影响，从而极大可能影响后续 Test 方法的执行。

> 因为对于第二个 Test 方法而言，数据库环境发生了变化，初始条件可能就已经不满足了。

spring-test 可以利用事务的回滚在测试方法结束后，撤销测试方法对数据库的影响。即，测试方法对数据库的写操作最终并未提交。

在测试方法，或测试类的头上加注 `@Transactional` 和 `@Rollback` 注解即可：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("test")
@Transactional
@Rollback   // 默认，可省略，与之对应的是 @Commit 注解
public class SpringBootTestExampleApplicationTests {
    ...
}
```

再执行测试方法时，你在输出日志中会看到类似如下的 INFO 信息：

```
Began transaction (1) for test context 
...
Rolled back transaction for test: 
```

`@Transactional` 和 `@Rollback` 不仅对 Dao 测试的回滚有效，对 Service 方法的测试，甚至是 Web 方法的测试也都有效。

## 4. @Sql 注解

**@Sql** 注解可以执行 SQL 脚本，也可以执行 SQL 语句。它既可以加上类上面，也可以加在方法上面。

默认情况下，方法上的 @Sql 注解会覆盖类上的 @Sql 注解。

例如：

```java
@Sql({ "/drop_schema.sql", "/create_schema.sql" })
@Sql(scripts = "/insert_data1.sql", statements = "insert into student(id, name) values (100, 'Shiva')")
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = AppConfig.class)
public class SqlTest {

	@Autowired
	private JdbcTemplate jdbcTemplate;

	@Test
	public void fetchRows1() {
		List<Map<String, Object>> students = jdbcTemplate.queryForList("SELECT * FROM student");
		assertEquals(3, students.size());
	}

	@Sql("/insert_more_data1.sql")
	@Test
	public void fetchRows2() {
		List<Map<String, Object>> students = jdbcTemplate.queryForList("SELECT * FROM student");
		assertEquals(5, students.size());
	}
} 
```

## 5. @WebMvcTest 注解

**@WebMvcTest** 是 Spring Boot 1.4 引入的 4 个新的测试注释之一：

```
       @JsonTest - for testing the JSON marshalling and unmarshalling
     @WebMvcTest - for testing the controller layer
    @DataJpaTest - for testing the repository layer
@RestClientTests - for testing REST clients
```

**@WebMvcTest** 声明来加载只包括了需要测试 web controller 的 bean 的应用上下文。

```txt
i.e. @Controller, @ControllerAdvice, @JsonComponent, Converter/GenericConverter, Filter, WebMvcConfigurer and HandlerMethodArgumentResolver beans but not @Component, @Service or @Repository beans
```

另外，**@WebMvcTest** 还自动激活、配置了 Spring Security和 MockMvc 。

```txt
By default, tests annotated with @WebMvcTest will also auto-configure Spring Security and MockMvc. 
```