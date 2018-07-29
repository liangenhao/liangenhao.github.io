---
title: 05 Java8新特性之时间日期API
date: 2018-05-13
categories: Java新特性
tags: [Java8新特性, 时间日期API]
---

Java8包含了全新的时间日期API，这些功能都放在了`java.time`包下。这套全新的时间日期API是**不可变且线程安全的**（This class is immutable and thread-safe）。

<!-- more -->

## 1 本地时间：LocalDate、LocalTime、LocalDateTime

`localDate`、`LocalTime`、`LocalDateTime`类的实例是**不可变的对象**，分别表示使用ISO-8601日历系统的日期、时间、日期和时间。它们提供了简单的日期或者时间，并不包含当前的时间信息。也不包含与时区相关的信息。

> `localDate`、`LocalTime`、`LocalDateTime`三个的使用的方式一模一样。

以`localDateTime`为例：

```java
// 获取当前日期时间
LocalDateTime nowDateTime = LocalDateTime.now();
System.out.println(nowDateTime);
// 获取指定的日期时间
LocalDateTime assignDateTime = LocalDateTime.of(2017, 11, 22, 15, 23, 55);
System.out.println(assignDateTime);
// 当前时间加两年
LocalDateTime nowDateTimePlus2Years = nowDateTime.plusYears(2);
System.out.println(nowDateTimePlus2Years);
// 当前时间减两年
LocalDateTime nowDateTimeMinus2Years = nowDateTime.minusYears(2);
System.out.println(nowDateTimeMinus2Years);
```

> 输出结果：
>
> 2018-05-13T16:56:04.346
> 2017-11-22T15:23:55
> 2020-05-13T16:56:04.346
> 2016-05-13T16:56:04.346

## 2 时间戳：Instant

`Instant`：时间戳，以1970年1月1日00:00:00 开始，到某个时间的毫秒值。

```java
// 默认获取UTC时区（UTC：世界统一时间）
Instant instant1 = Instant.now();
System.out.println(instant1);
// 获取时间戳
long milli = instant1.toEpochMilli();
System.out.println(milli);
// 带偏移量运算：偏移8小时：北京时间
OffsetDateTime offsetDateTime = instant1.atOffset(ZoneOffset.ofHours(8));
System.out.println(offsetDateTime);

// 时间戳加60秒
Instant instant2 = Instant.ofEpochSecond(60);
System.out.println(instant2);
```

> 输出结果：
>
> 2018-05-13T11:54:33.717Z
> 1526212473717
> 2018-05-13T19:54:33.717+08:00
> 1970-01-01T00:01:00Z

## 3 计算间隔：Duration、Period

`Duration`：计算两个“时间”之间的间隔。

`Period`：计算两个“日期”之间的间隔。

计算两个“时间”之间的间隔。

```java
Instant instant1 = Instant.now();
try {
  Thread.sleep(1000);
} catch (InterruptedException e) {
  e.printStackTrace();
}
Instant instant2 = Instant.now();

Duration duration = Duration.between(instant1, instant2);
System.out.println(duration.toMillis());

System.out.println("==================");

LocalTime time1 = LocalTime.now();
try {
  Thread.sleep(1000);
} catch (InterruptedException e) {
  e.printStackTrace();
}
LocalTime time2 = LocalTime.now();
System.out.println(Duration.between(time1, time2).toMillis());
```

计算两个“日期”之间的间隔。

```java
LocalDate date1 = LocalDate.of(2017, 1, 1);
LocalDate date2 = LocalDate.now();
Period period = Period.between(date1, date2);
System.out.println(period.getYears());
System.out.println(period.getMonths());
System.out.println(period.getDays());
```

> 1
> 4
> 12
>
> 相差1年4个月12天。

## 5 时间校正器：TemporalAdjuster

`TemporalAdjuster`：时间校正器，有时我们可能会需要获取例如：将日期调整到“下个周如”等操作。

`TemporalAdjusters`：该类通过静态方法提供了大量的常用`TemporalAdjuster`的实现。

例如：

```java
LocalDateTime dateTime = LocalDateTime.now();
// 指定日期
LocalDateTime dateTimeWithDayOfMonth = dateTime.withDayOfMonth(10);
// 使用时间校正器指定时间：下一个周末
LocalDateTime dateTimeNextSunday = dateTime.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));
System.out.println(dateTimeNextSunday);

// 自定义时间校正器：获取下一个工作日
LocalDateTime dateTime1 = dateTime.with((temporal) -> {
  LocalDateTime time = (LocalDateTime)temporal;
  DayOfWeek dayOfWeek = time.getDayOfWeek();
  if (dayOfWeek.equals(DayOfWeek.FRIDAY)) {
    return time.plusDays(3);
  } else if (dayOfWeek.equals(DayOfWeek.SATURDAY)) {
    return time.plusDays(2);
  } else {
    return time.plusDays(1);
  }
});
System.out.println(dateTime1);
```

## 6 时间日期格式化：dateTimeFormatter

```java
// 使用ISO标准日期格式
DateTimeFormatter formatter = DateTimeFormatter.ISO_DATE_TIME;
LocalDateTime dateTime = LocalDateTime.now();
// LocalDateTime的格式化方法
String dateStr = dateTime.format(formatter);
System.out.println(dateStr);


// 使用自定义日期格式
DateTimeFormatter formatter1 = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");
// DateTimeFormatter的格式化方法
String dateStr2 = formatter1.format(dateTime);
System.out.println(dateStr2);
```

## 7 时区的处理：ZonedDate、ZonedTime、ZonedDateTime

> 以`LocalDateTime`为例。

使用`LocalDateTime`的`static LocalDateTime now(ZoneId zone)`方法来指定时区：

```java
LocalDateTime dateTime = LocalDateTime.now(ZoneId.of("America/Los_Angeles"));
```

> 结果：2018-05-13T06:24:47.349。显示的是美国洛杉矶时区的时间。



使用`LocalDateTime`的`ZonedDateTime atZone(ZoneId zone)`方法转换成`ZonedDateTime`（带时区的时间日期对象）：

```java
LocalDateTime dateTime2 = LocalDateTime.now();
ZonedDateTime zonedDateTime = dateTime2.atZone(ZoneId.of("America/Los_Angeles"));
System.out.println(zonedDateTime);
```

> 结果：2018-05-13T06:24:47.446-07:00[America/Los_Angeles]

> `ZonedDateTime`是带时区的时间日期。

