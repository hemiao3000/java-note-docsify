# <font color="orange">使用 LocalDateTime 替代 Date</font>

JDK 中的 Date 的缺点：

- Date 如果不格式化，打印出的日期可读性差；

- 使用 SimpleDateFormat 可以对日期时间进行格式化，但是 SimpleDateFormat 并非线性安全；

- Date 对时间处理比较麻烦；

- Date 这个类名的命名并不严谨。

> 《阿里巴巴开发手册》中明确禁用 **static** 关键字修饰 **SimpleDateFormat** 。

Java 官方请著名的第三方日期时间包 *joda-time* 的作者重新设计了与日期时间有关的 API 部分，并把它们放在了 jdk 8 的 **java.time** 包下。

新增了：**LocalDate**、**LocalTime**、**LocalDateTime** 三个类及相关新的 API 用以替代 `Date` 及其旧的使用方式。

## 1. LocalDate

只会获取 `年月日` 信息。

**LocalDate** 中并不包含『**时区**』信息。即，一个单独的 LocalDate 对象所表达的含义是不严谨的。<small>LocalTime 和 LocalDateTime 也一样。</small>

- 创建 **LocalDate** 对象

  ```java
  LocalDate localDate = LocalDate.now();
  LocalDate localDate = LocalDate.of(2000, 1, 1);
  ```

- 获取年、月、日、星期几

  ```java
  LocalDate localDate = LocalDate.now();

  int year = localDate.getYear();
  int year = localDate.get(ChronoField.YEAR);

  Month month = localDate.getMonth();
  int month = localDate.get(ChronoField.MONTH_OF_YEAR);

  int day = localDate.getDayOfMonth();
  int day = localDate.get(ChronoField.DAY_OF_MONTH);

  DayOfWeek dayOfWeak = localDate.getDayOfWeek();
  int dayOfWeak = localDate.get(ChronoField.DAY_OF_WEEK);
  ```

## 2. LocalTime

只会获取 `时分秒` 信息。

- 创建 **LocalTime** 对象

  ```java
  LocalTime localTime = LocalTime.now();
  LocalTime localTime = LocalTime.of(17, 30, 0);
  ```

- 获取时分秒

  ```java
  int hour = localTime.getHour();
  int hour = localTime.get(ChronoField.HOUR_OF_DAY);
 
  int minute = localTime.getMinute();
  int minute = localTime.get(ChronoField.MINUTE_OF_HOUR);
 
  int second = localTime.getSecond();
  int second = localTime.get(ChronoField.SECOND_OF_MINUTE);
  ```

## 3. LocaDateTime

获取 `年月日时分秒`，等于 **LocalDate** + **LocalTime** 。

- 创建 `LocalDateTime`

  ```java
  LocalDateTime localDateTime = LocalDateTime.now();
  LocalDateTime localDateTime = LocalDateTime.of(2019, Month.SEPTEMBER, 10, 14, 46, 56);
  LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
  LocalDateTime localDateTime = localDate.atTime(localTime);
  LocalDateTime localDateTime = localTime.atDate(localDate);
  ```

- 获取 `LocalDate`

  ```java
  LocalDate localDate = localDateTime.toLocalDate();
  ```
  
- 获取 `LocalTime`

  ```java
  LocalTime localTime = localDateTime.toLocalTime();
  ```

## 4. Instant

在 Java 中，一个 Instant 对象标识时间轴上的一个点，代表着一个概念上的瞬间。

『**原点**』是众所周知的 1970 年 1 月 1 日的午夜，此时本初子午线正在穿过伦敦格林威治皇家天文台。从『**原点**』开始，时间按照每天 86400 秒进行正向、反向计算，向前、向后分别以纳秒为单位。

> 注意，Instant 是有时区概念的，在你未指定时，它的默认时区时 0 时区，即，格林威治时间。

虽然 **Instant** 内部是以纳秒为单位进行存储、运算的，但是很显然你可以将它转换成相较于『**原点**』的时、分、秒。

在编程世界中，我们日常生活中所说的『**秒**』使用的是 **second** ；而『**相较于原点的秒**』使用的是 **epoch second** 。<small>**epoch** 是时代、纪元等含义。</small>

另外，在 Java 8 中，使用 **Duration** 对象来代表两个 **Instant** 之间的时间跨度。

- 创建 `Instant` 对象

  ```java
  Instant instant = Instant.now();
  ```

- 获取秒数

   ```java
   long currentSecond = instant.getEpochSecond();
   ```

- 获取毫秒数

  ```java
  long currentMilli = instant.toEpochMilli();
  ```

> 个人觉得如果只是为了获取秒数或者毫秒数，使用 `System.currentTimeMillis()` 来得更为方便。


## 5. 修改 LocalDate、LocalTime、LocalDateTime、Instant

LocalDate、LocalTime、LocalDateTime、Instant 为『**不可变对象**』，修改这些对象对象会返回一个副本。

- 增加、减少年数、月数、天数等 以 LocalDateTime 为例

  ```java
  LocalDateTime localDateTime = LocalDateTime.of(2019, Month.SEPTEMBER, 10, 14, 46, 56);

  // 增加一年
  localDateTime = localDateTime.plusYears(1);
  localDateTime = localDateTime.plus(1, ChronoUnit.YEARS);

  // 减少一个月
  localDateTime = localDateTime.minusMonths(1);
  localDateTime = localDateTime.minus(1, ChronoUnit.MONTHS);
  ```

- 通过 `with` 修改某些值

  ```java
  // 修改年为 2019
  localDateTime = localDateTime.withYear(2020);
  // 修改为 2022
  localDateTime = localDateTime.with(ChronoField.YEAR, 2022);

  // 还可以修改月、日
  ```

## 6. 使用 TemporalAdjuster 和 TemporalAdjusters

有的时候，你需要进行一些更加复杂的日期操作，比如，将日期调整到下个周日、下个工作日，或者是本月的最后一天。这时，你可以使用重载版本的 `withXXX` 方法，向其传递一个提供了更多定制化选择的 `TemporalAdjuster` 对象， 更加灵活地处理日期。

比如有些时候想知道这个月的最后一天是几号、下个周末是几号，通过提供的时间和日期API可以很快得到答案。

对于最常见的用例，日期和时间 API 已经提供了大量预定义的 `TemporalAdjuster` 对象。你可以通过 `TemporalAdjusters`<small>（注意，此处有 `s`）</small>类的静态工厂方法访问它们。

- TemporalAdjusters 类中的常用工厂方法：

  - **dayOfWeekInMonth** 创建一个新的日期，它的值为同一个月中每一周的第几天

  - **firstDayOfMonth** 创建一个新的日期，它的值为当月的第一天

  - **firstDayOfNextMonth** 创建一个新的日期，它的值为下月的第一天

  - **firstDayOfNextYear** 创建一个新的日期，它的值为明年的第一天

  - **firstDayOfYear** 创建一个新的日期，它的值为当年的第一天

  - **firstInMonth** 创建一个新的日期，它的值为同一个月中，第一个符合星期几要求的值

  - **lastDayOfMonth** 创建一个新的日期，它的值为当月的最后一天

  - **lastDayOfNextMonth** 创建一个新的日期，它的值为下月的最后一天

  - **lastDayOfNextYear** 创建一个新的日期，它的值为明年的最后一天

  - **lastDayOfYear** 创建一个新的日期，它的值为今年的最后一天

  - **lastInMonth** 创建一个新的日期，它的值为同一个月中，最后一个符合星期几要求的值

  - **next** / **previous** 创建一个新的日期，并将其值设定为日期调整后或者调整前，第一个符合指定星 期几要求的日期

  - **nextOrSame** / **previousOrSame** 创建一个新的日期，并将其值设定为日期调整后或者调整前，第一个符合指定星 期几要求的日期，如果该日期已经符合要求，直接返回该对象


## 7. 格式化时间

`DateTimeFormatter` 默认提供了多种格式化方式，如果默认提供的不能满足要求，可以通过 `DateTimeFormatter` 的 `ofPattern` 方法创建自定义格式化方式。

```java
LocalDate localDate = LocalDate.of(2019, 9, 10);
String s1 = localDate.format(DateTimeFormatter.BASIC_ISO_DATE);
String s2 = localDate.format(DateTimeFormatter.ISO_LOCAL_DATE);

// 自定义格式化
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
String s3 = localDate.format(dateTimeFormatter);
```

## 8. 解析时间

相较于传统的 SimpleDateFormat，**DateTimeFormatter** 是线程安全的

```java
LocalDate localDate1 = LocalDate.parse("20190910", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate localDate2 = LocalDate.parse("2019-09-10", DateTimeFormatter.ISO_LOCAL_DATE);
```

### Date 与 LocalDate 互转 {docsify-ignore}

- Date 转 LocalDate

  ```java
  // Date 转 LocalDate
  public static LocalDate date2LocalDate(Date date) {
    return (date == null) ? (null) : (date.toInstant().atZone(ZoneId.systemDefault()).toLocalDate());
  }
  ```

- LocalDate 转 Date

  ```java
  //  LocalDate转Date
  public static Date localDate2Date(LocalDate localDate) {
    return (localDate == null) ? (null) : (Date.from(localDate.atStartOfDay(ZoneId.systemDefault()).toInstant()));
  }
  ```

## 9. 带时区的时间

**LocalDate**、**LocalTime** 和 **LocalDateTime** 是不带时区信息的。在 JDK 8 中，带有时区信息的日期时间类是 **ZonedDateTime** 。 

因特网编号管理局<small>（IANA）</small>维护着一份全球所有已知时区的数据库（[http://www.iana.org/time-zones](http://www.iana.org/time-zones)），每年会更新几次（这些更新主要处理夏令时规则的改变）。Java 就是使用了 IANA 的数据库。

在 Java 中，每个时区都有一个 ID，例如 `Asia/Shanghai` 。想要获得所有可用的时区，你可以调用 `ZoneId.getAvailableZoneIds()` 。<small>大概有 600 个。</small>

如果你想获得 ZonedDateTime 对象，你有 2 种途径：

- 直接使用 **ZonedDateTime.of()** 方法创建；

- 先创建一个 **LocalDateTime** 对象<small>（它不含时区概念）</small>，再调用它的 **atZone** 方法，赋予它时区的概念，生成 **ZonedDateTime** 对象。


## 10. SpringBoot 中应用 LocalDateTime

### 将 LocalDateTime 字段以时间戳的方式返回给前端。 {docsify-ignore}

添加日期转化类：

```java
public class LocalDateTimeConverter extends JsonSerializer<LocalDateTime> {

    @Override
    public void serialize(LocalDateTime value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeNumber(value.toInstant(ZoneOffset.of("+8")).toEpochMilli());
    }
}
```

并在 `LocalDateTime` 字段上添加 `@JsonSerialize(using = LocalDateTimeConverter.class)` 注解，如下：

```java
@JsonSerialize(using = LocalDateTimeConverter.class)
protected LocalDateTime gmtModified;
```

### 将 `LocalDateTime` 字段以指定格式化日期的方式返回给前端  {docsify-ignore}

在 LocalDateTime 字段上添加 `@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")` 注解即可，如下：

```java
@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")
protected LocalDateTime gmtModified;
```

### 将 LocalDateTime 字段以指定格式化日期的方式返回给前端  {docsify-ignore}

在 LocalDateTime 字段上添加 `@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")` 注解即可，如下：

```java
@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")
protected LocalDateTime gmtModified;
```

### 对前端传入的日期进行格式化  {docsify-ignore}

在 LocalDateTime 字段上添加 `@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")` 注解即可，如下：

```java
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
protected LocalDateTime gmtModified;
```
