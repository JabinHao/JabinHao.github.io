---
title: Chapter1 生产者消费者问题
excerpt: 摘要
tags:
  - java
  - juc
categories:
  - Java笔记
  - JUC
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: d0e8af76
date: 2020-11-19 23:43:53
updated: 2020-11-19 23:43:53
subtitle:
---
# JUC(Package java.util.concurrent包)
这部分看的是[狂神](https://www.bilibili.com/video/BV14W411u7gB?p=5)的视频，讲的很详细，[尚硅谷](https://www.bilibili.com/video/BV14W411u7gB)那个音质太差了耳朵受不了，不过老师的东北口音很带感
## 1. Lock锁
### 1.1 Synchronized 和 Lock区别

比较|`Synchronized`|`Lock`
:-:|:-|:-
属性|内置关键字|Java类
判断是否获取了锁|不能|可以
释放|自动|手动（不释放会死锁）
阻塞后|等待|不一定
其他|可重入锁，不可以中断，非公平|可重入锁，可以判断，非公平（可自行设置）
使用场景|少量代码同步|大量代码同步

### 1.2 生产者消费者问题
1. 传统synchronized方式
   
    ```java
    public class ProducerConsumerTest {
        public static void main(String[] args) {
            Data data = new Data();

            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"A").start();
            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"B").start();
            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"C").start();
            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"D").start();
        }
    }

    class Data{
        private int number = 0;

        // 生产者
        public synchronized void increment() throws InterruptedException {
            while (number!=0) {
                // 等待
                this.wait();
            }
            number++;
            System.out.println(Thread.currentThread().getName()+"-->"+number);
            this.notifyAll();
        }

        // 消费者
        public synchronized void decrement() throws InterruptedException {
            while (number == 0){
                // 等待
                this.wait();
            }
            number--;
            System.out.println(Thread.currentThread().getName()+"-->"+number);
            this.notifyAll();
        }

    }
    ```
    **注意虚假唤醒问题：**
  * 若`increment()`与`decrement()`函数中使用的不是 `while` 而是 `if` ，则容易出现虚假唤醒问题
  * `main` 函数中只开启两个线程不会有问题，开启多个则会出现
  * 例如：消费者消费结束，多个生产者线程同时被唤醒（`notifyAll()`），但只有一个拿到锁，而 `if` 只判断一次，其他线程也会继续执行`number++`操作，则会出现问题
2. `Lock` 方式
    ```java
    public class LockTest {
        public static void main(String[] args) {
            Data2 data = new Data2();

            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"A").start();
            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"B").start();
            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"C").start();
            new Thread( ()->{
                try {
                    for (int i = 0; i < 10; i++) data.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },"D").start();
        }
    }

    class Data2{
        private int number = 0;
        private Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        // 生产者
        public void increment() throws InterruptedException {
            lock.lock();
            try {
                while (number!=0) {
                    //
                    condition.await();
                }
                number++;
                System.out.println(Thread.currentThread().getName()+"-->"+number);
                condition.signalAll();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }

        // 消费者
        public synchronized void decrement() throws InterruptedException {

            try {
                while (number == 0){
                    //
                    condition.await();
                }
                number--;
                System.out.println(Thread.currentThread().getName()+"-->"+number);
                condition.signalAll();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }

    }
    ```

3. `Lock` 锁顺序唤醒

    ```java
    public class OrderLockTest {
        public static void main(String[] args) {
            Data3 data = new Data3();
            new Thread(()->{
                for (int i = 0; i < 10; i++) {
                    data.printA();
                }
            },"A").start();
            new Thread(()->{
                for (int i = 0; i < 10; i++) {
                    data.printB();
                }
            },"B").start();
            new Thread(()->{
                for (int i = 0; i < 10; i++) {
                    data.printC();
                }
            },"C").start();
        }
    }

    class Data3{
        private int number = 1;
        private Lock lock = new ReentrantLock();
        Condition condition1 = lock.newCondition();
        Condition condition2 = lock.newCondition();
        Condition condition3 = lock.newCondition();

        public void printA(){
            lock.lock();
            try {
                while (number != 1){
                    condition1.await();
                }
                number = 2;
                System.out.println(Thread.currentThread().getName()+" ---> AAAAAAAA");
                condition2.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }

        public void printB(){
            lock.lock();
            try {
                while (number != 2){
                    condition2.await();
                }
                System.out.println(Thread.currentThread().getName()+" ---> BBBBBBBB");
                number = 3;
                condition3.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }

        public void printC(){
            lock.lock();
            try {
                while (number != 3){
                    condition3.await();
                }
                number = 1;
                System.out.println(Thread.currentThread().getName()+" ---> CCCCCCCC");
                condition1.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
    ```
