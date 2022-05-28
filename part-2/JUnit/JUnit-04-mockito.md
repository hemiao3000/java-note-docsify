# <font color="orange">使用 Mockito 伪造对象</font>

> 如果是 SpringBoot 项目，SpringBoot 项目已经包含了它，无需引入。

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.21.0</version>
    <scope>test</scope>
</dependency>
```


孤立其他的方法<small>（或环境）</small>来测试每一个方法，是一个很好的目标。但是，有些方法总是和外部环境『紧密』联系的，例如，Servlet 类依赖于容器<small>（Tomcat）</small>、Service 层依赖于 Dao 层。

如果无法营造出一个独立的环境，那么当 Service 层中的方法有问题时，你就无法说清楚是 Service 方法本身有问题，还是其依赖的、调用的 Dao 方法有问题。

伪装一个外部环境<small>（或被依赖对象）</small>是隔离测试环境的关键。Mockito 库是一个可以实现模拟外部环境<small>（或被依赖测试对象）</small>的库，<small>当然，它是不是可以实现这个目的的唯一的库</small> 。

Mockito 中 `when(...).thenReturn(...)`  这样的语法来定义对象方法和参数（输入），然后在 `thenReturn` 中指定结果、输出。**它『强行』指定了一个对象在调用某个方法，并传入某个参数时，必定会返回什么值。**


```java
// 伪造对象
HttpServletRequest request = Mockito.mock(HttpServletRequest.class);

// 强行指定伪造对象的某个方法的返回
Mockito.when(request.getParameter("hello")).thenReturn("world");

log.info("{}", request.getParameter("hello"));  // 输出 world
log.info("{}", request.getParameter("hello"));  // 输出 world
log.info("{}", request.getParameter("hello"));  // 输出 world
```

Mockito 甚至可以『伪造』出一个接口的实现类的对象。

```java
List<String> list = Mockito.mock(List.class);

Mockito.when(list.get(0)).thenReturn("Hello World");

log.info("{}", list.get(0));
log.info("{}", list.get(0));
log.info("{}", list.get(0));
```

对于未设定伪造对象的方法，它都是返回默认值：null、0、false 之类。

---

需要『<font color="red">**注意**</font>』的是：

- 对于 **static** 和 **final** 方法， Mockito 无法对其 `when(...).thenReturn(...)` 操作。

- 当我们连续两次为同一个对象设定同一个方法的时，总是最新、最近的那次生效。即，后面的设置会『覆盖』前面的设置。


---

Mockito 还支持迭代风格的返回值设定，这个功能常常用于类似于『迭代器<small>（Iterator）</small>』这样的对象。


```java
Iterator<String> it = Mockito.mock(Iterator.class);

Mockito.when(it.next()).thenReturn("hello").thenReturn("world").thenReturn("goodbye");

System.out.println(it.next());
System.out.println(it.next());
System.out.println(it.next());
```

迭代风格的设定等价于：

```java
Mockito.when(it.next()).thenReturn("hello", "world", "goodbye");
```

---

`when(...).thenReturn(...)` 有另一种写法 `doReturn().when(...)`，这种写法更繁琐一些，但是它和后续的两个概念有统一的格式。

```java
Mockito.when(request.getParameter("hello")).thenReturn("world");
Mockito.doReturn("world").when(request).getParameter("hello");  // 等价于上面

Mockito.when(list.get(0)).thenReturn("Hello World");
Mockito.doReturn("Hello World").when(list).get(0);              // 等价于上面

Mockito.when(it.next()).thenReturn("hello").thenReturn("world").thenReturn("goodbye");
Mockito.doReturn("hello").doReturn("world").doReturn("goodbye").when(it).next(); // 等价于上面
```

---

Mockito 可以模拟调用对象的某个方法时，返回 void，即无返回值。

```java
Mockito.doNothing().when(list).clear(); // 当调用 list 的 clear() 方法时，什么都不干，即无返回值
```

Mockito 可以模拟调用对象的某个方法时，抛出某种异常。

```java
Mockito.doThrow(new SQLException()).when(it).next();
```

doReturn()、doNothing()、doThrow() 可以有迭代风格写法：

```java
Mockito.doReturn("Hello").doNothing().doThrow(new SQLException()).when(it).next();
```

---

如果我们不关心输入的具体内容，可以使用 Mockito 为我们提供的大量通配方法。如：`Mockito.anyInt()` 代表任意 int 型参数值。

```java
List<String> list = Mockito.mock(List.class);

Mockito.when(list.get(Mockito.anyInt())).thenReturn("hello");

System.out.println(list.get(0));        // 输出 hello
System.out.println(list.get(1));        // 输出 hello
System.out.println(list.get(10));       // 输出 hello
System.out.println(list.get(100));      // 输出 hello
System.out.println(list.get(1000));     // 输出 hello
```

---

thenReturn 是返回结果是我们写死的。如果要让被测试的方法不写死，即让方法的返回值具有逻辑，这需要自定义方法执行的返回结果，<strong>Answer 接口</strong>就是满足这样的需求而存在的。

```java
List<String> list = Mockito.mock(List.class);

Answer<String> answer = new Answer<String>() {    

    public String answer(InvocationOnMock invocation) {    
        Object[] args = invocation.getArguments();    

        int i = ((Integer)args[0]).intValue();
        if (i == 1)
            return "hello";
        else if (i == 2)
            return "world";
        else
            return "goodbye";
    }   
};

// Mockito.doAnswer(answer).when(list).get(0);     // 等价于下一行
Mockito.when(list.get(0)).thenAnswer(answer);
Mockito.when(list.get(1)).thenAnswer(answer);
Mockito.when(list.get(2)).thenAnswer(answer);
Mockito.when(list.get(3)).thenAnswer(answer);
Mockito.when(list.get(4)).thenAnswer(answer);

System.out.println(list.get(0));    // 输出 goodbye
System.out.println(list.get(1));    // 输出 hello
System.out.println(list.get(2));    // 输出 world
System.out.println(list.get(3));    // 输出 goodbye
System.out.println(list.get(4));    // 输出 goodbye
```
