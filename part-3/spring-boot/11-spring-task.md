# <font color="orange">Spring Task</font>

在项目开发中，经常需要定时任务来帮助我们来做一些内容，比如定时派息、跑批对账、业务监控等。

实现定时任务有 3 种方式：

- java 自带的 API：**java.util.Timer** 类和 **java.util.TimerTask** 类；

- Quartz 框架：开源 功能强大 使用起来稍显复杂；

- Spring 3.0 以后自带了 task 调度工具，也称 Spring Task，它比 Quartz 更加的简单方便。

## 1. pom 包配置

pom 包里面只需要引入 SpringBoot Starter 包即可，SpringBoot Starter 包中已经内置了定时的方法。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>
```

## 2. 启动类开启定时

在启动类上面加上 **@EnableScheduling** 即可开启定时:

```java
@Spring BootApplication
@EnableScheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class， args);
    }
}
```

## 3. 创建定时任务实现类

使用 SpringBoot 自带的定时非常的简单，只需要在方法上面添加 **@Scheduled** 注解即可。

### 3.1 定时任务 1:

```java
@Component
public class SchedulerTask {
    private static final Logger log = LoggerFactory.getLogger(SchedulerTask.class);
    private int count = 0;

    @Scheduled(cron="*/6 * * * * ?")
    private void process() {
        log.info("{}", "this is scheduler task running " + (count++));
    }
}
```

设置 *`process()`* 每隔六秒执行一次，并统计执行的次数。

我们还有另外的一种方案来设置，固定时间周期执行方法。

### 3.2 定时任务 2:

```java
@Component
public class Scheduler2Task {
    private static final Logger log = LoggerFactory.getLogger(Scheduler2Task.class);
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 6000)
    public void reportCurrentTime() {
        log.info("{}", "现在时间:" + dateFormat.format(new Date()));
    }
}
```

启动项目之后，就会在控制台看到打印的结果。

结果如下:

```
INFO | [   scheduling-1] com.example.timingtask.Scheduler2Task    : 现在时间:20:12:02
INFO | [   scheduling-1] com.example.timingtask.SchedulerTask     : this is scheduler task running 0
INFO | [   scheduling-1] com.example.timingtask.Scheduler2Task    : 现在时间:20:12:08
INFO | [   scheduling-1] com.example.timingtask.SchedulerTask     : this is scheduler task running 1
INFO | [   scheduling-1] com.example.timingtask.Scheduler2Task    : 现在时间:20:12:14
INFO | [   scheduling-1] com.example.timingtask.SchedulerTask     : this is scheduler task running 2
```

说明两个方法都按照固定 6 秒的频率来执行。

## 4. 参数说明

**@Scheduled** 参数可以接受两种定时的设置，一种是我们常用的 `cron="*/6 * * * * ?"`，一种是 `fixedRate = 6000`，两种都可表示固定周期执行定时任务。

### 4.1 fixedRate 说明

- **@Scheduled(fixedRate = 6000)** : 上一次开始执行时间点之后 6 秒再执行。
- **@Scheduled(fixedDelay = 6000)** : 上一次执行完毕时间点之后 6 秒再执行。
- **@Scheduled(initialDelay=1000, fixedRate=6000)** : 第一次延迟 1 秒后执行，之后按 fixedRate 的规则每 6 秒执行一次。

### 4.2 cron 说明

cron 一共有七位，最后一位是年，SpringBoot 定时方案中只需要设置六位即可:

- 第一位，表示秒，取值 0 ~ 59;
- 第二位，表示分，取值 0 ~ 59;
- 第三位，表示小时，取值 0 ~ 23;
- 第四位，日期天/日，取值 1 ~ 31;
- 第五位，日期月份，取值 1 ~ 12;
- 第六位，星期，取值 1 ~ 7，星期一，星期二，...。注，不是第 1 周、第 2 周的意思，另外，1 表示星期天，2 表示星期一;
- 第七位，年份，可以留空，取值 1970 ~ 2099。

cron 中，还有一些特殊的符号，含义如下:

- `(*)` 星号 
 
  可以理解为每的意思，每秒、每分、每天、每月、每年 ... 。

- `(?)` 问号 
 
  问号只能出现在日期和星期这两个位置，表示这个位置的值不确定，每天 3 点执行，因此第六位星期的位置，是不需要关注的，就是不确定的值;同时，日期和星期是两个相互排斥的元素，通过问号来表明不指定值，比如 1 月 10 日是星期一，如果在星期的位置另指定星期二，就前后冲突矛矛盾了。

- `(-)` 减号 
  
  表达一个范围，如在小时字段中使用“10 - 12”，则表示从 10 到 12 点，即 10、11、12。

- `(,)` 逗号 
  
  表达一个列表值，如在星期字段中使用“1,2,4”，则表示星期一、星期二、星期四。

- `(/)` 斜杠 

  如 `x/y`，x 是开始值，y 是步长，比如在第一位(秒)，`0/15` 就是从 0 秒开始，每隔 15 秒执行一次，最后就是 0、15、30、45、60，另 `*/y`，等同于 `0/y` 。

下面列举几个常用的例子子。

| 实例 | 说明 |
| :- | :- |
| `0 0 3 * * ?` | 每天 3 点执行 |
| `0 5 3 * * ?` | 每天 3 点 5 分执行 |
| `0 5 3 ? * *` | 每天 3 点 5 分执行，与上面作用相同 |
| `0 5/10 3 * * ?` |  每天 3 点的 5 分、15 分、25 分、35 分、45 分、55 分这几个时间点执行 |
| `0 10 3 ? * 1` |  每周星期天，3 点 10 分执行，注，1 表示星期天 |
| `0 10 3 ? * 1#3` | 每个月的第三个星期，星期天执行，# 号只能出现在星期的位置 |


## 5. 并行任务（了解、自学）

之前的的定时任务都是串行执行的。所谓串行执行指的是只由一个线程来执行任务。除了这种方式 Spring Task 还支持并行执行任务，即由多个线程来执行不同的任务。

要实现这样的功能，你需要去进行额外的配置：

```java

@Configuration
@EnableScheduling
public class TimingTaskConfig implements SchedulingConfigurer, AsyncConfigurer {

    // 线程池线程数量
    private static final int corePoolSize = 5;

    @Bean
    public ThreadPoolTaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.initialize(); // 初始化线程池
        scheduler.setPoolSize(corePoolSize); // 线程池容量
        return scheduler;
    }

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setTaskScheduler(taskScheduler());
    }

    @Override
    public Executor getAsyncExecutor() {
        return taskScheduler();
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}
```

注意，这时 **`@EnableScheduling`** 注解标注在了这个配置类上，因此，Spring Boot 的入口类上就不再需要标注它了。（当然，你硬要把它标注在入口类上其实也可以）。

其它的相关代码无需修改，运行项目你会看到类似如下的日志输出：

```
INFO | [taskScheduler-1] com.example.timingtask.Scheduler2Task    : 现在时间:20:08:57
INFO | [taskScheduler-1] com.example.timingtask.SchedulerTask     : this is scheduler task running 0
INFO | [taskScheduler-2] com.example.timingtask.Scheduler2Task    : 现在时间:20:09:03
INFO | [taskScheduler-3] com.example.timingtask.SchedulerTask     : this is scheduler task running 1
INFO | [taskScheduler-1] com.example.timingtask.Scheduler2Task    : 现在时间:20:09:09
INFO | [taskScheduler-4] com.example.timingtask.SchedulerTask     : this is scheduler task running 2
```