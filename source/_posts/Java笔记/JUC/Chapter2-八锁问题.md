---
title: Chapter2 八锁问题
excerpt: 摘要
tags:
  - java
  - juc
categories:
  - Java笔记
  - JUC
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: a5933a56
date: 2020-11-19 23:45:43
updated: 2020-11-19 23:45:43
subtitle:
---
看了一段时间狂神感觉讲的不够清晰，然后又发现了周阳老师，感觉找到真爱了。。。
### 2.1 简介  
   八锁问题即关于锁的八个问题，主要考察`synchronized` 同步方法的锁的对象问题。
### 2.2 问题详解  
1. 主程序
   ```java
    public static void main(String[] args) {
        Number number = new Number();

        new Thread(()->{number.one();},"A").start();
        new Thread(()->{number.two();},"B").start();
    }
   ```
2. 八个问题
   1. 两个普通同步方法，两个线程，标准打印。
        ```java
        class Number{
            public synchronized void one() {System.out.println("One");}

            public synchronized void two() {System.out.println("Two");}
        }
        ```
        结果：  
        ```
        One
        Two
        ```
        两者是同一把锁（number对象），所以顺序输出。
   2. 新增 Thread.sleep() 给 one() 
        ```java
        class Number{
            public synchronized void one(){
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("One");
            }

            public synchronized void two(){
                System.out.println("Two");
            }
        }
        ```
        结果：  
        ```
        One
        Two
        ```
        两者是同一把锁（number对象），所以还是顺序输出。
   3. 新增普通方法 three() 
        ```java
        class Number{
            public synchronized void one(){
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("One");
            }

            public synchronized void two(){
                System.out.println("two");
            }

            public void three(){
                System.out.println("Three");
            }
        }
        ```
        结果：  
        ```
        Three
        One
        Two
        ```
        `Three`方法不是同步方法，所以先输出（时间短）
   4. 两个普通同步方法，两个 Number 对象  
    主程序
        ```java
        public static void main(String[] args) {
            Number number1 = new Number();
            Number number2 = new Number();
            new Thread(()->{number1.one();},"A").start();
            new Thread(()->{number2.two();},"B").start();
        }
        ```
        ```java
        class Number{
            public synchronized void one(){
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("One");
            }

            public synchronized void two(){
                System.out.println("two");
            }
        }
        ```
        结果
        ```java
        Two 
        One
        ```
        两把锁，线程B快，所以先打印Two
        
   5. 修改 two() 为静态同步方法
        ```java
        class Number{
            public synchronized void one(){
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("One");
            }

            public static synchronized void two(){
                System.out.println("two");
            }
        }
        ```
        结果
        ```java
        Two
        One
        ```
        静态同步方法的锁为类本身（`Number.class`），所以是两把锁
    6. 修改两个方法均为静态同步方法
        ```java
        class Number{
            public static synchronized void one(){
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("One");
            }

            public static synchronized void two(){
                System.out.println("two");
            }
        }
        ```
        结果：  
        ```java
        One
        Two
        ```
        两个方法都是静态同步方法，锁都是Number类本身，所以是同一把锁。
   7. 一个静态同步方法，一个非静态同步方法，两个 Number 对象   
        主程序
        ```java
        public static void main(String[] args) {
            Number number1 = new Number();
            Number number2 = new Number();
            new Thread(()->{number1.one();},"A").start();
            new Thread(()->{number2.two();},"B").start();
        }
        ```
        ```java
        class Number{
            public synchronized void one(){
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("One");
            }

            public static synchronized void two(){
                System.out.println("two");
            }
        }
        ```
        结果
        ```java
        Two
        One
        ```
   8. 两个静态同步方法，两个 Number 对象?  
        主程序
        ```java
        public static void main(String[] args) {
            Number number1 = new Number();
            Number number2 = new Number();
            new Thread(()->{number1.one();},"A").start();
            new Thread(()->{number2.two();},"B").start();
        }
        ```
        ```java
        class Number{
            public static synchronized void one(){
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("One");
            }

            public static synchronized void two(){
                System.out.println("two");
            }
        }
        ```
        结果
        ```java
        One
        Two
        ```
        两个方法都是静态同步方法，锁都是Number类本身，所以是同一把锁，与谁调用无关。