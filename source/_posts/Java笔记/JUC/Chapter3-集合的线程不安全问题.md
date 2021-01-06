---
title: Chapter3 集合的线程不安全问题
excerpt: 摘要
tags:
  - java
  - juc
categories:
  - Java笔记
  - juc
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: e2095eb2
date: 2020-11-19 23:47:53
updated: 2020-11-19 23:47:53
subtitle:
---
### 3.1 `ArrayList`
1. 举例说明其线程不安全
   ```java
    public static void main(String[] args) {
            List<String> list = new ArrayList<>();
            for (int i = 0; i < 300; i++) {
                new Thread(()->{
                    list.add(UUID.randomUUID().toString().substring(0,8));
                    System.out.println(list);
                }, String.valueOf(i)).start();
            }
        }
   ```
    报错：
    > ConcurrentModificationException（并发修改异常）
2. 解决方法
   * 使用 `Vector` 代替 `ArrayList` ：Vector使用了synchronsized
   * 使用 `Collections`工具类：
        ```java
        List<String> list = Collections.synchronizedList(new ArrayList<>());
        ```
   * 使用`CopyOnWriteArrayList`代替 `ArrayList` ：
        ```java
        List<String> list = new CopyOnWriteArrayList();
        ```

### 3.2 `HashSet`
1. 举例说明其线程不安全  
   ```java
   public static void main(String[] args) {
        Set<String> set = new HashSet<>();
        for (int i = 0; i < 300; i++) {
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(set);
            }, String.valueOf(i)).start();
        }
    }
    ```
    `Hashset`底层为`HashMap`，`value`为常量。
2. 解决方法
   * 使用 `Collections`工具类：
        ```java
        Set<String> set = Collections.synchronizedSet(new HashSet<>());
        ```
   * 使用`CopyOnWriteArraySet`代替 `HashSet` ：
        ```java
        Set<String> set = new CopyOnWriteArraySet();
        ```

### 3.3 `HashMap`
1. 举例说明其不安全
    ```java
    public static void main(String[] args) {
        Map<String,String> map = new java.util.HashMap();
        for (int i = 0; i < 300; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0, 8));
                System.out.println(map);
            }).start();
        }
    }
    ```
2. 解决方法
    * Map<String,String> map = new ConcurrentHashMap<>();