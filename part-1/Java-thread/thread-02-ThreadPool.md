# <font color="orange">多线程：线程池</font>

线程池<small>（Thread Pool）</small>：把一个或多个线程通过统一的方式进行调度和重复使用的技术，避免了因为线程过多而带来使用上的开销。

为什么要使用线程池？

* 可重复使用已有线程，避免对象创建、消亡和过度切换的性能开销。
* 避免创建大量同类线程所导致的资源过度竞争和内存溢出的问题。
* 支持更多功能。比如，延迟任务线程池<small>（newScheduledThreadPool）</small>和缓存线程池<small>（newCachedThreadPool）</small>等。

创建线程池有两种方式：**ThreadPoolExecutor** 和 Executors<small>（后续讲解）</small> 。 

## 1. ThreadPoolExecutor

### 1.1 ThreadPoolExecutor 的使用

线程池使用代码如下：
    
```java    
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
    2, 10, 
    10L, TimeUnit.SECONDS, 
    new LinkedBlockingQueue(100));

threadPoolExecutor.execute(new Runnable() {
    @Override
    public void run() {
        // 执行线程池
        System.out.println("Hello, Java.");
    }
});

// 以上程序执行结果如下：
// Hello, Java.
``` 

ThreadPoolExecutor 构造方法有 4 个，其中参数最多的那个构造方法有 7 个入参，其它 3 个构造方法本质上就是这个『最长构造方法』的简写。

这 7 个参数名称如下所示：

```java    
public ThreadPoolExecutor(
        int corePoolSize,   // ①
        int maximumPoolSize,// ② 
        long keepAliveTime, // ③ 
        TimeUnit unit,      // ④
        BlockingQueue<Runnable> workQueue,  // ⑤
        ThreadFactory threadFactory,        // ⑥
        RejectedExecutionHandler handler    // ⑦
    ) {
        // ...
}
```    

其代表的含义如下：

1.  corePoolSize

    线程池中的核心线程数，默认情况下核心线程一直存活在线程池中。即，逻辑上，线程池中的最少线程数。
    
2.  maximumPoolSize

    线程池中最大线程数。如果活动的线程达到这个数值以后，ThreadPoolExcecutor 再接收到新任务时，将会阻塞等待，而非创建线程立即执行。

3.  keepAliveTime

    线程池中的线程可闲置的最大时长，<small>默认情况下对非核心线程生效</small>。如果闲置时间超过这个时间，非核心线程就会被回收。
    
    <small>如果 ThreadPoolExecutor 的 allowCoreThreadTimeOut 设为 true ，那么，核心线程也会受该时长影响。</small>

4.  unit

    配合 keepAliveTime 使用，keepAliveTime 的值的时间单位。

5.  workQueue

    线程池中的任务队列，使用 execute 方法或 submit 方法提交的任务都会存储在此队列中。

6.  threadFactory

    为线程池提供创建新线程的线程工厂。

7.  rejectedExecutionHandler

    线程池任务队列超过最大值之后的拒绝策略，RejectedExecutionHandler 是一个接口，里面只有一个 rejectedExecution 方法，可在此方法内添加任务超出最大值的事件处理。
    
    ThreadPoolExecutor 也提供了 4 种默认的拒绝策略：

    * `new ThreadPoolExecutor.DiscardPolicy()`：丢弃掉该任务，不进行处理
    * `new ThreadPoolExecutor.DiscardOldestPolicy()`：丢弃队列里最近的一个任务，并执行当前任务
    * `new ThreadPoolExecutor.AbortPolicy()`：直接抛出 RejectedExecutionException 异常
    * `new ThreadPoolExecutor.CallerRunsPolicy()`：既不抛弃任务也不抛出异常，直接使用主线程来执行此任务

包含所有参数的 ThreadPoolExecutor 使用代码示例：

```java
public class ThreadPoolExecutorTest {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
                    1, 1,
                    10L, TimeUnit.SECONDS, 
                    new LinkedBlockingQueue<Runnable>(2),
                    new MyThreadFactory(), 
                    new ThreadPoolExecutor.CallerRunsPolicy());

        threadPool.allowCoreThreadTimeOut(true);
        for (int i = 0; i < 10; i++) {
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName());
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}

class MyThreadFactory implements ThreadFactory {
    private AtomicInteger count = new AtomicInteger(0);
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            String threadName = "MyThread" + count.addAndGet(1);
            t.setName(threadName);
            return t;
    }
}
```    

### 1.2 线程池执行方法

execute 方法和 submit 方法都是用来执行线程池的。它们的区别在于 submit 方法可以接收线程池执行的返回值。

下面分别来看两个方法的具体使用和区别：

```java    
// 创建线程池
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
    2, 10, 
    10L, TimeUnit.SECONDS, 
    new LinkedBlockingQueue(100));
    // execute 使用
    threadPoolExecutor.execute(new Runnable() {
        @Override   // 逻辑代码
        public void run() {
            System.out.println("Hello, Java.");
        }
});

// submit 使用
Future<String> future = threadPoolExecutor.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        System.out.println("Hello, 老王.");
        return "Success";
    }
});

System.out.println(future.get());
```    

以上程序执行结果如下：

```
Hello, Java.
Hello, 老王.
Success
```

### 1.3 线程池关闭方法

线程池关闭，可以使用 **.shutdown** 或 **.shutdownNow** 方法，它们的区别是：

1. **.shutdown** 方法：不会立即终止线程池，而是要等所有任务队列中的任务都执行完后才会终止。执行完 shutdown 方法之后，线程池就不会再接受新任务了。

2. **.shutdownNow** 方法：执行该方法，线程池的状态立刻变成 STOP 状态，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务，执行此方法会返回未执行的任务。

下面用代码来模拟 shutdown() 之后，给线程池添加任务，代码如下：

    
```java
threadPoolExecutor.execute(() -> {
    for (int i = 0; i < 2; i++) {
        System.out.println("I'm " + i);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            System.out.println(e.getMessage());
        }
    }
});
threadPoolExecutor.shutdown();
threadPoolExecutor.execute(() -> {
    System.out.println("I'm Java.");
});
```    

以上程序执行结果如下：

```
I'm 0

Exception in thread "main" java.util.concurrent.RejectedExecutionException:
Task com.interview.chapter5.Section2`$$Lambda$2`/1828972342@568db2f2 rejected
from java.util.concurrent.ThreadPoolExecutor@378bf509[Shutting down, pool size
= 1, active threads = 1, queued tasks = 0, completed tasks = 0]

I'm 1
```

可以看出，shutdown() 之后就不会再接受新的任务了，不过之前的任务会被执行完成。


### 1.4 总结

**ThreadPoolExecutor** 是创建线程池最传统和最推荐使用的方式，创建时要设置线程池的核心线程数和最大线程数还有任务队列集合，如果任务量大于队列的最大长度，线程池会先判断当前线程数量是否已经到达最大线程数，如果没有达到最大线程数就新建线程来执行任务，如果已经达到最大线程数，就会执行拒绝策略（拒绝策略可自行定义）。

线程池可通过 submit() 来调用执行，从而获得线程执行的结果，也可以通过 shutdown() 来终止线程池。

## 2. Executors

    逻辑上，Executors 是 ThreadPoolExecutor 的工具类。

Executors 可以创建以下 6 种线程池。

| # | 方法 | 说明 |
| :-: | :- | :- |
| 1 | FixedThreadPool               | 创建一个数量固定的线程池，超出的任务会在队列中等待空闲的线程，可用于控制程序的最大并发数。|
| 2 | CachedThreadPool              | 短时间内处理大量工作的线程池，会根据任务数量产生对应的线程，并试图缓存线程以便重复使用，如果限制 60 秒没被使用，则会被移除缓存。|
| 3 | SingleThreadExecutor          | 创建一个单线程线程池。|
| 4 | ScheduledThreadPool           | 创建一个数量固定的线程池，支持执行定时性或周期性任务。|
| 5 | SingleThreadScheduledExecutor | 此线程池就是单线程的 newScheduledThreadPool 。|
| 6 | WorkStealingPool              | Java 8 新增创建线程池的方法，创建时如果不设置任何参数，则以当前机器处理器个数作为线程个数，此线程池会并行处理任务，不能保证执行顺序。|


### 2.1 FixedThreadPool 使用

创建固定个数的线程池，具体示例如下：

```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(2);
for (int i = 0; i < 3; i++) {
    fixedThreadPool.execute(() -> {
        System.out.println("CurrentTime - " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
```    

以上程序执行结果如下：

```
CurrentTime - 2019-06-27 20:58:58
CurrentTime - 2019-06-27 20:58:58
CurrentTime - 2019-06-27 20:58:59
```

根据执行结果可以看出，newFixedThreadPool(2) 确实是创建了两个线程，在执行了一轮<small>（2 次）</small>之后，停了一秒，有了空闲线程，才执行第三次。

### 2.2 CachedThreadPool 使用

根据实际需要自动创建带缓存功能的线程池，具体代码如下：

```java    
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

for (int i = 0; i < 10; i++) {
    cachedThreadPool.execute(() -> {
        System.out.println("CurrentTime - " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
```    

以上程序执行结果如下：

```java
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
CurrentTime - 2019-06-27 21:24:46
```

根据执行结果可以看出，newCachedThreadPool 在短时间内会创建多个线程来处理对应的任务，并试图把它们进行缓存以便重复使用。

### 2.3 SingleThreadExecutor 使用

创建单个线程的线程池，具体代码如下：

    
```java    
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();

for (int i = 0; i < 3; i++) {
    singleThreadExecutor.execute(() -> {
        System.out.println("CurrentTime - " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
```    

以上程序执行结果如下：

```
CurrentTime - 2019-06-27 21:43:34
CurrentTime - 2019-06-27 21:43:35
CurrentTime - 2019-06-27 21:43:36
```

### 2.4 ScheduledThreadPool 使用

创建一个可以执行周期性任务的线程池，具体代码如下：

```java   
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(2);
scheduledThreadPool.schedule(() -> {
    System.out.println("ThreadPool：" + LocalDateTime.now());
}, 1L, TimeUnit.SECONDS);
System.out.println("CurrentTime：" + LocalDateTime.now());
```    

以上程序执行结果如下：

```
CurrentTime：2019-06-27T21:54:21.881
ThreadPool：2019-06-27T21:54:22.845
```

根据执行结果可以看出，我们设置的 1 秒后执行的任务生效了。

### 2.5 SingleThreadScheduledExecutor 使用

创建一个可以执行周期性任务的单线程池，具体代码如下：

```java    
ScheduledExecutorService singleThreadScheduledExecutor = Executors.newSingleThreadScheduledExecutor();
singleThreadScheduledExecutor.schedule(() -> {
    System.out.println("ThreadPool：" + LocalDateTime.now());
}, 1L, TimeUnit.SECONDS);
System.out.println("CurrentTime：" + LocalDateTime.now());
```    

### 2.6 WorkStealingPool 使用

Java 8 新增的创建线程池的方式，可根据当前电脑 CPU 处理器数量生成相应个数的线程池，使用代码如下：

```java    
ExecutorService workStealingPool = Executors.newWorkStealingPool();
for (int i = 0; i < 5; i++) {
    int finalNumber = i;
    workStealingPool.execute(() -> {
        System.out.println("I：" + finalNumber);
    });
}
Thread.sleep(5000);
```    

以上程序执行结果如下：

```
I：0
I：3
I：2
I：1
I：4
```

根据执行结果可以看出，newWorkStealingPool 是并行处理任务的，并不能保证执行顺序。

### 2.7 总结

Executors 可以创建 6 种不同类型的线程池，其中 

-   newFixedThreadPool() 适合执行单位时间内固定的任务数

-   newCachedThreadPool() 适合短时间内处理大量任务

-   newSingleThreadExecutor() 和 newSingleThreadScheduledExecutor() 为单线程线程池。它们的区别在于：

    - newSingleThreadExecutor() 创建的线程池去执行任务时，都是一次性的，而

    - newSingleThreadScheduledExecutor() 可以执行周期性的任务。

-   newWorkStealingPool() 为 JDK 8 新增的并发线程池，可以根据当前电脑的 CPU 处理数量生成对比数量的线程池，但它的执行为并发执行不能保证任务的执行顺序。


### 2.8 其它

【强制】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 

说明：Executors 各个方法的弊端：

1.  newFixedThreadPool 和 newSingleThreadExecutor :

    主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至 OOM 。

2.  newCachedThreadPool 和 newScheduledThreadPool :

    主要问题是线程数最大数是 Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至 OOM 。