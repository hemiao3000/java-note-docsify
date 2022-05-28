# <font color="orange">多线程：锁</font>

## 1. 各种锁的概念

### 1.1 悲观锁和乐观锁

『**悲观锁**』和『**乐观锁**』并不是某个具体的 `锁` 而是一种并发编程的基本『概念』。乐观锁和悲观锁最早出现在数据库的设计当中，后来逐渐被 Java 的并发包所引入。

| # | 说明 |
| :- | :- |
| 悲观锁 | 悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观地认为，不加锁的并发操作一定会出问题。|
| 乐观锁 | 乐观锁正好和悲观锁相反，它获取数据的时候，并不担心数据被修改，每次获取数据的时候也不会加锁，只是在更新数据的时候，通过判断现有的数据是否和原数据一致来判断数据是否被其他线程操作，如果没被其他线程修改则进行数据更新，如果被其他线程修改则不进行数据更新。|

### 1.2 公平锁和非公平锁

根据线程获取锁的抢占机制，锁又可以分为『**公平锁**』和『**非公平锁**』。

| # | 说明 |
| :- | :- |
| 公平锁 | 公平锁是指多个线程按照申请锁的顺序来获取锁。 |
| 非公平锁 | 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。  |

ReentrantLock 提供了公平锁和非公平锁的实现。

```java
// 公平锁
Lock lock = new ReentrantLock(true);

// 非公平锁
Lock lock = new ReentrantLock(); // 或
Lock lock = new ReentrantLock(false);
```

### 1.3 独占锁和共享锁

根据锁能否被多个线程持有，可以把锁分为『**独占锁**』和『**共享锁**』。

| # | 说明 |
| :- | :- |
| 独占锁 | 独占锁是指任何时候都只有一个线程能执行资源操作。|
| 共享锁 | 共享锁指定是可以同时被多个线程读取，但只能被一个线程修改。| 

> Java 中 Lock 是一个接口。

Java 中的 **ReentrantReadWriteLock** 就是共享锁的实现方式，它允许一个线程进行写操作，允许多个线程读操作。

ReentrantReadWriteLock 共享锁演示代码如下：

```java
public static void main(String[] args) throws InterruptedException {

    ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    // 读线程
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + ": 写线程持有锁");
        lock.writeLock().lock();
        try {
            TimeUnit.SECONDS.sleep(30);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ": 写线程释放锁");
        lock.writeLock().unlock();
    }).start();

    // 读线程
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + ": 读线程持有锁");
        lock.readLock().lock();
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ": 读线程释放锁");
        lock.readLock().unlock();
    }).start();

    // 重复多个读线程
    // new Thread(...).start();
    // new Thread(...).start();

    // 挂起主线程
    Scanner scanner = new Scanner(System.in);
    scanner.next();
}
```    

以上程序执行结果如下：

```
Thread-2: 写线程持有锁
Thread-3: 读线程持有锁
Thread-4: 读线程持有锁
Thread-5: 读线程持有锁
Thread-2: 写线程释放锁
Thread-3: 读线程释放锁
Thread-5: 读线程释放锁
Thread-4: 读线程释放锁
```

### 1.4 可重入锁

『**可重入锁**』指的是该线程获取了该锁之后，可以无限次的进入该锁锁住的代码。我们常用的锁都是可重入锁。

```java
Lock lock = new ReentrantLock();

lock.lock();
lock.lock();
lock.lock();

lock.unlock();
lock.unlock();
lock.unlock();
```

### 1.5 自旋锁

『**自旋锁**』是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗 CPU 。

在 Java 中，自旋锁是通过 CAS（Compare and Swap）实现的。

-   简单实现

    ```java
    AtomicInteger cas = new AtomicInteger(0);

    new Thread(() -> {
        // 上锁逻辑
        while (!cas.compareAndSet(0, 1))
            ;
        System.out.println(Thread.currentThread().getName() + ": 获得锁");
        TimeUnitSecondsQuietlySleep(5);
        System.out.println(Thread.currentThread().getName() + ": 释放锁");

        // 解锁逻辑
        cas.compareAndSet(1, 0);
    }).start();

    new Thread(() -> {
        // 上锁逻辑
        while (!cas.compareAndSet(0, 1))
            ;
        System.out.println(Thread.currentThread().getName() + ": 获得锁");
        TimeUnitSecondsQuietlySleep(5);
        System.out.println(Thread.currentThread().getName() + ": 释放锁");
        // 解锁逻辑
        cas.compareAndSet(1, 0);
    }).start();

    Scanner scanner = new Scanner(System.in);
    scanner.next();
    ```

-   优化：自定义 SpinLock 类

    ```java
    public class SpinLock {

        private AtomicReference<Thread> cas = new AtomicReference<Thread>();

        public void lock() {
            Thread current = Thread.currentThread();
            // 利用 CAS
            while (!cas.compareAndSet(null, current))
                ;
        }

        public void unlock() {
            Thread current = Thread.currentThread();
            cas.compareAndSet(current, null);
        }

    }
    ```

## 2. CAS 与 ABA 问题

CAS<small>（Compare and Swap）</small>比较并交换，是一种『乐观锁』的实现，是用非阻塞算法来代替锁定，其中 java.util.concurrent 包下的 **AtomicInteger** 就是借助 CAS 来实现的。  

但 CAS 也不是没有任何副作用，比如著名的 ABA 问题就是 CAS 引起的。

### 2.1 ABA 问题描述

老王去银行取钱，余额有 200 元，老王取 100 元，但因为程序的问题，启动了两个线程，线程一和线程二进行比对扣款，线程一获取原本有 200 元，扣除 100 元，余额等于 100 元，此时阿里给老王转账 100 元，于是启动了线程三抢先在线程二之前执行了转账操作，把 100 元又变成了 200 元，而此时线程二对比自己事先拿到的 200 元和此时经过改动的 200 元值一样，就进行了减法操作，把余额又变成了 100 元。这显然不是我们要的正确结果，我们想要的结果是余额减少了 100 元，又增加了 100 元，余额还是 200 元，而此时余额变成了 100 元，显然有悖常理，这就是著名的 ABA 的问题。

执行流程如下。

  * 线程一：取款，获取原值 200 元，与 200 元比对成功，减去 100 元，修改结果为 100 元。
  * 线程二：取款，获取原值 200 元，阻塞等待修改。
  * 线程三：转账，获取原值 100 元，与 100 元比对成功，加上 100 元，修改结果为 200 元。
  * 线程二：取款，恢复执行，原值为 200 元，与 200 元对比成功，减去 100 元，修改结果为 100 元。

最终的结果是 100 元。

### 2.2 ABA 问题的解决

常见解决 ABA 问题的方案加版本号，来区分值是否有变动。以老王取钱的例子为例，如果加上版本号，执行流程如下。

  * 线程一：取款，获取原值 200_V1，与 200_V1 比对成功，减去 100 元，修改结果为 100_V2。
  * 线程二：取款，获取原值 200_V1 阻塞等待修改。
  * 线程三：转账，获取原值 100_V2，与 100_V2 对比成功，加 100 元，修改结果为 200_V3。
  * 线程二：取款，恢复执行，原值 200_V1 与现值 200_V3 对比不相等，退出修改。

最终的结果为 200 元，这显然是我们需要的结果。  

---

在程序中，要怎么解决 ABA 的问题呢？  

在 JDK 1.5 的时候，Java 提供了一个 **AtomicStampedReference** 原子引用变量，通过添加版本号来解决 ABA 的问题，具体使用示例如下：

    
```java    
String name = "老王";
String newName = "Java";
AtomicStampedReference<String> as = new AtomicStampedReference<String>(name, 1);

log.debug("值：{} | Stamp：{}", as.getReference(), as.getStamp());

as.compareAndSet(name, newName, as.getStamp(), as.getStamp() + 1);

log.debug("值：{} | Stamp：{}",  as.getReference(), as.getStamp());
```    

以上程序执行结果如下：

```
值：老王 | Stamp：1
值：Java | Stamp：2
```

## 3. synchronized 和 volatile

synchronized 是悲观锁的实现，因为 synchronized 修饰的代码，每次执行时会进行加锁操作，同时只允许一个线程进行操作，所以它是悲观锁的实现。

synchronized 使用的是非公平锁，并且是不可设置的。这是因为非公平锁的吞吐量大于公平锁，并且是主流操作系统线程调度的基本选择，所以这也是 synchronized 使用非公平锁原由。


> 为什么非公平锁吞吐量大于公平锁？
> 
> 比如 A 占用锁的时候，B 请求获取锁，发现被 A 占用之后，堵塞等待被唤醒，这个时候 C 同时来获取 A 占用的锁，如果是公平锁 C 后来者发现不可用之后一定排在 B 之后等待被唤醒，而非公平锁则可以让 C 先用，在 B 被唤醒之前 C 已经使用完成，从而节省了 C 等待和唤醒之间的性能消耗，这就是非公平锁比公平锁吞吐量大的原因。


volatile 是 Java 虚拟机提供的最轻量级的同步机制。当变量被定义成 volatile 之后，具备 2 种特性：

1. 保证此变量对所有线程的可见性，当一条线程修改了这个变量的值，修改的新值对于其他线程是可见的（可以立即得知的）；

2. 禁止指令重排序优化，普通变量仅仅能保证在该方法执行过程中，得到正确结果，但是不保证程序代码的执行顺序。

synchronized 既能保证可见性，又能保证原子性，而 volatile 只能保证可见性，无法保证原子性。比如，i++ 如果使用 synchronized 修饰是线程安全的，而 volatile 会有线程安全的问题。

