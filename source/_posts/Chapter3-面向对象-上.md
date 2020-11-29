---
title: Chapter3 面向对象(上)
excerpt: 类的概念、JVM内存结构、匿名对象、封装与隐藏、关键字this import package等
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 656389f2
date: 2020-10-04 04:03:02
updated: 2020-10-21 01:25:02
subtitle:
---
## 3.1 类和对象
**局部变量无初始化值，需显式赋值。**
### 1. 属性
属性有默认初始化值，局部变量没有
### 2. 方法
1. 默认使用public权限。Java有四种权限修饰符：`private、public、缺省、protected`
2. 重载  
   方法名相同，参数列表不同
3. 可变个数形参  
   * 参数可以是零个、一个...
   * 可以重载
   * 与方法名相同，形参为数组的方法不能共存
   * 可变形参必须声明在末尾
   ```java
   method(type ... name){}
   // 例：
   public void show(String ... strs)

   //以下不能与上面共存
   public void show(String[] strs)
   ```

### 3. 内存结构

1. 虚拟机栈：  
   局部变量
2. 堆：  
  new出来的结构（包括对象的非static属性）
3. 方法区：
   类的加载信息、常量池、静态域

### 4. 匿名对象
代码示例：
```java
new classname().method();
```
### 5. 参数传递机制
* 基本数据类型：值传递
* 引用数据类型：地址传递

## 3.2 封装与隐藏
### 1. 封装的体现
* 将属性私有化，提供公共的方法来获取和设置属性值
* 私有方法
* ...

### 2. 权限修饰符
封装性的体现需要权限修饰符来实现
1. private、缺省、protected、public    

修饰符|类内部|同一个包|不同包的子类|同一个工程    
:-:|:-:|:-:|:-:|:-:  
private|Yes| | | |
缺省|Yes|Yes
protected|Yes|Yes|Yes|
public|Yes|Yes|Yes|Yes|
---

2. 修饰符的使用
* 可以用来修饰类的内部结构、属性、方法、构造器、内部类
* class只能使用public和default

## 3.3 构造器（constructor）
1. 作用  
   创建对象，初始化
2. 说明  
   如果没有显式定义类的构造器，则系统默认提供一个空参的构造器
3. 构造器格式  
   `权限修饰符 类名(形参列表){}`
4. 构造器可以重载
5. 一旦显式定义构造器，则系统不再提供默认空参构造器

## 3.4 JavaBean
* 由Java语言写成的可重用组件
* 符合以下标准的类：
  * public
  * 有一个无参的构造器
  * 有属性及其对应的get、set方法


## 3.5 UML图
如图：
![Django的MVT模式](https://cdn.jsdelivr.net/gh/JabinHao/mihs/img/xiaoxin/UML.png)

## 3.6 this关键字
使用：属性、方法、构造器    
   在类中使用：
   * 调用属性：`this.属性`
   * 调用构造器：`this()`（用于在构造器内部调用其他构造器）
   * 一个构造器中只能调用一个其他构造器，必须在首句。

## 3.7 package与import关键字 

### 1. package
1. 为了更好的实现项目中类的管理
2. 使用 `package` 声明类或接口所属的包，声明在源文件的首行
3. 包属于标识符（命名规范）
4. 每个.代表一层文件目录
5. 同一个包下不能命名同名接口或类

#### Java主要包：
1. java.lang----包含一些Java语言的核心类， 如String、 Math、 Integer、 System和
Thread， 提供常用功能
2. java.net----包含执行与网络相关的操作的类和接口。
3. java.io ----包含能提供多种输入/输出功能的类。
4. java.util----包含一些实用工具类， 如定义系统特性、 接口的集合框架类、 使用与日
期日历相关的函数。
5. java.text----包含了一些java格式化相关的类
6. java.sql----包含了java进行JDBC数据库编程的相关类/接口
7. java.awt----包含了构成抽象窗口工具集（abstract window toolkits） 的多个类， 这
些类被用来构建和管理应用程序的图形用户界面(GUI)

#### MVC设计模式
> MVC是常用的设计模式之一，将整个程序分为三个层次： 视图模型层，控制器层，与
数据模型层。 这种将程序输入输出、数据处理，以及数据的展示分离开来的设计模式
使程序结构变的灵活而且清晰，同时也描述了程序各个对象间的通信方式，降低了程
序的耦合性。
### 2. import
1. 在源文件中显式使用导入指定包下的类、接口
2. 声明在包声明之后
3. 可使用 `import xxx.*`导入该包下所有结构 **(不包含子包)**
4. `java.lang` 包和同一个包下定义的类或接口可省略 `import`
5. 可使用全类名而不必导入: `com.xxx.class n = new com.xxx.class()`
6. `import static`: 导入指定类或接口中的静态结构



