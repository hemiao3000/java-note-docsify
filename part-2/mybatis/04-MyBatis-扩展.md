# <font color="orange"> MyBatis 扩展</font>

## 1. 一个个人习惯引发的问题

### 1.1 有些人的个人习惯

由于 mybatis 的 mapper 映射文件和 dao 的一一对应关系，有些人习惯将 mapper 映射文件<small>（ *.xml* ）</small>和 dao 接口放在一起，并且命名为统一的名字。例如：

```
src
└── main
    └── java
        └── xxx
            └── yyy
                └── zzz
                    └── dao
                        ├── XxxDAO.java
                        ├── XxxDAOMapper.xml
                        ├── YyyDAO.java
                        ├── YyyDAOMapper.xml
                        ├── ZzzDAO.java
                        └── ZzzDAOMapper.xml
```

通常认为这不是一个很好的习惯，因为在 Maven 项目中，**.xml** 和 **.properties** 一类的资源文件应该统一放在 **resources** 目录下，而非放在代码目录<small>（ *java* ）</small>下。

但是这么干也是有好处的，因为 DAO 和 DAOMapper 在一起，很方便一起找到它们俩，开发者多少还是能省点事的。



### 1.2 问题及其原因

按照上述结构来组织代码时，在 spring 和 mybatis 的相关配置都正确的情况下，你运行项目会失败，报错信息会告诉你项目中的 Mapper<small>（ *.xml* ）</small>文件找不到！

> 如果你对 **classpath** 的概念很熟悉的话，那么你很显然会立刻想到到哪里去看、去验证是否真的没有 mapper 文件。

这里的原因在于，对于代码文件夹<small>（ *java* ）</small>目录下的文件，maven 默认只对 *.java* 文件做相关工作：编译成 *.class* 文件，并『搬』到 classpath 下。对于代码文件夹<small>（ *java* ）</small>目录下的非 *.java* 文件，maven 默认一概视而不见，不会帮你把它们『搬』到 classpath 下，就好似这些非 *.java* 文件不存在。

此时，需要你通过配置『告诉』maven 代码文件夹<small>（ *java* ）</small>目录，也承担资源文件夹<small>（ *resources* ）</small>的功能，要求 maven 帮我们『搬』文件。


### 1.3 解决办法

在 *pom.xml* 文件的 **\<build>...\</build>** 中添加如下配置：

```xml
<resources>
    <resource>
       <directory>src/main/java</directory>
       <includes><!-- 你可以试试去掉这一段的效果 -->
            <include>**/*.xml</include>
       </includes>
    </resource>
    <resource>
       <directory>src/main/resources</directory>
    </resource>
</resources>
```


## 2. 注解

> 首先需要说明的是通过注解的方式虽然省略掉了 mapper 配置文件，但是注解的局限性很大。主要体现在使用 Example 对象实现通用的条件查询。
>
> 虽然，mapper 配置文件很繁琐，但是 mybatis 官方提供了 mybatis-generator 来帮我们生成 mapper 配置文件，实际上需要人亲自干的事情并不多。
> 
> 但是使用注解时，我们需要将 mapper 配置文件中的动态 SQL 语句以代码的形式手工重写一遍，这个工作就繁琐得令人讨厌了。

### 2.1 简单使用

在 MyBatis 的核心配置文件中，你需要配置的不是 mapper 映射文件，而是 Mapper 接口所在的包路径。

```xml
<!-- 在配置文件中 关联包下的 接口类-->
<mappers>
    <package name="com.example.dao"/>
</mappers>
```

另外，我们也不再需要 mapper 映射文件。对于 DAO 中的方法所对应的 SQL 语句，我们直接以注解的形式标注在方法上。

```java
public interface DepartmentMapper {

    @Select("select * from dept where deptno = #{id}")
    Department selectByPK(int id);

    @Select("select * from dept")
    List<Department> select();

    @Delete("delete from dept where deptno = #{id}")
    int delete(int id);

    @Insert("insert into dept values(NULL, #{name}, #{location})")
    @Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "deptno")
    int insert(Department dept);
}
```

上述代码中的注解很好理解。唯一需要注意的是，如果在执行 insert 语句时，需要启用 MyBatis 的主键回填功能，需要多使用一个 **@Options** 注解。


### 2.2 结果集映射

我们在使用 MyBatis 不可能都是遇到最简单的情况：表的列名与类的属性名一致。当表的列明与类的属性名不一致时，需要去配置结果集映射。

通过注解进行结果集的映射是通过使用 **@Results**、**@Result** 和 **@ResultMap** 注解完成的。其中，

- **@Results** 和 **@Result** 结合使用进行结果集映射；

- **@ResultMap** 则是在别处『调用』映射规则。

- **@Results** 和 **@Result** 只需要配置一次，而 **@ResultMap** 会在多出使用。

例如：

```java
@Select("select * from dept where deptno=#{id}")
@Results(id = "department", value = {
        @Result(property = "id", column = "deptno"),
        @Result(property = "name", column = "name"),
        @Result(property = "location", column = "loc")
})
public Department selectByPK(int id);

@Select("select * from dept")
@ResultMap("department")
public List<Department> select();
```



### 2.3 关系映射

一对一、一对多和多对多的关系映射就是在结果集映射的基础上再使用 `@One` 和 `@Many` 注解。

```java
@Select("select * from emp where empno=#{id}")
@Results(id = "employee", value = {
        @Result(property = "empno", column = "empno"),
        @Result(property = "ename", column = "ename"),
        @Result(property = "job", column = "job"),
        @Result(property = "mgr", column = "mgr"),
        @Result(property = "hiredate", column = "hiredate"),
        @Result(property = "sal", column = "sal"),
        @Result(property = "comm", column = "comm"),
        @Result(property = "dept", column = "deptno", one = @One(select = "dao.DepartmentMapper.selectByPK"))
})
public Employee selectByPK(int id);

@Select("select * from emp where deptno = #{id}")
@ResultMap("employee")
public List<Employee> selectByEmployeeID(int deptno);
```

```java
@Select("select * from dept where deptno=#{id}")
@Results(id = "department", value = {
        @Result(property = "id", column = "deptno"),
        @Result(property = "name", column = "deptno"),
        @Result(property = "location", column = "loc"),
        @Result(property = "employeeList", column = "deptno", many = @Many(select = "dao.EmployeeMapper.selectByDepartmentID"))
})
public Department selectByPK(int id);
```

### 2.4 @SelectProvider 单独提供 SQL 语句

@SelectProvider 功能就是用来单独写一个类与方法，用来提供一些 XML 或者注解中不好写的 SQL 。

写一个简单的 @SelectProvider 的用法，新建类，添加一个根据 userId 查询 User 的方法。

```sql
public class MySelectSqlProvider {

    public String selectByUserId(Long id) {
        StringBuffer buffer = new StringBuffer();
        buffer.append("SELECT * FROM user where ");
        buffer.append("id = ").append(id).append(";");
        return buffer.toString();
    }
}
```

MySelectSqlProvider 中提供了一个很简单的查询方法，根据 userId 返回 User 对象，里面就是用了一个 StringBuffer 对象来拼接一个 SQL 语句。更多、更优雅的写法是通过 MyBatis 中的 SQL Builder 的拼接一个 SQL 语句。SQL Builder 写法在[官方网站地址](http://www.mybatis.org/mybatis-3/zh/statement-builders.html) 。

UserMapper/UserDao 中使用它:

```java
@ResultMap("BaseResultMap")
@SelectProvider(type = MySelectSqlProvider.class, method = "selectByUserId")
User getUserByUserId(long id);
```

需要说的就是这一个，这一个方法在 XML 中没有对应的 SQL，在该方法上也没有 @Select 注解修饰，只有 @SelectProvider 注解，@SelectProvider 中两个属性，type 为提供 SQL 的类，method 为指定方法。


另外，除了有 @SelectProvider 之外，还有 @InsertProvider、@UpdateProvider、@DeleteProvider 。












### 2.4 常用功能注解汇总

| 注解 | 目标 | 相对应的 XML | 描述 |
| :-  | :-   | :- | :- |
| **@Param** | 参数 | N/A | 如果你的映射器的方法需要多个参数，这个注解可以被应用于映射器的方法参数来给每个参数一个名字。<br>否则，多参数将会以它们的顺序位置来被命名<small>（不包括任何 RowBounds 参数）</small>比如。*#{param1}* , *#{param2}* 等，这是默认的。<br>使用 **@Param("person")**，参数应该被命名为 **#{person}**。|
| **@Insert** <br><br> **@Update** <br><br> **@Delete** <br><br> **@Select** | 方法 | **\<insert>** <br><br> **\<update>** <br><br> **\<delete>** <br><br> **\<select>** | 这些注解中的每一个代表了执行的真实 SQL。它们每一个都使用字符串数组<small>（或单独的字符串）</small>。<br><br>如果传递的是字符串数组，它们由每个分隔它们的单独空间串联起来。|
| **@Results** | 方法 | **\<resultMap>** | 结果映射的列表，包含了一个特别结果列如何被映射到属性或字段的详情。属性有 *value*，*id* 。<br>**value** 属性是 Result 注解的数组。<br>**id** 的属性是结果映射的名称。|
| **@Result** |  N/A | **\<result>** <br> **\<id>** | 在列和属性或字段之间的单独结果映射。属性有 id，column，property，javaType，jdbcType，typeHandler，one，many。<br>**id** 属性是一个布尔值，表示了应该被用于比较<small>（和在 XML 映射中的 `<id>` 相似）</small>的属性。<br>**one** 属性是单独的联系，和 **\<association>** 相似 , 而 **many** 属性是对集合而言的 , 和 **\<collection>** 相似。|
| **@ResultMap** | 方法 | N/A | 这个注解给 **@Select** 或者**@SelectProvider** 提供在 XML 映射中的 **\<resultMap>** 的id。<br>这使得注解的 select 可以复用那些定义在 XML 中的 ResultMap。<br>如果同一 select 注解中还存在 **@Results** 或者 **@ConstructorArgs** ，那么这两个注解将被此注解覆盖。|
| **@One** | N/A | **\<association>** | 复杂类型的单独属性值映射。属性有 **select**，已映射语句<small>（也就是映射器方法）</small>的完全限定名，它可以加载合适类型的实例。<br>注意：联合映射在注解 API 中是不支持的。这是因为 Java 注解的限制，不允许循环引用。<br> **fetchType** 会覆盖全局的配置参数 **lazyLoadingEnabled** 。|
| **@Many** | N/A | **\<collection>** | 映射到复杂类型的集合属性。属性有 select，已映射语句<small>（也就是映射器方法）</small>的全限定名，它可以加载合适类型的实例的集合，**fetchType** 会覆盖全局的配置参数 **lazyLoadingEnabled** 。 注意联合映射在注解 API 中是不支持的。这是因为 Java 注解的限制，不允许循环引用。 |
| **@InsertProvider** <br><br> **@UpdateProvider** <br><br> **@DeleteProvider** <br><br> **@SelectProvider** | 方法 | **\<insert>** <br><br> **\<update>** <br><br> **\<delete>** <br><br> **\<select>** | 这些可选的 SQL 注解允许你指定一个类名和一个方法在执行时来返回运行允许创建动态的 SQL。基于执行的映射语句，MyBatis 会实例化这个类，然后执行由 provider 指定的方法。<br> You can pass objects that passed to arguments of a mapper method, "Mapper interface type" and "Mapper method" via theProviderContext(available since MyBatis 3.4.5 or later) as method argument. (In MyBatis 3.4 or later, it's allow multiple parameters) <br>属性有 **type** ，**method** 。**type** 属性是类。**method** 属性是方法名。|


### 2.5 mybatis-generator 对注解的支持

mybatis-generator 除了支持自动生成 XML 文件之外，也支持自动生成基于注解的 Mapper 接口。

只需要将 mybatis-generator 的配置文件中的 `<javaClientGenerator ... type="...">` 的属性的值从 XMLMAPPER 改为 **ANNOTATEDMAPPER** 即可：

```xml
<javaClientGenerator targetProject="src/main/java"
                            targetPackage="xxx.yyy.zzz.dao.mysql"
                            type="ANNOTATEDMAPPER"> <!-- 主要就是这一项有变化 -->
```

另外，配置文件中所有与 mapper 映射文件有关的内容就都可以移除了。

## 3. MyBatis<small>（3.4.5 以下）</small>中使用 LocalDateTime

MyBatis 从 **3.4.5** 版本开始就完全支持这种类型了，不再需要自己去写类型转换。因此以下内容仅作了解。

### 3.1 背景

项目中使用 MySQL 数据库，然后用 mybatis 做数据持久化。最近使用时，想把一些 model 类的 gmtCreate、gmtModified 等字段从 **java.util.Date** 改成 Java 8 的 **java.time.LocalDateTime**，此类是不可变类，且自带很好用的日期函数 api 。

原本依赖如下：

``` 
compile "org.mybatis:mybatis:3.3.0"
compile "org.mybatis:mybatis-spring:1.2.5"
compile "org.mybatis.generator:mybatis-generator-core:1.3.2"
```

直接修改代码有两个问题：

1. mapper、model、xml 文件都是用 generator 插件生成，维护成本高。

2. 会报错如下：

   ```java
   Caused by: java.lang.IllegalStateException: No typehandler found for property createTime
     at org.apache.ibatis.mapping.ResultMapping$Builder.validate(ResultMapping.java:151)
     at org.apache.ibatis.mapping.ResultMapping$Builder.build(ResultMapping.java:140)
     at org.apache.ibatis.builder.MapperBuilderAssistant.buildResultMapping(MapperBuilderAssistant.java:382)
     at org.apache.ibatis.builder.xml.XMLMapperBuilder.buildResultMappingFromContext(XMLMapperBuilder.java:378)
     at org.apache.ibatis.builder.xml.XMLMapperBuilder.resultMapElement(XMLMapperBuilder.java:280)
     at org.apache.ibatis.builder.xml.XMLMapperBuilder.resultMapElement(XMLMapperBuilder.java:252)
     at org.apache.ibatis.builder.xml.XMLMapperBuilder.resultMapElements(XMLMapperBuilder.java:244)
     at org.apache.ibatis.builder.xml.XMLMapperBuilder.configurationElement(XMLMapperBuilder.java:116)
   ```

### 3.2 修改

首先，添加了以下依赖：（jsr310 是 java 规范 310）

```
compile "org.mybatis:mybatis-typehandlers-jsr310:1.0.2"
```

接着，在 *`mybatis-config.xml`* 中添加如下配置：（该配置来源于[官网](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)）

```xml
<typeHandlers>
  <!-- ... -->
  <typeHandler handler="org.apache.ibatis.type.InstantTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.LocalDateTimeTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.LocalDateTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.LocalTimeTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.OffsetDateTimeTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.OffsetTimeTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.ZonedDateTimeTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.YearTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.MonthTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.YearMonthTypeHandler" />
  <typeHandler handler="org.apache.ibatis.type.JapaneseDateTypeHandler" />
</typeHandlers>
```

看了文档发现需要 *`mybatis 3.4.0`* 版本

不过这里有个问题了，会出现版本不兼容问题，报错如下：

```
java.lang.AbstractMethodError: org.mybatis.spring.transaction.SpringManagedTransaction.getTimeout()Ljava/lang/Integer;
    at org.apache.ibatis.executor.SimpleExecutor.prepareStatement(SimpleExecutor.java:85)
    at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:62)
    at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:325)
    at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:156)
    at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:109)
    at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:83)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
```

查看发现 *`SpringManagedTransaction`* 未实现 *`Transaction`* 中的 *`getTimeout()`* 方法。

```java
public interface Transaction {
    Connection getConnection() throws SQLException;
    void commit() throws SQLException;
    void rollback() throws SQLException;
    void close() throws SQLException;
} 
```

查了文档发现：Spring 版本跟 Mybatis 版本有个对照表要满足，详情见[官网](http://mybatis.org/spring/zh/)

而我需要的 `mapper.xml` 文件类似如下：

```xml
<mapper namespace="com.my.MyMapper">
  <resultMap id="BaseResultMap" type="com.my.MyModel">
    <id column="id" jdbcType="BIGINT" property="id"/>
    <result column="gmt_create" jdbcType="OTHER" property="gmtCreate" typeHandler="org.apache.ibatis.type.LocalDateTimeTypeHandler"/>
 
  </resultMap>
  <!-- 省略 -->
</mapper>
```

之后，查了 generator 的官方文档，发现还是可以通过 *`generator.xml`* 的配置文件做到的。

```xml
<table tableName="my_table" domainObjectName="MyModel">
  <generatedKey column="id" sqlStatement="JDBC" identity="true"/>
  <columnOverride column="gmt_create" property="gmtCreate" typeHandler="org.apache.ibatis.type.LocalDateTimeTypeHandler" jdbcType="OTHER" javaType="java.time.LocalDateTime" />
  <columnOverride column="gmt_modified" property="gmtModified" typeHandler="org.apache.ibatis.type.LocalDateTimeTypeHandler" jdbcType="OTHER" 
<!-- 省略其他 -->
</table>
```


