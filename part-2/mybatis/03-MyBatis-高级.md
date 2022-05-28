# <font color="orange">MyBatis 高级</font>

## 1. MyBatis Generator 和 Example 对象

### 1.1 Mybatis Generator

mybatis 属于『**半自动 ORM 框架**』，在使用这个框架中，工作量最大的就是书写 mapper 的映射文件，由于手动书写很容易出错，为此 mybatis 官方提供了 ***mybatis-generator*** 来帮我们自动生成文件。

早期的 mybatis-generator 的使用十分原始，使用起来非常麻烦。在开源社区的努力下，现在有多种方案能简化我们对 myBatis-generator 的使用：

1. 使用图形化工具，例如：[mybatis-generator-gui](https://github.com/zouzg/mybatis-generator-gui)

2. 如果使用的是 Jetbrains IDEA 开发工具，那么可以使用 ***Free Mybatis plugin*** 插件来进行图形化操作。

3. 使用 maven 的 mybatis-generator 插件，通过命令直接生成映射文件等内容。

这里推荐第三种方案：使用 maven 的 myabtis-generator 插件。

---

在 Maven 中，有一个名为 ***mybatis-generator-maven-plugin*** 的第三方插件，它能够将 mybatis-genetrator 的功能纳入到 Maven 体系中，允许你通过一条 maven 命令去生成相关的 JaveBean、DAO 接口和 Mapper 映射文件，以简化对 mytabis-generator 的使用。

#### 2.1.1 pom.xml 配置

在项目的 ***pom.xml*** 文件中的 ***build*** > ***plugins*** 中加入如下内容：

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.4.0</version> <!-- 不要低于 1.3.7 版本 -->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.4.0</version> <!-- 不要低于 1.3.7 版本 -->
        </dependency>
    </dependencies>
    <configuration>
        <!-- 具体的、更多的参数说明见官网 http://mybatis.org/generator/running/runningWithMaven.html -->
        <verbose>true</verbose> <!-- 允许移动生成的文件 -->
        <overwrite>true</overwrite> <!-- 是否覆盖 -->
        <!-- 自动生成的配置 -->
        <configurationFile>src/main/resources/mybatis/mybatis-generator-config.xml</configurationFile>
    </configuration>
</plugin>
```

mybatis-generator-maven-plugin 依赖于两个包：

- mybatis-generator-core

  mybatis-generator-core 是『自动生成』功能的实现者和提供者，mybatis-generator-maven-plugin 是在『利用』它的这个功能。

- mysql-connector-java
  
  本质上是因为 mybatis-generator-core 在『自动生成』JavaBean 时，需要去数据库中查询表的相关信息，去『求』表的列名并以此为依据命名你的 JavaBean 的属性。很显然因为需要连接数据库，所以这里需要有数据库驱动包。


mybatis-generator 要能正常工作，需要你提供一个配置文件。在这个配置文件中，你去『告诉』mybatis-generator 去『自动生成』时的相关细节。

> 另外，你还可以将 mybatis-generator:generate 命令绑定到 maven 的 package 命令中。这样，当你去执行 *`mvn package`* 命令时也会触发 mybatis-generator:generate 命令：
> 
> ```xml
> <plugin>
>     <groupId>org.mybatis.generator</groupId>
>     <artifactId>mybatis-generator-maven-plugin</artifactId>
>     <version>1.4.0</version> 
>     <dependencies> ... </dependencies>
>     <executions>
>         <execution>
>             <id>Generate MyBatis Artifacts</id>
>             <phase>package</phase>
>             <goals>
>                 <goal>generate</goal>
>             </goals>
>         </execution>
>     </executions>
>     <configuration> ... </configuration>
> </plugin>
> ```



#### 2.1.2 mybatis-generator-config.xml 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>

    <!-- 数据库驱动包位置 -->
    <!-- 由于在 pom.xml 中加入插件时已经配置数据库驱动包，所以此处不必配置了 -->
    <!--     
        <classPathEntry location="C:\Users\59960549\.m2\repository\mysql\mysql-connector-java\5.1.47\mysql-connector-java-5.1.47.jar" />
    -->

    <context id="scott" targetRuntime="MyBatis3">

        <!-- 覆盖生成 XML 文件的 Bug 解决 -->
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />

        <commentGenerator>
            <!-- 去掉注解中的生成日期 -->
            <property name="suppressDate" value="true"/>
            <!--关闭注释 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--数据库链接地址账号密码-->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/scott?useSSL=false&amp;serverTimezone=UTC"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver>
            <!--
            true： 无论什么情况，都是使用BigDecimal对应DECIMAL和 NUMERIC数据类型
            false： 默认值,
                scale>0;length>18：使用BigDecimal;
                scale=0;length[10,18]：使用Long；
                scale=0;length[5,9]：使用Integer；
                scale=0;length<5：使用Short；
            -->
            <property name="forceBigDecimals" value="true"/>
        </javaTypeResolver>

        <!-- PO 类的相关设置 -->
        <javaModelGenerator targetProject="src/main/java"
                            targetPackage="com.demo.pojo">
            <!-- 在 targetPackage 的基础上，根据数据库的 schema 再生成一层 package，最终生成的类放在这个 package下，默认为false  -->
            <property name="enableSubPackages" value="false"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- Mapper.xml 配置文件的相关设置 -->
        <sqlMapGenerator targetProject="src/main/resources"
                         targetPackage="mapper">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- DAO 接口的相关设置-->
        <javaClientGenerator targetProject="src/main/java"
                             targetPackage="com.demo.dao"
                             type="XMLMAPPER">
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>

        <!--生成对应表及类名-->
        <table tableName="department" catalog="scott"
               domainObjectName="Department"
               mapperName="DepartmentDao"
               domainObjectName="Department"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false">
            <property name="ignoreQualifiersAtRuntime" value="true"/>
        </table>
    </context>
</generatorConfiguration>
```

### 1.2 Mybatis Example 对象

Example 对象是一种简化条件查询的方案。通过它，你避免去编写大量的 DAO 中的 ***selectByXxx()*** 方法。

#### 2.1 简单的情况

这里的简单情况指的是：没有，或只有 ***and*** 关系：

```java
EmployeeExample example = new EmployeeExample();

example.createCriteria()
    .andEmpnoEqualTo(7369)
    .andSalGreaterThanOrEqualTo(1500)
    .andSalLessThan(2000);

List<Employee> list = dao.selectByExample(example);
System.out.println(list.size());
```

***or()*** 方法是一个更通用的形式，可以用于实现任意的查询条件。其原理在于，任何一个复杂的查询语句，总能写成如下形式：

```sql
where (... and ... and ...) or (... and ... and ...) or (...)
```


#### 2.2 复杂的情况

这里的复杂情况指的是：有 ***or*** 关系，甚至是 ***and*** 和 ***or*** 混用。


```java
TestTableExample example = new TestTableExample();

// 第 1 个括号中的两个并列条件
example.or()
    .andAaaEqualTo(5)
    .andBbbIsNull();

// 第 2 个括号中的两个并列条件
example.or()
    .andCccNotEqualTo(9)
    .andDddIsNotNull();

// 第 3 个括号中的唯一的条件
List<Integer> list = new ArrayList<Integer>();
list.add(8);
list.add(11);
list.add(14);
list.add(22);
example.or()
    .andEeeIn(field5Values);

// 第 4 个括号中的唯一的条件
example.or()
    .andFffBetween(3, 7);
```

相当于

```sql
where (aaa = 5 and bbb is null)
     or (ccc != 9 and ddd is not null)
     or (eee in (8, 11, 14, 22))
     or (fff between 3 and 7);
```

## 2. 延迟加载

如果一个对象关联另一个对象，那么在查询 A 对象的时候，会去关联查询 B 对象。

何时查询（加载）B 对象分为三种时机：

- 立即加载
- 激进式延迟加载
- 延迟加载

### 3.1 立即加载

MyBaits 默认是立即加载，即在查询 A 对象的时候，会立即查询其关联的 B 对象。如果，B 对象也有关联对象，例如 C 对象，那么还会立即查询 C 对象，... 因此类推，直到把所有有关联关系的数据全部查询出来。


### 3.2 激进式延迟加载

通过设置，可以启用延迟加载：

```xml
<settings>
  <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

启用延迟加载之后，Mybatis 又是默认的激进地延迟加载。

Mybatis 内部会进行某种规则判断，从而使得激进式的延迟加载，有时候等同于立即加载，有时候等同于普通的延迟加载。

### 3.3 真-延迟加载

可以再通过配置关闭掉激进地延迟加载，从而进入普通的延迟加载：

```xml
<settings>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

普通的延迟加载只会在你真正用到 A 对象的 B 属性时，再去查询/加载 B 对象。

## 3. Mybatis 缓存

### 3.1 Mybatis 一级缓存

#### 3.1.1 概述

MyBatis 提供了查询缓存来缓存数据，从而达到提高查询新能的要求。MyBatis 的缓存分为一级缓存和二级缓存。

MyBatis 的一级缓存是 SqlSession 级别的缓存（<small>在操作数据库时需要构造 SqlSession 对象</small>），每个 SqlSession 对象中都有一个 HashMap 对象用于缓存数据，不同的 SqlSession 之间缓存的数据互不影响。

> 提前说明一下，当 mybatis 与 spring 整合后，**如果没有事务，一级缓存是失效的！**
> 
> 原因就是两者结合后，sqlsession 如果发现当前没有事务，那么每执行一个 mapper 方法之后，sqlsession 就被关闭了<small>（ *`session.close()`* ）</small>。
> 
> 所以记得给 Service 的方法的脑袋上面加 ***@Transactional*** 。


在参数和 SQL 完全相同的情况下，使用同一个 SqlSession 对象调用同一个 Mapper 方法，往往只执行一次 SQL 语句。因为如果没有声明需要刷新缓存并且缓存没有超时，SqlSession 都只会取出当前缓存的数据<small>，而不是执行 SQL 语句</small>。

如果 SqlSession 执行了 DML 操作，并提交数据库，Mybatis 会清空 SqlSession 中的一级缓存，这样做的目的是保证缓存中存储的数据是最新的，避免出现脏读现象。

#### 3.1.2 刷新缓存的时机

- 如果 SqlSession 调用了 ***close*** 方法，则一级缓存不可用/销毁。

- 如果 SqlSession 调用了 ***clearCache*** 方法，则一级缓存中缓存的数据被清空。

- 如果 SqlSession 执行了一个 DML 操作，则一级缓存中缓存的数据被清空。


### 3.2 Mybatis 二级缓存

二级缓存是 Mapper 级别<small>（也叫 namespace 级别）</small>的缓存，同样使用 HashMap 进行数据存储。

- 二级缓存是以 namespace 为单位的，不同的 namespace 下的操作相互隔离。

- 增删改操作会清空 namespace 下的全部缓存。

如果开启了二级缓存，那么在关闭 sqlsession 后，会把该 sqlsession 一级缓存中的数据添加到 namespace 的二级缓存中。

二级缓存比一级缓存作用域更大，多个 sqlsession 可以共用二级缓存，即，二级缓存是跨 sqlsession 的。


> <small>例如，关闭一个 sqlsession 之后，打开一个新的 sqlsession，执行同一条 sql 语句，会利用上一次的缓存数据。</small>

mybatis **默认没有开启二级缓存** ，需要在配置中进行配置才能使用。打开二级缓存分为三步：

1. 打开二级缓存总开关

2. 打开需要使用二级缓存的 mapper 的开关。

3. POJO 序列化

#### 3.2.1 打开二级缓存总开关
 
打开总开关，只需要在 mybatis 总配置文件中加入一行设置

```xml
<settings>
  <!--开启二级缓存-->
  <setting name="cacheEnabled" value="true"/>
</settings>
```


#### 3.2.2 打开需要使用二级缓存的 mapper 的开关。
 
在需要开启二级缓存的 ***mapper.xml*** 中加入 caceh 标签

```xml
<cache />
```

#### 3.2.3 POJO 序列化

让需要使用二级缓存的 POJO 类实现 Serializable 接口，如

```java
public class Department implements Serializable {
  ...
}
```

通过之前三步操作就可以使用二级缓存了。
