---
title: Chapter10 集合(下)
excerpt: Map接口及其实现类、Map常用方法、HashMap实现原理、Collections工具类等
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 858e7b84
date: 2020-10-30 23:34:50
updated: 2020-10-30 23:34:50
subtitle:
---
## 10.6 `Map`接口
### 1. `Map`框架
1. `Map`:双列数据，存储key-value对的数据   
   * `HashMap`:作为Map的主要实现类；线程不安全的，效率高；存储null的key和value
     * `LinkedHashMap`:保证在遍历map元素时，可以按照添加的顺序实现遍历。  
     原因：在原有的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素。对于频繁的遍历操作，此类执行效率高于`HashMap`。
   * `TreeMap`:保证按照添加的`key-value`对进行排序，实现排序遍历。此时考虑key的自然排序或定制排序，底层使用红黑树
   * `Hashtable`:作为古老的实现类；线程安全的，效率低；不能存储`null`的`key`和`value`
     * `Properties`:常用来处理配置文件。`key`和`value`都是`String`类型

2. HashMap的底层：
* 数组+链表  （jdk7及之前）
* 数组+链表+红黑树 （jdk 8）

### 2. Map结构的理解：
* Map中的key:无序的、不可重复的，使用Set存储所有的key  ---> key所在的类要重写equals()和hashCode() （以HashMap为例）
* Map中的value:无序的、可重复的，使用Collection存储所有的value --->value所在的类要重写equals()
* 一个键值对：key-value构成了一个Entry对象。
* Map中的entry:无序的、不可重复的，使用Set存储所有的entry

### 3. HashMap的底层实现原理
1. jdk7中
   ```java
    HashMap map = new HashMap():
    ```
    在实例化以后，底层创建了长度是16的一维数组Entry[] table。  
    ..可能已经执行过多次put...
    ```java
    map.put(y1,value1):
    ```
    * 首先，调用key1所在类的`hashCode()`计算key1哈希值，此哈希值经过某种算法计算以后，得到在Entry数组中的存放位。
        * 如果此位置上的数据为空，此时的`key1-value1`添加成功。 ----情况1
        * 如果此位置上的数据不为空，(意味着此位置上存在一个或多个数据(以链表形式存在)),比较key1和已经存在的一个或个数据的哈希值：
            * 如果`key1`的哈希值与已经存在的数据的哈希值都不相同，此时`key1-value1`添加成功。----情况2
            * 如果`key1`的哈希值和已经存在的某一个数据(`key2-value2`)的哈希值相同，继续比较：调用key1所在类的`equals(key2)`方法，比较：
                * 如果`equals()`返回`false`:此时`key1-value1`添加成功。----情况3
                * 如果`equals()`返回`true`:使用`value1`替换`value2`。

    * 补充：关于情况2和情况3：此时`key1-value1`和原来的数据以链表的方式存储。

    * 在不断的添加过程中，会涉及到扩容问题，当超出临界值(且要存放的位置非空)时，扩容。默认的扩容方式：扩容为原容量的2倍，并将原有的数据复制过来。

2. jdk8 相较于jdk7在底层实现方面的不同：
   1. `new HashMap()`:底层没有创建一个长度为16的数组
   2. jdk 8底层的数组是：Node[], 而非Entry[]
   3. 首次调用`put()`方法时，底层创建长度为16的数组
   4. jdk7底层结构只有：数组+链表。jdk8中底层结构：数组+链表+红黑树。
      1. 形成链表时，七上八下（jdk7:新的元素指向旧的元素。jdk8：旧的元素指向新的元素）
      2. 当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8 且当前数组的长度 > 64时，此时此索引位上的所数据改为使用红黑树存储。
3. API中的一些常量
    ```java
   DEFAULT_INITIAL_CAPACITY : HashMap的默认容量，16
   DEFAULT_LOAD_FACTOR：HashMap的默认加载因子：0.75
   threshold：扩容的临界值，=容量*填充因子：16 * 0.75 => 12
   TREEIFY_THRESHOLD：Bucket中链表长度大于该默认值，转化为红黑树:8
   MIN_TREEIFY_CAPACITY：桶中的Node被树化时最小的hash表容量:64
   ```

### 4. `LinkedHashMap`的底层实现原理（了解）
1. `HashMap`中的内部类： `Node`
   ```java
   static class Node<K,V> implements Map.Entry<K,V> {
      final int hash;
      final K key;
      V value;
      Node<K,V> next;
   }
   ```
2. `LinkedHashMap`中的内部类： `Entry`
   ```java
   static class Entry<K,V> extends HashMap.Node<K,V> {
      Entry<K,V> before, after;
      Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
      }
   }

### 5. `Map`中定义的方法：
1. 添加、删除、修改操作：
    * `Object put(Object key,Object value)`：将指定key-value添加到(或修改)当前map对象中
    * `void putAll(Map m)`:将m中的所有key-value对存放到当前map中
    * `Object remove(Object key)`：移除指定key的key-value对，并返回value
    * `void clear()`：清空当前map中的所有数据
2. 元素查询的操作：
    * `Object get(Object key)`：获取指定key对应的value
    * `boolean containsKey(Object key)`：是否包含指定的key
    * `boolean containsValue(Object value)`：是否包含指定的value
    * `int size()`：返回map中key-value对的个数
    * `boolean isEmpty()`：判断当前map是否为空
    * `boolean equals(Object obj)`：判断当前map和参数对象obj是否相等
3. 元视图操作的方法：
    * `Set keySet()`：返回所有key构成的Set集合
    * `Collection values()`：返回所有value构成的Collection集合
    * `Set entrySet()`：返回所有key-value对构成的Set集合
4. 遍历操作
    ```java
     //方式一：entrySet()
     Set entrySet = map.entrySet();
     Iterator iterator1 = entrySet.iterator();
     while (iterator1.hasNext()){
         Object obj = iterator1.next();
         //entrySet集合中的元素都是entry
         Map.Entry entry = (Map.Entry) obj;
         System.out.println(entry.getKey() + "---->" + entry.getValue());

     }
     System.out.println();
     //方式二：
     Set keySet = map.keySet();
     Iterator iterator2 = keySet.iterator();
     while(iterator2.hasNext()){
         Object key = iterator2.next();
         Object value = map.get(key);
         System.out.println(key + "=====" + value);

     }
    ```

### 6. TreeMap
1. 向TreeMap中添加key-value，要求key必须是由同一个类创建的对象
2. 按照key进行排序：自然排序 、定制排序
3. 定制排序示例：
    ```java
   TreeMap map = new TreeMap(new Comparator() {
      @Override
      public int compare(Object o1, Object o2) {
          if(o1 instanceof User && o2 instanceof User){
              User u1 = (User)o1;
              User u2 = (User)o2;
              return Integer.compare(u1.getAge(),u2.getAge());
          }
          throw new RuntimeException("输入的类型不匹配！");
      }
   });
    ```

### 7. Map实现类之`Properties`
* `Properties` 类是 `Hashtable` 的子类，该对象用于处理属性文件
* 由于属性文件里的 `key`、 `value` 都是字符串类型，所以 `Properties` 里的 `key` 和 `value` 都是字符串类型
* 存取数据时，建议使用 `setProperty(String key,String value)` 方法和 `getProperty(String key)` 方法
```java
Properties pros = new Properties();
pros.load(new FileInputStream("jdbc.properties"));
String user = pros.getProperty("user");
System.out.println(user);
```

## 10.7 `Collections`工具类
### 1. 常用方法
1. `reverse(List)`：反转 List 中元素的顺序
2. `shuffle(List)`：对 List 集合元素进行随机排序
3. `sort(List)`：根据元素的自然顺序对指定 List 集合元素按升序排序
4. `sort(List，Comparator)`：根据指定的 Comparator 产生的顺序对 List 集合元素进行排序
5. `swap(List，int， int)`：将指定 list 集合中的 i 处元素和 j 处元素进行交换
6. `Object max(Collection)`：根据元素的自然顺序，返回给定集合中的最大元素
7. `Object max(Collection，Comparator)`：根据 Comparator 指定的顺序，返回给定集合中的最大元素
8. `Object min(Collection)`
9. `Object min(Collection，Comparator)`
10. `int frequency(Collection，Object)`：返回指定集合中指定元素的出现次数
11. `void copy(List dest,List src)`：将src中的内容复制到dest中
12. `boolean replaceAll(List list，Object oldVal，Object newVal)`：使用新值替换 List 对象的所有旧值

    **`copy()`使用注意：**
    ```java
    List dest = Arrays.asList(new Object[list.size()]);
    System.out.println(dest.size());//list.size();
    Collections.copy(dest,list);
    ```
### 2. 其他
`Collections` 类中提供了多个 `synchronizedXxx()` 方法，该方法可使将指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题


