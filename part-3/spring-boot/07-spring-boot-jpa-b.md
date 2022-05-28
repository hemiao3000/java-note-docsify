# <font color="orange">Spring Data JPA 的高级使用</font>

## 1. 关系映射

### 1.1 被省略调的默认设置

由于 JPA 的默认设置在起作用，我们之前对 **`@Entity`** 中的属性的设置，『有些注解被省略掉了』。

- 与主键列对应的属性，除了使用 **`@Id`** 注解，还要使用 **`@Column`** 注解。<small>（**`@GeneratedValue`** 注解的作用是另一码事，和我们这里说的无关）</small>

- 与其它列对应的属性，除了使用 **`@Basic`** 注解，还要使用 **`@Column`** 注解。

完整的形式应该如下：

```java
@Entity
@Table(name = "department", schema = "scott")
public class Department {

    @Id
    @Column
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Basic
    @Column
    private String name;

    @Basic
    @Column
    private String location;

    // getter / setter
}
```

这里【默认】的规则如下：

- Entity 的属性默认就是 **`@Basic`**。因此，逻辑上是 **`@Basic`** 的属性头上的 **`@Basic`** 就都可以省略。不是 **`@Basic`** 的属性，例如 **`@Id`**，自然就要明确标明 **`@Id`**。

- 如果属性名和列名是一致的，或只是驼峰命名法和下划线命名法这种命名风格的差异，那么，**`@Column`** 注解可以省略。反而言之，**`@Column`** 注解只有在双方命名不一致的情况下，才会出来干活。


### 1.2 一对多关系映射/配置

这里有个概念可以便于理解和记忆以下配置：<strong>**`JoinColumn`** 指的就是外键列</strong>，只不过一个是编程领域中的叫法，一个是数据库领域中的叫法。**`@JoinColumn`** 的 **`name`** 自然就是外键列的列名。

如果没有映射成【引用】关系，那么 Entity 中的与外键列对应的属性，使用的注解自然就是 **`@Basic`** + **`@Column`**。例如：

```java
@Basic
@Column
private Integer departmentId;
```

当 **`Integer departmentId`** 属性要衍变为 **`Department department`** 属性时，自然不再适合使用 **`@Basic`** + **`@Column`** 注解。

在一对多的关系中：

- 在多方<small>（例如员工、学生）</small>的属性上，使用 **`@ManyToOne`** + **`@JoinColumn`** 注解。

- 在一方<small>（例如部门、老师）</small>的属性上，使用 **`@OneToMany`** 注解。

需要注意的是，**`@ManyToOne`** 和 **`@OneToMany`** 两个注解并非必须成对出现，只有在双向的关系中，一方和多方需要互相引用对象时，才会成对出现。如果是单向的关系，通常只是对多方使用 **`@ManyToOne`** 。

-   Employee 类

    ```java
    @ManyToOne
    @JoinColumn(name = "department_id")  // 外键列列名字
  private Department department;
  ```

-   Department
  ```java
  @OneToMany(mappedBy = "department") // 对应对端的相关属性名。
  private Set<Employee> employeeSet;  // 可以使用 List
  ```

另外，需要强调的是，由于在一对多关系中，外键列是在『**从表**』一方。也就是『**多方**』。因此，是【多方】的配置中出现 **`@JoinColumn`**。如果是双向关系的话，主表方/一方使用 **`mappedBy`** 。



-   通用形式：

    ```java
    多方/从表方 : Employee {
	    ...
	    @ManyToOne
	    @JoinColumn(
		    name = "外键列 name（这一列是在从表中的）",
		    referencedColumnName="与外键列对应的列 name（通常是主表的主键列）"
	    )
	    private Department department;
    }

    一方/主表方 : Department {
	    ...
	    @OneToMany(mappedBy="对端对应的属性的 name（就是用上了 @ManyToOne 的那个属性）")
	    private Set<Employee> employeeSet;
    }
    ```


### 1.3 一对多双向关系的注意事项

> 再次强调，如无必要，尽量不要使用双向关系。

1.  在『**多方**』 **`@ManyToOne`** 中要使用 **`FetchType.LAZY`**，否则会导致性能降低（1 + N 问题）。
   
    ```java
    class Employee {
      ...
      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "department_id")
      private Department department;
    }
    ```

2. 『**一方**』中要增加了 2 个方法，***`.addXxx()`*** 和 ***`.removeXxx()`*** 。

    ```java
    class Department {
      ...

      public void addEmployee(Employee employee) {
        if (employeeSet.contains(employee))
            return;

        employeeSet.add(employee);
        employee.setDepartment(this);
      }

      public void removeEmployee(Employee employee) {
        if (!employeeSet.contains(employee))
            return;

        employeeSet.remove(employee);
        employee.setDepartment(null);
      }
    }
   ```

3.  『**多方**』中的 *`.setXxx()`* 方法要重新设计。

    ```java
    class Employee {
      ...
      public void setDepartment(Department department) {
        if (this.department == department)
          return;

        if (this.department != null)
          this.department.removeEmployee(this);

        this.department = department;

        if (department != null)
          department.addEmployee(this);
      }
    }
   ```

4.  在使用 JSON 来序列化对象或生成默认的 *`toString()`* 方法时，会产生无限递归<small>（Infinite recursion）</small>问题：StackOverFlow。

再次强调一点，<span clss="strong">最理想的情况应该是【有向无环】</strong>。如非必要，尽量不要使用双向关系。因为一不小心容易出现逻辑错误。


### 1.4 多对多关系映射/配置


这里有个无关的小问题，由于下面的例子中使用到的素材里有个表名为 **`order`**，这与数据库关键字冲突，因此为了【告知】Hibernate/JPA 在【帮】我们执行 SQL 语句时要为它加返单引号，因此在 Entity 的 **`@Table`** 中做一点小改动：

```java
@Table(name = "`order`", schema = "scott")
```

如果充分理解 **`@ManyToOne`**，那么接下来理解多对多关系中的 **`@ManyToMany`** 就很容易。

Order 类：

```java
@ManyToMany
@JoinTable(name = "orderitem",  // 指定中间表
    joinColumns = {@JoinColumn(      // 【我】（Order）是如何和中间表关联
        name = "order_id",           // 中间表中的对应我的主键列的外键列
        referencedColumnName = "id") // 【我】的主键列
    },
    inverseJoinColumns = {@JoinColumn( // 【我的对端】（Product）是如何和中间表关联
        name = "prod_id",              // 中间表中的对应我的对端的主键列的外键列
        referencedColumnName = "id")   // 【我的对端】的主键列
    }
)
private Set<Product> productSet;
```

由于多对多的关系中，**外键列** 是在中间表中（A B 并没有对方的外键列），因此 **`@JoinColumn`** 自然是出现在 **`@JoinTable`** 【里面】的。并且，**`@JoinColumn`** 的 **`name`** 指的就是外键列的列名。


!FILENAME Product
```java
@ManyToMany(mappedBy = "productSet")  // 对端的对应属性名
private Set<Order> orderSet;
```

<strong>一对多双向关系中需要关注的那几点，在多对多双向关系中一样也要关注！</strong>

- 通用形式：

  ```java
  A方 : Order {
	  ...
	  @ManyToMany
	  @JoinTable(name="中间表name",
		  joinColumns = { 	// 配置【我】（A方）与中间表的关系
			  @JoinColumn(
				  name = "中间表中A方的外键列name",
				  referencedColumnName = "A方表中与之对应的列的name（通常就是A方的主键列）"
			  )
		  }
		  inverseJoinColumns = { // 配置【对方】（B方）与中间表的关系
			  @JoinColumn(
				  name = "中间表中B方的外键列name",
				  referencedColumnName = "B方表中与之对应的列的name（通常就是B方的主键列）"
			  )
		  }
	  )
	  private Set<Product> productSet;
  }

  B方 : Product {
	  ...

	  @ManyToMany(mappedBy="对方用上了@ManyToMany和@JoinTable写了一大坨配置的那个属性的name")
	  private Set<Order> orderSet;
  }
  ```


### 1.5 一对一关系映射/配置

```java
class Product { // 主表
  ...

  @OneToOne(mappedBy="product")
  private Productnote productnote;
  ...
}
```

```java
class Productnote {
  ...
  @OneToOne(optional = false)
  @JoinColumn(name = "prod_id") // 外键列
  private Product product;
  ...
}
```

上面用到的 *`optional`* 并非必须，这里只是故意显示其作用。optional 的默认值为 true，在 **`@ManyToOne`** 和 **`@ManyToMany`** 中也可使用。

当 optional 的属性值为 true 时，Hibernate/JPA 执行的是内连（inner join）查询，因此 product 属性值必定null null 。为 false 时，Hibernate/JPA 执行的是左外连接（left join）查询，因此 product 属性的值有可能为 null （是否真为 null，取决于数据库的实际情况）。

- 通用形式

  ```java
  主表 : Product {
	  ...
	  @OneToOne(mappedBy="从表方用上了 @OneToOne 和 @JoinColumn 的那个属性的name")
	  private ProductNode node;
  }

  从表 : ProductNote {
	  ...
	  @OneToOne
	  @JoinColumn(
		  name="外键列name（这一列肯定是在从表中的）",
		  referencedColumnName="与外键列对应的列name（通常是主表的主键列）"
	  )
	  private Product product;
  }
  ```


## 2. 分页、排序、限制查询

Spring Data JPA 已经帮我们内置了分页功能，在查询的方法中，需要传入参数 Pageable，当查询中有多个参数的时候 Pageable 建议 ***作为最后一个参数传入***。

```java
// Pageable pageable = PageRequest.of(0, 3);
Page<User> findByNickNameAndEmail(String nickName, String email, Pageable pageable);
```

具有分页功能的方法会返回一个 **`Page<T>`** 对象，其中封装了与分页有关的相关信息，及其所获取的数据。


Pageable 是 Spring 封装的分页实现类，使用的时候需要传入页数、每页条数和排序规则，Page 是 Spring 封装的分页对象，封装了总页数、分页数据等。返回对象除使用 Page 外，还可以使用 Slice 作为返回值。

Page 和 Slice 的区别如下:

- Page 接口继承自 Slice 接口，而 Slice 继承自 Iterable 接口。

- Page 接口扩展了 Slice 接口，添加了获取总页数和元素总数量的方法，因此，返回 Page 接口时，必须执行两条 SQL，一条复杂查询分页数据，另一条负责统计数据数量。

- 返回 Slice 结果时，查询的 SQL 只会有查询分页数据这一条，不统计数据数量。

- 用途不一样：Slice 不需要知道总页数、总数据量，只需要知道是否有下一页、上一页，是否是首页、尾页等，比如前端滑动加载一页可用；而 Page 知道总页数、总数据量，可以用于展示具体的页数信息，比如后台分页查询。

实例：

```java
@Test
public void testPageQuery() {
    int page=1, size=2;
    Sort sort = new Sort(Sort.Direction.DESC, "id");  // 排序
    Pageable pageable = PageRequest.of(page, size, sort);
    userRepository.findALL(pageable);
    userRepository.findByNickName("aa", pageable);
}
```

- Sort，控制分页数据的排序，可以选择升序和降序。

- PageRequest，控制分页的辅助类，可以设置页码、每页的数据条数、排序等。

### 限制查询 {docsify-ignore}

有时候我们只需要查询前 N 个元素，或者只取前一个实体。

```java
// order by salary
List<Employee> findFirstByOrderBySalary(); 

// order by salary desc limit 2
List<Employee> findFirst2ByOrderBySalaryDesc();

// where job = ? order by salary desc limit 2 
List<Employee> findFirst2ByJobEqualsOrderBySalaryDesc(String job);
```

## 3. 更复杂的条件查询

我们可以通过 **`AND`** 或者 **`OR`** 等连接词来不断拼接属性来构建多条件查询，但如果参数大于 6 个时，方法名就会变得非常的长，并且还不能解决动态多条件查询的场景。到这里就需要给大家介绍另外一个利器 **`JpaSpecificationExecutor`** 了。

`JpaSpecificationExecutor` 是 JPA 2.0 提供的 **`Criteria API`** 的使用封装，可以用于动态生成 Query 来满足我们业务中的各种复杂场景。我们只需要为我们的自定义接口多继承一个父接口：`JpaSpecificationExecutor` 。

- 我们的自定义的接口多继承了 `JpaSpecificationExecutor` 后，我们的接口中自然就【多】出来一些入参为 **`Specification`** 类型的方法。

- `Specification` 是一个接口，其中的 **`toPredicate`** 方法的返回值 **`Predicate`** 对象就代表着查询条件。

- 简单来说，我们要去实现 `Specification` 接口，并通过实现它的 `toPredicate` 方法返回一个 `Predicate` 对象。`JpaSpecificationExecutor` 需要这个 `Predicate` 对象来执行查询操作。

在使用 JpaSpecificationExecutor 构建复杂查询场景之前，我们需要了解几个概念：

| 概念 | 说明 |
| :- | :- |
| `Root<T> root` | 代表了可以查询和操作的实体对象的根，可以通过 `get("属性名")` 来获取对应的值。 |
| `CriteriaQuery<?> query` |代表一个 `specific` 的顶层查询对象，它包含着查询的各个部分，比如 `select `、`from`、`where`、`group by`、`order by` 等。|
| `CriteriaBuilder cb` | 来构建 `CritiaQuery` 的构建器器对象，其实就相当于条件或者是条件组合，并以 `Predicate` 的形式返回。 |


### 使用案例

- 为 `EmployeeRepository` 【多】添加接口：

  ```java
  @Repository
  public interface EmployeeRepository extends
        JpaRepository<Employee, Integer>,
        JpaSpecificationExecutor<Employee> {
            ...
  }
  ```

- 在 Service（或 Junit）中使用 JpaSpecificationExecutor 的具体使用。

  ```java
  // 1. Specification 的 toPredicate() 方法返回的 Predicate 对象负责描述【要找的人的标准】。
  Specification<Employee> specification = new Specification() {
    @Override
    public Predicate toPredicate(Root root, CriteriaQuery query, CriteriaBuilder criteriaBuilder) {
      Predicate predicate1 = criteriaBuilder.equal(root.get("job"), "SALESMAN");
      Predicate predicate2 = criteriaBuilder.greaterThan(root.get("salary"), 1000);
      Predicate predicate3 = criteriaBuilder.equal(root.get("job"), "MANAGER");
      Predicate predicate4 = criteriaBuilder.lessThan(root.get("salary"), 25000);
      Predicate predicate5 = criteriaBuilder.like(root.get("name"), "A%");

      /*
       * where (job = 'SALESMAN' and salary > 1000) 
       *   or (job = 'MANAGER' and sal < 2500) 
       *   or name like 'A%'
       */
      return criteriaBuilder.or(
        criteriaBuilder.and(predicate1, predicate2),
        criteriaBuilder.and(predicate3, predicate4),
        predicate5
      );
    }
  };

  // 2. findAll() 负责干【找人】这个活。
  List<Employee> list = repository.findAll(specification);

  list.forEach(System.out::println);
  ```

JpaSpecificationExecutor 接口中的方法也支持分页和排序等功能。


## 4. JPA 高级查询：QBC（自学、了解）

### @Query

使用 Spring Data 大部分的 SQL 都可以根据方法名定义的方式来实现，但是由于某些原因必须使用自定义的 SQL 来查询，Spring Data 也可以完美支持。

在 SQL 的查询方法上面使用 ***`@Query`*** 注解，在注解内写 Hql 来查询内容。

```java
@Query("select e from Employee e where e.job = ?1")
List<Employee> xxx(String job);

@Query("select e from Employee e where e.job = :hello")
Page<Employee> yyy(@Param("hello") String job, Pageable pageable);
```

当然如果感觉使用原生 SQL 更习惯，它也是支持的，需要再添加一个参数 *`nativeQuery = true`* 。

```java
@Query(value = "select * from employee where job = ?1", nativeQuery = true)
Page<Employee> zzz(String job, Pageable pageable);
```

### QBC 多表查询

多表查询在 Spring Data JPA 中有两种实现方式：

1. 利用 Hibernate 的级联查询来实现；
2. 是创建一个结果集的接口来接收连表查询后的结果。

这里主要介绍第二种方式。

定义一个结果集的<span class="strong">接口</span>，接口的内容来自于员工表和部门表。

```java
public interface EmployeeInfo {
    Integer getId();
    String getName();
    String getJob();
    Integer getSalary();
    Integer getDepartmentId();
    String getDepartmentName();
    String getDepartmentLocation();
}
```

在运行中 Spring 会给接口（EmployeeInfo）自动生产一个代理类来接收返回的结果，代码中使用 `getXXX()` 的形式来获取。

在 **`EmployeeRepository`** 中添加查询的方法，返回类型设置为 UserInfo：

```java
@Query(value = "select e.id as id," +
        " e.name as name," +
        " e.job as job," +
        " e.salary as salary," +
        " e.departmentId as departmentId," +
        " d.name as departmentName," +
        " d.location as departmentLocation" +
        " from Employee e, Department d" +
        " where e.departmentId = d.id and e.job = ?1")
List<EmployeeInfo> findEmployeeInfo(String job);
```

特别注意这里的 SQL 是 HQL，需要写类的名和属性，这块很容易出错。

- 测试验证：

  ```java
  List<EmployeeInfo> list = repository.findEmployeeInfo("SALESMAN");
  list.forEach(info -> {
    System.out.println(info.getName() + ", " + info.getDepartmentName());
  });
  ```

## 5. open session in view

<span class="strong">因为延迟加载</span>，你有可能在执行代码时，发现报如下错误。

```java
org.hibernate.LazyInitializationException: could not initialize proxy [com.softeem.bean.po.Employee#1] - no Session
	at org.hibernate.proxy.AbstractLazyInitializer.initialize(AbstractLazyInitializer.java:169) ~[hibernate-core-5.3.15.Final.jar:5.3.15.Final]
	at org.hibernate.proxy.AbstractLazyInitializer.getImplementation(AbstractLazyInitializer.java:309) ~[hibernate-core-5.3.15.Final.jar:5.3.15.Final]
	at org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor.intercept(ByteBuddyInterceptor.java:45) ~[hibernate-core-5.3.15.Final.jar:5.3.15.Final]
	at org.hibernate.proxy.ProxyConfiguration$InterceptorDispatcher.intercept(ProxyConfiguration.java:95) ~[hibernate-core-5.3.15.Final.jar:5.3.15.Final]
```

例如，去查询一个员工信息，在 Service 层向 Web 层返回 Employee 对象后，去打印员工的部门的名字。

> 不过，在 Spring Boot 中看到这个问题并不是很容易，因为，Spring Boot 的默认配置启用了 *`opne-session-in-view`* 功能，解决了这个问题。因此，你需要将这个功能关掉，才能看到这个错。
> 
> 关闭的方法是：在配置文件中添加配置：spring.jpa.open-in-view=false 。很显然，这个配置项的默认值是 true 。

出现这个问题的原因在于，当 Service 层的方法执行结束返回 Employee 对象时，与数据库之间的连接已经断开。因为延迟加载的原因，当 Web 层因为 *`.getDepartment()`* 而导致需要查询 Department 时，自然就没有连接/会话来支持这个操作的执行。

> 从 Hibernate 的角度解释是，Employee 对象已经从持久态变为游离态，此时，无法再通过 Employee 对象去查询相关的 Department 对象了。

*`open-session-in-view`* 功能本质上就是延迟 session 的关闭，将原本在 Service 层结束时就该关闭的 Session，推迟到 Web 层结束时再关闭。因此，在 Web 层的方法中，你触发延迟加载时，仍有 session 可用，以支持背后的自动的查询行为。

启用 *`open-session-in-view`* 功能需要进行两步配置。不过这里 Spring Boot 都【帮】我们干完了。

启动类<small>（或某个配置类）</small>中加入 

```java
@Bean
public OpenEntityManagerInViewFilter openEntityManagerInViewFilter() {
   return new OpenEntityManagerInViewFilter();
}
```

配置文件中加入 *`spring.jpa.open-in-view=true`* 。

## 6. @Entity 对象转 JSON 时的一个异常

在 RESTful 风格的项目中，当 Web 层将 Service 层返回的 Entity 对象转换成 JSON 格式字符串时，有可能会报如下错误:

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: com.childdream.price.entity.Price$HibernateProxy$FGboGgyG["hibernateLazyInitializer"])
  ...
```

直接分析原因的话，是因为 jackson 库在将 Entity 序列化为 JSON 格式字符串时，发现其中有一个名为 `hibernateLazyInitializer` 为 null 。这种情况许下下，jackson 不知道该如何处理这个属性，因此它就抛出异常抱错。

但是，问题是我们的 Entity 中好似并没有 `hibernateLazyInitializer` 属性!

这里的关键还是也延迟加载有关。实际上 Hibernate 并未真的去直接使用我们的 Entity，而是利用类似于 AOP 代理机制，去创建并使用了 Entity 的代理对象。从 Service 层返回到 Web 层的并不是真正的 Entity 对象，而是 Entity 的代理对象。

Entity 的代理对象中有这个 `hibernateLazyInitializer` 属性！

解决掉这个异常有两种办法：

1. 在 Entity 的头上标注 `@JsonIgnoreProperties(value = {"hibernateLazyInitializer"}) ` 。

   实际上就是在告诉 jackson，去序列化 Entity 时忽略调其中的 `hibernateLazyInitializer` 属性。

2. 在 Spring Boot 配置文件中配置 `spring.jackson.serialization.FAIL_ON_EMPTY_BEANS=false` 。

   这是在告诉 jackson，去序列化对象时，如果遇到为 null 的属性不要抛出异常，而是继续序列化。

   这种情况下，Web 层返回给前端的 JSON 数据中会多一项：`hibernateLazyInitializer=""` 。


『The End』
