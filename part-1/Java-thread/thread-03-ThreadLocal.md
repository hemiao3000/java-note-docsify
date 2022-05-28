# <font color="orange">多线程：ThreadLocal</font>

ThreadLocal 诞生于 JDK 1.2 ，用于解决多线程间的数据隔离问题。也就是说 ThreadLocal 会为每一个线程创建一个单独的变量副本。

ThreadLocal 最典型的使用场景有两个：

- ThreadLocal 可以用来管理 Session，因为每个人的信息都是不一样的，所以就很适合用 ThreadLocal 来管理；

- 数据库连接，为每一个线程分配一个独立的资源，也适合用 ThreadLocal 来实现。

其中，ThreadLocal 也被用在很多大型开源框架中，比如 Spring 的事务管理器，还有 Hibernate 的 Session 管理等。


## 1. ThreadLocal 基础使用

ThreadLocal 常用方法有 set(T)、get()、remove() 等，具体使用请参考以下代码。

    
```java    
ThreadLocal threadLocal = new ThreadLocal();

// 存值
threadLocal.set(Arrays.asList("hello", "world"));

// 取值
List list = (List) threadLocal.get();
System.out.println(list.size());
System.out.println(threadLocal.get());

// 删除值
threadLocal.remove();
System.out.println(threadLocal.get());
```    

以上程序执行结果如下：

```
2
[hello, world]
null
```

## 2. ThreadLocal 数据共享

既然 ThreadLocal 设计的初衷是解决线程间信息隔离的，那 ThreadLocal 能不能实现线程间信息共享呢？  

答案是肯定的，只需要使用 ThreadLocal 的子类 InheritableThreadLocal 就可以轻松实现，来看具体实现代码：

    
```java    
ThreadLocal inheritableThreadLocal = new InheritableThreadLocal();
inheritableThreadLocal.set("老王");
new Thread(() -> System.out.println(inheritableThreadLocal.get())).start();
```    

以上程序执行结果如下：

> 老王

从以上代码可以看出，主线程和新创建的线程之间实现了信息共享。


## 3. ThradLocal 的正确使用方式（内存泄漏问题）


```java
// jvm 内存参数调低以便于快速触发内存不足：-Xmx50M
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor t = new ThreadPoolExecutor(
        4, 4, 
        0, TimeUnit.SECONDS, 
        new ArrayBlockingQueue<>(1000));
    for (int i = 0; i < 100; i++) {
        Thread.sleep(200);
        t.execute(() -> {
                System.out.println(Thread.currentThread().getName());
                ThreadLocal<int[]> threadLocal = new ThreadLocal<>();
                int[] arr = new int[1024 * 1024];
                threadLocal.set(arr);
                //threadLocal.remove();
            });
    }
    t.shutdown();
}
```

执行上述代码，你会发现 `java.lang.OutOfMemoryError: Java heap space` 异常。原因在于：

ThreadLocal 并不存储数据，而是依靠每个线程中的 ThreadLocalMap 来存储数据。毫无疑问，ThreadLocalMap 是一个 Map 结构，其中存的是键值对：键是 ThreadLocal 对象，而值就是你所要存储的值。

简而言之，你以为存进 ThreadLocal 中的数据实际上并没有存到 ThreadLocal 中，而是存到了一个 Map 中的 Value 部分。例如，你有 3 个 ThreadLocal 分别村的是 String 类型的值、Integer 类型的值和 Double 类型的值：那么 ThreadLocalMap 中的内容类似如下：

| Key    | Value   |
| :----- | :------ |
| local1 | String 对象  |
| local2 | Integer 对象 |
| local3 | Double 对象  |


但是，问题在于：**每个程中有且仅有一个 ThreadLocalMap ，并且，它的 Key 是 ThraedLocal 的软引用** ！

也就是意味着，当 ThreadLocal 对象没有其它的强引用时，在 JVM 进行垃圾回收时，它们会被释放掉，那么这就意味着，ThreadLocalMap 中会出现 Key 为 null 的键值对，并且，键值对中的值将无法被 JVM 回收，从而最终出现内存溢出。

解决这个问题的关键在于：在不再需要的时候，一定要记得调用 `.remove` 方法做 `.set` 方法的反向操作，移除存储的数据。这就是正确使用 ThreadLocal 的方式。




