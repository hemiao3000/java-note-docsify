# <font color="orange">FutureTask</font>

使用 Thread 和 Runnable 实现线程来有一个小问题：不方便获得线程的执行结果。

为此，J.U.C 中提供了 Callable 接口和 Future 接口解决这个问题。

Callable 接口和 Runnable 接口很像，其中的 `call()` 方法是需要我们自己实现的线程执行方法。

不同的是 Runnable 接口的 *`run()`* 方法没有返回值，而 Callable 接口的 *`call()`* 方法有返回值，而这个返回值需要通过一个 Futrue 对象来获取。

Callable 对象不像 Runnable 对象那样可以作为 *`new Thread()`* 的参数来创建一个线程来运行它的 *`call()`* 方法。它需要【另一种】方式来使用。


## 1. Callable 使用方式一：FutureTask

FutureTask 对象类似于 Thread 对象，它是 Callable 对象的运行的执行者。创建 FutureTask对象，并传入 Callable 对象，再调用它的 *`run()`* 方法，就可以让一个新的线程去执行 Callable 对象的 *`call`* 方法。

至于如何获得 Callable 方法的返回值，这就需要调用 FutureTask 对象的 *`get()`* 方法。注意，这个方法有可能导致当前线程的阻塞，这取决于运行 Callable 的 *`run()`* 方法的线程当前是否执行结束了。

```java
Callable<String> c = () -> {
    System.out.println("hello world");
    Thread.sleep(5000);
    return "Hello World";
};

FutureTask<String> task = new FutureTask<>(c);
task.run();

System.out.println(task.get());
```

## 2. Callable 使用方式二：线程池和Future

第二种用法和第一种用法本质上并没有太大区别，只不过是将 Callable 对象交过线程池，由线程池中的线程去执行而已。

```java
ExecutorService service =  Executors.newFixedThreadPool(3);

Callable<String> callable = new Callable<String>() {
    @Override
    public String call() throws Exception {
        System.out.println("HELLO WORLD");
        Thread.sleep(3000);
        return "hello world";
    }
};

Future<String>  future = service.submit(callable);
System.out.println(future.get());
```

