---
title: Chapter8 线程池
excerpt: Executors工具类、ThreadPoolExecutor底层原理、如何使用ThreadPoolExecutor创建线程池
tags:
  - java
  - juc
categories:
  - Java笔记
  - JUC
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: fe7f18c4
date: 2020-11-21 23:37:00
updated: 2020-11-21 23:37:00
subtitle:
---
### 8.1 概述
1. 线程池的优势  
   线程池做的工作只要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。
2. 特点  
    线程复用; 控制最大并发数; 管理线程
    * 降低资源消耗
    * 提高响应速度
    * 提高线程的可管理性

### 8.2 线程池的使用
1. 架构  
   Java中的线程池是通过Executor框架实现的，该框架中用到了Executor，Executors，ExecutorService，ThreadPoolExecutor这几个类
   * 父接口 `Executor`
   * 子接口 `ExecutorService`、` ScheduledExecutorService`
   *  `ExecutorService`接口的实现类：`ThreadPoolExecutor`
   *  工具类 `Executors`
2. `Executors`主要方法
   * `public static ExecutorService newFixedThreadPool(int nThreads) `：创建指定数量的线程
   * `public static ExecutorService newSingleThreadExecutor()`：创建单个线程
   * `public static ExecutorService newCachedThreadPool() `：动态创建线程（动态调整线程数）
3. 代码示例
   ```java
    public class Test1 {
        public static void main(String[] args) {
    //        ExecutorService threadPool = Executors.newFixedThreadPool(3); //三个窗口
    //        ExecutorService threadPool = Executors.newSingleThreadExecutor(); //单个窗口
            ExecutorService threadPool = Executors.newCachedThreadPool(); //三个窗口

            try {
                for (int i = 0; i < 10; i++) {
                    threadPool.execute(()->{
                        System.out.println(Thread.currentThread().getName()+"\t办理业务");
                    });
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                threadPool.shutdown();
            }
        }
    }
   ```
4. `ThreadPoolExecutor` 底层原理  
   三个API源码
   ```java
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
   ```
   ```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
   ```
   ```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
   ```
5. 三个方法的问题 
   * `FixedThreadPool` 和 `SingleThreadPool`:  
        允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
   * `CachedThreadPool` 和 `ScheduledThreadPool`:  
        允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

### 8.3 线程池的参数
1. `ThreadPoolExecutor` 构造器源码
    ```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
    ```

2. 参数
   1. `corePoolSize`：线程池中的常驻核心线程数
   2. `maximumPoolSize`：线程池中能够容纳同时执行的最大线程数，此值必须大于等于1
   3. `keepAliveTime`：多余的空闲线程的存活时间当前池中线程数量超过corePoolSize时，当空闲时间达到keepAliveTime时，多余线程会被销毁直到只剩下corePoolSize个线程为止
   4. `unit`：keepAliveTime的单位
   5. `workQueue`：任务队列，被提交但尚未被执行的任务
   6. `threadFactory`：表示生成线程池中工作线程的线程工厂，用于创建线程，**一般默认的即可**
   7. `handler`：拒绝策略，表示当队列满了，并且工作线程大于等于线程池的最大线程数（maximumPoolSize）时如何来拒绝请求执行的runnable的策略

### 8.4 线程池底层工作原理
1. 在创建了线程池后，开始等待请求。
2. 当调用`execute()`方法添加一个请求任务时，线程池会做出如下判断：
   1. 如果正在运行的线程数量小于`corePoolSize`，那么马上创建线程运行这个任务；
   2. 如果正在运行的线程数量大于或等于`corePoolSize`，那么将这个任务放入队列；
   3. 如果这个时候队列满了且正在运行的线程数量还小于`maximumPoolSize`，那么还是要创建非核心线程立刻运行这个任务；
   4. 如果队列满了且正在运行的线程数量大于或等于`maximumPoolSize`，那么线程池会启动饱和拒绝策略来执行。
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
4. 当一个线程无事可做超过一定的时间（`keepAliveTime`）时，线程会判断：
    * 如果当前运行的线程数大于`corePoolSize`，那么这个线程就被停掉。
    * 所以线程池的所有任务完成后，它最终会收缩到`corePoolSize`的大小。
 
### 8.5 使用线程池
1. 线程池的拒绝策略(4个)
   * `AbortPolicy()`：默认，直接抛出RejectedExecutionException异常阻止系统正常运行
   * `CallerRunsPolicy()`：“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量。
   * `DiscardOldestPolicy()`：抛弃队列中等待最久的任务，然后把当前任务加人队列中尝试再次提交当前任务。
   * `DiscardPolicy()`：该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种策略。
   * 以上内置拒绝策略均实现了`RejectedExecutionHandle`接口
2. 示例
   ```java
    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(2,5,2L, TimeUnit.SECONDS, new LinkedBlockingDeque<>(3),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());

        try {
            for (int i = 0; i < 10; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"\t办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
   ```
    * 这里使用了默认的 `AbortPolicy()` 策略，总任务数为10，而允许的最大任务数 = `maximumPoolSize`(5) + `workQueue`(3) = 8，所以会抛出RejectedExecutionException异常
    * 若使用`CallerRunsPolicy()` 策略，则会执行8个任务，将剩下的两个退回给main线程（也有可能处理较快，处理了9个，main只处理了一个）
    * 若使用`DiscardOldestPolicy()`策略，则会抛弃两个时间最长的
    * 若使用`DiscardPolicy()`策略，则会抛弃两个
3. 在工作中如何使用线程池，是否自定义过线程池
   * 获取电脑核数：`Runtime.getRuntime().availableProcessors()`
   * CPU密集型任务：最大线程数(maximumPoolSize)=CPU核心数+1
   * IO密集型任务：CPU核心数/（1-阻塞系数）（参考《Java 虚拟机并发编程》2.1节 12页）、





