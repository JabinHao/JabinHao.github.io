---
title: Chapter4 Callable的使用
excerpt: 摘要
tags:
  - java
  - juc
categories:
  - Java笔记
  - JUC
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: a338db42
date: 2020-11-20 21:15:27
updated: 2020-11-20 21:15:27
subtitle:
---
### 4.1 与Runnable区别
1. 方法不同，一个是`run`，一个是`call`
2. `call`有返回值，`run`没有
3. `call`抛异常，`run`没有

### 4.2 `Callable`的使用
1. `Runnable`->`RunnableFuture`->`FutureTask`(实现类)
2. `FutureTask`有构造方法：`public FutureTask(Callable<V> callable)`
3. 示例
    ```java
    public class CallableDemo {
        public static void main(String[] args) throws ExecutionException, InterruptedException {
            FutureTask futureTask = new FutureTask(new MyThread());
            new Thread(futureTask,"A").start();
            System.out.println(futureTask.get());
    //        new Thread(new FutureTask<Integer>(new MyThread())).start();
        }
    }

    class MyThread implements Callable<Integer>{

        @Override
        public Integer call() throws Exception {
            System.out.println("******************");
            return 1024;
        }
    }
    ```
4. `get()`方法的使用
    ```java
    public class CallableDemo {
        public static void main(String[] args) throws ExecutionException, InterruptedException {
            FutureTask futureTask = new FutureTask(new MyThread());
            new Thread(futureTask,"A").start();
            System.out.println(Thread.currentThread().getName()+"线程完成");
            System.out.println(futureTask.get());
    //        new Thread(new FutureTask<Integer>(new MyThread())).start();
        }
    }

    class MyThread implements Callable<Integer>{

        @Override
        public Integer call() throws Exception {
            System.out.println("******************");
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 1024;
        }
    }
    ```
    输出结果：
    ```java
    ******************
    main  main线程完成
    1024
    ```
    `get`方法要在后面调用，因为一旦调用，程序会一直等待到该线程完成
5. 多个分线程
    ```java
    public class CallableDemo {
        public static void main(String[] args) throws ExecutionException, InterruptedException {
            FutureTask futureTask = new FutureTask(new MyThread());
            new Thread(futureTask,"A").start();
            new Thread(futureTask,"B").start();
            System.out.println(Thread.currentThread().getName()+"线程完成");
            System.out.println(futureTask.get());
    //        new Thread(new FutureTask<Integer>(new MyThread())).start();
        }
    }

    class MyThread implements Callable<Integer>{

        @Override
        public Integer call() throws Exception {
            System.out.println(Thread.currentThread().getName());
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 1024;
        }
    }
    ```
    结果：
    ```
    A
    main线程完成
    1024
    ```