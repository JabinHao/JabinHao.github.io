---
title: Chapter8 常用类(下)
excerpt: jdk8之前的日期时间API，jdk8中新的日期时间API，Java比较器、其他常用类等
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: fa0e57b9
date: 2020-10-28 23:31:22
updated: 2020-10-29 20:04:22
subtitle:
---
## 8.2 JDK8之前的日期时间API
### 1. `java.lang.System` 类  
1. `currentTimeMillis()`方法：  
   时间戳，返回当前时间与1970.1.1，0:0:0之间的时间差，单位为毫秒

### 2. `java.util.Date`类  
1. 构造器
   * `Date()`：无参构造器，获取本地当前时间
        ```java
        Date d = new Date();
        ```
    * `Date(long date)`：创建指定毫秒数的date对象
2. 方法
   * `toString()`：将date转换为可读模式：Wed Oct 28 23:50:55 CST 2020
   * `getTime()` 时间戳，返回当前对象与1970.1.1相差的毫秒数

### 3. `java.sql.Date`类
1. 数据库中的日期变量类型，`java.util.Date`类的子类
2. 转换：利用`getTime()`方法
    ```java
    Date date1 = new Date();
    java.sql.Date date2 = new java.sql.Date(date1.getTime());
    ```

### 4. `java.text.SimpleDateFormat`类
对`Date`类的格式化和解析
1. 默认空参构造器
   ```java
   Date date = new Date();//创建日期类
   SimpleDateFormat sdf = new SimpleDateFormat();// 实例化该类

   // 1. 字符串-->日期
   String format = sdf.format(date); // 2020/10/29 下午3:47

   // 2. 日期-->字符串
   String str = "2020/10/29 下午3:48"; //只能是这种格式
   Date date1 = sdf.parse(str);
   ```
2. 自定义格式  
   使用有参数的构造器，可以自定义输入格式
   ```java
   Date date = new Date();//创建日期类
   SimpleDateFormat sdf = new SimpleDateFormat("yyy-MM-dd hh:mm:ss");// 实例化该类,参数自定义

   // 1. 字符串-->日期
   String format = sdf.format(date); // 2020-10-29 03:52:52

   // 2. 日期-->字符串
   String str = "2020/10/29 下午3:48"; //只能是这种格式
   Date date1 = sdf.parse("2021-10-29 03:52:52"); //要使用对应格式
   ```

### 5. `java.util.Calendar`类
抽象类
1. 实例化——两种方式
   * 创建其子类(GregorianCalendar)的对象
   * 调用其静态方法getInstance(): `Calendar calendar = Claendar.getInstance()`
2. 常用方法
   1. `get()`: 获取日期、时间等：
      ```java
      Calendar calendar = Calendar.getInstance();
      int days = calendar.get(Calendar.DAY_OF_MONTH);
      ```
   2. `set()`：修改···
      ```java
      calendar.set(Calendar.DAY_OF_MONTH,22);
      ```
   3. `add()`增加
      ```java
      calendar.add(Calendar.DAY_OF_MONTH,15);
      ```
   4. `getTime()`:用于与`Date`类转换
      ```java
      Date date = calendar.getTime();
      ```
   5. `setTime()`:
      ```java
      Date date = new Date();
      calendar.setTime()date;
      ```
   6. 注意事项
      * 获取月份时，一月是0，二月是1；
      * 获取星期时，周日是1，周一是2；

## 8.3 JDK8中新日期时间API
### 1. `localDate`、`localTime`、`localDateTime`类
1. 实例化方式
   1. 调用`now()`方法
      ```java
      LocalDate localDate = LocalDate.now();
      LocalTime localTime = LocalTime.now();
      LocalDateTime localDateTime = LocalDateTime.now();
      ```
   2. `of()`方法
      ```java
      LocalDateTime localDateTime1 = LocalDateTime.of(2020, 10, 6, 13, 23, 43);
      ```
2. 常用方法
   1. `getXxx()`方法
      ```java
      System.out.println(localDateTime.getDayOfMonth());
      System.out.println(localDateTime.getDayOfWeek());
      System.out.println(localDateTime.getMonth());
      System.out.println(localDateTime.getMonthValue());
      System.out.println(localDateTime.getMinute());
      ```
      返回值一般为`int`
   2. `withXxx()`:设置相关属性
      ```java
      LocalDate localDate1 = localDate.withDayOfMonth(22);//原对象不变，返回新的，体现了不可变性
      LocalDateTime localDateTime2 = localDateTime.withHour(4);
      ```
   3. `plusXxx()`:增加日期、时间等
      ```java
      LocalDateTime localDateTime3 = localDateTime.plusMonths(3);
      ```
   4. `minusXxx()`:减少···
      ```java
      LocalDateTime localDateTime4 = localDateTime.minusDays(6);
      ```

### 2. `Instant`类
时间戳
1. 实例化
   1. `now()`方法
      ```java
      Instant instant = Instant.now();
      ```
2. 常用方法
   1. `atOffset()`  
      获取自1970年1月1日0时0分0秒（UTC）开始的毫秒数  ---> Date类的getTime()
      ```java
      OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
      ```
   2. `toEpochMilli()`
      ```java
      long milli = instant.toEpochMilli();
      ```
   3. `ofEpochMilli()`  
      通过给定的毫秒数，获取Instant实例  -->Date(long millis)
      ```java
      Instant instant1 = Instant.ofEpochMilli(1550475314878L);
      ```

### 3. `DateTimeFormatter`类
* `java.time.format`
* 格式化及解析日期与时间，类似于SimpleDateFormat

1. 实例化
   1. 预定义标准格式，如：ISO_LOCAL_DATE_TIME; ISO_LOCAL_DATE; ISO_LOCAL_TIME
      ```java
      DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
      //格式化:日期-->字符串
      LocalDateTime localDateTime = LocalDateTime.now();
      String str1 = formatter.format(localDateTime); //2020-10-29T21:05:31.4965446

      //解析：字符串 -->日期
      TemporalAccessor parse = formatter.parse("2020-10-29T21:05:31");
      ```
2. 本地化相关的格式。
   1. `ofLocalizedDateTime()`  
      可用常量：`FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT`
      ```java
      DateTimeFormatter formatter1 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
      //格式化
      String str2 = formatter1.format(localDateTime);
      System.out.println(str2);//2020年10月29日 CST 下午9:40:55
      ```
   2. `ofLocalizedDate()`  
      可用常量：`FormatStyle.FULL / FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT`
      ```java
      DateTimeFormatter formatter2 = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM);
      //格式化
      String str3 = formatter2.format(LocalDate.now());
      System.out.println(str3);//2020年10月29日
      ```
   3. 自定义格式。如：`ofPattern(“yyyy-MM-dd hh:mm:ss”)`
      ```java
      DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
      //格式化
      String str4 = formatter3.format(LocalDateTime.now());
      System.out.println(str4);//2020-10-29 09:47:34

      //解析
      TemporalAccessor accessor = formatter3.parse("2019-02-18 03:52:09");
      System.out.println(accessor);
     ```

## 8.4 java 比较器
1. 说明：  
   * Java中的对象，正常情况下，只能进行比较：==  或  != 。不能使用 > 或 < 
   * 在开发场景中，我们需要对多个对象进行排序，则需使用两个接口中的一个：Comparable或Comparator
2. `Comparable`接口与`Comparator`的使用的对比：  
   * `Comparable`接口的方式一旦一定，保证Comparable接口实现类的对象在任何位置都可以比较大小。
   * `Comparator`接口属于临时性的比较。
### 1. Comparable接口的使用举例： 自然排序  
1. 像String、包装类重写compareTo()方法以后，进行了从小到大的排列
2. **重写compareTo(obj)的规则：**
   * 如果当前对象this大于形参对象obj，则返回正整数，
   * 如果当前对象this小于形参对象obj，则返回负整数，
   * 如果当前对象this等于形参对象obj，则返回零。
3. 自定义类  
   * 实现`Comparable`接口，重写`compareTo(obj)`方法, 在`compareTo(obj)`方法中指明如何排序:
   * 在类内部重写`compareTo()`方法（类需要`implements Comparable`）
   ```java
   //指明商品比较大小的方式:按照价格从低到高排序,再按照产品名称从高到低排序
    @Override
    public int compareTo(Object o) {
        if(o instanceof Goods){
            Goods goods = (Goods)o;
            //方式一：
            if(this.price > goods.price){
                return 1;
            }else if(this.price < goods.price){
                return -1;
            }else{
               return -this.name.compareTo(goods.name);
            }
            //方式二：
   //           return Double.compare(this.price,goods.price);
        }
        throw new RuntimeException("传入的数据类型不一致！");
    }
    ```

### 2. Comparator接口的使用：定制排序
1. 背景：  
    当元素的类型没有实现java.lang.Comparable接口而又不方便修改代码，或者实现了java.lang.Comparable接口的排序规则不适合当前的操作，那么可以考虑使用 Comparator 的对象来排序
2. 重写compare(Object o1,Object o2)方法，比较o1和o2的大小：
   * 如果方法返回正整数，则表示o1大于o2；
   * 如果返回0，表示相等；
   * 返回负整数，表示o1小于o2。
3. 使用  
直接用匿名实现类的匿名对象
   ```java 
   Arrays.sort(arr, new Comparator(){
      @Override
      public int compare(Object o1, Object o2) {
          if(o1 instanceof String && o2 instanceof  String){
              String s1 = (String) o1;
              String s2 = (String) o2;
              return -s1.compareTo(s2);  //按照字符串从大到小的顺序排列
          }
         return 0;
          throw new RuntimeException("输入的数据类型不一致");
      }
   })
   ```

## 8.5 其他常用类
### 1. `System`类
常用方法
* `currentTimeMillis()`
* `void exit(int status)`：结束程序
* `void gc()`：请求垃圾回收
* `String getProperty(String key)`：获取相应属性值

### 2. `Math`类
提供了一系列静态方法
```java
public BigInteger abs()：返回此 BigInteger 的绝对值的 BigInteger。
BigInteger add(BigInteger val) ：返回其值为 (this + val) 的 BigInteger
BigInteger subtract(BigInteger val) ：返回其值为 (this - val) 的 BigInteger
BigInteger multiply(BigInteger val) ：返回其值为 (this * val) 的 BigInteger
BigInteger divide(BigInteger val) ：返回其值为 (this / val) 的 BigInteger。整数相除只保留整数部分。
BigInteger remainder(BigInteger val) ：返回其值为 (this % val) 的 BigInteger。
BigInteger[] divideAndRemainder(BigInteger val)：返回包含 (this / val) 后跟(this % val) 的两个 BigInteger 的数组。
BigInteger pow(int exponent) ：返回其值为 (exponent次方) 的 BigInteger。
```

### 3. `BigInteger` 与 `BigDecimal`类
* BigInteger可以表示不可变的任意精度的整数
* BigDecimal类支持不可变的、任意精度的有符号十进制定点数