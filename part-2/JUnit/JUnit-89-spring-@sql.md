# <font color="orange">Spring 中的 @Sql 注解</font>

> 大概我们在别处也用不上 Spring 的 @Sql 注解，主要也就是在单元测试中它才有用武之地。

## @Sql 注解

**@Sql** 注解可以执行 SQL 脚本，也可以执行 SQL 语句。它既可以加上类上面，也可以加在方法上面。

默认情况下，方法上的 @Sql 注解会覆盖类上的 @Sql 注解。<small>但可以通过 @SqlMergeMode 注解来修改此默认行为。</small>

**@Sql** 有下面的属性：

| 属性 | 说明 |
| :-             | :- |
| config         | 与注解 @SqlConfig 作用一样，用来配置 `注释前缀`、`分隔符` 等。 |
| executionPhase | 决定 SQL 脚本或语句什么时候会执行，默认是 BEFORE_TEST_METHOD 。|
| **statements** | 配置要一起执行的 SQL 语句。                                    |
| **scripts**    | 配置 SQL 脚本路径。                                            |
| value             | scripts 的别名，它不能和 scripts 同时配置，但 statements 可以。|

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

-   drop_schema.sql ：

    ```sql
    drop table if exists student; 
    ```
-   create_schema.sql ：

    ```sql
    CREATE TABLE student (
        id INT NOT NULL,
        name VARCHAR(50) NOT NULL,
        PRIMARY KEY(id)
    ); 
    ```
-   insert_data1.sql ：

    ```sql
    insert into student(id, name) values (101, 'Mohan');
    insert into student(id, name) values (102, 'Krishna'); 
    ```

-   insert_more_data1.sql ：

    ```sql
    insert into student(id, name) values (103, 'Indra');
    insert into student(id, name) values (104, 'Chandra'); 
    ```

## @SqlConfig（了解）

@SqlConfig 用于配置如何去解释 @Sql 注解中指定的 Sql 脚本。

> @SqlConfig 用于配置 @Sql 的一些个性化的行为，通常，我们不会那么有个性。

@SqlConfig 可以用于类上，也可以用于方法上。

> @Sql 注解也有一个 config 属性，作用与 @SqlConfig 相同，不同的是作用域只在对应的 @Sql 注解范围。它的优先级也大于类注解的 @SqlConfig 。

| 属性 | 说明 |
| :- | :- |
| blockCommentStartDelimiter | 多行注释开始字符，默认是 `/*` 。 |
| blockCommentEndDelimiter | 多行注释结束字符，默认是 `*/` 。 |
| commentPrefix | 单行注释前缀，默认是 `–` 。|
| commentPrefixes | 指定多个单行注释前缀，默认是 `["–"]` |
| dataSource | 指定脚本执行的数据库的名字，只有在多个数据源时需要指定 |
| encoding | 指定 sql 脚本文件的字符编码。|
| errorMode | 配置错误模式，默认是 SqlConfig.ErrorMode 的 DEFAULT |
| separator | 配置脚本语句分隔符，默认是 `\n` |
| transactionManager | 指定 transactionManager bean，只有有多个 transactionManager 时需要指定 |
| transactionMode | 指定脚本执行的事务模式，默认是 SqlConfig.TransactionMode 的 DEFAULT|

例子：

```java
@SqlConfig(commentPrefix = "#")
@Sql({ "/drop_schema.sql", "/create_schema.sql" })
@Sql(scripts = { "/insert_data2.sql" })
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = AppConfig.class)
public class SqlConfigTest {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    public void fetchRows1() {
        List<Map<String, Object>> students = jdbcTemplate.queryForList("SELECT * FROM student");
        assertEquals(2, students.size());
    }

    @Sql(scripts = "/insert_more_data2.sql", config= @SqlConfig(commentPrefix = "~"))
    @Test
    public void fetchRows2() {
        List<Map<String, Object>> students = jdbcTemplate.queryForList("SELECT * FROM student");
        assertEquals(4, students.size());
    }
}
```

insert_data2.sql ：

```sql
-- Insert initial data
insert into student(id, name) values (101, 'Mohan');
insert into student(id, name) values (102, 'Krishna'); 
```

## @SqlMergeMode（了解）

@SqlMergeMode 可以加在类上，也可以加在方法上。用于指示方法上的 @Sql 和类上 @Sql 注解配置是否合并。方法上的 @SqlMergeMode 注解优先级更高。默认值是 SqlMergeMode.MergeMode 的 OVERRIDE 。

> 和 @SqlConfig 注解一样，@SqlMergeMode 注解也是用于配置 @Sql 的一些个性化的行为，通常，我们不会那么有个性。

```java
@SqlMergeMode(MergeMode.MERGE)
@Sql({ 
    "/drop_schema.sql", 
    "/create_schema.sql", 
    "/insert_data1.sql" 
})
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = AppConfig.class)
public class SqlMergeModeTest {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Sql(statements = "insert into student(id, name) values (100, 'Shiva')")    
    @Test
    public void fetchRows1() {
        List<Map<String, Object>> students = jdbcTemplate.queryForList("SELECT * FROM student");
        assertEquals(3, students.size());
    }

    @SqlMergeMode(MergeMode.OVERRIDE)    
    @Sql("/insert_more_data1.sql")
    @Test
    public void fetchRows2() {
        List<Map<String, Object>> students = jdbcTemplate.queryForList("SELECT * FROM student");
        assertEquals(5, students.size());
    }
} 
```


