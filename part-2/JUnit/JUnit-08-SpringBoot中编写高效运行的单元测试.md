# <font color="orange">在 SpringBoot 中如何编写高效运行的单元测试</font>

在很多的项目中单元测试都极度匮乏，或许大家都未意识到单元测试的作用，亦或是懒得编写或是不会写。但是单元测试是必要的，可以这么说，程序员有责任编写功能代码，同时也就有责任为自己的代码编写单元测试，最终受益的也是程序员自己。

本篇文章通过深入研究 Spring 的测试框架，来探讨如何编写高效运行的单元测试。先来看功能代码，这是一个创建用户的接口<small>（省略无关代码）</small>：

```java
@Service
public class UsersServiceImpl implements UsersService {

    // ......
    @Override
    @Transactional
    public Users createUsers(CreateUsersReqVo reqVo, UserDetails userDetails) {
        ...
    }
}
```

## 1. 第一版针对 Service 的单元测试

```java
// 添加事务注解，则默认情况下测试方法执行完后事务会被回滚，结合 @commit 注解使用可让事务被提交
@Transactional
@RunWith(SpringRunner.class)          // for JUnit 4 
// @ExtendWith(SpringExtension.class) // for Junit 5 用这个
@SpringBootTest(classes = {
    CoreApplication.class
})
public class UsersServiceNormalTests {

    @Autowired
    private UsersService usersService;

//  添加此注解则事务不会回滚
//  @Commit
    @Test
    @Sql("classpath:sql-script/users.sql")
//  @Sql(scripts = "classpath:sql-script/xxx.sql", statements = "insert into users values (...)")
    public void createUsersTest() {
        CreateUsersReqVo reqVo = new CreateUsersReqVo();
        CreateUsersForm form = new CreateUsersForm();
        form.setUsername("jufeng98");
        form.setPassword("admin");
        form.setEmail("jufeng98@qq.com");
        form.setGender("M");
        reqVo.setCreateOrEditUsersForm(form);
        Users users = usersService.createUsers(reqVo, mockUserDetails());
        Assert.assertEquals(users.getUsername(), "jufeng98");
    }
}
```

注意 **@Transactional** 和 **@Sql** 注解的使用，这样就能让测试方法可以重复执行而不污染数据库。


## 2. 单独的测试配置

单元测试这样写目前没什么问题，但是随着业务的发展，我们的应用会变得越来越庞大，会引入各种各样的框架，如 Spring Cloud、Redis、RabbitMQ 和 ElasticSearch 等等，导致了我们应用启动越来越慢。而单元测试这里使用 @SpringBootTest 注解时，该注解相当于完整启动了整个应用，这会让执行测试类的耗时随着应用的变大而不断变长，在我接触的实际项目中有些项目启动就要耗时一分多钟，这意味着执行测试类也要耗时一分多钟，这就变得无法接受，所以此种写法不推荐。

理论上，如果我们仅仅只是需要测试 UsersService 类，那么我们可以不让 Spring 组装其他无关的 bean，这种情况下，我们只需要将 UsersService 类所依赖的 bean 组装起来就行了。

> 在这里，我们要重新捡起 SSM Java 代码配置那套东西。

所以，先编写一个 mybatis 的测试配置类：

```java
@TestConfiguration 
@MapperScan(basePackages = "com.woniu.example.department.outlet.dao.mysql")
@Profile("PROFILE_UNIT_TEST")
public class MybatisTestConfig {

    @Bean(destroyMethod = "close")
    public HikariDataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        ds.setJdbcUrl("jdbc:mysql://127.0.0.1:3306/scott?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC");
        ds.setUsername("root");
        ds.setPassword("123456");
        return ds;
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource ds) {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(ds);
        // factoryBean.setConfigLocation(new ClassPathResource("mybatis/mybatis-config.xml"));
        // factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("mybatis/mapper/*.xml"));
        return factoryBean;
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer configurer = new MapperScannerConfigurer();
        configurer.setBasePackage("com.woniu.example.departmentservice.outlet.dao.mysql");
        return configurer;
    }

    @Bean
    public DataSourceTransactionManager transactionManager(DataSource ds) {
        DataSourceTransactionManager manager = new DataSourceTransactionManager();
        manager.setDataSource(ds);
        return manager;
    }

}
```

接着还有其他的相关 bean 依赖以此类推来编写，最终得到的写法：

```java
@Transactional
@ContextConfiguration(classes = {
        DatasourceTestConfig.class,
        MybatisTestConfig.class,
        RedissonTestConfig.class,
        WebTestConfig.class,
        UsersServiceImpl.class
})
@ActiveProfiles("PROFILE_UNIT_TEST")
public class UsersServiceBestTests extends CommonTestCode {

    @Autowired
    private UsersService usersService;

    @Test
    // @Commit
    @Sql("classpath:sql-script/users.sql")
    public void createUsersTest() {
        CreateUsersReqVo reqVo = new CreateUsersReqVo();
        CreateUsersForm form = new CreateUsersForm();
        form.setUsername("jufeng98");
        form.setPassword("admin");
        form.setEmail("jufeng98@qq.com");
        form.setGender("M");
        reqVo.setCreateOrEditUsersForm(form);
        Users users = usersService.createUsers(reqVo, mockUserDetails());
        Assert.assertEquals(users.getUsername(), "jufeng98");
    }
}
```

此时执行测试类就非常快了，因为我们只组装了必要的 bean，只要 Service 依赖的东西不变，那么执行时间就基本不会有太多变化，避免了第一种写法的问题。

这种方式唯一的缺点就是需要清楚知道依赖了哪些 bean，并将他们组装起来，虽然麻烦了点，但这是值得的，避免了组装无关的bean，让测试类能快速启动执行。

## 3. 测试 Controller 

最后这里是针对 Service 层<small>（接口）</small>的测试，对于 Controller 层的测试是缺失的，所以为了能快速测试 Controller，我又研究了针对 Controller 的测试类，先来看看Controller的功能代码：

```java
@RestController
@RequestMapping("/admin/users")
public class UsersController {

    @Autowired
    private UsersService usersService;  

    /**
     * 创建用户
     */
    @PostMapping("/createUsers")
    public Result<Users> createUsers(@RequestBody CreateUsersReqVo reqVo,
                                     @AuthenticationPrincipal UserDetails userDetails) {
        return new Result<>(usersService.createUsers(reqVo, userDetails));
    }
}
```

针对 Controller 的测试类写法，如果使用 @SpringBootTest 注解，也有执行耗时随着应用的变大而不断变长的问题。我们可以通过自己组装 Spring 应用的上下文，而避免 @SpringBootTest 注解的 All In：

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = {
        DatasourceTestConfig.class,
        MybatisTestConfig.class,
        RedissonTestConfig.class,
        WebTestConfig.class,
        UsersServiceImpl.class,
        SecurityTestConfig.class,
        UsersController.class
})
// 下面这三个注解用于配置 SpringMVC 的测试上下文
// @AutoConfigureMockMvc
// @AutoConfigureWebMvc
@WebMvcTest // 等于上面两个
@ActiveProfiles("PROFILE_UNIT_TEST")
@Transactional
public class UsersControllerBestTests extends CommonTestCode {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @SneakyThrows
    @WithMockUser(
            username = "admin",
            password = "admin",
            authorities = "ROLE_ADMIN"
    )
    // @Commit
    @Sql("classpath:sql-script/users.sql")
    public void createUsersTest() {
        ObjectNode reqVo = objectMapper.createObjectNode();
        ObjectNode createOrEditUsersForm = reqVo.putObject("createOrEditUsersForm");

        createOrEditUsersForm.put("username", "jufeng98");
        createOrEditUsersForm.put("password", "admin");
        createOrEditUsersForm.put("email", "jufeng98@qq.com");
        createOrEditUsersForm.put("gender", "M");

        mockMvc.perform(post("/admin/users/createUsers")
                            .contentType(MediaType.APPLICATION_JSON_UTF8)
                            .content(objectMapper.writeValueAsString(reqVo))
                            .accept(MediaType.APPLICATION_JSON_UTF8))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(jsonPath("$.data.username").value("jufeng98"));
    }
}
```

所以，掌握此类写法就能让单元测试高效运行，于此同时，我们也可利用单元测试来快速调试代码，这也大大提高了我们的开发效率，可谓一举两得。

另外，Spring 也提供了两个非常有用的测试注解：**@MockBean** 和 **@SpyBean**，还有一个辅助类：**MockRestServiceServer** 。

下面依次介绍其用法，首先是 @MockBean，此注解会代理 bean 的所有方法，对于未 mock 的方法调用均是返回 null：

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = {
        WebTestConfig.class,
        DatasourceTestConfig.class,
        SecurityTestConfig.class,
        UsersController.class
})
@AutoConfigureMockMvc
@AutoConfigureWebMvc
@ActiveProfiles("PROFILE_UNIT_TEST")
public class MockBeanTests extends CommonTestCode {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private UsersService usersService;

    @Test
    @SneakyThrows
    @WithMockUser(
            username = "admin",
            password = "admin",
            authorities = "ROLE_ADMIN"
    )
    public void createUsersTest() {
        Users users = new Users();
        users.setUsername("jufeng98");
        // @MockBean 注解会代理 bean 的所有方法,对于未 mock 的方法调用均是返回null,这里的意思是针对调用createUsers方法
        // 的任意入参，均返回指定的结果
        given(usersService.createUsers(any(), any())).willReturn(users);

        ObjectNode reqVo = objectMapper.createObjectNode();
        ObjectNode createOrEditUsersForm = reqVo.putObject("createOrEditUsersForm");

        createOrEditUsersForm.put("username", "jufeng98");
        createOrEditUsersForm.put("password", "admin");
        createOrEditUsersForm.put("email", "jufeng98@qq.com");
        createOrEditUsersForm.put("gender", "M");

        mockMvc.perform(
            post("/admin/users/createUsers")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(objectMapper.writeValueAsString(reqVo))
                .accept(MediaType.APPLICATION_JSON_UTF8)
        )
        .andDo(print())
        .andExpect(status().isOk())
        .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
        .andExpect(jsonPath("$.data.username").value("jufeng98"));
    }
}
```

@SpyBean 可达到部分 mock 的效果，未被 mock 的方法会被真实调用：

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = { 
        MybatisTestConfig.class,
        WebTestConfig.class,
        DatasourceTestConfig.class,
        SecurityTestConfig.class,
        UsersController.class,
        UsersServiceImpl.class
})
@AutoConfigureMockMvc
@AutoConfigureWebMvc
@ActiveProfiles("PROFILE_UNIT_TEST")
public class SpyBeanTests extends CommonTestCode {

    @Autowired
    private MockMvc mockMvc;
    @Autowired
    private ObjectMapper objectMapper;

    @SpyBean
    private UsersService usersService;

    @Test
    @SneakyThrows
    @WithMockUser(
            username = "admin",
            password = "admin",
            authorities = "ROLE_ADMIN"
    )
    public void createUsersTest() {
        Users users = new Users();
        users.setUsername("jufeng98");
        // @SpyBean可达到部分mock的效果,仅当 doReturn("").when(service).doSomething() 时，doSomething方法才被mock，
        // 其他的方法仍被真实调用。
        // 未发生实际调用
        doReturn(users).when(usersService).createUsers(any(), any());

        ObjectNode reqVo = objectMapper.createObjectNode();
        ObjectNode createOrEditUsersForm = reqVo.putObject("createOrEditUsersForm");

        createOrEditUsersForm.put("username", "jufeng98");
        createOrEditUsersForm.put("password", "admin");
        createOrEditUsersForm.put("email", "jufeng98@qq.com");
        createOrEditUsersForm.put("gender", "M");

        mockMvc
                .perform(
                        post("/admin/users/createUsers")
                                .contentType(MediaType.APPLICATION_JSON_UTF8)
                                .content(objectMapper.writeValueAsString(reqVo))
                                .accept(MediaType.APPLICATION_JSON_UTF8)
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(jsonPath("$.data.username").value("jufeng98"));
    }
}
```

最后是 **MockRestServiceServer** 类，用于 mock 使用 RestTemplate 调用 http 接口的返回，假设我们有个接口是这样的，使用了 RestTemplate 调用 http 接口获取信息：

```java
@Validated
@RestController
@RequestMapping("/admin/test")
public class TestController {
    @Autowired
    private TestService testService;
    @PostMapping("/getOrderPayType")
    public Result<String> getOrderPayType(@RequestBody JsonNode jsonNode) {
        return new Result<>(testService.getOrderPayType(jsonNode.get("orderCode").asText()));
    }
}

@Service
public class TestServiceImpl implements TestService {

    @Autowired
    private RestTemplate restTemplate;

    @Override
    public String getOrderPayType(String orderCode) {
        JsonNode jsonNode = restTemplate.getForObject("http://b2c-cloud-order-service/getOrderPayType?orderCode={1}", JsonNode.class, orderCode);
        return Objects.requireNonNull(jsonNode).get("payType").asText();
    }
}
```

那么单元测试就可以这样写：

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = {
        DatasourceTestConfig.class,
        SecurityTestConfig.class,
        WebTestConfig.class,
        TestController.class,
        TestServiceImpl.class
})
@AutoConfigureMockMvc
@AutoConfigureWebMvc
@WebAppConfiguration
@ActiveProfiles("PROFILE_UNIT_TEST")
public class MockRestServiceServerTests extends CommonTestCode {

    @Autowired
    protected MockMvc mockMvc;

    @Autowired
    private RestTemplate restTemplate;

    @Test
    @WithMockUser(
            username = "admin",
            password = "admin",
            authorities = "ROLE_ADMIN"
    )
    @SneakyThrows
    public void test() {
        MockRestServiceServer server = MockRestServiceServer.bindTo(restTemplate).build();
        // mock http接口的返回
        server
                .expect(requestTo("http://b2c-cloud-order-service/getOrderPayType?orderCode=C93847639357"))
                .andRespond(withSuccess("{\"orderCode\":\"C93847639357\",\"payType\":\"alipay\"}", MediaType.APPLICATION_JSON_UTF8));

        mockMvc
                .perform(
                        post("/admin/test/getOrderPayType")
                                .contentType(MediaType.APPLICATION_JSON_UTF8)
                                .content("{\"orderCode\":\"C93847639357\"}")
                                .accept(MediaType.APPLICATION_JSON_UTF8)
                )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(jsonPath("$.data").value("alipay"));
    }

}
```

最后 SpringBoot 还提供了大量的各类辅助测试的注解例如 @JdbcTest、@DataRedisTest、@DataJpaTest 等等，大家有兴趣可以去研究。


附上 Spring Testing 和 SpringBoot Testing 的官方文档：

- [spring Testing](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing)

- [spring-boot Testing](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-testing.html)














