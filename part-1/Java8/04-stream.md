# <font color="orange">Stream API</font>

Stream 是 Java 8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作。

当你使用 Stream 时，你会通过三个阶段来建立一个操作流水线。

| # | 阶段 | 说明 |
| :-: | :- | :- |
| 1 | 生成 | 创建一个 Stream 。|
| 2 | 操作 | 在一个或多个步骤中，指定将初始 Stream 转换为另一个 Stream 的中间操作。 |
| 3 | 数据收集 | 使用一个终止操作来产生一个结果。该操作会强制它之前的延迟操作立即执行。在这之后，该 Stream 就不会再被使用了。|

例如：

```java
String[] array = new String[]{"hello", "world", "goodbye"};
Stream<String> stream = 
List<String> list = Arrays
    .stream(array)                  // 1. 生成
    .map(String::toUpperCase)       // 2. 操作
    .collect(Collectors.toList());  // 3. 数据收集
```

## 1. 生成：创建 Stream

### 1.1 由集合&数组生成 Stream 

Stream 作为元素的『序列』，自然而然，我们想到通过集合、数组生成 Stream 。

Java 8 在 Collection 接口中新添加了 **.stream** 方法，可以将任何集合转化为一个 Stream 。

```java
List<String> list = new ArrayList<>();
...

Stream<String> stream = list.stream();
```

如果你面对的是一个数组，也可以用静态的 **Stream.of** 方法将它转化为一个 Stream 。

```java
String[] array = new String[]{"hello", "world", "goodbye"};
Stream<String> stream = Stream.of(array);
```

**Stream.of** 方法有一个重载形式，接收可变长度的参数：

```java
Stream.of(xxx, xxx, xxx, xxx);
```

另外，JDK 的 **Arrays** 工具类中也提供了一个 **.stream** 方法用以将数组转化为 Stream 。

```java
stream = Stream.of(array);
```

特别地，当数组元素类型 T 是原始类型，静态方法 **.stream(T[])** 将返回原始类型的 Stream ：

```java
IntStream stream = Arrays.stream(array);
```

### 1.2 直接创建 Stream 

除了由集合和数组生成 Stream，Stream API 为 Stream 提供了静态方法 **.generate(Supplier)** 方法、**.iterator(Final T, final UnaryOperator\<T> f)** 方法，直接创建 Stream 。

> 这里需要注意的是，上一章节中，通过数组&集合生成的 Stream 对象中的数据的数量是确定的，即为数组和集合中的数据的数量。 而通过 **.generate** 和 **.iterator** 方法生成的 Stream 对象中的数据的数量是无限的，即，你向 Stream 对象每次『要』一个对象时它都会每次生成一个返回给你，从而达到『无限个』的效果。

通常直接创建的 Stream 对象会结合 **.limit()** 方法使用。

*`Stream.generate(Supplier)`* 通过参数 Supplier 获取 Stream 序列的新元素。

```java
Stream<Double> stream = Stream.generate(Math::random).limit(10);
```

**.iterator** 方法要求你提供两个参数：

- 数据序列中的第一个数。这个数字需要使用者认为指定，通常也被称为『种子』。

- 根据『前一个数字』计算『下一个数字』的计算规则。

```java
Stream<Integer> stream = Stream.iterate(1, n -> n += 2);
// 1, 3, 5, 7, 9, 11, 13, 15, 17, 19, ...
```

整个序列的值就是：x, f(x), f(f(x)), f(f(f(x))), ...


## 2. 中间操作：处理 Stream 中的数据序列 

所谓的中间操作，指的就是对 Stream 中的数据使用流转换方法。

filter 和 map 方法是 stream 最常见的两个流转换方法。

### 2.1 filter 方法

**.filter** 是过滤转换，它将产生一个新的流，其中『**包含符合某个特定条件**』的所有元素。逻辑上就是『**选中**』的功能。

例如：

```java
String[] array = new String[]{"hello", "world", "goodbye"};

Stream<String> stream = Arrays.stream(array);
Stream<String> newStream = stream.filter((item) -> item.startsWith("h"));

System.out.println(newStream.count());  // 1
```

**.filter** 方法的参数是一个 **Predicate\<T>** 对象——即一个从 T 到 boolean 的函数。

### 2.2 map 方法

我们经常需要对一个流中的值进行某种形式的转换。这是可以考虑使用 **.map** 方法，并传递给它一个负责进行转换的函数。

例如：

```java
newStream = stream.map(String::toUpperCase);
```

### 2.3 anyMatch / allMatch / noneMatch 判断

anyMatch / allMatch / noneMatch 三个方法用于判断 Stream 中的元素是否『**至少有一个**』、『**全部都**』、『**全部都不**』满足某个条件。因此，这三个方法的返回值都是 boolean 值。

```java
boolean b = stream.anyMatch((item) -> item.length() == 5);
```

## 3. 数据收集

当经过第 2 步的操作之后，你肯定会需要查看或收集流中<small>（经过处理）</small>的数据。

### 3.1 iterator 方法

Stream 的 iterator 方法会返回一个传统风格的迭代器，用于访问元素。

```java
Iterator<String> it = stream.iterator();
while (it.hasNext()) {
    String str = it.next();
    System.out.println(str);
}
```

### 3.2 toArray 方法

Stream 的 **.toArray** 方法会返回一个包含了流中所有元素的数组。

```java
Object[] array = stream.toArray();
```

不过，**.toArray** 方法返回的是一个 `Object[]` 数组。如果想获得一个正确类型的数组，可以将数组类型的构造函数传递给它：

```java
String[] array = stream.toArray(String[]::new); // 注意，这里是 String[] 而不是 String
```

### 3.3 collect：收录到集合

如果你想将 Stream 中的数据收录到集合中，那么你可以使用 **.collect** 方法。


> 不过最原始最底层的 **.collect** 方法看起来比较『奇怪』：
> 
> ```java
> stream.collect(HashSet::new, HashSet::add, HashSet::addAll);
> stream.collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
> ```
> 
> 这里的第一第二个参数好理解，比较奇怪的是第三个参数。这里需要用到集合的 **.addAll** 方法是因为会将 Stream 中的数据先存放于多个集合中，最后再将多个集合合并成一个总的集合中再返回<small>（这种『奇怪』的行为是和 Stream API 的并发特性有关）</small>。

为此，Stream 提供了几个重载的 **.collect** 方法简化使用：

```java
stream.collect(Collectors.toCollection())
stream.collect(Collectors.toList());
stream.collect(Collectors.toSet())
```

### 3.4 collect：收录到 Map

假设你有一个 `Stream<Student>`，并且你想将其中的元素收集到一个 map 中，这样你随后可以通过他们的 ID 来进行查找。为了实现这个目的，你可以再 **.collect** 方法中使用 **Collectors.toMap** 。

**Collectors.toMap** 有两个函数参数，分别用来生成 map 的 key 和 value。例如：

```java
Map<Integer, String> map = stream.collect(
    Collectors.toMap(Student::getId, Student::getName)
);
```

一般来说，Map 的 value 通常会是 Student 元素，这样的话可以实使用 `Function.identity()` 作为第二个参数，来实现这个效果。（你可以将它视为固定用法）。

```java
Map<Integer, Student> map = stream.collect(
    Collectors.toMap(Student::getId, Function.identity())
);
```

