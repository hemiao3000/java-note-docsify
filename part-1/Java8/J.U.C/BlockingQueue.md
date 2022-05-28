<span class="title">BlockingQueue</span>

BlockingQueue 是 J.U.C 中提供的一个阻塞队列，用以简化并发编程，特别是简化生产者消费者模型的多线程使用场景。例如下面这样的任务：

> 有两个线程 A、B，  A 线程每 200ms 就生成一个 [0, 100] 之间的随机数， B 线程每 2s 中打印出 A 线程所产生的增量随机数。

```java
public class BlockingQueueTest {

    /**
     * 实例化一个队列，队列中的容量为 10
     */
    private static BlockingQueue<Integer> blockingQueue = new LinkedBlockingQueue<>(10);
    public static void main(String[] args) {
        ScheduledExecutorService product = Executors.newScheduledThreadPool(1);
        Random random = new Random();
        product.scheduleAtFixedRate(() -> {
            int value = random.nextInt(101);
            try {
                blockingQueue.offer(value);  // offer() 方法就是向队列的尾部添加值
            } catch(Exception ex) {
                ex.printStackTrace();
            }
        }, 0, 200, TimeUnit.MILLISECONDS);  // 每 100 毫秒执行线程

        new Thread(() -> {
            while(true){
                try {
                    Thread.sleep(2000);
                    System.out.println("开始取值");
                    List<Integer> list = new LinkedList<>();
                    blockingQueue.drainTo(list);  // drainTo() 将队列中的值全部从队列中移除，并添加到结果参数（集合）中
                    list.forEach(System.out::println);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```



