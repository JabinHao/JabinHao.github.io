---
title: Chapter15 JDK8新特性
excerpt: 摘要
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 35d81cec
date: 2020-11-15 21:48:41
updated: 2020-11-15 21:48:41
subtitle:
---
## 15.1 Lambda表达式
1. 格式：
   * -> ：lambda操作符 或 箭头操作符
   * 左边：lambda形参列表 （其实就是接口中的抽象方法的形参列表）
   * 右边：lambda体 （其实就是重写的抽象方法的方法体）
2. Lambda表达式的使用：
   * ->左边：lambda形参列表的参数类型可以省略(类型推断)；如果lambda形参列表只有一个参数，其一对()也可以省略
   * ->右边：lambda体应该使用一对{}包裹；如果lambda体只有一条执行语句（可能是return语句），省略这一对{}和return关键字
3. Lambda表达式的本质：作为函数式接口的实例
4. 匿名实现类表示的都可以用Lambda表达式来写
5. . 示例
   ```java
   (o1,o2) -> Integer.compare(o1,o2);
   ```

## 15.2 函数式接口
1. 如果一个接口中，只声明了一个抽象方法，则此接口就称为函数式接口。我们可以在一个接口上使用 `@FunctionalInterface` 注解，这样做可以检查它是否是一个函数式接口。
2. java内置的4大核心函数式接口
    * 消费型接口 Consumer<T>     void accept(T t)
    * 供给型接口 Supplier<T>     T get()
    * 函数型接口 Function<T,R>   R apply(T t)
    * 断定型接口 Predicate<T>    boolean test(T t)

## 15.3 方法引用与构造器引用
### 1. 方法引用
1. 使用情境：  
   当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！
2. 方法引用，本质上就是Lambda表达式，而Lambda表达式作为函数式接口的实例。所以方法引用，也是函数式接口的实例。
3. 使用格式：  
   ```sql
   类(或对象) :: 方法名
   ```
4. 分类
   * 对象 :: 非静态方法
   * 类 :: 静态方法
   * 类 :: 非静态方法
5. 使用要求  
   接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同！（针对于情况1和情况2）
6. 代码示例
   ```java
   public class MethodRefTest {

	// 情况一：对象 :: 实例方法
	//Consumer中的void accept(T t)
	//PrintStream中的void println(T t)
	@Test
	public void test1() {
		Consumer<String> con1 = str -> System.out.println(str);
		con1.accept("北京");

		System.out.println("*******************");
		PrintStream ps = System.out;
		Consumer<String> con2 = ps::println;
		con2.accept("beijing");
	}
	
	//Supplier中的T get()
	//Employee中的String getName()
	@Test
	public void test2() {
		Employee emp = new Employee(1001,"Tom",23,5600);

		Supplier<String> sup1 = () -> emp.getName();
		System.out.println(sup1.get());

		System.out.println("*******************");
		Supplier<String> sup2 = emp::getName;
		System.out.println(sup2.get());

	}

	// 情况二：类 :: 静态方法
	//Comparator中的int compare(T t1,T t2)
	//Integer中的int compare(T t1,T t2)
	@Test
	public void test3() {
		Comparator<Integer> com1 = (t1,t2) -> Integer.compare(t1,t2);
		System.out.println(com1.compare(12,21));

		System.out.println("*******************");

		Comparator<Integer> com2 = Integer::compare;
		System.out.println(com2.compare(12,3));

	}
	
	//Function中的R apply(T t)
	//Math中的Long round(Double d)
	@Test
	public void test4() {
		Function<Double,Long> func = new Function<Double, Long>() {
			@Override
			public Long apply(Double d) {
				return Math.round(d);
			}
		};

		System.out.println("*******************");

		Function<Double,Long> func1 = d -> Math.round(d);
		System.out.println(func1.apply(12.3));

		System.out.println("*******************");

		Function<Double,Long> func2 = Math::round;
		System.out.println(func2.apply(12.6));
	}

	// 情况三：类 :: 实例方法  (有难度)
	// Comparator中的int comapre(T t1,T t2)
	// String中的int t1.compareTo(t2)
	@Test
	public void test5() {
		Comparator<String> com1 = (s1,s2) -> s1.compareTo(s2);
		System.out.println(com1.compare("abc","abd"));

		System.out.println("*******************");

		Comparator<String> com2 = String :: compareTo;
		System.out.println(com2.compare("abd","abm"));
	}
   ```
### 2. 构造器引用
1. 使用  
   和方法引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一致。抽象方法的返回值类型即为构造器所属的类的类型
2. 格式
   ```java
   类 :: new;
   ```
### 3. 数组引用
1. 说明  
   把数组看做是一个特殊的类，则写法与构造器引用一致
2. 格式
   ```java
   类型[] :: new;
   ```
## 15.4 强大的 `Stream API`
### 1. 概述
1. `Stream`与集合
   * `Stream`关注的是对数据的运算，与CPU打交道
   * 集合关注的是数据的存储，与内存打交道
2. `Stream`特点
   * `Stream` 自己不会存储元素。
   * `Stream` 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
   * `Stream` 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行
3. `Stream` 执行流程
   * ① `Stream` 的实例化
   * ② 一系列的中间操作 (过滤、映射、...)
   * ③ 终止操作
4. 说明：
   * 一个中间操作链，对数据源的数据进行处理
   * 一旦执行终止操作，就执行中间操作链，并产生结果。之后，不会再被使用
### 2. `Stream` 的实例化
1. 创建 `Stream`方式一：通过集合
   ```java
   List<Employee> employees = EmployeeData.getEmployees();

   // default Stream<E> stream() : 返回一个顺序流
   Stream<Employee> stream = employees.stream();

   // default Stream<E> parallelStream() : 返回一个并行流
   Stream<Employee> parallelStream = employees.parallelStream();
   ```
2. 创建 Stream方式二：通过数组
   ```java
   int[] arr = new int[]{1,2,3,4,5,6};
   // 调用Arrays类的static <T> Stream<T> stream(T[] array): 返回一个流
   IntStream stream = Arrays.stream(arr);

   //其他流
   Employee e1 = new Employee(1001,"Tom");
   Employee e2 = new Employee(1002,"Jerry");
   Employee[] arr1 = new Employee[]{e1,e2};
   Stream<Employee> stream1 = Arrays.stream(arr1);
   ```
3. 创建 Stream方式三：通过Stream的of()方法
   ```java
   Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6);
   ```
4. 创建 Stream方式四：创建无限流
   ```java
   迭代
   // public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)
   // 遍历前10个偶数
   Stream.iterate(0, t -> t + 2).limit(10).forEach(System.out::println);

   // 生成
   // public static<T> Stream<T> generate(Supplier<T> s)
   Stream.generate(Math::random).limit(10).forEach(System.out::println);
   ```
### 3. `Stream` 的中间操作
1. 筛选与切片
   方法|描述
   :-:|:-
   `filter(Predicate p)`|接收 Lambda ， 从流中排除某些元素
   `distinct()`|筛选，通过流所生成元素的 hashCode() 和 equals() 去除重复元素
   `limit(long maxSize)`|截断流，使其元素不超过给定数量
   `skip(long n)`|跳过元素，返回一个扔掉了前 个空流。与 limit(n) 互补 n 个元素的流。若流中元素不足 n 个，则返回一
2. 映射
   方法|描述
   :-:|:-
   `map(Function f)`|接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
   `mapToDouble(ToDoubleFunction f)`|接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 `DoubleStream`。
   `mapToInt(ToIntFunction f)`|接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 `IntStream`。
   `mapToLong(ToLongFunction f)`|接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 LongStream。
   `flatMap(Function f)`|接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
3. 排序
   方法|描述
   :-:|:-
   `sorted()`|产生一个新流，其中按自然顺序排序
   `sorted(Comparator com)`|产生一个新流，其中按比较器顺序排序
### 4. `Stream` 的终止操作
1. 匹配与查找
   方法|描述
   :-:|:-
   `count()`|返回流中元素总数
   `max(Comparator c)`|返回流中最大值
   `min(Comparator c)`|返回流中最小值
   `forEach(Consumer c)`|内部迭代(使用 `Collection` 接口需要用户去做迭代，称为外部迭代。相反，`Stream API` 使用内部迭代)
2. 归约
   方法|描述
   :-:|:-
   `reduce(T iden, BinaryOperator b)`|可以将流中元素反复结合起来，得到一个值。返回 T
   `reduce(BinaryOperator b)`|可以将流中元素反复结合起来，得到一个值。返回 `Optional<T>`
   这里的 `BinaryOperator`是一个两参数函数，参数类型为T
3. 收集
   方法|描述
   :-:|:-
   `collect(Collector c)`|将流转换为其他形式。接收一个 `Collector` 接口的实现，用于给`Stream` 中元素做汇总的方法
   * `Collector` 接口中方法的实现决定了如何对流执行收集的操作(如收集到 List、 Set、Map)
   * `Collectors` 实用类提供了很多静态方法，可以方便地创建常见收集器实例：

      方法 | 返回类型 | 作用
      :-:|:-:|:-
      `toList` | `List<T>` | 把流中元素收集到`List`
      `toSet` | `Set<T>` | 把流中元素收集到`Set`
      `toCollection` | `Collection<T>` | 把流中元素收集到创建的集合
      `counting` | `Long` | 计算流中元素的个数
      ...|...|...

## 15.5 `Optional` 类
### 1. 说明  
   * `Optional<T>` 类(`java.util.Optional`) 是一个容器类， 它可以保存类型T的值， 代表这个值存在。或者仅仅保存`null`，表示这个值不存在。
   * `Optional` 可以避免空指针异常
### 2. 常用方法
1. 创建对象
   * `Optional.of(T t)`：创建一个 Optional 实例， t必须非空；
   * `Optional.empty()`：创建一个空的 Optional 实例
   * `Optional.ofNullable(T t)`：t可以为null
2. 获取Optional容器的对象
   * `T get()`: 如果调用对象包含值，返回该值，否则抛异常
   * `T orElse(T other)` ： 如果有值则将其返回，否则返回指定的other对象。
   * `T orElseGet(Supplier<? extends T> other)` ： 如果有值则将其返回，否则返回由`Supplier`接口实现提供的对象
   * `T orElseThrow(Supplier<? extends X> exceptionSupplier)` ： 如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。
3. 判断Optional容器中是否包含对象
   * `boolean isPresent()` : 判断是否包含对象
   * `void ifPresent(Consumer<? super T> consumer)` ： 如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它