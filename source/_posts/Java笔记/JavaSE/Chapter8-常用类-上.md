---
title: Chapter8 常用类(上)
excerpt: String类及StringBuffer、StringBuilder的使用
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: afaff3d
date: 2020-10-27 23:59:53
updated: 2020-10-28 00:13:53
subtitle:
---
## 8.1 字符串相关类
### 1. String的使用
1. `String`类概述
   1. `String`声明为`final`，不可被继承
   2. `String`实现了`Serializable`接口：表示字符串是支持序列化的；实现了`Comparable`接口，表明可以比较大小
   3. `String`内部定义了`final char[] value`用于存储字符串数据
   4. `String` 代表不可变字符序列：
      * 对字符串重新赋值时，会重新指定内存区域赋值，不能使用原有`value`
      * 对字符修改（连接、`replace`）时，也需重新指定内存区域赋值
   5. 通过字面量的方式给字符串赋值（区别于new），此时字符串声明在字符串常量池中
   6. 字符串常量池中不会储存相同内容的字符串（即相同则为同一个地址）

    `String` 类**API:**
   ```java
   public final class String
       implements java.io.Serializable, Comparable<String>, CharSequence,
                  Constable, ConstantDesc {

       @Stable
       private final byte[] value;
       ....
   ```
2. String实例化方式
   1. 通过字面量定义：保存在字符串常量池中
        ```java
        String str = "java";
        ```
   2. 通过new + 构造器：保存在堆空间中
        ```java
        String str = new String("java");
        ```
        此时创建了两个对象，new了一个对象，该对象又在常量池中创建了字符串对象
   3. 字面量与变量拼接：在堆中创建
        ```java
        String s1 = "Hello"; //常量池
        String s2 = s1 + "world"; // 堆
        String s3 = "Hello" + "World"; //常量池
        ```
    4. 调用`intern`方法：//返回常量池中地址
        ```java
        String s4 = s2.intern(); //常量池
        ```
3. String类常用方法  
   常用方法：
   ```java
   1. int length()   //字符串长度
   2. char charAt(int index) // s索引处字符
   3. boolean isEmpty() // 是否是空字符串
   4. String toLowerCase()  // 返回小写字符串
   5. String toUpperCase()  //
   6. String trim() //去除字符串首尾空格
   7. boolean equals(obj) //比较内容是否相同
   8. boolean equalsIgnoreCase(String str) //忽略大小写比较
   9. String contact(String str) //等价与 +
   10. int comparTo(String str)  // 比较字符串大小，0相同，负数则当前小("abc"<"abe":-2)
   11. String subString(int start, int end) //切片，左闭右开， end可缺省
   ```
    常用方法2：
    ```java
    1. boolean endsWith(String str) //是否以str结尾
    2. boolean startsWith(String str, int toffset) // 从toffset索引处开始判断，是否以str开始，toffset可缺省
    3. boolean contains(CharSequence s) //判断当前字符串是否包含字符串 s
    4. int indexOf(String str, int index) //从index位置开始字符串str首次出现的位置，没有则为-1,index可缺省
    5. int lastIndexOf(String str, int index) // 从后往前找，索引仍是正着数，index缺省则从最后开始
    ```
    常用方法3：
    ```java
    1. String replace(char c1, char c2)// 使用字符c2替换c1
    2. String replace(CharSequence s1, CharSequence s2) // 字符串替换
    3. String replaceAll(String regex, String replacement)  //用replacement替换所有匹配正则表达式的子字符串
    4. String replaceFirst(String regex, String replacement) //···························的第一个子字符串
    5. boolean matches(String regex) //告知此字符串是否匹配给定的正则表达式
    6. String[] split(String regex) //根据给定正则表达式的匹配拆分此字符串。
    7. String[] split(String regex, int limit) //···,最多不超过limit个，若超过，剩下的全部放到最后一个元素中
    ```
4. `String`类与其他类转换
    1. String与基本数据类型、包装类
        ```java
        int num = Integer.parseInt(str);
        String str = String.valueOf(num)
        ```
    2. String与char[]之间转换
        ```java
        // 1. String --> char[]: String的toCharArrey()
        String str = "java";
        char[] charArray = str.toCharArray();
        // 2. char[] --> String: 调用String的构造器
        String str = new String(charArray);
        ```
    3. String与byte[]
        ```java
        // 1. String --> byte[]: 调用String的getBytes[]
        byte[] bytes = str.getBytes();//可以有参数，编码方式：str.getBytes(gbk)
        // 2. bytes[] --> String: Arrays.toString(bytes)
        String str = Arrays.toString(bytes);   //得到的是数组，并未转换，只是方便输出
        String str = new String(bytes, "gbk");  //解码，得到的是字符串
        ```

### 2. `StringBuffer`与`StringBuilder`的使用
1. `String`、`StringBuffer`、`StringBuilder`三者异同
   * `String`：不可变字符序列
   * `StringBuffer`：可变字符序列，线程安全
   * `StringBuilder`：可变字符序列，线程不安全，效率高
   * 三者底层均使用`char[]`存储
2. 源码分析
   ```java
    String str = new String();//char[] value = new char[0];
    String str1 = new String("abc");//char[] value = new char[]{'a','b','c'};

    StringBuffer sb1 = new StringBuffer();//char[] value = new char[16];底层创建了一个长度是16的数组。
    System.out.println(sb1.length());//
    sb1.append('a');//value[0] = 'a';
    sb1.append('b');//value[1] = 'b';

    StringBuffer sb2 = new StringBuffer("abc");//char[] value = new char["abc".length() + 16];

    //问题1. System.out.println(sb2.length());//3
    //问题2. 扩容问题:如果要添加的数据底层数组盛不下了，那就需要扩容底层的数组。
    
    //默认情况下，扩容为原来容量的2倍 + 2，同时将原有数组中的元素复制到新的数组中。
    //指导意义：开发中建议大家使用：StringBuffer(int capacity) 或 StringBuilder(int capacity)
   ```
3. 常用方法
   ```java
   StringBuffer append(xxx) //提供了很多的append()方法，用于进行字符串拼接
   StringBuffer delete(int start,int end) //删除指定位置的内容
   StringBuffer replace(int start, int end, String str) //把[start,end)位置替换为str
   StringBuffer insert(int offset, xxx) //在指定位置插入xxx
   StringBuffer reverse() //把当前字符序列逆转
   public int indexOf(String str)
   public String substring(int start,int end) //返回一个从start开始到end索引结束的左闭右开区间的子字符串
   public int length()
   public char charAt(int n )
   public void setCharAt(int n ,char ch)

    //  总结：
    //  增：append(xxx)
    //  删：delete(int start,int end)
    //  改：setCharAt(int n ,char ch) / replace(int start, int end, String str)
    //  查：charAt(int n )
    //  插：insert(int offset, xxx)
    //  长度：length();
    //  遍历：for() + charAt() / toString()
   ```
