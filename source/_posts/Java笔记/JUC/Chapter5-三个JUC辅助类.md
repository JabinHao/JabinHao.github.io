---
title: Chapter5 三个JUC辅助类
excerpt: 摘要
tags:
  - java
  - juc
categories:
  - Java笔记
  - JUC
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 8484ed1d
date: 2020-11-20 21:15:46
updated: 2020-11-20 21:15:46
subtitle:
---
### 5.1 `CountDownLatch`类
1. 说明  
    * jdk1.5中引入的，位于java.util.cucurrent包下的工具类
    * 使一个线程等待其他线程各自执行完毕后再执行。
    * 类似于做减法，最后一个人走后再关灯（关灯人独自等待）
2. 常用方法
    * `public CountDownLatch(int count)`：构造器，count为要执行的线程数（计数器）
    * `public void countDown()`：将count值减1
    * `public void await()`：调用该方法的线程会被挂起，直到count值为0才继续执行
    * `public boolean await(long timeout, TimeUnit unit)`：当前线程等待，一定的时间后count值还没变为0则继续执行
3. 代码示例
    ```java
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i < 7; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName());
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName());
    }
    ```

### 5.2 `CyclicBarrier` 类
1. 说明
    * 线程被阻塞，当所有线程都完成后再被集体唤醒
    * 类似于人齐了再开会（先来的一起等）
2. 构造器
    * `public CyclicBarrier(int parties)`：`parties` 是参与线程的个数
    * `public CyclicBarrier(int parties, Runnable barrierAction)`：第二个参数是最后一个到达线程要做的任务（使用`lambda`函数）
3. 常用方法
    * `public int await()`：表示当前线程已到达栅栏
    * `public int await(long timeout, TimeUnit unit)`：指定等待的时间，即使未全部到达栅栏，到指定的时间，所在的线程就继续执行
3. 代码示例
    ```java
        public static void main(String[] args) {
            CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
                System.out.println("******* 召唤神龙");
            });

            for (int i = 1; i < 8; i++) {
                int finalI = i;
                new Thread(()->{
                    System.out.println(Thread.currentThread().getName()+" 收集到第 "+ finalI +" 颗龙珠");
                    try {
                        cyclicBarrier.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } catch (BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                },String.valueOf(i)).start();

            }
        }
    ```

### 5.3 `Semaphore`类
1. 说明
    * 信号量， 可以用来控制同时访问特定资源的线程数量
    * 相当于多辆车抢有限的停车位
2. 方法
    * `public Semaphore(int permits)`：构造器，参数为令牌数
    * `public void acquire()`：获取一个令牌，抢到则执行，否则阻塞等待其他线程释放令牌。
    * `public void acquire(int permits)`：获取指定数量的令牌，...
    * `public void release()`：释放一个令牌，唤醒一个获取令牌不成功的阻塞线程
3. 代码示例
    ```java
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);

        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"\t抢到了车位");
                    try {
                        TimeUnit.SECONDS.sleep(2);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                finally {
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
    ```