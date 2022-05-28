# <font color="orange">多线程：synchronized 和 ReentrantLock</font>

线程安全问题指的是在多线程中，各线程之间因为同时操作所产生的数据污染或其他非预期的程序运行结果。

## 1. 线程安全

### 1.1 非线程安全事例

比如 A 和 B 同时给 C 转账的问题，假设 C 原本余额有 100 元，A 给 C 转账 100 元，正在转的途中，此时 B 也给 C 转了 100 元，这个时候 A 先给 C 转账成功，余额变成了 200 元，但 B 事先查询 C 的余额是 100 元，转账成功之后也是 200 元。当 A 和 B 都给 C 转账完成之后，余额还是 200 元，而非预期的 300 元，这就是典型的线程安全的问题。

| 时间线 | 线程 A | 线程 B | C 的余额 |
| :- | :- | :- | :- |
| 1 | 查看 C 的余额 | | 100 |
| 2 | | 查看 C 的余额 | 100 |
| 3 | 转账：余额 += 100 | | 200 | 
| 4 | | 转账：余额 += 100 | 200 |


### 1.2 非线程安全代码示例

上面的内容没看明白没关系，下面来看非线程安全的具体代码：

```java
class ThreadSafeTest {

    static int number = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(() -> addNumber());
        Thread thread2 = new Thread(() -> addNumber());
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("number：" + number);
    }

    public static void addNumber() {
        for (int i = 0; i < 10000; i++) {
            ++number;
        }
    }
}
```    

以上程序执行结果如下：

```
number：12085
```

每次执行的结果可能略有差异，不过几乎不会等于<small>（正确的）</small>累计之和 20000 。

### 1.3 线程安全的解决方案

线程安全的解决方案有以下几个维度：

- 数据不共享，单线程可见，比如 ThreadLocal 就是单线程可见的；

- 使用线程安全类，比如 StringBuffer 和 JUC<small>（java.util.concurrent）</small>下的安全类；

- 使用同步代码或者锁。

## 2. synchronized

**synchronized** 是 Java 提供的同步机制，当一个线程正在操作同步代码块<small>（synchronized 修饰的代码）</small>时，其他线程只能阻塞等待原有线程执行完再执行。

synchronized 可以修饰代码块或者方法，示例代码如下：

```java
// 修饰代码块
synchronized (this) {
    // do something
}

// 修饰方法
synchronized void method() {
    // do something
}
```    

使用 synchronized 完善本文开头的非线程安全的代码。

**方法一：使用 synchronized 修饰代码块** ，代码如下：

    
```java    
class ThreadSafeTest {

    static int number = 0;

    public static void main(String[] args) 
            throws InterruptedException {

        Thread thread1 = new Thread(() -> {
            // 同步代码
            synchronized (ThreadSafeTest.class) {
                addNumber();
            }
        });

        Thread thread2 = new Thread(() -> {
            // 同步代码
            synchronized (ThreadSafeTest.class) {
                addNumber();
            }
        });

        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("number：" + number);
    }

    public static void addNumber() {
        for (int i = 0; i < 10000; i++) {
            ++number;
        }
    }
}
```

以上程序执行结果如下：

> number：20000

**方法二：使用 synchronized 修饰方法** ，代码如下：

```java    
class ThreadSafeTest {
    static int number = 0;
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> addNumber());
        Thread t2 = new Thread(() -> addNumber());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("number：" + number);
    }

    public synchronized static void addNumber() {
        for (int i = 0; i < 10000; i++) {
            ++number;
        }
    }
}
```    

以上程序执行结果如下：

```
number：20000
```

## 3. synchronized 实现原理

synchronized 本质是通过进入和退出的 Monitor 对象来实现线程安全的。  

以下面代码为例：


```java    
public class SynchronizedTest {
    public static void main(String[] args) {
        synchronized (SynchronizedTest.class) {
            System.out.println("Java");
        }
    }
}
```    


JVM<small>（Java 虚拟机）</small>采用 **monitorenter** 和 **monitorexit** 两个指令来实现同步的：

- monitorenter 指令相当于加锁；

- monitorexit 相当于释放锁。


## 4. ReentrantLock

ReentrantLock<small>（再入锁、可重入锁）</small>是 Java 5 提供的锁实现，它的功能和 synchronized 基本相同。ReentrantLock 通过调用 **lock** 方法来获取锁，通过调用 **unlock** 来释放锁。

### 4.1 ReentrantLock 使用

**ReentrantLock 基础使用** ，代码如下：
    
```java
Lock lock = new ReentrantLock();
lock.lock();    // 加锁
// 业务代码...
lock.unlock();    // 解锁
```

使用 ReentrantLock 完善本文开头的非线程安全代码：

```java
public class LockTest {

    static int number = 0;

    public static void main(String[] args) 
            throws InterruptedException {
        // ReentrantLock 使用
        Lock lock = new ReentrantLock();
        Thread thread1 = new Thread(() -> {
            try {
                lock.lock();
                addNumber();
            } finally {
                lock.unlock();
            }
        });

        Thread thread2 = new Thread(() -> {
            try {
                    lock.lock();
                    addNumber();
            } finally {
                    lock.unlock();
            }
        });

        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("number：" + number);
    }

    public static void addNumber() {
        for (int i = 0; i < 10000; i++) {
            ++number;
        }
    }
}
```

-   尝试获取锁

    ReentrantLock 可以无阻塞尝试访问锁，使用 **ReentrantLock#tryLock** 方法。

    ```java 
    Lock reentrantLock = new ReentrantLock();

    // 线程一
    new Thread(() -> {
        try {
            reentrantLock.lock();
            Thread.sleep(2 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }).start();

    // 线程二
    new Thread(() -> {
        try {
            Thread.sleep(1 * 1000);
            System.out.println(reentrantLock.tryLock());    // false
            Thread.sleep(2 * 1000);
            System.out.println(reentrantLock.tryLock());    // true
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
    ```


-   **尝试一段时间内获取锁**

    ReentrantLock#lock 默认是以非阻塞方式获取锁，如果获取不到，则立即以 false 返回。

    **ReentrantLock#tryLock** 有一个重载方法 **ReentrantLock#tryLock(long timeout, TimeUnit unit)**，用于尝试一段时间内获取锁<small>（而非立即返回）</small>。

    注意，在此期间，ReentrantLock 是一直尝试，而非等待一段时间后再试。

    ```java 
    Lock reentrantLock = new ReentrantLock();

    // 线程一
    new Thread(() -> {
        try {
            reentrantLock.lock();
            System.out.println(LocalDateTime.now());
            Thread.sleep(2 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }).start();

    // 线程二
    new Thread(() -> {
        try {
            Thread.sleep(1 * 1000);
            System.out.println(reentrantLock.tryLock(3, TimeUnit.SECONDS));
            System.out.println(LocalDateTime.now());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
    ```

    以上代码执行结果如下：

        2019-07-05 19:53:51
        true
        2019-07-05 19:53:53

    可以看出锁在休眠了 2 秒之后，就被线程二直接获取到了，所以说 **tryLock(long timeout, TimeUnit unit)** 方法内的 timeout 参数指的是获取锁的最大等待时间。

### 4.2 ReentrantLock 注意事项

- 使用 ReentrantLock 一定要记得释放锁，否则该锁会被永久占用。

- lock - unlock 应该成对出现，即，lock 了多少次，就要有相应多少次的 unlock 。

  <small>如果有需要，你可以通过 **ReentrantLock#getHoldCount** 方法查询当前线程执行 lock的次数</small>

## 5. synchronized 和 ReentrantLock 有什么区别？

synchronized 和 ReentrantLock 都是保证线程安全的，它们的区别如下：

* ReentrantLock 使用起来比较灵活，但是必须有释放锁的配合动作；
* ReentrantLock 必须手动获取与释放锁，而 synchronized 不需要手动释放和开启锁；
* ReentrantLock 只适用于代码块锁，而 synchronized 可用于修饰方法、代码块等；
* ReentrantLock 性能略高于 synchronized。

## 6. 条件（Condition）等待

Java 中条件变量都实现了 *java.util.concurrent.locks.Condition* 接口，条件变量的实例化是通过一个 Lock 对象上调用 `newCondition()` 方法来获取的，这样，条件就和一个锁对象绑定起来了。因此，Java 中的条件变量只能和锁配合使用，来控制并发程序访问竞争资源的安全。


在 Condition 中，用 **await()** 替换 wait()，用 **signal()** 替换 notify()，用 **signalAll()** 替换 notifyAll()，传统线程的通信方式，Condition 都可以实现。再次提醒，Condition 是被绑定到 Lock 上的，要创建一个 Lock 的 Condition 必须用 newCondition() 方法。 这样看来，Condition 和传统的线程通信没什么区别，Condition 的强大之处在于它可以为多个线程间建立不同的 Condition 。



## 7. 信号量（Semaphore）


Java 通过 Semaphore 类实现了经典的信号量。信号量通过计数器来控制对共享资源的访问。

- 如果计数器大于 0， 则允许访问；
- 如果计数器为 0，则不允许访问。

计数器的计数逻辑上代表着当前共享资源的许可证的数量。

Semaphore 类具有如下所示的两个构造函数：

```java
Semaphore(int num)
Semaphore(int num, boolean how)
```

其中，`num` 指定了初始的许可证计数大小。因此，num 指定了任意时刻可以访问共享资源的线程数量。如果，num 是 1，那么任意时刻只有一个线程能够访问资源。

默认情况下，等待获取许可证的线程以随机的方式『抢夺』许可证。通过将 `how` 设置为 true ，可以确保等待的线程以前后顺序获得许可证。<small>即，是否是公平锁。</small>

为了得到许可证，可以调用 **Semaphore#acquire** 方法，该方法具有以下 2 种形式：

```java
// 获得 1 个许可证
void acquire() throws InterruptedException

// 获得 num 个许可证
void acquire(int num) throws InterruptedException
```

如果调用时无法获得许可证，就会挂起线程<small>（阻塞）</small>，直到许可证可以获得为止。

为了释放许可证，可以调用 **Semaphore#release** 方法，该方法具有以下 2 种形式：

```java
// 释放 1 个许可证
void release()

// 释放 num 个许可证
void relase(int num)
```

注意：为了使用信号量控制对资源的访问，在访问资源之前，希望使用资源的每个线程必须先调用 **acquire** 方法；当线程使用完资源时，必须调用 **release** 方法。
