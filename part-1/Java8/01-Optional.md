# <font color="orange">如何正确使用 Optional</font>

Java 8 增加了一些很有用的 API，其中一个就是 Optional。目的是从 JDK 级别解决 NPE 问题。大多数新语言都从语法层面上解决了 NPE 问题。

你『**不应该**』以如下方式使用 Optional：

- 调用它的 **isPresent** 方法

- 调用它的 **get** 方法

- 用 Optional 类型作为类/实例属性

- 用 Optional 类型作为方法参数

一句话小结: 使用 Optional 时尽量不要直接调用 Optional 的 **get** 方法, Optional 的 **isPresent** 方法更应该被视为一个私有方法。

## 1. 基本使用

Optional 的 3 种构造方式:

- 明确的 **Optional.empty()**

  它是用来构造一个 Optional 空容器。实际上使用它的机会几乎没有。

- **Optional.of(obj)**

  它要求传入的 obj 不能是 null 值的, 否则还没开始进入角色就倒在了 NullPointerException 异常上了。

- **Optional.ofNullable(obj)**

  它以更智能、宽容的方式来构造一个 Optional 实例，来者不拒。传 null 进到就等价于调用 `Optional.empty()`, 非 null 就等价于调用 `Optional.of(obj)` 。 


使用 **Optional.of(obj)** 来构造 Optional 实例的场景：

  1. 当我们非常的明确将要传给 `Optional.of(obj)` 的 obj 参数不可能为 null 时, 比如它是一个刚 new 出来的对象（例如，`Optional.of(new User(...))`）, 或者是一个非 null 常量时;  

  2. 逻辑上 obj 明确不允许为 null 。一旦为 null ，立即报错，抛出 NPE 。不允许接续执行。

那怎么去使用一个已有的 Optional 实例？假定我们有一个实例 Optional\<User> user。**应避免 if(user.isPresent()) { ... } else { ... } 的方式进行应用**。

存在即返回, 无则提供默认值：

```java
return user.orElse(null);
return user.orElse(UNKNOWN_USER);
```

存在即返回, 无则由函数来产生：

```java
return user.orElseGet(() -> fetchAUserFromDatabase()); 
```

存在才对它做点什么：

```java
user.ifPresent(System.out::println);
 
/* 而不要下边那样
if (user.isPresent()) {
  System.out.println(user.get());
}
*/
```

## 2. map 函数隆重登场

当 **user.isPresent()** 为真, 获得它关联的 orders, 为假则返回一个空集合时, 我们用上面的 **orElse()**, **orElseGet()** 方法都乏力时, 那原本就是 **map** 函数的责任, 我们可以这样一行：

```java
return user.map(u -> u.getOrders()).orElse(Collections.emptyList())

/* 上面避免了我们类似 Java 8 之前的做法
if(user.isPresent()) {
  return user.get().getOrders();
} else {
  return Collections.emptyList();
}
*/
```

map  是可能无限级联的, 比如再深一层, 获得用户名的大写形式

```java
return user.map(u -> u.getUsername())
           .map(name -> name.toUpperCase())
           .orElse(null);
```

这要搁在以前, 每一级调用的展开都需要放一个 null 值的判断

```java
User user = .....
if (user != null) {
    String name = user.getUsername();
    return name != null ? name.toUpperCase() : null;
} else {
      return null;
}
```


## 3. Optional 的使用场景和使用原则

Optional 通常（只应该）出现在方法的返回值类型部分。

```java
public Optional<Department> selectDepartmentByPrimaryKey(Integer deptno) {
    ...
}
```

`Optional<Department>` 和 `Department` 两种不同类型的返回值，代表着方法的作者对方法的使用者的两种『暗示』和『约定』：

- `Department` 类型的返回值

  - 由方法的作者确保该方法 **一定会/必须要** 返回一个非空的 Department 对象。方法的使用者无须判断/担心该方法的返回值为 null 的问题。

  - 即便是出现了返回 null 的问题，责任在方法的作者一方，由他负责修改/调整代码，务必确保该方法返回值为非空。

- `Optional<Department>` 类型的返回值

  - 方法的作者在暗示使用者，本方法的返回值 Optional 容器中存放的 Department 对象可能存在，也可能不存在（为 Null）。逻辑上，相当于本方法的返回值有两种可能。

  - 由方法的使用者，在从 Optional 中取出 Department 对象时来判断-处理有可能为 null 的情况。

  - 此时，方法的使用者，必须通过 `.orElseGet()` 或 `.orElseThrow()` 方法从 Optional 中取值（一定不要使用 `get()` 方法从中取 Department 。

简而言之，`Optional<T>` 类型的返回值，就是在强迫你思考：**这个方法的返回值有可能是 NULL，接下来你需要对这种情况作出处理（给它个默认值，或者抛出异常）** 。
