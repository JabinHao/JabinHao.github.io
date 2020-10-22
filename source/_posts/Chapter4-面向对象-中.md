---
title: Chapter4 面向对象(中)
excerpt: 继承与多态、向下转型、Object类、包装类的使用
tags:
  - java
categories:
  - Java笔记
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.jpg
abbrlink: f598b0dc
date: 2020-10-19 22:12:14
updated: 2020-10-21 01:25:14
subtitle:
---
## 4.1 继承基础
### 1. 继承的优点
1. 减少了代码的冗余
2. 便于功能扩展
3. 为多态性的使用提供了前提
### 2. 代码格式
`class A extends B{}`
* A: 子类、派生类，subclass
* B: 父类、基类，superclass
### 3. 规则 
* 子类继承父类所有结构、属性、方法
* **子类不能直接调用继承到的私有属性、方法**
* 子类可以声明自己的属性、方法
* **同名属性不会覆盖**
* Java只能单继承，即只能有一个父类
* Java可以多层继承 (A->B->C): 直接父类、间接父类
* 所有类都继承自 `java.lang.object` 类
## 4.2 方法重写
定义：子类对父类中同名同参方法进行覆盖  
### 规则：
1. 同名同参
2. 子类方法权限修饰符大于等于父类方法
3. 私有方法不能重写
4. 返回值类型
   1. 父类为void则子类也为void
   2. 父类为A类则子类返回值可以为A的子类：父类返回`object`，子类可以返回`String`
   3. 基本数据类型则子父类必须相同
5. 子类重写抛出异常类型不大于父类（异常处理）
6. 静态方法(`static`)不能被重写

## 4.3 四种权限修饰符
1. 不同包下的普通类只能调用`public`的属性、方法
2. 不同包下的子类可以调用`public`、`protected`

## 4.4 `super`关键字
1. 在子类中调用父类结构(与`this`相对)
2. 子类中有父类同名属性时:
   * `this.property`: 子类属性
   * `super.property`: 父类属性
3. 子类重写了父类方法时，可以用`super`调用父类方法
4. 调用父类构造器：`super(参数列表);`：
   * 必须声明在子类构造器首行
   * 与`this()`不能同时使用
5. 子类构造器未显示调用或重载时，默认调用父类**空参**构造器

## 4.4 子类对象实例化过程
创建对象时会调用父类构造器(多层)，但只生成一个对象。

## 4.5 多态性
### 1. 定义
对象的多态性：父类的引用指向子类的对象
```java
superclass p = new subclass();
```
可以认为是向上转型：子类指针给了父类
### 2. 使用
* 不能调用子类特有方法
* 调用同名同参方法时执行的是子类重写的方法
* 即“编译看左边，执行看右边”
* 多态性不适用于属性，即只能调用父类属性
  
**多态是运行时行为**  
### 3. 向下转型
多态调用子类方法：强制类型转换将指针转换为子类
  ```
  superclass p1 = new subclass(); // 多态
  subclass p2 = (subclass)p1; // 强制类型转换
  ```
转换后的新指针可以调用子类方法、属性  

**父类指针可以指向子类对象，反之不行**  
例子：父类指针指向子类对象
* 指针转换为其他子类：编译通过，运行出错
* 指针转换为其他父类：编译运行都通过

### 4. `instanceof`关键字
1. 作用  
判断对象是否为某个类的实例，返回值为 `Boolean`
    ```java
    对象 instanceof 类;
    ```
2. 使用情景  
向下转型前判断，避免出现 `ClassCastException` 异常

## 4.6 `Object` 类的使用
`java.lang.Object`
### 1. 说明
1. Object类是所有Java类的根父类
2. Object类无属性，只有一个空参构造器
3. Object 类属性：
   * `clone()`
   * `equals(obj)`
   * `finalize()`: 垃圾回收，自动调用
   * `getClass()`: 返回当前对象所属的类
   * `toString()`: 
### 2. 常用属性
1. `equals(obj)`  
    **`==`回顾：**  
      * 用于基本数据类型与引用数据类型
      * 比较基本数据类型时直接比较值
      * 不同类型可以互相比较：`int`、`double`、`char` 之间可以比较  
      * 引用数据类型比较时比较的是地址值  

    **`equals()`方法：**
    * 适用于引用数据类型
    * `Object` 中定义为比较地址值(与`==`相同)
    * `String`、`Data`、`File`、包装类等重写了该方法，不再比较地址，而是比较具体内容

    **自定义类重写`equals()`方法**
    ```java
    public boolean equals(Object obj) {
		if (obj == this) {    // 首先比较地址是否
			return true;     //  相同
		}

		if (obj instanceof Person) {                                // 是否是同一个类的实例
			Person p = (Person) obj;                                
			return this.age == p.age && this.name.equals(p.name);  // 参数是否相同
		}

		return false;
	}
    ```
    一般直接调用IDE的重写方法
2. `toString()`
   * 当直接输出一个对象时，，实际调用的是对象的`toString()`方法
   * Object中的定义：
        ```java
        public String toString() {
            return getClass().getName() + "@" + Integer.toHexString(hashCode());
        }
        ```
   * `String`、`Data`、`File`、包装类等重写了该方法
   * 自定义类可以重写该方法

## 4.7 包装类的使用
### 1. 定义
针对八种基本数据类型定义的相应引用类型——包装类（封装类）
基本数据类型|引用数据类型|父类
:-:|:-:|:-:
byte|Byte|Number
short|Short|Number
int|Integer|Number
long|Long|Number
float|Float|Number
double|Double|Number
boolean|Boolean|-
char|Character|-

### 2. 基本数据类型转换为包装类
调用包装类构造器：
```java
int n1 = 10;
Integer in1 = new Integer(n1);
Integer in1 = new Integer("n1"); //这种写法也可
```
**`boolean` 转换为 `Boolean` 后，不再是 `false` 而是 `null`**

### 3. 包装类转换为基本数据类型
调用包装类的 `xxxValue()` 方法：
```java
Integer in1 = new Integer(12);

int n1 = in1.intValue();
```
### 4. 新特性：自动装箱与拆箱
```java
int n1 = 10;
Integer in1 = n1; //自动装箱
int n2 = in1;         //自动拆箱
```
### 5. 基本数据类型、包装类 转换为String类型
1. 连接运算 `+`:
   ```java
    int n1 = 10;
    String str1 = "" + n1;
   ```
2. 调用`String.valueof()`方法
   ```java
    float f1 = 10.5f;
    String str1 = String.valueOf(f1);
   ```
### 6. String类型转换为基本数据类型
调用包装类的 `parseXxx()` 方法:
```java
String str1 = "123";
int n1 = Integer.parseInt(str1);

String str2 = "true1";
boolean b1 = Boolean.parseBoolean(str2);//此时b1为 false
```

总结：![](https://cdn.jsdelivr.net/gh/JabinHao/mihs/img/xiaoxin/convert.png)

### 特殊情况
Integer内部定义了IntegerCache结构，其中定义了integer[], 保存了-128~127范围的整数，使用自动装箱时，在此范围内则不再new，提高了效率：
```java
Integer n1 = 1;
Integer n2 = 1;
n1 == n2; // true

Integer n1 = 128;
Integer n2 = 128;
n1 == n2; // false

Integer n1 = new Integer(1);
Integer n2 = new Integer(1);
n1 == n2; // false
```