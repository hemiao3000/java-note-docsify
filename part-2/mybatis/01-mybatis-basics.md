# <font color="orange">MyBatis 基础</font>

## 1. MyBatis

『**持久层**』可以将业务数据存储到磁盘，具备长期存储能力，只要磁盘不损坏，即便实在断电情况下，重新开启系统仍然可以读取到这些数据。

『**数据库系统**』是最常见的执行持久化工作的工具。

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO<small>（Plain Old Java Objects，普通老式 Java 对象）</small>为数据库中的记录。

MyBatis 的成功主要有 3 点：

- 不屏蔽 SQL，意味着可以更为精准地定位 SQL 语句，可以对其进行优化和改造。

- 提供强大、灵活的映射机制，方便 Java 开发者使用。提供了动态动态 SQL 的功能，允许使用者根据不同条件组装 SQL 语句。

- 在 MyBatis 中，提供了使用 Mapper 的接口编程，进一步简化了使用者的工作，使开发者能集中于业务逻辑，而非 Dao 层的编写。

MyBatis 的持久化解决方案将用户从原始的 JDBC 访问中解放出来，用户只需要定义需要操作的 SQL 语句，无须关注底层的 JDBC 操作，就能以面向对象的方式进行持久化层操作。底层数据库连接的获取、数据访问的实现、事务控制等都无须用户关心。


```xml
<!-- mysql 数据库驱动包 -->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>${mysql.version}</version> <!-- 8.0.21 -->
</dependency>

<!-- mybatis -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>${mybatis.version}</version> <!-- 3.5.1 -->
</dependency>
```



## 2. 基本概念

### 2.1 MyBatis 的体系结构

MyBatis 中的常用对象有 **SqlSessionFactory** 和 **SqlSession** 。

SqlSessionFactory 对象是 MyBatis 的关键对象，它对应着单个数据库。

```
XML 配置文件
└── SqlSessionFactoryBuilder
    └── SqlSessionFactory
        └── SqlSession
```

整个关系可以如下述这样『反推』：

- 最终是需要获得一个 **SqlSession** 对象来操作数据库。<small>SqlSession 对象代表着与数据库之间的连接。</small>

- 要『弄』到 **SqlSession** 对象，首先要先『弄』到一个 **SqlSessionFactory** 对象。

- 要『弄』到 **SqlSessionFactory** 对象，首先要先『弄』到一个 **SqlSessionFactoryBuilder** 对象。

- 而在这个整个过程中，需要用到 **`1 + N`** 个配置文件。

```java
// 这是一个相对于 classpath 的文件路径名。而且，不需要使用 / 。
InputStream is = Resources.getResourceAsStream("mybatis-config.xml");

SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(is);
SqlSession session = factory.openSession(true);

...
```

**注意**：使用完 SqlSession 之后『**关闭 Session 很重要**』，应该确保使用 **finally** 块来关闭它。

- 一个 MyBatis 应用程序只需要一个 **SqlSessionFactory** 的对象。因此，SqlSessionFactory 对象应该是『**单例对象**』。<small>在将 Mybatis 和 Spring 整合后，毫无疑问，**SqlSessionFactory** 单例对象的创建工作就交到了 Spring 手里。</small>
`
- **SqlSession** 是线程不安全的，所以 **SqlSession** 对象是非单例的。



### 2.2 使用 XML 构建 SqlSessionFactory

MyBatis 中的 XML 文件分为两类，一类是『**基础配置文件**』<small>（也叫『**核心配置文件**』）</small>，它只有一个。另一类是『**映射文件**』，它至少有一个。<small>合计是 `1 + N` 个配置文件</small>。

『**基础配置文件**』通常叫做 **mybatis-config.xml** 文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

<!-- 别名。非必须。
  <typeAliases>
    <typeAlias alias="dept" type="com.xja.scott.bean.Department"/>
  </typeAliases>
-->
  <!-- 数据库环境。必须。-->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC" />
        <dataSource type="POOLED">
        <property name="driver" value="com.mysql.cj.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/scott?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false&amp;serverTimezone=UTC"/>
        <property name="username" value="root" />
        <property name="password" value="123456" />
      </dataSource>
    </environment>
  </environments>

  <!-- 映射文件。必须。-->
  <mappers>
    <!--这是一个相对于 classpath 的路径名。另外，不需要使用 / 。 -->
    <mapper resource="mapper/DeptMapper.xml" />
  </mappers>

</configuration>
```

- **\<typeAlias>** 元素为一个类定义了一个别名，这样在后续使用该类时，可以直接使用别名，而不是它的完全限定名。

- **\<environment>** 元素描述了一个数据库相关信息。

    - 它里面的 **\<transactionManager>** 元素配置了『**事务管理器**』 ，这里采用的是 MyBatis 的 JDBC 管理器方式。

    - 它里面的 **\<dataSource>** 元素配置了数据库连接的相关信息，其中属性 **type="POOLED"** 表示采用 MyBatis 内部提供的连接池方式。

- **\<mapper>** 元素代表引入指定的 Mapper 配置文件。

为了加载 XML 配置文件来构建 **SqlSessionFactory** 对象。MyBaits 专门提供了 **Resources** 类来加载配置文件。

```java
String resource = "mybatis-config.xml";
SqlSessionFactory factory = null;
InputStream is = null;

try {
    is = Resources.getResourceAsStream(resource);
    factory = new SqlSessionFactoryBuilder().build(is);
} catch (IOException e) {
    e.printStackTrace();
}
```

**注意**，Mybatis 对核心配置文件中的内容<small>（子元素）</small> 出现的『**先后顺序有要求**』，你可以没有使用到某个子元素，但是如果你用到了，那么必须符合固定的先后顺序：


- properties（属性）
- settings（设置）
- typeAliases（类型别名）
- typeHandlers（类型处理器）
- objectFactory（对象工厂）
- plugins（插件）
- environments（环境配置）
  - environment（环境变量）
    - transactionManager（事务管理器）
    - dataSource（数据源）
- databaseIdProvider（数据库厂商标识）
- mappers（映射器）


### 2.3 SqlSession

**SqlSession** 是 MyBatis 的核心接口。**SqlSession** 的作用类似于 JDBC 中的 **Connection** 对象，代表着一个数据库的连接。

它的作用有 3 个：

- 获取 Mapper 接口。

- 发送 SQL 给数据库。

- 控制数据库事务。

有了 **SqlSessionFactory** 创建 **SqlSession** 就十分简单了：

```java
SqlSession sqlSession = factory.openSession();
// 相当于
SqlSession sqlSession = factory.openSession(false);
```

由此可见，SqlSession 默认『**未开启**』事务的自动提交<small>（autoCommit）</small>功能。因此需要程序员手动操作事务。

> 另外，如果在建表时，有意或无意使用的是 **MyIsam** 引擎，那么此处无论是 **true** ，或者 **false** ，都无法回滚，因为 **MyIsam** 数据库引擎本身就不支持事务功能<small>（这是它与 **InnoDB** 引擎的重要区别之一）</small>。
> 
> 对初学者而言，建表是错误地使用了数据库引擎，而导致『**事务不回滚**』的常见原因。

```java
SqlSession session = null;

try {
    session = factory.openSession();
    // some code ...
    session.commit();		// 提交事务
} catch (Exception e) {
    session.rollback();		// 回滚事务
} finally {
    if (session != null)
        session.close();	// 务必确保关闭 session
}
```

### 2.4 默认的别名

|     别名  |  Java 类型  | 是否支持数组   |       别名  | Java 类型   | 是否支持数组  |
| --------: | :---------- | :------------: | ----------: | :---------- | :------------: |
|    _byte  | byte       |      Y       |       byte | Byte       |      Y       |
|   _short  | short      |      Y       |      short | Short      |      Y       |
|     _int  | int        |      Y       |        int | Integer    |      Y       |
| _integer  | int        |      Y       |    integer | Integer    |      Y       |
|    _long  | long       |      Y       |       long | Long       |      Y       |
|   _float  | float      |      Y       |      float | Float      |      Y       |
|  _double  | double     |      Y       |     double | Double     |      Y       |
| _boolean  | boolean    |      Y       |    boolean | Boolean    |      Y       |
|  decimal  | BigDecimal |      Y       | bigdecimal | BigDecimal |      Y       |
|   string  | String     |      Y       |       date | Date       |      Y       |
|   object  | Object     |      Y       | collection | Collection |      —      |
|      map  | Map        |      ——    |    hashmap | HashMap    |      ——    |


### 2.5 补充

```xml
<environments default="...">
  <environment id="...">
    <transactionManager type="..."/>
      <dataSource type="...">
        <property name="driver" value="..."/>
        <property name="url" value="..."/>
        <property name="username" value="..."/>
      <property name="password" value="..."/>
    </dataSource>
  </environment>
</environments>
```

**\<transactionManager type="..."/>** 表示事务管理器配置，可选值有：**JDBC** 和 **MANAGED** 。

| 属性值  | 说明 |
| :------- | :--------|
| JDBC    | 这个配置表示 MyBatis 底层使用 JDBC 中的 Connection 对象进行事务的提交和回滚。|
| MANAGED | 这个配置表示 MyBatis 底层不进行任何事物的提交和回滚操作，而是由『别人』<small>（容器）</small>来进行事务的操作。<br> 不过，默认情况下它会关闭连接，而有些容器并不希望如此，<br>所以通常使用子元素 `<property name=closeConnection" value="false"/>` 来取消这种行为。|

在整合 Spring 和 MyBaits 时，不需要在此配置事务管理器，因为 Spring 会使用其自身的事务管理器来覆盖此处的配置。

**\<dataSource type="...">** 表示数据源配置，其可选值有：**UNPOOLED** 、**POOLED** 和 **JNDI** 。

| 属性值   | 说明                                                    |
|:---|:---------------------|
| UNPOOLED | 表示不使用连接池，因此每次请求都会打开/关闭连接。|
| POOLED   | 表示使用 MyBatis 内部的数连接池功能，此时在底层 Connection 对象会被复用。|
| JNDI     | 这表示这数据库连接由容器维护。使用较少。|

## 3. 执行 SQL 语句

Mapper 是 MyBatis 最强大的工具与功能，它用于执行 SQL 语句。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="...">
  <insert id="..."> ... </insert>
  <delete id="..."> ... </delete>
  <update id="..."> ... </update>
  <select id="..."> ... </select>
</mapper>
```

| 元素      | 描述                             | 备注                            |
|:--------- |:------------------------------- |:------------------------------- |
| select    | 查询语句，常用又复杂              | 可以自定义参数，返回结果集等      |
| insert    | 插入语句                         | 执行后返回一个整数，代表插入的条数 |
| update    | 更新语句                         | 执行后返回一个整数，代表更新的条数 |
| delete    | 删除语句                         | 执行后返回一个整数，代表删除的条数 |
| resultMap | 定义查询结果映射关系，常用又复杂   | 它将提供映射规则                  |



### 3.1. 增删改操作

#### 3.1.1 insert 元素

insert 元素的必要属性有 ：

| 元素名        | 说明 |
|:------------- | :--------------------------------------------------------- |
| id            | 和 Mapper 的 namespace 组合起来是唯一的，提供给 MyBatis 调用。|
| parameterType | 类的完全限定名，或内置/自定义的类的别名（Alias）。 <br> 可以向 SQL 传递 JavaBean 和 Map 等复杂参数类型，<small>但 Map 参数不建议使用</small>。|

MyBatis 在执行插入之后会返回一个 **整数**，以表示插入的记录数。

```xml
<insert id="insertDepartment" parameterType="com.xja.hemiao.bean.Department">
  INSERT INTO dept(dname, loc) VALUES(#{dname}, #{loc});
</insert>
```

> 在传递参数时，MyBatis 中可用的占位符有两种：**`#{}`** 和 **`${}`** 。
> - **`#{}`**：MyBatis 使用的是 JDBC 中的 **PreparedStatement** 。
> - **`${}`**：MyBatis 使用的是 JDBC 中的 **Statement** 。

```java
// 这是一个相较于 classpath 的文件路径名。而且，不需要使用 / 。
InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
SqlSession session = factory.openSession();

Department dept = new Department("Test", "BeiJing");
int n = session.insert("xxx.yyy.zzz.insertDepartment", dept);

System.out.println(n);      // 输出打印 1

session.commit();
session.close();
```

#### 3.1.2 insert 过程中的主键回填

大多数情况下，插入信息的主键是由数据库底层生成的，在插入数据后，我们往往需要这个主键，以便于未来的操作。为此，MyBatis 提供了主键回填功能。<small>这个功能需要数据库和数据库驱动支持，MyBatis 才能正常使用它。</small>

开启主键回填功能的 insert 必要属性：

| 属性             | 说明                                                               |
| :--------------- | :---------------------------------------------------------------  |
| useGeneratedKeys | 启用主键回填功能的开关属性。`"true"` |
| keyProperty      | 指定需要回填的 Bean 属性<small>（对应数据库主键列的那个属性）</small>  |

如果你有大量的 insert 都要用到主键回填功能，而你又觉得要在所有的这些 `<insert ...>` 中每一个都去写 `userGeneratedKeys="true"` 很麻烦，MyBatis 的核心配置文件中的 **`<settings>`** 中有一个全局设置，可以帮你批量开启所有的 `<insert ...>` 的主键回填功能：

```xml
<settings>
  ...
  <!-- 默认值是 false 。因此你要一个个地写。 -->
  <setting name="useGeneratedKeys" value="true" />
  ...
</settings>
```

> 注意 **\<settings>** 的位置。MyBatis 对配置文件的内容的先后顺序有要求。

#### 3.1.3 delete 元素 和 update 元素

和 insert 元素一样，MyBatis 执行完 update 元素 和 delete 元素后会返回一个整数，标示执行后影响的记录条数。

```xml
<update id="updateDepartment" parameterType="dept">
  UPDATE dept SET dname = #{dname}, loc = #{loc} WHERE deptno = #{deptno}
</update>

<delete id="deleteDepartment" parameterType="int">
  DELETE FROM dept WHERE deptno = #{deptno}
</delete>
```

```java
...
session.insert("xxx.yyy.zzz.insertDepartment", dept);
...
session.update("xxx.yyy.zzz.updateDepartment", dept);
...
session.delete("xxx.yyy.zzz.deleteDepartment", 41);
...
```







### 3.2 getMapper 方法

通过 SqlSession 的 insert、upate、delete 和 selectOne、selectList 方法可以去调用 Mapper.xml 中定义的配置文件，进而操作数据库。不过，MyBatis 提供了更『高端』的操作，『帮』程序员去实现 DAO 层代码。

如果将 Mapper.xml 配置文件的 namespace『故意』写的和一个 DAO 接口的完全路径名一样，并且该接口中的方法名有『碰巧』和 Mapp.xml 配置文件中的各个 SQL 语句的 id 值一样，那么 MyBatis 就会去为该接口动态生成一个实现类。

通过 SqlSession 的 **getMapper** 方法传入接口的类对象，就可以获得这个由 MyBatis 动态生成的 DAO 接口的实现类。

3 个『保持一致』：

- *Mapper.xml* 的 **namespace** 与接口的完全限定名保持一致。

- SQL 语句的 **id** 属性值与接口中的方法名保持一致。

- SQL 语句的 **parameterType** 属性值与接口的方法的参数类型保持一致。


```java
package xxx.yyy.zzz.dao;

public interface DepartmentDao {
    public List<Department> listDepartments();
    ...
}
```

```xml
<mapper namespace="xxx.yyy.zzz.dao.DepartmentDao"> <!-- 注意，此处是接口的完全限定名字。-->
  <select id="listDepartments" resultType="dept">
    SELECT * FROM dept
  </select>
  ...
</mapper>
```

```java
DepartmentDao dao = session.getMapper(DepartmentDao.class);

List<Department> list = dao.listDepartments();
```

### 3.3 查操作

#### 3.3.1 select 元素

通过 MyBatis 执行 SQL 后，MyBatis 通过其强大的映射规则，可以自动地将返回的结果集绑定到 JavaBean 中。

select 元素的必要属性有：

| 属性 | 说明 |
| :- | :- |
| id | 和 Mapper 的 namespace 组合起来必须唯一 |
| parameterType | 类的完全限定名，或内置/自定义的类的别名（Alias）<br>。可以向 SQL 传递 JavaBean 和 Map 等复杂参数类型。|
| resultType    | 类的完全限定名，查询结果将通过固定规范进行映射；<br>或者定义为 int、double、float 等参数。|
| resultMap     | resultType 的“高级版”，允许我们自定义映射规则。<br><small>**不能与 resultType 同时使用**。</small>|

#### 3.3.2 select 与 聚合函数

并不是所有的 select 语句都会返回一行，或多行记录。例如，在 select 中使用聚合函数。这种情况对于 MyBatis 而言最为简单，因为不需要将结果集映射成 JavaBean ，它只需要返回一行一列的单个数据。

```xml
<select id="getMaxSal" resultType="int">
  SELECT * FROM dept WHERE deptno = #{deptno}
</select>

<!-- 此处特意是 string 类型，以验证效果 -->
<select id="getEmployeeCount" resultType="string">
  SELECT count(empno) FROM emp;
</select>
```

```java
int n = session.selectOne("xxx.yyy.zzz.getMaxSal");
System.out.println(n);

String str = session.selectOne("xxx.yyy.zzz.getemployeeCount");
System.out.println(str);
```

MyBatis 可以很智能地将返回结果转换为你所指定的类型，如：int、String 等。


### 3.4 传递多个参数

多参数的传递有三种方法：

| 传递方式 | 说明 |
| :- | :- |
| 使用 Map 传参 | <font color="red">不建议使用</font>（已被 JavaBean 传参替代）|
| 使用 JavaBean 传参 | <font color="#0088dd">**大量多参**</font> 传递时使用 |
| 使用 注解 传参 |<font color="#0088dd">**少量多参**</font> 传递时使用 |


#### 3.4.1 使用 Map 传递多参数

MyBatis 支持 Map 对象作为参数，此时，要求 select 元素的 `parameterType` 值为 `map` 。

```java
List<Employee> selectBySal1(Map<String, Integer> salMap);
```

```xml
<select id="selectBySal1" parameterType="map" resultType="Employee">
  select * from emp where sal >= #{minSal} and sal &lt; #{maxSal}
</select>
```

```java
Map<String, Integer> map = new HashMap<>();
map.put("minSal", 1000);
map.put("maxSal", 2000);

List<Employee> list = dao.selectBySal1(map);
for (Employee emp : list) {
    log.info("{}", emp);
}
```

#### 3.4.2 使用 JavaBean 传递多参

由于 Map 的无语义性，因此官方 <font color="red">不建议使用</font> Map 传参！

此时，要求 `select` 元素的 `parameterType` 属性值为 JavaBean 的完全限定名（或别名）。

```java
@Data
public class SallaryRegion {
    private Integer minSallary;
    private Integer maxSallary;
}
```

```
List<Employee> selectBySal2(SallaryRegion region);
```

```xml
<select id="selectBySal2"
        parameterType="com.microboom.bean.po.SallaryRegion"
        resultType="Employee">
    select * 
    from emp 
    where sal >= #{minSallary} 
      and sal &lt; #{maxSallary}
</select>
```


#### 3.4.3 使用注解方式传递多参数

如果所有的多参数传递都通过定义并使用 JavaBean 来进行，那么项目中会出现大量的参数 JavaBean 的定义，显然这也并不太合理。

为此，Mybatis 提供了参数注解，以减少参数 JavaBean 的定义。

```java
List<Employee> selectBySal3(
    @Param("xxx") Integer minSallary,
    @Param("yyy") Integer MaxSallary);
```

```xml
<select id="selectBySal3" resultType="Employee">
  select * 
  from emp 
  where sal >= #{xxx} 
    and sal &lt; #{yyy}
</select>
```

`补充`，<small>MyBatis 框架的注解功能相对而言比较薄弱，官方推荐使用 XML 配置，而非注解，但是少量的多参数传递，是 <font color="#0088dd">必须使用注解</font> 的场景。</small>



