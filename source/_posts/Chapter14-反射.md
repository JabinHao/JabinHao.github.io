---
title: Chapter14 反射
excerpt: 摘要
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 5aacf62
date: 2020-11-02 00:23:38
updated: 2020-11-02 00:23:38
subtitle:
---
## 14.1 Java反射机制概述
### 1. 概念
1. Reflection（反射）是被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。
2. 加载完类之后， 在堆内存的方法区中就产生了一个Class类型的对象（ 一个类只有一个Class对象） ， 这个对象就包含了完整的类的结构信息。 我们可以通过这个对象看到类的结构

### 2. 反射相关的主要API
* `java.lang.Class`:代表一个类
* `java.lang.reflect.Method`:代表类的方法
* `java.lang.reflect.Field`:代表类的成员变量
* `java.lang.reflect.Constructor`:代表类的构造器

## 14.2 理解`Class`类并获取`Class`的实例
### 1. `Class`类
1. 类的加载过程：
   * 程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)。
   * 使用java.exe命令对某个字节码文件进行解释运行。相当于将某个字节码文件加载到内存中。此过程就称为类的加载。
   * 加载到内存中的类，我们就称为运行时类，此运行时类，就作为`Class`的一个实例。
2. 即`Class`的实例就对应着一个运行时类。
3. 加载到内存中的运行时类，会缓存一定的时间。在此时间之内，我们可以通过不同的方式来获取此运行时类。

### 2. 获取`Class`的实例的方式（前三种方式需要掌握）
1. 调用运行时类的属性：.class
   ```java
   Class clazz1 = Person.class;
   ```
2. 通过运行时类的对象,调用getClass()
   ```java
   Person p1 = new Person();
   Class clazz2 = p1.getClass();
   ```

3. 调用Class的静态方法：forName(String classPath)
   ```java
   Class clazz3 = Class.forName("com.atguigu.java.Person");
   ```
4. 使用类的加载器：ClassLoader  (了解)
   ```java
   ClassLoader classLoader = ReflectionTest.class.getClassLoader();
   Class clazz4 = classLoader.loadClass("com.atguigu.java.Person");
   ```

### 3. 哪些类型可以有Class对象
或者说Class实例可以是哪些结构的：
1. class：外部类， 成员(成员内部类， 静态内部类)， 局部内部类， 匿名内部类
2. interface： 接口
2. []：数组
2. enum：枚举
2. annotation：注解@interface
2. primitive type：基本数据类型
2. void

## 14.3 类的加载与`ClassLoader`的理解
### 1. 概述
1. 类加载的过程  
   加载(Load)-->链接(Link)-->初始化(Initialize)
2. 类加载器作用  
   把类(class)装载进内存
### 2. 获取类加载器
1. 对于自定义类，使用系统类加载器进行加载
   ```java        
   ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
   ```
2. 调用系统类加载器的getParent()：获取扩展类加载器
   ```java
   ClassLoader classLoader1 = classLoader.getParent();
   ```
3. 调用扩展类加载器的getParent()：无法获取引导类加载器   
   引导类加载器主要负责加载java的核心类库，无法加载自定义类的。
   ```java
   ClassLoader classLoader2 = classLoader1.getParent();
   ```
4. 读取配置文件  
    配置文件默认识别为：当前module的src下
   ```java
   ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
   InputStream is = classLoader.getResourceAsStream("jdbc1.properties");
   pros.load(is);
   ```

## 14.4 创建运行时类的对象
1. 调用Class对象的`newInstance()`方法  
要 求：
   * 类必须有一个无参数的构造器。
   * *类的构造器的访问权限需要足够。
2. 没有无参构造器的类：
   ```java
   //1.根据全类名获取对应的Class对象
   String name = "atguigu.java.Person";
   Class clazz = null;
   clazz = Class.forName(name);
   //2.调用指定参数结构的构造器，生成Constructor的实例
   Constructor con = clazz.getConstructor(String.class,Integer.class);
   //3.通过Constructor的实例创建对应类的对象，并初始化类属性
   Person p2 = (Person) con.newInstance("Peter",20)
   ```

## 14.5  获取运行时类的完整结构
### 1. 如何获取   
1. Class类的getXxx()方法可以获取类的各种结构，返回值为一下类：`Field`、 `Method`、 `Constructor`、 `Superclass`、 `Interface`、 `Annotation`
2. 相应类内定义了各种方法
### 2. 获取全部属性：`Field`
1. 获取属性
   * `public Field[] getFields()` 返回此Class对象所表示的类或接口及其父类中的`public`的`Field`。
   * `public Field[] getDeclaredFields()`返回此Class对象所表示的类或接口的全部`Field`, (不包含父类中声明的属性)
2. Field方法中：
   * `public int getModifiers()` 以整数形式返回此`Field`的修饰符
   * `public Class<?> getType()` 得到`Field`的属性类型
   * `public String getName()` 返回`Field`的名称
3. 代码示例
   ```java
   Class clazz = Person.class;
   Field[] declaredFields = clazz.getDeclaredFields();
   for(Field f : declaredFields){
      //1.权限修饰符
      int modifier = f.getModifiers();
      System.out.print(Modifier.toString(modifier) + "\t");
      //2.数据类型
      Class type = f.getType();
      System.out.print(type.getName() + "\t");
      //3.变量名
      String fName = f.getName();
   ```

### 3. 获取全部的方法`Method`
1. 
   * `public Method[] getDeclaredMethods()` 返回此Class对象所表示的类或接口的全部方法
   * `public Method[] getMethods()` 返回此Class对象所表示的类或接口及其所有父类中的public的方法
2. `Method`类中：
   * `public String getName()` 获取方法名
   * `public Class<?> getReturnType()` 取得全部的返回值
   * `public Class<?>[] getParameterTypes()` 取得全部的参数
   * `public int getModifiers()` 取得修饰符
   * `public Class<?>[] getExceptionTypes()` 取得异常信息
   * `public Annotation[] getAnnotations()` 获取方法声明的注解

### 4. 获取全部的构造器
1. 两个方法
   * `public Constructor<T>[] getConstructors()`返回此 `Class` 对象所表示的类的所有public构造方法。
   * `public Constructor<T>[] getDeclaredConstructors()`返回此 `Class` 对象表示的类声明的所有构造方法。
2. `Constructor`类中：
   * 取得修饰符: `public int getModifiers()`;
   * 取得方法名称: `public String getName()`;
   * 取得参数的类型： `public Class<?>[] getParameterTypes()`;

### 5. 所继承的父类
1. `public Class<? Super T> getSuperclass()`返回表示此`Class`所表示的实体（类、接口、基本类型）的父类的
`Class`。
2. `public Type getGenericSuperclass()` 获取运行时类的带泛型的父类
3. 获取运行时类的带泛型的父类的泛型
   ```java
   Class clazz = Person.class;
   Type genericSuperclass = clazz.getGenericSuperclass();
   ParameterizedType paramType = (ParameterizedType) genericSuperclass;
   //获取泛型类型
   Type[] actualTypeArguments = paramType.getActualTypeArguments();
   System.out.println(((Class)actualTypeArguments[0]).getName());
   ```

### 6. 获取运行时类实现的接口
1. `public Class<?>[] getInterfaces()` 确定此对象所表示的类或接口实现的接口
2. 获取运行时类的父类实现的接口
   ```java
   Class[] interfaces1 = clazz.getSuperclass().getInterfaces();
   ```

### 7. 获取运行时类所在的包
1. `public Package getPackage()`

### 8. 获取运行时类声明的注解
1. `get Annotation(Class<T> annotationClass)`
2. `public Annotation[] getDeclaredAnnotations()`

## 14.6 调用运行时类的指定结构
### 1. 调用指定属性
1. 可以调用私有、默认属性
2. 步骤
```java
Class clazz = Person.class;
//创建运行时类的对象
Person p = (Person) clazz.newInstance();
//1. getDeclaredField(String fieldName):获取运行时类中指定变量名的属性
Field name = clazz.getDeclaredField("name");
//2.保证当前属性是可访问的
name.setAccessible(true);
//3.获取、设置指定对象的此属性值
name.set(p,"Tom");
System.out.println(name.get(p)); //Tom
```

### 2. 调用指定方法
1. 静态方法与非静态
2. 步骤
```java
Class clazz = Person.class;
//创建运行时类的对象
Person p = (Person) clazz.newInstance();

/*
1.获取指定的某个方法
getDeclaredMethod():参数1 ：指明获取的方法的名称  参数2：指明获取的方法的形参列表
*/
Method show = clazz.getDeclaredMethod("show", String.class);
//2.保证当前方法是可访问的
show.setAccessible(true);

/*
3. 调用方法的invoke():参数1：方法的调用者  参数2：给方法形参赋值的实参
invoke()的返回值即为对应类中调用的方法的返回值。
*/
Object returnValue = show.invoke(p,"CHN"); //对于static的方法，参数一可以使用null
System.out.println(returnValue);
```

### 3. 调用运行时类中的指定的构造器
1. 步骤
   ```java
   Class clazz = Person.class;

   //private Person(String name)
   /*
   1.获取指定的构造器
   getDeclaredConstructor():参数：指明构造器的参数列表
   */
   Constructor constructor = clazz.getDeclaredConstructor(String.class);

   //2.保证此构造器是可访问的
   constructor.setAccessible(true);

   //3.调用此构造器创建运行时类的对象
   Person per = (Person) constructor.newInstance("Tom");
   System.out.println(per);
   ```

## 14.7 反射的应用：动态代理
















