---
title: Chapter11 泛型
excerpt: 泛型概述，自定义泛型结构：泛型类、泛型接口、泛型方法
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 41c5fad1
date: 2020-10-31 11:24:21
updated: 2020-10-31 11:24:21
subtitle:
---
## 11.1 泛型概述
1. jdk 5.0新增的特性
2. 在集合中使用泛型：
   * 集合接口或集合类在jdk5.0时都修改为带泛型的结构。
   * 在实例化集合类时，可以指明具体的泛型类型
   * 指明完以后，在集合类或接口中凡是定义类或接口时，内部结构（比如：方法、构造器、属性等）使用到类的泛型的位,都指定为实例化的泛型类型。  
    比如：`add(E e) ` --->实例化以后：`add(Integer e)`
3. 注意点：
    * 泛型的类型必须是类，不能是基本数据类型。需要用到基本数据类型的位置，拿包装类替换
    * 如果实例化时，没有指明泛型的类型。默认类型为java.lang.Object类型。
    * jdk7新特性：类型推断, 后面无需指定
        ```java
        Map<String,Integer> map = new HashMap<>();
        ```
## 11.2 自定义泛型结构：泛型类、泛型接口、泛型方法
### 1. 泛型类、泛型接口
1. 泛型的声明
   ```java
   interface List<T> 和 class GenTest<K,V>
   ```
    * 其中， T,K,V不代表值，而是表示类型。 这里使用任意字母都可以。
    * 常用T表示，是Type的缩写。
2. 泛型的实例化：
   * 一定要在类名后面指定类型参数的值（类型）。如：
    ```java
    List<String> strList = new ArrayList<String>();
    Iterator<Customer> iterator = customers.iterator();
    ```
   * T只能是类，不能用基本数据类型填充，但可以使用包装类填充。
   * 把一个集合中的内容限制为一个特定的数据类型，这就是generics背后的核心思想
3. 注意事项
   1. 泛型不同的引用不能相互赋值
   2. 异常类不能声明为泛型类
   3. 不能使用`new E[]`。但是可以：`E[] elements = (E[])new Object[capacity]`;
   4. 静态方法中不能使用类的泛型
   5. 父类有泛型，子类可以选择保留泛型也可以选择指定泛型类型：
      * 子类不保留父类的泛型：按需实现
        * 没有类型 擦除
        * 具体类型
      * 子类保留父类的泛型：泛型子类
        * 全部保留
        * 部分保留
       * 结论：子类必须是“富二代”，子类除了指定或保留父类的泛型，还可以增加自己的泛型
      ```java
      class Father<T1, T2> {
      }
      // 子类不保留父类的泛型
      // 1)没有类型 擦除
      class Son1 extends Father {// 等价于class Son extends Father<Object,Object>{
      }
      // 2)具体类型
      class Son2 extends Father<Integer, String> {
      }
      // 子类保留父类的泛型
      // 1)全部保留
      class Son3<T1, T2> extends Father<T1, T2> {
      }
      // 2)部分保留
      class Son4<T2> extends Father<Integer, T2> {
      }
      ```
### 2. 泛型方法
1. 格式
    ```java
    [访问权限] <泛型> 返回类型 方法名([泛型标识 参数名称]) 抛出的异常
    public <E> List<E> Order(E[] arr) {
        //
    }
    ```
2. 泛型方法可以声明为静态的。原因：泛型参数是在调用方法时确定的。并非在实例化类时确定。

## 11.3 泛型在继承上的体现
1. 类A是类B的父类，但是G\<A> 和G\<B>二者不具备子父类关系，二者是并列关系。
2. 类A是类B的父类，A\<G> 是 B\<G> 的父类
```java
List<Object> list1 = null;
List<String> list2 = new ArrayList<String>();
//此时的list1和list2的类型不具有子父类关系
list1 = list2;//编译不通过
```
```java
AbstractList<String> list1 = null;
List<String> list2 = null;
ArrayList<String> list3 = null;

list1 = list3;//编译通过
list2 = list3;//编译通过
```

## 11.4 通配符的使用
1. 通配符?  
   类A是类B的父类，G\<A>和G\<B>是没有关系的，二者共同的父类是：G\<?>
   ```java
   List<Object> list1 = null;
   List<String> list2 = null;
   List<?> list = null;
   
   list = list1;
   list = list2;
   ```
2. 添加(写入)：  
   对于`List<?>`就不能向其内部添加数据，除了添加null之外。

3. 获取(读取)：  
允许读取数据，读取的数据类型为`Object`。  
    ```java
    List<?> list = null
    Object o = list.get(0);
    System.out.println(o);
    ```
4. 有限制的通配符
* `<?>`：允许所有泛型的引用调用
* 通配符指定上限
  * 上限extends：使用时指定的类型必须是继承某个类，或者实现某个接口，即<=
* 通配符指定下限
  * 下限super：使用时指定的类型不能小于操作的类，即>=
* 举例：
  * `<? extends Number>` (无穷小 , Number]  
    只允许泛型为Number及Number子类的引用调用
  * `<? super Number>` [Number , 无穷大)  
    只允许泛型为Number及Number父类的引用调用
  * `<? extends Comparable>`  
    只允许泛型为实现Comparable接口的实现类的引用调用 
* 代码示例：
   ```java
   List<? extends Person> list1 = null;
   List<? super Person> list2 = null;

   List<Student> list3 = new ArrayList<Student>();
   List<Person> list4 = new ArrayList<Person>();
   List<Object> list5 = new ArrayList<Object>();

   list1 = list3;
   list1 = list4;

   list2 = list4;
   list2 = list5;

   //读取数据：
   list1 = list3;
   Person p = list1.get(0);
   Student s = list1.get(0);   //编译不通过

   list2 = list4;
   Object obj = list2.get(0);
   Student s = list1.get(0);  //编译不通过

   //写入数据：
   list1.add(new Student());//编译不通过
   //编译通过
   list2.add(new Person());
   list2.add(new Student());
   ```


