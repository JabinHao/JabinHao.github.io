---
title: Java BigDecimal的使用
excerpt: Java中浮点数存在精度损失，而 BigDecimal 可以表示一个任意大小且精度完全准确的浮点数。
tags:
  - java
  - BigDecimal
categories:
  - Java笔记
  - 其它
banner_img: /img/post/banner/Levi Ackerman.png
index_img: /img/post/index/limbo.jpg
category: Java笔记/其它
abbrlink: dbac2c57
date: 2021-06-11 17:40:00
updated: 2021-06-11 21:04:37
subtitle:
---
## 1. 简介

### 1.1 java中浮点数的存储

1. 形式  
   * java中浮点数的存储遵循 IEEE 754 标准，由 `符号位 S`、 `阶码 E`、 `尾数 M `三部分组成，以float为例，组成为：   `SEEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM` ，其中
      * 符号位只占一位，0表示正，1表示负
      * 阶码即指数部分，不过是二进制形式的科学计数，而且采用移码
      * 尾数部分决定了一个数的精度，整数部分都是1， 所以只用记录小数部分
   * java中的 float 大小为 4 字节，也就是32`bit`，double 为 8 字节， 也就是64`bit`，其中SEM三部分的大小如下：
        符号位| 阶码 |尾数
        :-:|:-:|:-:
        1|8|23
        1|11|52

2. 举例说明  

    下面我们举例说明一下，对于一个十进制数，首先要将其转换为二进制数，具体来讲就是整数部分除 2 取余，结果反转， 小数部分乘 2 取余，我们以 20.625 为例：
    整数部分|商|余数
    :-:|:-:|:-:
    20|10|0
    10|5|0
    5|2|1
    2|1|0
    1|0|1

    结果需要反转，因此整数部分为： 10100

    小数部分|积|取整数部分
    :-:|:-:|:-:
    0.625|1.25|1
    0.25|0.5|0
    0.5|1.0|1
    0.0|0|0
    
    小数部分结果为 .101  

    最终结果为 10100.101，科学计数法表示：
    $1.0100101 \times 2^4$
    * 正数，因此符号位为0
    * 阶码（指数）为4，float移码要加127，因此为131，即
    * 尾数即小数点后面部分, 0100101，23位，后面补0

    因此 float型10进制数 20.375 在内存中的格式为：
    ```java
    01000001 10100101 00000000 00000000
    ```
   
3. 精度损失  

   上面的例子似乎并没有什么问题，但是如果我们的小数部分是 0.36，则转换成二进制就会是 `0.01011100001010001111011...`，这里计算机存储时必须进行截取，也就产生了精度损失。

### 1.2 BigDecimal

1. 为了解决浮点数精度损失问题，Java在java.math包中提供的API类BigDecimal，用来对超过16位有效位的数进行精确的运算。
2. 双精度浮点型变量double可以处理16位有效数，对于那些不需要准确计算精度的数字，我们可以直接使用Float和Double处理，但是Double.valueOf(String) 和Float.valueOf(String)会丢失精度。
3. 在开发中，如果我们需要精确计算的结果，则必须使用BigDecimal类来操作。

## 2. 使用

### 2.1 构造函数

1. 常用构造函数
   *  `BigDecimal(int)`：创建一个具有参数所指定整数值的对象
   *  `BigDecimal(double)`：创建一个具有参数所指定双精度值的对象
   *  `BigDecimal(long)`：创建一个具有参数所指定长整数值的对象
   *  `BigDecimal(String)`：创建一个具有参数所指定以字符串表示的数值的对象

2. 问题

    按照API说明，参数为 double 的构造函数具有不确定性：
    > Notes:  
    The results of this constructor can be somewhat unpredictable. One might assume that writing new BigDecimal(0.1) in Java creates a BigDecimal which is exactly equal to 0.1 (an unscaled value of 1, with a scale of 1), but it is actually equal to 0.1000000000000000055511151231257827021181583404541015625. This is because 0.1 cannot be represented exactly as a double (or, for that matter, as a binary fraction of any finite length). Thus, the value that is being passed in to the constructor is not exactly equal to 0.1, appearances notwithstanding.

    也就是说通过new BigDecimal(0.1)获得的数字未必等于0.1：
    ```java
    BigDecimal d1 = new BigDecimal(0.1);
    System.out.println(d1);
    >>> 0.1000000000000000055511151231257827021181583404541015625
    ```
    因此使用时应尽量使用 String 类型的构造函数：
    ```java
    BigDecimal d1 = new BigDecimal("0.1");
    System.out.println(d1);
    >>> 0.1
    ```

### 2.2 常用方法
1. 运算
   * 加法 a + b：`a.add(b)`
   * 减法 a - b：`a.subtract`
   * 乘法 a * b：`a.multiply(b)`
   * 除法 a / b：`a.divide(b)`

2. 转换
   * `toString()`：将BigDecimal对象中的值转换成字符串
   * `doubleValue()`：将BigDecimal对象中的值转换成双精度数
   * `floatValue()`：将BigDecimal对象中的值转换成单精度数
   * `longValue()`：将BigDecimal对象中的值转换成长整数
   * `intValue()`：将BigDecimal对象中的值转换成整数

3. 比较大小：`a.compareTo(b)`
   * -1 : a < b
   *  0 : a == b
   *  1 : a > b
4. 设置有效数字 `setScale(int newScale, RoundingMode roundingMode)`  

    其中，`newScale`表示有效数字，`roundingMode`设置舍弃部分的处理方式，`RoundingMode`是一个枚举类。
    * `RoundingMode.UP`：最后一位加 1（1.243 -> 1.25）
    * `RoundingMode.DOWN`：最后一位不变 (1.376 -> 1.37)
    * `RoundingMode.CEILING`：接近正无穷大的舍入模式(1.243 -> 1.25, -1.243 -> -1.24)
    * `RoundingMode.FLOOR`：接近正无穷小的舍入模式
    * `RoundingMode.HALF_UP`：四舍五入
    * `RoundingMode.HALF_DOWN`：五舍六入
    * `RoundingMode.HALF_EVEN`：
        * 舍弃位有多位则直接晋位(2.1251 -> 2.13)
        * 舍弃位只有一位且为 5，则原则是让最后一位变成偶数(2.125 -> 2.12, 2.115 -> 2.12)
   
