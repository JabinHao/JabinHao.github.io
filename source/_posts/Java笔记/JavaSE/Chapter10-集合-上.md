---
title: Chapter10 集合(上)
excerpt: 集合框架、Collection接口及子接口List与Set、集合实现类、Iterator迭代器
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 2871df29
date: 2020-10-30 14:58:50
updated: 2020-10-30 14:58:50
subtitle:
---
## 10.1 集合框架概述
### 1. 背景
1. 集合、数组都是对多个数据进行存储操作的结构，简称Java容器。  
 说明：此时的存储，主要指的是内存层面的存储，不涉及到持久化的存储（.txt, .jpg, .avi，数据库中）
2. 数组存在一些缺点

### 2. 集合框架
1. `Collection`接口： 单列数据， 定义了存取一组对象的方法的集合
    1. `List`：元素有序、可重复的集合
        * `ArrayList`
        * `LinkedList`
        * `Vector`
    2. `Set`： 元素无序、不可重复的集合
        * `HashSet`
        * `LinkedHashSet`
        * `TreeSet`
2. `Map`接口： 双列数据，保存具有映射关系“`key-value`对”的集合
   * `HashMap`
   * `LinkedHashMap`
   * `TreeMap`
   * `Hashtable`
   * `Properties`

## 10.2 Collection接口方法
### 1. 常用方法1
1. `add(obj)`：添加
2. `size()`: 获取添加的元素的个数
3. `addAll(Collection coll1)`: 将coll1集合中的元素添加到当前的集合中
4. `clear()`: 清空集合元素
5. `isEmpty()`: 判断当前集合是否为空

### 2. 常用方法2
1. `contains(obj)`：判断当前集合中是否包含obj，调用obj对象所在类的`equals()`。
2. `containsAll(Collection coll1)`: 判断形参coll1中的所有元素是否都存在于当前集合中。
3. `remove(Object obj)`: 从当前集合中移除obj元素。
4. `removeAll(Collection coll1)`: 差集：从当前集合中移除coll1中所有的元素。 

**向Collection接口的实现类的对象中添加数据obj时，要求obj所在类要重写`equals()`.**

### 3. 常用方法3
1. `retainAll(Collection coll1)`: 交集：获取当前集合和coll1集合的交集，并返回给当前集合(即删掉不同，保留相同)
2. `equals(Object obj)`: 要想返回true，需要当前集合和形参集合的元素都相同。
3. `hashCode()`: 返回当前对象的哈希值
4. `toArray()`: 集合 --->数组
5. `iterator()`:返回`Iterator`接口的实例，用于遍历集合元素。

数组-->集合：`Arrays.asList()`

## 10.3 `Iterator`迭代器接口
用于遍历 `Collection` 集合中的元素
### 1. `hasNext()` 和  `next()`方法
    ```java
    Iterator iterator = coll.iterator();
    while(iterator.hasNext()){
        //next():①指针下移 ②将下移以后集合位置上的元素返回
        System.out.println(iterator.next());
    }
    ```

### 2. 说明
1. 集合对象每次调用`iterator()`方法都得到一个全新的迭代器对象，
2. 默认游标都在集合的第一个元素之前。

### 3. `remove()`方法
在遍历的时候，删除集合中的元素。此方法不同于集合直接调用`remove()`  
```java
Iterator iterator = coll.iterator();
    while (iterator.hasNext()){
        // iterator.remove();
        Object obj = iterator.next();
        if("Tom".equals(obj)){
            iterator.remove();
        //  iterator.remove();
        }

    }
    //遍历集合
    iterator = coll.iterator();
    while (iterator.hasNext()){
        System.out.println(iterator.next());
    }
```

如果还未调用`next()`或在上一次调用 next 方法之后已经调用了 `remove()` 方法，再调用`remove()`都会报`IllegalStateException`。

### 4. `foreach`循环
* jdk 5.0 新增了foreach循环，用于遍历集合、数组
* 格式：`for(数组元素的类型 局部变量 : 数组对象)`
  ```java
  for(String s : arr){
    //
  }
  ```

## 10.4 `Collection`子接口之一：`List`接口
### 1. `List`接口实现类
1. `ArrayList`：作为List接口的主要实现类；线程不安全的，效率高；底层使用`Object[] elementData`存储
2. `LinkedList`：对于频繁的插入、删除操作，使用此类效率比ArrayList高；底层使用**双向链表**存储
3. `Vector`：作为`List`接口的古老实现类；线程安全的，效率低；底层使用`Object[] elementData`存储

### 2. `ArrayList`的源码分析
1. jdk 7情况下
   ```java
    ArrayList list = new ArrayList();//底层创建了长度是10的Object[]数组elementData
   list.add(123);//elementData[0] = new Integer(123);
   ...
   list.add(11);//如果此次的添加导致底层elementData数组容量不够，则扩容。默认情况下，扩容为原来的容量的1.5倍，同时需要将原有数组中的数据复制到新的数组中。  
   ```
    结论：建议开发中使用带参的构造器：ArrayList list = new ArrayList(int capacity)
2. jdk 8中`ArrayList`的变化：
    ```java
    ArrayList list = new ArrayList();//底层Object[] elementData初始化为{}.并没有创建长度为10的数组
    list.add(123);//第一次调用add()时，底层才创建了长度10的数组，并将数据123添加到elementData[0]
    ...
    后续的添加和扩容操作与jdk 7 无异。
    ```
3. 小结：  
   jdk7中的ArrayList的对象的创建类似于单例的饿汉式，而jdk8中的ArrayList的对象的创建类似于单例的懒汉式，延迟了数组的创建，节省内存。

### 3. LinkedList的源码分析
```java
LinkedList list = new LinkedList(); 内部声明了Node类型的first和last属性，默认值为null
list.add(123);//将123封装到Node中，创建了Node对象。
```
其中，Node定义为：
```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
    this.item = element;
    this.next = next;
    this.prev = prev;
    }
}
```
体现了`LinkedList`的双向链表的说法

### 4. `Vector`的源码分析
* jdk7和jdk8中通过Vector()构造器创建对象时，底层都创建了长度为10的数组。
* 在扩容方面，默认扩容为原来的数组长度的2倍。

### 5. `List`接口方法
1. `void add(int index, Object ele)`:在index位置插入ele元素
2. `boolean addAll(int index, Collection eles)`:从index位置开始将eles中的所有元素添加进来
3. `Object get(int index)`:获取指定`index`位置的元素
4. `int indexOf(Object obj)`:返回`obj`在集合中首次出现的位置
5. `int lastIndexOf(Object obj)`:返回`obj`在当前集合中末次出现的位置
6. `Object remove(int index)`:移除指定`index`位置的元素，并返回此元素
7. `Object set(int index, Object ele)`:设置指定`index`位置的元素为`ele`
8. `List subList(int fromIndex, int toIndex)`:返回从`fromIndex`到`toIndex`位置的子集合(左闭右开)

## 10.5 `Collection`子接口之二：`Set`接口
### 1. `Set`接口实现类
1. `HashSet`：作为`Set`接口的主要实现类；线程不安全的；可以存储`null`值
    * `LinkedHashSet`：作为HashSet的子类；遍历其内部数据时，可以按照添加的顺序遍历对于频繁的遍历操作，`LinkedHashSet`效率高于`HashSet`.
2. `TreeSet`：可以按照添加对象的指定属性，进行排序。

### 2. 说明
1. Set接口中没有额外定义新的方法，使用的都是Collection中声明过的方法。
2. 向Set(主要指：`HashSet`、`LinkedHashSet`)中添加的数据，其所在的类一定要重写`hashCode()`和`equals()`
   1. 重写的`hashCode()`和`equals()`尽可能保持一致性：相等的对象必须具有相等的散列码
   2. 重写技巧：对象中用作 `equals()` 方法比较的 `Field`，都应该用来计算 `hashCode` 值

### 3. `HashSet()`类
1. `Set`：存储无序的、不可重复的数据，对于`HashSet`：
   1. 无序性：不等于随机性。存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值决定的。
   2. 不可重复性：保证添加的元素按照equals()判断时，不能返回true.即：相同的元素只能添加一个。
2. 添加元素的过程:
   1. 首先调用元素a所在类的`hashCode()`方法，计算其哈希值，从而得到底层数组中的索引位置，判断数组此位置上是否已经有元素：
      1. 如果此位置上没有其他元素，则元素a添加成功。 --->情况1
      2. 如果此位置上有其他元素b(或以链表形式存在的多个元素），则比较元素a与元素b的hash值：
         1. 如果`hash`值不相同，则元素a添加成功。--->情况2
         2. 如果hash值相同，进而需要调用元素a所在类的`equals()`方法：
            1. `equals()`返回`true`,元素a添加失败
            2. `equals()`返回`false`,则元素a添加成功。--->情况3
   2. 对于添加成功的情况2和情况3而言：元素a 与已经存在指定索引位置上数据以链表的方式存储。
       * jdk 7 :元素a放到数组中，指向原来的元素。
       * jdk 8 :原来的元素在数组中，指向元素a
       * 总结：七上八下

### 4. `LinkedHashSet`类
1. LinkedHashSet作为HashSet的子类，在添加数据的同时，每个数据还维护了两个引用，记录此数据前一个数据和后一个数据。
2. 优点：对于频繁的遍历操作，LinkedHashSet效率高于HashSet

### 5. `TreeSet`类
1. 向TreeSet中添加的数据，要求是相同类的对象。
2. 两种排序方式：自然排序（实现`Comparable`接口） 和 定制排序（`Comparator`）
3. 自然排序中，比较两个对象是否相同的标准为：`compareTo()`返回0.不再是`equals()`.
4. 定制排序中，比较两个对象是否相同的标准为：`compare()`返回0.不再是`equals()`.
    ```java
    //定制排序，首先实现Comparator接口
    Comparator com = new Comparator() {
      //按照年龄从小到大排列
        @Override
        public int compare(Object o1, Object o2) {
            //
        }
    };
    // 作为参数传入
    TreeSet set = new TreeSet(com);
    ```


