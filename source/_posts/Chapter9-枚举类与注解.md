---
title: Chapter9 枚举类与注解
excerpt: 枚举类的使用、自定义枚举类，元注解与自定义注解、jdk8新特性等
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: a663daf0
date: 2020-10-29 23:06:34
updated: 2020-10-30 14:56:34
subtitle:
---
## 9.1 枚举类的使用
### 1. 概述
1. 枚举类的理解：类的对象只有有限个，确定的。我们称此类为枚举类
2. 当需要定义一组常量时，强烈建议使用枚举类
3. 如果枚举类中只有一个对象，则可以作为单例模式的实现方式。

###  2. 定义枚举类
1. jdk5.0之前：自定义
2. jdk5.0，可以使用`enum`关键字自定义枚举类
3. 代码示例——自定义
   ```java
   //自定义枚举类
   class Season{
       //1.声明Season对象的属性:private final修饰
       private final String seasonName;
       private final String seasonDesc;

       //2.私有化类的构造器,并给对象属性赋值
       private Season(String seasonName,String seasonDesc){
           this.seasonName = seasonName;
           this.seasonDesc = seasonDesc;
       }

       //3.提供当前枚举类的多个对象：public static final的
       public static final Season SPRING = new Season("春天","春暖花开");
       public static final Season SUMMER = new Season("夏天","夏日炎炎");
       public static final Season AUTUMN = new Season("秋天","秋高气爽");
       public static final Season WINTER = new Season("冬天","冰天雪地");

       //4.其他诉求1：获取枚举类对象的属性
       public String getSeasonName() {
           return seasonName;
       }

       public String getSeasonDesc() {
           return seasonDesc;
       }
       //4.其他诉求1：提供toString()
       @Override
       public String toString() {
           return "Season{" +
                   "seasonName='" + seasonName + '\'' +
                   ", seasonDesc='" + seasonDesc + '\'' +
                   '}';
       }
    }
    ```
4. 代码示例 —— `enum`关键字  
    定义的枚举类默认继承于`java.lang.Enum`类
    ```java
    enum Season{
       //1.提供当前枚举类的对象，多个对象之间用","隔开，末尾对象";"结束
       SPRING("春天","春暖花开"),
       SUMMER("夏天","夏日炎炎"),
       AUTUMN("秋天","秋高气爽"),
       WINTER("冬天","冰天雪地");

       //2.声明Season对象的属性:private final修饰
       private final String seasonName;
       private final String seasonDesc;

       //3.私有化类的构造器,并给对象属性赋值
       private Season1(String seasonName,String seasonDesc){
           this.seasonName = seasonName;
           this.seasonDesc = seasonDesc;
       }

       //4.其他诉求1：获取枚举类对象的属性
       public String getSeasonName() {
           return seasonName;
       }

       public String getSeasonDesc() {
           return seasonDesc;
       }
    ```
### 3. `Enum`类常用方法
* `values()`方法：返回枚举类型的对象数组。该方法可以很方便地遍历所有的枚举值。
* `valueOf(String str)`：可以把一个字符串转为对应的枚举类对象。要求字符串必须是枚举类对象的“名字”。如不是，会有运行时异常：`IllegalArgumentException`。
* `toString()`：返回当前枚举类对象常量的名称

### 4. 使用`enum`关键字定义的枚举类实现接口的情况
1. 说明
    * 情况一：实现接口，在enum类中实现抽象方法
    * 情况二：让枚举类的对象分别实现接口中的抽象方法

2. 代码示例
     ```java
     enum Season implements Info{
         SPRING("春天","春暖花开"){
             @Override
             public void show() {
                 System.out.println("春天在哪里？");
             }
         },
         SUMMER("夏天","夏日炎炎"){
             @Override
             public void show() {
                 System.out.println("宁夏");
             }
         },
         AUTUMN("秋天","秋高气爽"){
             @Override
             public void show() {
                 System.out.println("秋天不回来");
             }
         },
         WINTER("冬天","冰天雪地"){
             @Override
             public void show() {
                 System.out.println("大约在冬季");
             }
         };

         // ...
    }
     ```

## 9.2 注解(Annotation)
### 1. 说明
* jdk5.0新增功能
* 使用 `Annotation` 时要在其前面增加 @ 符号, 并把该 `Annotation` 当成一个修饰符使用，用于修饰它支持的程序元素。

### 2.使用示例：
* 生成文档相关的注解
* 在编译时进行格式检查(JDK内置的三个基本注解)
    ```java
    @Override: 限定重写父类方法, 该注解只能用于方法
    @Deprecated: 用于表示所修饰的元素(类, 方法等)已过时。通常是因为所修饰的结构危险或存在更好的选择
    @SuppressWarnings: 抑制编译器警告
    ```
* 跟踪代码依赖性，实现替代配置文件功能

### 3. 自定义注解
1. 注解声明为 `@interface`
2. 内部定义成员，通常使用`value`表示
3. 可以指定默认值，使用`dafault`指定
4. 如果没有成员，则表明是一个标识作用
```java
public @interface MyAnnotation {

    String value() default "hello";
}
```

### 4. 元注解
1. 元注解：对现有的注解进行解释说明的注解
2. jdk 提供4种元注解
   * Retention：指定所修饰的 Annotation 的生命周期：SOURCE\CLASS（默认行为）\RUNTIME 只有声明为RUNTIME生命周期的注解，才能通过反射获取。
   * Target:用于指定被修饰的 Annotation 能用于修饰哪些程序元素
   * Documented:表示所修饰的注解在被javadoc解析时，保留下来。
   * Inherited:被它修饰的 Annotation 将具有继承性。
    ```java
    @Inherited
    @Repeatable(MyAnnotations.class)
    @Retention(RetentionPolicy.RUNTIME)
    @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE,TYPE_PARAMETER,TYPE_USE})
    public @interface MyAnnotation {

        String value() default "hello";
    }
    ```
### 5. jdk8新特性：可重复注解、类型注解
1. 可重复注解
   1. 在MyAnnotation上声明@Repeatable，成员值为MyAnnotations.class
   2. MyAnnotation的Target和Retention等元注解与MyAnnotations相同。
    ```java
    @Inherited
    @Repeatable(MyAnnotations.class)
    @Retention(RetentionPolicy.RUNTIME)
    @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE,TYPE_PARAMETER,TYPE_USE})
    public @interface MyAnnotation {

        String value() default "hello";
    }
    ```
    ```java
    @Inherited
    @Retention(RetentionPolicy.RUNTIME)
    @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
    public @interface MyAnnotations {

        MyAnnotation[] value();
    }
    ```
2. 类型注解
   * ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中（如：泛型声明）。
   * ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中。
  
    使用：
    ```java
    // 首先将ElementType.TYPE_USE声明在MyAnnotation的Target中
    // 声明后该注解能写在使用类型的任何语句中
    class Generic<@MyAnnotation T>{

      public void show() throws @MyAnnotation RuntimeException{

          ArrayList<@MyAnnotation String> list = new ArrayList<>();

          int num = (@MyAnnotation int) 10L;
      }

    }
    ```


