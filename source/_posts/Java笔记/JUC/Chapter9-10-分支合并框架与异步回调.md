---
title: Chapter9-10 分支合并框架与异步回调
excerpt: 分支合并框架ForkJoinTask、ForkJoinPool，CompletableFuture实现异步回调
tags:
  - java
  - juc
categories:
  - Java笔记
  - JUC
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 51900d2d
date: 2020-11-21 23:39:34
updated: 2020-11-21 23:39:34
subtitle:
---
## 9. 分支合并框架
### 9.1 架构
1. 主要由`ForkJoinPool`类实现：  
   `Executor`(父接口)->`ExecutorService`(子接口)->`AbstractExecutorService`(抽象类)->`ForkJoinPool`(实现类)
2. `ForkJoinTask` 抽象类
   * 实现类 `Future`、`Serializable`两个接口
   * 类似于`FutureTask`
3. `RecursiveTask`类：`ForkJoinTask`的抽象子类
4. 理解
   * `ForkJoinTask`需要通过`ForkJoinPool`来执行。
   * `ForkJoinTask`可以理解为类线程但比线程轻量的实体, 在`ForkJoinPool`中运行的少量`ForkJoinWorkerThread`可以持有大量的`ForkJoinTask`和它的子任务.
   * `ForkJoinTask`同时也是一个轻量的`Future`,使用时应避免较长阻塞和`io`.

### 9.2 计算案例
```java
public class ForkJoinDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinWork mywork = new ForkJoinWork(0,100);
        ForkJoinPool forkJoinPool = new ForkJoinPool(); //实现ForkJoin需要ForkJoinPool的支持
        ForkJoinTask<Integer> forkJoinTask = forkJoinPool.submit(mywork);

        System.out.println(forkJoinTask.get());
        forkJoinPool.shutdown();
    }
}

class ForkJoinWork extends RecursiveTask<Integer>{
    private int begin; //起始值
    private int end;   // 结束值
    private int result;
    private static final int ADJUST_VALUE = 10; //临界值

    public ForkJoinWork(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int length = end-begin; //判断长度
        if(length<ADJUST_VALUE){
            // 拆分完毕就计算
            for (int i = begin; i < end; i++) {
                result+=i;
            }
        }else{
            // 没有拆分完就拆分
            int middle = (begin+end)/2; // 计算中间值
            ForkJoinWork right = new ForkJoinWork(begin,middle);
            ForkJoinWork left = new ForkJoinWork(middle,end);
            // 拆分并压入线程
            right.fork();
            left.fork();

            result = right.join()+left.join();// 合并
        }

        return result;
    }
}
```

## 10. 异步回调
### 10.1 同步回调与异步回调
1. 同步回调  
   多线程中，同步回调是阻塞的，单个的线程需要等待结果的返回才能继续执行。
2. 异步回调  
   * 如果不希望程序在某个执行方法上一直阻塞，需要先执行后续的方法，那就是这里的异步回调。
   * 在调用一个方法时，传入一个回调的方法，当方法执行完时，让被调用者执行给定的回调方法，获得返回值

### 10.2 架构
1. 通过`CompletableFuture`类来完成，该类实现了`CompletionStage<T>`, `Future<T>`两个接口
2. `CompletableFuture`实现了`CompletionStage<T>`接口中的诸多方法：
    ```java
    public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn)
    public CompletableFuture<T> whenComplete(BiConsumer<? super T, ? super Throwable> action)
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
    ```
3. 代码示例
   ```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        /*// 同步调用
        CompletableFuture<Void> completableFuture1 = CompletableFuture.runAsync(()->{
            System.out.println(Thread.currentThread().getName()+"\t completableFuture1");
        });
        completableFuture1.get();*/

        // 异步回调
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+"\t completableFuture2");
            int i = 10/0; //异常点
            return 1024;
        });

        int result = completableFuture2.whenComplete((t,u)->{
            System.out.println("-------t "+t);
            System.out.println("-------u "+u);
        }).exceptionally(f->{
            System.out.println("--------exception:" + f.getMessage());
            return 444;
        }).get();
        System.out.println(result);
    }
   ```
    * `CompletableFuture.supplyAsync()`参数为一个`Runnable`，将异步方法放进去
    * `whenComplete()`方法参数为`BiConsumer`（双消费型接口），即两个参数，无返回值
    * t代表成功，u代表异常，即无异常则t为`supplyAsync()`的参数的结果
    * 若无异常，则t为1024，u为`null`，`get()`值为1024
    * 若有异常，则t为`null`，u为异常信息，`get()`值为444
