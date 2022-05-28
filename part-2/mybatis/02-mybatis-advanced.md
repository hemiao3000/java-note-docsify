# <font color="orange">MyBatis 进阶</font>

## 1. 动态 SQL

简而言之，动态 SQL 就是在 Mapper 中使用分支、循环等逻辑。常见的动态 SQL 元素包括：

- if 元素
- choose-when-otherwise 元素
- where 元素
- set 元素
- foreach 元素

### 1.1 if 元素

if 元素是我们最常见的元素判断语句，相当于 Java 中的 if 语句。它的 test 属性是它的必要属性。

```xml
<select id="select" parameterType="int" resultType="Department">
    SELECT * FROM dept
    <if test="deptno != null">
        WHERE deptno = #{deptno}
    </if>
</select>
```

### 1.2 choose-when-otherwise 元素

MyBatis 并未提供类似 if-else 元素来处理分支情况，if 元素可出现多次，但它们是并列的判断，而非互斥的判断。

choose-when-otherwise 元素类似于 Java 中的 switch-case，用于处理多个条件间的互斥判断。

```xml
<select id="selectBySallary" resultType="Employee">
    SELECT * FROM emp
    <choose>
        <when test="min != null and max != null">
            WHERE sal >= #{min} AND sal <= #{max}
        </when>
        <when test="min != null">
            WHERE sal >= #{min} 
        </when>
        <when test="max != null">
            WHERE sal <= #{max}
        </when>
        <otherwise></otherwise>
    </choose>
</select>
```

### 1.3 where 元素

如果我们强行规定，上述 choose-when-otherwise 所实现的功能必须使用 if 实现，那么将会写成如下形式：

```xml
<select id="selectBySallary" resultType="Employee">
    SELECT * FROM emp
    WHERE 1 = 1
    <if test="min != null">
        AND sal >= #{min}
    </if>
    <if test="max != null">
        AND sal <= #{max}
    </if>
</select>
```

注意体会上面 `WHERE 1 = 1` 的位置及其作用。

由于判断条件有可能有，也可能没有，所有在 if 元素中，WHERE 关键字出现的地方就有些“尴尬”。`WHERE 1 = 1` 就是此问题的 ***非典型*** 解决方案。

MyBatis 提供了 Where 元素以解决上述尴尬问题。

```xml
<select id="selectBySallary" resultType="Employee">
    SELECT * FROM emp
    <where>
        <if test="min != null">
            AND sal >= #{min}
        </if>
        <if test="max != null">
            AND sal <= #{max}
        </if>
    </where>
</select>
```

### 1.4 set 元素

类似于 where 的元素，set 元素对应于 SQL 语句中的 SET 子句。它专用于 update 语句，用于包含所需更新的列。

set 元素常常和 if 元素联合使用。因为在“选择性更新”功能中，有一个`最后一个逗号`问题。

`注意`，<small>更新行为务必要保证更新至少一个属性，否则 MyBatis 更新语句提示 update 语句错误</small>

```xml
<update id="updateByPrimaryKeySelective" parameterType="Department">
    UPDATE dept
    <set>
        <if test="dname != null"> dname = #{dname}, </if>
        <if test="loc != null"> loc = #{loc}, </if>
    </set>
    WHERE deptno = #{deptno}
</update>
```

### 1.5 foreach 元素

foreach 元素使用不多，但是当需要构建包含 IN 子句的查询时，则必用到。

```xml
<select id="selectInDeptnos" resultType="Employee">
    SELECT * FROM emp 
    WHERE deptno IN 
    <foreach collection="list" item="deptno" open="(" separator="," close=")">
        #{deptno}
    </foreach>
</select>
```

collection 属性表示集合类型，其属性值可以是 list 或 array，对应参数类型为 List 或 数组。


## 2. 映射结果集（基本）

在前面的内容中，由于我们的 PO 类的属性名与数据库中表的列名是一致的，因此，在 Mapper.xml 配置文件中，Mybatis 省略/简化 掉了一块配置。

```xml
<resultMap id="xxx" type="demo.bean.po.Department">
  <id column="id" jdbcType="INTEGER" property="id" />
  <result column="name" jdbcType="VARCHAR" property="name" />
  <result column="location" jdbcType="VARCHAR" property="location" />
</resultMap>

...

<select id="..." resultMap="xxx"> ... </select>
```

很容易猜得到这个块配置的作用就是<small>（在查询功能中）</small>**指定数据库的表的列与 PO 类的属性之间的对应关系** 。

实际上，Mybatis 需要有这样的一个配置来 指导/告诉 它如何将结果集（ResultSet）中的数据映射成对象，或对象的集合。这是任何一个 ORM 框架的基本功能（重要功能）之一。

### 2.1 &lt;resultMap>

resultMap 元素必要的两个属性有：

| 属性名 | 说明 |
| :- | :- |
| id | resultMap 的唯一标识符。|
| type | 它表示映射所返回的实际类型。|


### 2.2 &lt;id> 和 &lt;result>

resultMap 最常见的两个子元素有：

- <big>id 子元素 </big>
  
  表示数据库表的主键列。 其中，
  
  *`column`* 属性表示表的列名；

  *`property`* 属性，表示映射对象的属性名 

- <big>result 子元素</big>
  
  表示数据库的普通列。其中，

  *`column`* 属性，表示数据库表的列名；
  
  *`property`* 属性，表示映射对象的属性名 



### 2.3 jdbcType

将 ResultSet 数据映射成对象时，会涉及到两种数据类型：数据库类型<small>（varchar) </small>和 Java 类型<small>（String）</small>。MyBatis 使用 <font color="#0088dd">**类型转换器**</font>（typeHandler）来处理两种类型数据的转换问题。

> `补充`，不同的数据库对于同一个数据类型的概念可能会使用不同的『单词』。例如：
> 
> **整型**，在 MySQL 中是 ***INT*** ，在 Oracle 中是 ***INTEGER*** 。


在 Java 的 JDBC 中，对不同数据库的各种类型的『称呼』进行了统一：<font color="#0088dd">**JDBC 类型**</font> 。例如：

『整型』的 JDBC Type 表示为 ***INTEGER*** ，即表示 MySQL 中的 ***INT*** ，又表示 Oracle 中的 ***INTEGER*** 。

常见的有：

| JDBC Type  |  Mysql Type | Java Type |  
| :-- | :-- | :-- |
| SMALLINT  | SMALLINT  | short <br>java.lang.Short     | 
| INTEGER   | INTEGER   | int <br> java.lang.Integer    | 
| BIGINT    | BIGINT    | long <br> java.lang.Long      |
| FLOAT     | FLOAT     | float <br> java.lang.Float    | 
| DOUBLE    | DOUBLE    | double <br>java.lang.Double   | 
| DECIMAL   | DECIMAL   | java.math.BigDecimal          | 
| CHAR      | CHAR      | java.lang.String              | 
| VARCHAR   | VARCHAR   | java.lang.String              | 
| DATE      | DATE      | java.util.Date                | 
| TIME      | TIME      | java.util.Date                | 
| TIMESTAMP | TIMESTAMP | java.util.Date                | 

**注意**：对于 ***java.lang.Date*** 和 ***java.sql.Date*** ，是两种不同的类型。在写 JavaBean 一定要确认你所使用的是哪个 Date 类型<small>（ 一般都是使用 ***java.lang.Date*** ）</small>。


### 2.4 自动映射原理

在 MyBatis 的配置文件<small>（*`settings`* 元素部分）</small>中，有一个 `autoMappingBehavior` 配置，其默认值为 `PARTIAL` ，表示 MyBatis 会自动映射（简单的，没有嵌套关系的）结果集。

```xml
<configuration>

  <properties> ... </properties>

  <settings>
    ...
    <setting name="autoMappingBehavior" value="PARTIAL"/>
    ...
  </settings>

  <typeAliases> ... </typeAliases> 

  ...

</configuration>
```

如果你的类的属性名与表的字段名一致，那么 MyBatis 会自动将结果集的一行封装成一个 JavaBean 。

一般而言，数据库表和字段的命名风格是以下划线为分隔符，而 Java 中命名风格是驼峰命风格。

如果，PO 类的属性名和 Table 的列名仅仅是命名风格的不同，那么此时你可以使用 `mapUnderscoreToCamelCase` 进行控制，以便于自动转换或不转换。

```xml
<configuration>

  <properties> ... </properties>

  <settings>
    ...
    <setting name="mapUnderscoreToCamelCase" value="false"/>
    ...
  </settings>

  <typeAliases> ... </typeAliases> 

  ...

</configuration>
```

## 3. 映射结果集（高级）

Table 的列名与 PO 类的属性名的不同，只是一个小 case。`<resultMap>` 真正的作用和价值并非在此，而是在于『**关系映射**』。

### 1. 一对一映射

使用自定义映射的第二个，也是更重要的原因是：***关联映射*** 。

两张表的一对一映射在类与类的关系上体现为 <font color="#0088dd">一个类“持有”另一个类</font>。双向的一对一映射则表现为 <font color="#0088dd">两个类互相“持有”</font>。

持有方所对应的表为『从表』，被持有方所对应的表为『主表』。`association` 元素用于 <font color="#0088dd">从表</font> 方。

`association` 元素的常用属性有：

| 属性 | 说明 |
| :- | :- |
| property 属性|表示所映射对象的属性名。|
| column 属性|表示表的列名，此处必为 ***外键列*** 。|
| javaType 属性|表示属性所对应的类型名称。|
| select 属性|表示为了获取关联对象，所调用的另一条 MyBatis select 语句的 id 。|

**第一种写法：传统写法**

```xml
<mapper namespace="demo.dao.EmployeeMapper">

  <resultMap id="BaseResultMap" type="demo.bean.po.Employee">
    <id ... />
    <result ... />
    ...

    <association property="department" javaType="demo.bean.po.Department">
      <id column="deptno" property="deptno" jdbcType="INTEGER"/>
      <result column="dname" property="dname" jdbcType="VARCHAR"/>
      <result column="loc" property="loc" jdbcType="VARCHAR"/>
    </association>

  <select id="..." resultMap="BaseResultMap">

</mapper>
```


**第二种写法：调用法**

```xml
<mapper namespace="demo.dao.EmployeeMapper">

  <resultMap id="BaseResultMap" type="demo.bean.po.Employee">
    <id ... />
    <result ... />
    ...
    <association property="department" 
        column="deptno" 
        javaType="demo.bean.po.Department"
        select="demo.dao.DepartmentMapper.selectByPrimaryKey"/>
    </resultMap>

  <select id="..." resultMap="BaseResultMap">

</mapper>
```

### 2. 一对多映射

一对多的映射在类与类的关系上体现为：一个类持有另一个类的一个对象，另一个类持有这个类的对象集合。

如果一对多映射是单向的，则只会使用到 collection 元素；如果一对多映射是双向的，那么还会使用到前面所说的 association 元素。

collection 元素的常用属性有：

| 属性 | 说明 |
| :- | :- |
| property 属性 | 表示所映射对象的属性名。|
| column 属性 | 表示表的列名，此处必为 ***外键列*** |
| javaType 属性 | 表示属性所对应的类型名称，此处必为某种集合类型 |
| ofType 属性 | 表示集合当中的单元类型 |
| select 属性 | 表示为了获取关联对象，所调用的另一条 MyBatis select 语句的 id |

**第一种写法：**

```xml
<resultMap id="xxx" type="Department">
  <id column="deptno" property="deptno"/>
  <result column="dname" property="dname"/>
  <result column="loc" property="loc"/>

  <collection property="employeeList" ofType="Employee">
    <id column="empno" property="empno"/>
    <result column="ename" property="ename"/>
    <result column="job" property="job"/>
    <result column="mgr" property="mgr"/>
    <result column="hiredate" property="hiredate"/>
    <result column="sal" property="sal"/>
    <result column="comm" property="comm"/>
    <result column="deptno" property="deptno"/>
  </collection>
</resultMap>

  <select id="selectByPK" parameterType="int" resultMap="xxx">
    select dept.deptno  deptno,
      dept.dname   dname,
      dept.loc     loc,
      emp.empno    empno,
      emp.ename    ename,
      emp.job      job,
      emp.mgr      mgr,
      emp.hiredate hiredate,
      emp.sal      sal,
      emp.comm     com
    from dept,
      emp
    where dept.deptno = emp.deptno
      and dept.deptno = #{deptno}
  </select>
```

**第二种写法：**

```xml
<mapper namespace="demo.dao.DepartmentMapper">

  <resultMap id="BaseResultMap" type="demo.bean.po.Department">
    <id ... />
    <result ... />
    ...
    <collection column="deptno" 
        javaType="java.util.List"
        ofType="demo.bean.po.Employee" 
        property="employeeSet"
        select="demo.dao.EmployeeMapper.selectByPK"/>
  </resultMap>

  <select ...>
```

### 3. 多对多映射

MyBatis 并没有提供专门的元素用于多对多方案，采取了一种“聪明”的办法，将“三张表”的多对多关系转化为“两张表”的一对多关系。

所以，在多对多关系中，使用的仍然是 `collection` 元素。

例如：

```xml
<mapper namespace="demo.dao.DepartmentMapper">
  <resultMap id="BaseResultMap" type="demo.bean.po.Department">
    <id ... />
    <result ... />
    ...
    <collection column="deptno" 
        javaType="java.util.Set"  
        ofType="demo.bean.po.Employee" 
        property="employeeSet"
        select="demo.dao.EmployeeMapper.selectByDeptno"/>
  </resultMap>

  <select ...>
```

『完』

## 4. 分页

Mybatis 中实现分页功能有三种途径：

- RowBounds 分页<small>（不建议使用）</small>

- Example 分页<small>（简单情况可用)</small>

- PageHelper 分页

### 4.1 RowBounds 分页

MyBatis 本身通过 RowBounds 对象提供了分页功能，你仅需为你的 dao 的查询方法多添加 RowBounds 类型的一个参数，并且不需要对配置文件做任何调整。

> <small>RowBounds 也称原生分页、逻辑分页。</small>

```java
RowBounds bounds = new RowBounds(0, 4);
List<Employee> list = dao.select(bounds);
```

但是这种分页是一种 **逻辑分页**，MyBatis 并未使用 **limit** 子句，查询的仍是 *所有数据*，只是它仅给你“看”到了所有数据中的一部分，逻辑分页虽然完成了分页功能，但是它并未通过分页功能的对性能问题有所提升。

### 4.2 PageHelper 分页

PageHelper 是一款被广泛使用的 MyBatis 插件。它通过 Mybatis 的插件机制，巧妙地通过机制，在不需要配置文件（不需要写 limit 子句）的情况下，动态去修改你所执行的 SQL，在其后动态添加 limit 子句。

为了使用 PageHelper 需要引入相应的包：

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.1.8</version>
</dependency>
```

PageHelper 是一款 MyBaits 插件，使用它需要向 Mybatis 注册 PageHelper，并对它作出相关配置（`mybatis-config.xml`）。

```xml
<plugins>
  <!-- com.github.pagehelper 为 PageHelper 类所在包名 -->
  <plugin interceptor="com.github.pagehelper.PageInterceptor">
    <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
    <property name="helperDialect" value="mysql" />
    <property name="..." value="..."/>
  </plugin>
</plugins>
```

注意：pagehelper 有 4.x 和 5.x 两个版本，用法有所不同，并不是向下兼容，同样的配置在使用 4.x 或 5.x 版本中可能会报错。例如，上面的 `helperDialect` 就是 5.x 中的配置，在 4.x 中使用的是 `dialect` 。

如果 Mybatis 整合进了 Spring，除了上述这样配置外，还可以将相应的注册-配置工作就在 Spring 的配置文件中进行：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property .../>

  <property name="plugins">
    <array>
      <bean class="com.github.pagehelper.PageInterceptor">
        <property name="properties">
          <!--使用下面的方式配置参数，一行配置一个 -->
          <value>
            param1=value1
            param2=value2
            ...
          </value>
        </property>
      </bean>
    </array>
  </property>
</bean>
```

**插件的属性配置**：

- helperDialect

  用于指明底层数据库类型：oracle, mysql, mariadb, sqlite, hsqldb, postgresql, db2, sqlserver, informix, h2, sqlserver2012, derby

- reasonable

  是否启用『合理化』功能<br>

  启用（true）时，如果 `pageNum < 1`，会返回第一页内容；如果 `pageNum > pages`，会返回查询最后一页。<br>

  禁用（false）时，超出合理的范围会直接返回空数据。


在使用 PageHelper 时，PageHelper 提供了两种风格来描述分页：

- offset / limit 组合

  `PageHelper.offsetPage(0, 3);`<br>

  很显然，这种风格就是 SQL 语句的分页写法

- pageNum / pageSize 组合

  `PageHelper.startPage(1, 10);`<br>

  这种风格实际上是在模拟“人的语气”<br>

  插件作者建议推荐方式

在你调用查询方法之前，调用 PageHelper 的上述两个方法中的任意一个，都可激活 PageHelper 插件的分页功能，使其动态地『帮』你修改SQL语句<small>（添加 *limit* 子句）</small>。而 Mybatis 的 select 返回的结果就返回的是一页数据。

```java
PageHelper.startPage(4, 2);
List<Employee> list = empDao.selectByExample(null);
PageInfo<Employee> info = new PageInfo<>(list, 5);

System.out.println(info);
```

> <small>`注意`，由于 PageHelper 插件的实现涉及到 ThreadLocal 原理，这导致一旦 PageHelper 生产了一个分页参数，但是没有被消费，这个参数就会一直保留在这个线程上。当这个线程再次被使用时，就可能导致不该分页的方法去消费这个分页参数，这就产生了莫名其妙的分页。所以，分页参数的创建代码，和查询方法的调用代码，必须“紧密的在一起”。</small>

PageHelper 插件流行的原因在于，它不仅仅能实现分页功能，而且还进一步封装了页面上的『**分页导航条**』所需要的所有相关信息。

使用这一个功能时，你需要创建 ***PageInfo*** 对象，并为其设置 4 个关键数据：

- 页面上所需显示的数据<small>（分页查询的结果）</small>

- 导航栏上导航数字的个数

- 当前页号

- 页面大小

- 总数据量

例如：

```java
// << < 2 3 [4] 5 6 > >>
PageInfo<Employee> pageInfo = new PageInfo<>(empList, 5);
pageInfo.setPageNum(4); // 当前页的页号
pageInfo.setPageSize(2);// 页面大小，即，每页显示的数据量
// pageInfo.setTotal(32);  // 数据库中的总数据量。现在可省略，PageHelper 会自己去查。
```

创建 PageInfo 对象，并为其 4 个核心属性赋值后，便可以使用它：

```java
log.info("是否有下一页：{}", pageInfo.isHasNextPage());
log.info("是否有上一页：{}", pageInfo.isHasPreviousPage());
log.info("导航栏上第一个页号：{}", pageInfo.getNavigateFirstPage());
log.info("导航栏上最后一个页号：{}", pageInfo.getNavigateLastPage());
log.info("导航栏上的五个导航数字：{}", Arrays.toString(pageInfo.getNavigatepageNums()));
log.info("共有 {} 页", pageInfo.getPages());
log.info("{} / {} ", pageInfo.getPageNum(), pageInfo.getPages());
```


### 4.4 PageInfo 对象属性描述 

| 属性 | 说明 | 举例 |
| :- | :-| :- |
| int pageNum | 当前页 | 比如，当前为第 `5` 页 |
| int pageSize | 每页的数量 | 比如，每页（计划/预期）显示 `10` 条数据 |
| int size | 当前页的数量 | 比如，以 98 条总数据为例，每页最多显示 `10` 条（最后一页显示 `8` 条数据） |
| int startRow | 当前页面第一个元素在数据库中的行号 | 比如，以 98 条总数据的最后一页为例，第一条数据是第 `91` 条 |
| int endRow | 当前页面最后一个元素在数据库中的行号 | 比如，以 98 条总数据的最后一页为例，最后一条数据是第 `98` 条 |
| int pages | 总页数 | 比如，以 98 条总数据为例，每页显示 10 条（其中最后一页 8 条），因此共 `10` 页 |
| int prePage | 前一页 | 比如，当前是第 5 页，所以前一页为 `4` |
| int nextPage | 下一页 | 比如，当前是第 5 页，所以下一页为 `6` |
| boolean isFirstPage | 是否为第一页 | 比如，当前是第 5 页，不是第 1 页，所以为 `false` |
| boolean isLastPage | 是否为最后一页 | 比如，当前是第 5 页，不是最后 1 页，所以为 `false` |
| boolean hasPreviousPage | 是否有前一页 | 比如，当前是第 5 页，有前一页，所以为 `true` |
| boolean hasNextPage | 是否有下一页 | 比如，当前是第 5 页，有后一页，所以为 `true` |
| int navigatePages | 导航页码数 | 比如，页面导航栏显示 [3 4 5 6 7] 共 `5` 个数字 |
| int[] navigatepageNums | 所有导航页号 | 比如，页面导航栏显示 `[3 4 5 6 7]` 这 5 个数字 |
| int navigateFirstPage | 导航条上的第一页 | 比如，页面导航栏显示 [3 4 5 6 7] 时，第一页是第 `3` 页 |
| int navigateLastPage | 导航条上的最后一页 | 比如，页面导航栏显示 [3 4 5 6 7] 时，第一页是第 `7` 页|

