---
title: 'Java 中常用的日期类'
author: ehzyil
tags:
  - Java
categories:
  - 技术
date: 2023-10-09
---
# Java 中常用的日期类

## 一、 java.util.Date

**`System.currentTimeMillis()`**：返回当前时间与格林威治时间 1970-01-01 00:00:00 （北京时间：1970-01-01 08:00:00）之间以毫秒为单位的时间差。此方法适用于计算时间差

```java
        // 1687936118101  时间戳
        Long time = System.currentTimeMillis();
```



`System.currentTimeMillis()` 方法的返回类型是 Long 类型。一般用于获取某个方法或其它的执行时间差。即：在开始前获取一次，在结束时获取一次，结束时间减去开始时间，得到执行时间。如：

```java
 Long startTime = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            System.out.print("*");
        }
        Long endTime = System.currentTimeMillis();

        Long result = endTime - startTime;

        System.out.println("用时："+result);
```

时间戳与 `java.util.Date` 类互相转换：

```java
// 时间戳转换为 Date 类型
        Date date = new Date(time);
        System.out.println(date);

        // Date 类型转换为时间戳
        Long time = new Date().getTime();
        System.out.println(time);
```

一般 java.util.Date 类与 java.text.SimpleDateFormat 类一起用，用于格式化日期。

如：将日期与字符串互转：

```java
 // 格式化时间
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH时mm分ss秒");
        System.out.println(sdf.format(new Date()));


        // 将字符串转换为日期：
        String pattern= "yyyy-MM-dd HH:mm:ss";
        String time = "2022-07-18 22:53:22";
        SimpleDateFormat simpleDateFormat=new SimpleDateFormat(pattern);
        try {
            Date date=simpleDateFormat.parse(time);
            System.out.println(date);
        } catch (Exception e) {
            throw new RuntimeException("日期格式化错误！！，日期为：" + time);
        }
```

注意：将字符串转换为日期时 pattern要对应。

## 二. java.sql.Date

java.sql.Date 是 java.util.Date 类的子类；是针对 SQL 语句使用的，它只包含日期而没有时间部分，一般在读写数据库的时候用，PreparedStament#setDate() 方法的参数和 ResultSet#getDate() 方法的都是 java.sql.Date。

```java
  Date date=new Date(0L);
        System.out.println(date);
        //1970-01-01
```

java.util.Date 与 java.sql.Date 互相转换：

```java
	        java.util.Date utilDate=new java.util.Date();
        System.out.println(utilDate);
        //Wed Jun 28 15:45:45 CST 2023

        java.sql.Date sqldate=new Date(utilDate.getTime());
        System.out.println(sqldate);
        //2023-06-28
```

## 三. java.util.Calendar

java.util.Calendar：日历类，是个抽象类

Calendar 实例化的方法：

1. 创建其子类 GregorianCalendar 的对象

2. 调用其静态方法 getInstance()

   从下列方法可以得出两种实例化方法一样

```
        Calendar calendar=Calendar.getInstance();
        System.out.println(calendar.getClass());
        //class java.util.GregorianCalendar
        Calendar GregorianCalendar=new GregorianCalendar();
        System.out.println(calendar.getClass()==GregorianCalendar.getClass());
        //true
```

常用的方法：

get(int field)：获取某个时间
set()：设置某个时间
add()：在某个时间上，加上某个时间
getTime()：将 Calendar 类转换为 Date 类
setTime()：将 Date 类转换为 Calendar 类
field 取值有：

DAY_OF_MONTH：这个月的第几天
DAY_OF_YEAR：这年的第几天
…
注意：

获取月份时：一月是 0，二月是1，…
获取星期时：周日是1，周一是2，…

`getTime()` 方法

源码

```java
public final Date getTime() {
    return new Date(getTimeInMillis());
}
```

```java
        Calendar calendar = Calendar.getInstance();
        System.out.println(calendar.getTime());
        //Wed Jun 28 16:04:30 CST 2023
```



`public int get(int field)`方法

```java
        Calendar calendar=Calendar.getInstance();
        System.out.println(calendar.getTime());
        //Wed Jun 28 15:59:38 CST 2023
        calendar.set(Calendar.DAY_OF_MONTH, 5);
        //将当前时间设置成某个时间
        System.out.println(calendar.get(Calendar.DAY_OF_MONTH));
        //1
        System.out.println(calendar.getTime());
        //Thu Jun 01 15:59:38 CST 2023
```

 `set(int field, int value)`方法

```java
       Calendar calendar=Calendar.getInstance();
        System.out.println(calendar.getTime());
        calendar.set(Calendar.DAY_OF_MONTH, 1);
        //将当前时间设置成某个时间
        System.out.println(calendar.get(Calendar.DAY_OF_MONTH));
        System.out.println(calendar.getTime());
```

`add(int field, int amount)`方法

```java
        Calendar calendar = Calendar.getInstance();
        System.out.println(calendar.getTime());
        //Wed Jun 28 16:04:30 CST 2023

        // 给当前时间加上 5 天
        calendar.add(Calendar.DAY_OF_MONTH, +5);
        System.out.println(calendar.getTime());
        //Mon Jul 03 16:04:30 CST 2023
        
        // 给当前时间加上 2 月
        calendar.add(Calendar.MONTH, 2);
        System.out.println(calendar.getTime());
        //Sun Sep 03 16:04:30 CST 2023
        
        // 给当前时间减去 3 天
        calendar.add(Calendar.DAY_OF_MONTH, -3);
        System.out.println(calendar.getTime());
        //Thu Aug 31 16:04:30 CST 2023
```
`setTime(Date date)` 方法

源码

```java
public final void setTime(Date date) {
    setTimeInMillis(date.getTime());
}
```

## 四、`java.time API`

### `java.time API` 背景

`java.util.Date、java.util.Calendar` 面临的问题：

-  1.可变性：像日期、时间这样的类应该是不可变的 
- 2.偏移性：Date 中的年份是从 1900 年开始的；月份是从 0 开始的
-  3.格式化：格式化只对 Date 有用，而对 Calendar 则不行 4.线程安全：它们都不是线程安全



java.time 中包含了：

- LocalDate：本地日期。yyyy-MM-dd 格式的日期。可以存储生日、纪念日等
- LocalTime：本地时间。代表的是时间，不是日期
- LocalDateTime：本地日期时间
- ZonedDateTime：时区
- Duration：持续时间
- 大大地简化了对日期、时间的操作

### java.time API 中常用的 API

- now()：获取当前时间、日期
- of()：获取指定的日期、时间
- getXxx()：获取值
- withXxx()：设置值
- plusXxx()：加
- minusXxx()：减

now() 获取当前日期

```java
LocalDate localDate = LocalDate.now();
        LocalTime localTime = LocalTime.now();
        LocalDateTime localDateTime = LocalDateTime.now();

        System.out.println(localDate);//2023-06-28
        System.out.println(localTime);//16:17:21.720
        System.out.println(localDateTime);//2023-06-28T16:17:21.720
```
`localDate`：表示当前的日期
`localTime`：表示当前的时间
`localDateTime`：表示当前的日期时间

of() 指定一个特定的日期



```java
 //给出时间参数，表示出日期和时间 再生成localDateTime对象返回
public static LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute) {
        LocalDate date = LocalDate.of(year, month, dayOfMonth);
        LocalTime time = LocalTime.of(hour, minute);
        return new LocalDateTime(date, time);
    }
        LocalDateTime localDateTime = LocalDateTime.of(2035,10,1,10,00);
        System.out.println(localDateTime);
        //2035-10-01T10:00

```
getXxx()

```java

```

withXxx()

每次修改，返回的是一个新对象，部分源码如下。

```java
    private LocalDateTime with(LocalDate newDate, LocalTime newTime) {
     ...
        return new LocalDateTime(newDate, newTime);
    }
```

```java
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime);
        //2023-06-28T16:35:38.764
        
        LocalDateTime dayOfMonth = localDateTime.withDayOfMonth(25);
        System.out.println(dayOfMonth);
        //2023-06-25T16:35:38.764
```
plusXxx()

该方法每次调用返回时调用withXxx()方法，因此每次修改，返回的也是一个新对象。

```java
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime);
        //2023-06-28T16:38:55.066
        
        LocalDateTime localDateTime1 = localDateTime.plusDays(2);
        System.out.println(localDateTime1);
        //2023-06-30T16:38:55.066
        
        LocalDateTime localDateTime2 = localDateTime.plusMonths(-1);
        System.out.println(localDateTime2);
        //2023-05-28T16:38:55.066
```
minusXxx()

调用plusXxx()把传入的值取反。

```java
    public LocalDateTime minusDays(long days) {
        return (days == Long.MIN_VALUE ? plusDays(Long.MAX_VALUE).plusDays(1) : plusDays(-days));
    }
```

## 五、java.time.format.DateTimeFormatter



java.time.format.DateTimeFormatter：格式化、解析日期或时间（类似于：SimpleDateFormat）

此类提供了三种格式化方法：

- 预定义的标准格式：ISO_LOCAL_DATE_TIME…
- 本地化相关的格式：
- 自定义格式：ofPattern(“yyyy-MM-dd HH:mm:ss”)

```java
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime);
        //2023-06-28T16:50:06.649
        
        DateTimeFormatter dtf=DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
        System.out.println(dtf.format(localDateTime));
        //2023/06/28 16:50:06
```





## 六、java.time.Instant

Instant：时间线上的一个瞬时点。 在UNIX中，这个数从1970年开始，以秒为的单位；同样的，在Java中，也是从1970年开始，但以毫秒为单位。
java.time包通过值类型Instant提供机器视图，不提供处理人类意义上的时间单位。 Instant表示时间线上的一点，而不需要任何上下文信息，例如，时区。概念上讲， 它只是简单的表示自1970年1月1日0时0分0秒（ UTC）开始的秒数。 因为java.time包是基于纳秒计算的，所以Instant的精度可以达到纳秒级。

| 方法                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| now()                         | 静态方法， 返回默认UTC时区的Instant类的对象                  |
| ofEpochMilli(long epochMilli) | 静态方法，返回在 1970-01-01 00:00:00基础上加上指定毫秒数之后的Instant 类的对象 |
| atOffset(ZoneOffset offset)   | 结合即时的偏移来创建一个 OffsetDateTime                      |
| toEpochMilli()                | 返回1970-01-01 00:00:00到当前时间的毫秒数， 即为时间戳       |



```java
        //now():获取本初子午线对应的标准时间，所以和我们当前的时间约有8个小时的时差。
        Instant now = Instant.now();
        System.out.println(now);//2023-06-28T09:52:51.124Z

        //添加时间的偏移量
        OffsetDateTime offsetDateTime = now.atOffset(ZoneOffset.ofHours(8));
        System.out.println(offsetDateTime);//2023-06-28T17:52:51.124+08:00

        //toEpochMilli():获取自1970年1月1日0时0分0秒（UTC）开始的毫秒数  ---> Date类的getTime()
        long milli = now.toEpochMilli();
        System.out.println(milli);//1687945971124
        
        //ofEpochMilli():通过给定的毫秒数，获取Instant实例  -->Date(long millis)
        Instant instant = Instant.ofEpochMilli(System.currentTimeMillis());
        System.out.println(instant);//2023-06-28T09:52:51.131Z
```



## 七、java.time.temporal.Temporal

Java的`Temporal`类是Java 8引入的日期和时间API的一部分，它是一个抽象类，用于处理各种日期和时间对象。下面是`Temporal`类的一些常用方法以及对应的例子：

1. `of(TemporalField field, long value)`：使用指定的字段和值创建一个`Temporal`对象。
```java
LocalDate date = LocalDate.of(2023, 6, 28);
```

2. `get(TemporalField field)`：获取指定字段的值。
```java
int year = LocalDate.now().get(ChronoField.YEAR);
```

3. `with(TemporalField field, long newValue)`：将指定字段的值设置为新值，返回一个新的`Temporal`对象。
```java
LocalDate newDate = LocalDate.now().with(ChronoField.MONTH_OF_YEAR, 7);
```

4. `plus(TemporalAmount amountToAdd)`：在当前`Temporal`对象上添加指定的时间量。
```java
LocalDateTime dateTime = LocalDateTime.now().plus(Duration.ofHours(2));
```

5. `minus(TemporalAmount amountToSubtract)`：在当前`Temporal`对象上减去指定的时间量。
```java
LocalTime newTime = LocalTime.now().minus(Duration.ofMinutes(30));
```

6. `isSupported(TemporalUnit unit)`：检查指定的时间单位是否支持。
```java
boolean isSupported = LocalDateTime.now().isSupported(ChronoUnit.DAYS);
```

7. `until(Temporal endExclusive, TemporalUnit unit)`：计算当前`Temporal`对象与指定`Temporal`对象之间的时间间隔。
```java
long daysBetween = LocalDate.of(2023, 12, 31).until(LocalDate.now(), ChronoUnit.DAYS);
```

8. `format(DateTimeFormatter formatter)`：使用指定的日期时间格式化器将`Temporal`对象格式化为字符串。
```java
String formattedDate = LocalDate.now().format(DateTimeFormatter.ofPattern("dd-MM-yyyy"));
```

这些只是`Temporal`类的一些常用方法和示例。`Temporal`类还有其他方法，可以根据具体的需求进行查阅和使用。



## 案例

### 在 Java 8 中获取年、月、日信息

```java
//在 Java 8 中获取年、月、日信息
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println(localDateTime);
        //2023-06-28T16:56:25.874
        
        int year = localDateTime.getYear();
        System.out.println(year);
        //2023
        
        Month month = localDateTime.getMonth();
        System.out.println(month);
        //JUNE
        
        int day = localDateTime.getDayOfMonth();
        System.out.println(day);
        //28

        //某日期 一年中的第几天
        int dayOfYear = localDateTime.getDayOfYear();
        System.out.println(dayOfYear);
        //179
```


### 在 Java 8 中处理特定日期

```java
        LocalDateTime dateTime = LocalDateTime.of(2023, 6, 28, 16, 0, 0);
        // 2023-06-28T16:00
        System.out.println(dateTime);
```

### 在 Java 8 中判断两个日期是否相等
```java
    	LocalDate localDate=LocalDate.now();
        LocalDate nowDate=LocalDate.of(2023,6,28);
        System.out.println(localDate.equals(nowDate));
        //true
```

### 在 Java 8 中检查像生日这种周期性事件
```java
        LocalDate now = LocalDate.now();
        LocalDate dateOfBirth = LocalDate.of(2001, 2, 17);

        MonthDay birthday = MonthDay.of(dateOfBirth.getMonth(), dateOfBirth.getDayOfMonth());
        MonthDay currentMonthDay = MonthDay.from(now);

        if (currentMonthDay.equals(birthday)) {
            System.out.println("Happy Birthday");
        } else {
            System.out.println("Sorry, today is not your birthday");
        }
        //Sorry, today is not your birthday
```

### 如何计算一周后的日期

`LocalDate` 日期不包含时间信息，它的 `plus()` 方法用来增加天、周、月，`ChronoUnit` 类声明了这些时间单位。由于 `LocalDate` 也是不变类型，返回后一定要用变量赋值。`minus()` 方法用来减去天、周、月等

```java
        //1.8之前的方法
        //一周后的日期
        SimpleDateFormat formatDate = new SimpleDateFormat("yyyy-MM-dd");
        Calendar ca = Calendar.getInstance();
        ca.add(Calendar.DATE, 7);
        Date d = ca.getTime();
        String after0 = formatDate.format(d);
        System.out.println("一周后日期：" + after0);

        //1.8之后的方法
        //一周后的日期
        LocalDate localDate = LocalDate.now();
        //方法1
        LocalDate after = localDate.plus(1, ChronoUnit.WEEKS);
        //方法2
        LocalDate after2 = localDate.plusWeeks(1);
        System.out.println("一周后日期：" + after);

        /**
         * 一周后日期：2023-07-07
         * 一周后日期：2023-07-07
         */

```

### 如何用 Java 判断日期或时间是早于还是晚于另一个日期或时间

```java
        //日期的比较
        LocalDate localDate=LocalDate.now();
        System.out.println(localDate);//2023-06-28

        LocalDate dateTime = LocalDate.of(2023, 6, 29);

        boolean before = dateTime.isBefore(localDate);
        System.out.println(before);
        boolean after = dateTime.isAfter(localDate);
        System.out.println(after);
        
        //时间的比较
        LocalTime localTime=LocalTime.now();
        System.out.println(localTime);//17:13:12.341

        LocalTime time = LocalTime.of(19, 00, 00);
        
        boolean timeAfter = localTime.isAfter(time);
        System.out.println(timeAfter);
        boolean timeBefore = localTime.isBefore(time);
        System.out.println(timeBefore);
```

### 如何表示信用卡到期这类固定日期
```java
 YearMonth currentYearMonth = YearMonth.now();

        System.out.printf("Days in month year %s has %d days %n", currentYearMonth, currentYearMonth.lengthOfMonth());
        //Days in month year 2023-06 has 30 days

        //返回到期日期
        YearMonth creditCardExpiry = YearMonth.of(2023, Month.OCTOBER);
        System.out.printf("Your credit card expires on %s %n", creditCardExpiry);
        //Your credit card expires on 2023-10 
        
        //返回月底的本地日期
        System.out.println(currentYearMonth.atEndOfMonth());
        //2023-06-30
        
        //获取有多少月份
        System.out.println(currentYearMonth.lengthOfMonth());
        //30
```

### 计算两个日期之间的天数和月数



1.8之前

```java
        //算两个日期间隔多少天，计算间隔多少年，多少月方法类似
        String dates1 = "2023-12-23";
        String dates2 = "2023-02-26";
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        Date date1 = format.parse(dates1);
        Date date2 = format.parse(dates2);
        int day = (int) ((date1.getTime() - date2.getTime()) / (1000 * 3600 * 24));
        System.out.println(dates1 + "和" + dates2 + "相差" + day + "天");
        //2023-12-23和2023-02-26相差300天
```

1.8之后

用 `java.time.Period `类来做计算

```java
        LocalDate now = LocalDate.now();
        LocalDate date = LocalDate.of(2025, Month.JUNE, 5);

        //计算两日期间相差多少天
        long day = Math.abs(now.toEpochDay() - date.toEpochDay());
        System.out.println(now + "和" + date + "相差" + day + "天");

        System.out.println("====================");

        Period period = Period.between(now, date);
        //若第一个日期晚于第二个日期 则返回早日期距较晚日期的天数
        //这里period.getDays()得到的天是抛去年月以外的天数，并不是总天数
        //月是抛去年以外的月数
        System.out.printf("离下个时间还有 %s年,%s月,%s天", period.getYears() ,period.getMonths(),period.getDays());
```



### 计算两日期间相差的月份

 ChronoUnit.MONTHS.between(temporal1, temporal2)本质上是调用Temporal的util方法。

```java
        Temporal temporal1 = LocalDate.parse("2023-06-10", DateTimeFormatter.ofPattern("yyyy-MM-dd"));

        Temporal temporal2 = LocalDate.now();

        //方法返回为相差月份
        long monthsDifference = ChronoUnit.MONTHS.between(temporal1, temporal2);
        //月份差转为正数
        monthsDifference = Math.abs(monthsDifference);

        if (monthsDifference == 0) {
            System.out.println("当月为宽带最后一个月");
        } else if (monthsDifference == 1) {
            System.out.println("下月为宽带最后一个月");
        } else {
            System.out.println("宽带还有更多时间");
        }
```

可以计算两日期间相差的天数，但要注意两日期

```java
  	    Temporal temporal1 = LocalDate.parse("2023-06-10", DateTimeFormatter.ofPattern("yyyy-MM-dd"));
        Temporal temporal2 = LocalDate.now();
        long daysDifference = ChronoUnit.DAYS.between(temporal1, temporal2);
        daysDifference = Math.abs(daysDifference);
        System.out.println(daysDifference);
		//18
```

###  获取指定日期

**Java 8 之前:**

```java
public void getDay() {
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        //获取当前月第一天：
        Calendar c = Calendar.getInstance();
        c.set(Calendar.DAY_OF_MONTH, 1);
        String first = format.format(c.getTime());
        System.out.println("first day:" + first);

        //获取当前月最后一天
        Calendar ca = Calendar.getInstance();
        ca.set(Calendar.DAY_OF_MONTH, ca.getActualMaximum(Calendar.DAY_OF_MONTH));
        String last = format.format(ca.getTime());
        System.out.println("last day:" + last);

        //当年最后一天
        Calendar currCal = Calendar.getInstance();
        Calendar calendar = Calendar.getInstance();
        calendar.clear();
        calendar.set(Calendar.YEAR, currCal.get(Calendar.YEAR));
        calendar.roll(Calendar.DAY_OF_YEAR, -1);
        Date time = calendar.getTime();
        System.out.println("last day:" + format.format(time));
}
```

**Java 8 之后:**

```java
        LocalDate today = LocalDate.now();
        //获取当前月第一天：
        LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
        // 取本月最后一天
        LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());

        //下一个月的第一天
        LocalDate firstDayOfNextMonth = today.with(TemporalAdjusters.firstDayOfNextMonth());

        //取下一天：
        LocalDate nextDay = lastDayOfThisMonth.plusDays(1);
        LocalDate nextDay1 = lastDayOfThisMonth.plus(1,ChronoUnit.DAYS);
        
        //当年的第一天
        LocalDate firstDayOfYear = today.with(TemporalAdjusters.firstDayOfYear());
        
        //当年最后一天
        LocalDate lastday = today.with(TemporalAdjusters.lastDayOfYear());

        //下一年的第一天
        LocalDate firstDayOfNextYear = today.with(TemporalAdjusters.firstDayOfNextYear());

        //2021年最后一个周日，如果用Calendar是不得烦死。
        LocalDate lastMondayOf2021 = LocalDate.parse("2024-6-30").with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
    
```

### 获取指定日期的0点以及24点



```java
	// 返回时间格式如：2020-02-17 00:00:00
	public static String getStartOfDay(Date time) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(time);
        calendar.set(Calendar.HOUR_OF_DAY, 0);
        calendar.set(Calendar.MINUTE, 0);
        calendar.set(Calendar.SECOND, 0);
        calendar.set(Calendar.MILLISECOND, 0);
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(calendar.getTime());
    }
	// 返回时间格式如：2020-02-19 23:59:59
    public static String getEndOfDay(Date time) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(time);
        calendar.set(Calendar.HOUR_OF_DAY, 23);
        calendar.set(Calendar.MINUTE, 59);
        calendar.set(Calendar.SECOND, 59);
        calendar.set(Calendar.MILLISECOND, 999);
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(calendar.getTime());
    }
    // 获取30天以前的时间，同样是比较常用的
    public static String getThirtyDaysAgo(Date time) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(time);
        calendar.add(calendar.DATE, -30);
        calendar.set(Calendar.HOUR_OF_DAY, 0);
        calendar.set(Calendar.MINUTE, 0);
        calendar.set(Calendar.SECOND, 0);
        calendar.set(Calendar.MILLISECOND, 0);
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(calendar.getTime());
    }

```



```java
	// 获得某天最大时间 2020-02-19 23:59:59
    public static Date getEndOfDay(Date date) {
        LocalDateTime localDateTime = LocalDateTime.ofInstant(Instant.ofEpochMilli(date.getTime()), ZoneId.systemDefault());;
        LocalDateTime endOfDay = localDateTime.with(LocalTime.MAX);
        return Date.from(endOfDay.atZone(ZoneId.systemDefault()).toInstant());
    }

    // 获得某天最小时间 2020-02-17 00:00:00
    public static Date getStartOfDay(Date date) {
        LocalDateTime localDateTime = LocalDateTime.ofInstant(Instant.ofEpochMilli(date.getTime()), ZoneId.systemDefault());
        LocalDateTime startOfDay = localDateTime.with(LocalTime.MIN);
        return Date.from(startOfDay.atZone(ZoneId.systemDefault()).toInstant());
    }

```



```
Date now = new Date(); //获取当前时间
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
String nowStr = sdf.format(now)+" 00:00:00"; //得到今天凌晨时间

Calendar calendar = Calendar.getInstance();
calendar.setTime(now);
calendar.add(Calendar.DAY_OF_MONTH, +1);//+1今天的时间加一天
String tomorrow = sdf.format(calendar.getTime())+" 00:00:00"; //得到明天凌晨的时间
```

