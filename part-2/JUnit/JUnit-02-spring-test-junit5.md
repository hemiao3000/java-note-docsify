# <font color="orange">Spring 整合 JUnit5</font>

## 1. 基本使用

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>${spring.version}</version>
  <scope>test</scope>
</dependency>

<dependency><!--偷懒，一口气引入『一堆』junit5 相关依赖 -->
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.6.2</version>
  <scope>test</scope>
</dependency>
```

spring-text 并不是一个『新』的测试框架！它和 JUnit 的关系类似于 slf4j 和 logback，我们是直接使用 spring-text，间接使用 JUnit ，在整个测试工作中真正『干活』的仍然是 JUnit 。

之所以这么『绕』一下再使用 JUnit 是因为我们直接使用 JUnit 的话，我们至少有两件事情是需要亲自去干：

1. 手动加载 `.xml` 配置文件，构建 Spring IoC 容器。

2. 手动向 Spring『要』Spring IoC 容器中的单例对象。

例如：

```java
/**
 * 麻烦的写法
 */
public class Test1 {

  @Test
  public void demo() {
    ApplicationContext context = new ClassPathXmlApplicationContext("demo1.xml");

    DataSource ds = context.getBean(DataSource.class);
    ds.getConnection();
    ...
  }
}
```

如果我们是直接使用 spring-text，而间接使用 JUnit，那么至少上述两件事情我们就不用『手动』处理了。

```java
@ExtendWith(SpringExtension.class) // ①
@ContextConfiguration("classpath:demo1.xml")  // ②
public class Test2 {

  @Autowired // ③
  private DataSource ds;

  @Test
  public void demo() throws SQLException {
    ds.getConnection();
    ...
  }
}
```

上述 spring-test 代码有 3 处注解：

1. **@ExtendWith** 指定使用 Spring JUnit 的测试运行器，也就是『真正』干活的那个测试框架。<small>上例中指定为 JUnit，如果有特殊需要和偏好，你也可以指定为别的测试框架，例如 TestNG 。</small>

2. **@ContextConfiguration** 注解指定测试用的 Spring 配置文件的位置。根据你的配置，spring-test 会去加载该文件构建 IoC 容器。

3. **@Autowired** 任何 Spring IoC 容器中有的，你所需要的对象，你都可以要求 spring-test 注入到你的测试类的属性中。

> 注意，在 Maven 项目中，spring-test 代码加载的是 test 配套的资源文件，即，`项目 > src > test > resources` 下的配置文件。

另外，上述代码还有一个可优化之处：使用 **@SpringJUnitConfig** 注解，一个顶俩。

```java
// @ExtendWith(SpringExtension.class)
// @ContextConfiguration("classpath:demo.xml")
@SpringJUnitConfig(locations = "classpath:demo.xml")
```

## 2. spring-test 利用事务避免污染数据库

如果涉及测试增删改的 DAO 方法，或者是测试涉及这些 DAO 的 Service 方法，每一个 Test 方法执行结束后都会对数据库造成影响，从而极大可能影响后续 Test 方法的执行。<small>（因为对于第二个 Test 方法而言，数据库环境发生了变化，初始条件可能就已经不满足了）</small>。

为此，spring-test 利用事务的回滚可以在 Test 方法执行结束后，撤销 Test 方法对数据库造成的影响。注意，你的 spring 配置文件中，必须有相关的事务<small>（spring-tx）</small>配置部分。即，必须引入 `spring-service.xml` 配置文件

- **@Transactional**：表示启用事务。

- **@Rollback**：表示本方法执行结束后事务回滚。

```java
@Test
@Transactional
@Rollback
public void demo2() {
    Department dept = departmentDao.selectByPrimaryKey(40);
    log.info("{}", dept);

    dept.setDname(dept.getDname() + ".");
    departmentDao.updateByPrimaryKey(dept);

    dept = departmentDao.selectByPrimaryKey(40);
    log.info("{}", dept);
}
```

`@Transactional` 和 `@Rollback` 还可以直接标注在类上，相当于对测试类下的所有的测试方法进行了统一设置。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {
    "classpath:spring/spring-dao.xml", 
    "classpath:spring/spring-service.xml"
})
@Transactional
@Rollback
public class AppTest {
    ...
}
```

`@Transactional` 和 `@Rollback` 不仅对 Dao 测试的回滚有效，对 Service 方法的测试，甚至是 Web 方法的测试也都有效。

<br>

---

<br>

初始化数据库可以借助于 `spring-dao.xml` 中 `<jdbc:initialize-database>` 配置：

```xml
<jdbc:initialize-database data-source="dataSource"  ignore-failures="DROPS">  
    <jdbc:script location="classpath:sql/schema.sql" encoding="UTF-8"/>  
    <jdbc:script location="classpath:sql/import-data.sql" encoding="UTF-8" />  
</jdbc:initialize-database>  
```

`jdbc:initialize-database` 这个标签的作用是在工程启动时，去执行一些 sql，也就是初始化数据库。比如向数据库中建立一些表及插入一些初始数据等。这些 sql 的路径需要在其子标签 `jdbc:script` 中去指定。

> <small>在这里，我们通过 jdbc:initialize-database 去初始化 H2 数据库，在测试代码期间，我们又通过 spring-tx 的回滚机制，确保各个测试方法不会对数据库中的数据造成实质性的修改，从而保证测试环境的干净。</small>

<big>**`jdbc:initialize-database` 标签**</big>

| 属性 | 说明 |
| :- | :- |
| `dataSource` | 引一个配置好的数据库连接池 |
| `ignore-failures="NONE"` | 不忽略任何错误，有错即终止执行 |
| `ignore-failures="DROPS"` | 忽略因表不存在而导致的删表错误 |
| `ignore-failures="ALL"` | 忽略任何错误 |

