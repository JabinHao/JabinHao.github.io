---
title: Chapter5 面向对象(下)
excerpt: static关键字、final关键字、main方法及代码块
tags:
  - java
categories:
  - Java笔记
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.jpg
abbrlink: e1a4b3a0
date: 2020-10-21 01:25:23
updated: 2020-10-21 01:25:23
subtitle:
# abbrlink: chapter5
---
## 5.1 `static`关键字
可以修饰属性、方法、代码块、内部类

### 1. 属性
* static修饰的属性称为静态属性（类变量），与之相对的是非静态属性（实例变量）
* 相当于类的属性，某个对象修改该属性时，其他对象的该属性也发生变化
* 随着类的加载而加载，早于对象的创建，存在于方法区的静态域中
* 可以直接通过`class.property`调用

### 2. 方法
* 随着类的加载而加载，可以通过`class.method()`调用
* 静态方法中只能调用静态方法或属性
* 静态方法中不能使用`this`、`super`关键字

### 3. 使用场景
* 操作静态属性的方法，通常设置为static
* 工具类中的属性，习惯上声明为static，比如Math、Arrays、Collections

### 4. 单例设计模式
1. 定义  
   指采取一定方法保证整个软件系统中，某个类只能存在一个对象实例
2. 实现方式
   * 饿汉式
   * 懒汉式
3. 单例模式的饿汉式实现
   ```java
   ls
   class Order(){
    //1.私有化类的构造器
    private Order(){

    }

    // 2. 内部创建类的静态对象
    private static Order instance = new Order();

    // 3. 提供公共的静态方法，返回类的对象
    public static Order getInstance(){
      return instance;
    }
   }
   ```


4. 单例模式的懒汉式实现
   
   ```java
   class Order(){
     // 1. 私有化类的构造器
     private Order(){

     }
     //2. 声明当前对象，但不初始化（静态对象）
     private static Order instance = null;

     // 3. 声明public、static的返回当前类对象的方法
     public static Order getInstance(){
       if(instance == null){
         instance = new Order();
       }

       return instance;
     }
   }
   ```

5. 两种方式对比
   * 饿汉式
     * 加载时间过长
     * 线程安全
   * 懒汉式
     * 延迟对象的创建

6. 应用场景
   * 网站计数器
   * 应用程序的日志应用
   * 数据库连接池
   * 项目中读取配置的类
   * Application
   * Windows的任务管理器
   * Windows的回收站

## 5.2 `main`方法
### 使用说明
1. 作为程序的入口
2. 也是一个普通的静态方法，可以在程序中调用
3. 可以作为与控制台交互的方式

## 5.3 类的成员四：代码块
1. 结构：一对大括号 `{}`
2. 作用：初始化类、对象
3. 修饰词：只能使用`static`修饰
   * 静态代码块
     * 随着类的加载而加载并执行，只执行一次
     * 初始化类的信息
     * 多个静态代码块时，按顺序执行
     * 静态代码块早于非静态代码块执行
   * 非静态代码块
     * 随着对象的创建而加载执行
     * 每创建一个对象执行一次
     * 可以在创建对象时对其进行初始化
     * 先于构造器执行

## 5.4 `final`关键字
1. 可以修饰类、方法、变量
2. 修饰类，则类不能被继承
3. 修饰方法，则方法不能重写
4. 修饰变量，则变量值不能修改(一般大写)
   * 修饰属性：可以定义后在代码块、构造器中赋值
   * 修饰局部变量
   * 修饰形参：方法内不能修改该变量
5. `static final`: 全局常量
6. 


