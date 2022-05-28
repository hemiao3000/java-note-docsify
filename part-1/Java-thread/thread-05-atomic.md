# <font color="orange">多线程：原子值</font>

    原子值（atomic）也是 JDK 1.5 的 J.U.C 特性引入的知识点。

如果多个线程更新一个共享计数器，那么你就需要保证更新操作是以线程安全的方式进行的。因为 `i++` 、`i--` 这样的操作是非原子性的，它们是线程不安全的。

JDK<small>（从 1.5 开始）</small> 在 <prev>java.util.concurrent.atomic</prev> 包下面为我们准备了很多可以高效、简洁地『**对 int、long 和 boolean 值、对象的引用和数组进行原子性操作**』的类。

## 1. 简单使用

    以 AtomicLong 为例。

AtomicLong 的 **.incrementAndGet** 方法可以将 AtomicLong 对象的值加 1 ，并返回增加之后的值。即，实现 `++i` 的逻辑，只不过比 `++i` 更高级的是整个操作不能被打断，即，它是原子性的。

```java
AtomicLong i = new AtomicLong(0);

System.out.println( i.incrementAndGet() );
```

AtomicLong 的各个方法的功能都是显而易见的，此处就不一一展示。

不过，需要注意的是，如果你想先读后写 AtomicLong 的值，**不要使用 <prev>.get</prev> 和 <prev>.set</prev> 方法，因为它两的组合不是原子性的**。你要使用的一个 **.updateAndGet** 方法来替代它们两个。<prev>.updateAndGet</prev> 方法要求你传入一个 lambda 表达式，在表达式中它会将 AtomicLong 的原值传进来，你在 lambda 表达式中返回新值。

## 2. AtomicReference

AtomicReference 类提供了一个可以原子读写的对象引用变量。 原子意味着尝试更改相同 AtomicReference 的多个线程<small>（例如，使用比较和交换操作）</small>不会使 AtomicReference 最终达到不一致的状态。 AtomicReference 甚至有一个先进的 **.compareAndSet** 方法，它可以将引用与预期值<small>（引用）</small>进行比较，如果它们相等，则在 AtomicReference 对象内设置一个新的引用。




