---
title: Chapter7 多线程
excerpt: 多线程的四种创建方法，解决线程安全问题的三种方法，线程的通信
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: e4d260dd
date: 2020-10-24 20:38:47
updated: 2020-10-24 20:38:47
subtitle:
---
## 7.1 基本概念
### 1. 线程与进程
1. 进程：程序的一次执行过程，或是正在运行的一个程序。是一个动态的过程：有它自身的产生、存在和消亡的过程。 ——生命周期
2. 线程：进程可进一步细化为线程，是一个程序内部的一条执行路径

### 2. 并行与并发
1. 并行  
   多个CPU同时执行多个任务
2. 并发  
   一个CPU(采用时间片)同时执行多个任务

### 3. 多线程优点
* 提高应用程序的响应。对图形化界面更有意义，可增强用户体验。
* 提高计算机系统CPU的利用率
* 改善程序结构。将既长又复杂的进程分为多个线程，独立运行，利于理解和修改

## 7.2 线程的创建与使用
`java.lang.Thread` 类
### 1. 多线程的创建——方法一
1. 步骤
   1. 创建 `Thread` 类的子类
   2. 重写 `run()`方法，将此线程操作声明在`run()`方法中
   3. 实例化该子类
   4. 通过对象调用 `start()`方法
2. 注意事项
   1. 多个线程则创建多个对象(若内容不同则需创建多个子类)
   2. 可以使用匿名子类的方式:
        ```java
        new Thread(){
            @overwrite
            run()
            {
                //方法体
            }
        }.start()
        ```
3. `Thread` 常用方法
    1. `currentThread()`：静态方法，返回执行当前代码的线程
    2. `getName()`：获取当前进程名字
    3. `setName()`：设置当前进程名字
    4. `yield()`：主动释放当前CPU执行（有时可能下一刻可能又分配到CPU，看上去没有效果）
    5. `join()`：在当前线程中调用其他线程该方法，则该线程进入阻塞状态，知道另一个线程执行完结束
    6. `stop()`：强制结束当前线程（已过时）
    7. `sleep(long millitime)`：让当前线程睡眠指定毫秒数（在run中使用）
    8. `isalive()`：判断当前线程是否存活

4. 线程优先级
   1. 相关常量
      ```java
      MAX_PRIORITY: 10
      MIN_PRIORITY: 1
      NORM_PRIORITY: 5
      ```
   2. 如何设置
      * `getPriority()`：获取线程优先级
      * `setPriority(int p)`：设置线程优先级
   3. 说明：  
      高优先级线程先执行的概率大，但不一定

### 2. 多线程的创建——方法二
1. 步骤
   1. 创建实现了 `Runnable` 接口的类
   2. 实现类中实现 `run()` 方法
   3. 实例化实现类
   4. 将对象作为参数创建 `Thread` 类的对象
   5. 通过Thread类的对象调用 `start` 方法
2. 说明
   1. 可以用同一个实现类对象创建多个Thread对象

### 3. 两种方式对比
1. 实际开发中  
   优先选择方式二（实现`Runnable`接口的方式）
2. 不同点
   1. 实现Runnable接口的方式（方式二）没有单继承的局限性
   2. 方式二更适合多线程共享数据（同一个对象创建线程）
3. 相同点  
   两种方式都要重写`run()`方法，将要执行的代码声明在该方法中

## 7.3 线程的生命周期
如图：  
![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/lifetime.png)

## 7.4 线程的同步
通过同步机制解决线程安全问题  
### 1. 同步代码块
1. 代码示例
   ```java
   synchronized(同步监视器){
      //需要被同步的代码
   }
   ```
2. 说明
   * 操作共享数据的代码即为需要被同步的代码
   * 共享数据：多个线程共同操作的变量
   * 同步监视器：俗称锁，任何类的对象都可以充当锁，**多个线程必需共用同一把锁**。
   * 对于实现Runnable接口的多线程，可以使用当前对象作为锁(`this`)
   * 对于继承Thread的方式，则需创建静态对象，或使用`当前类名.class`（类也是对象）
3. 特点
   * 解决了线程安全问题
   * 执行同步操作时，只有一个线程操作，其他在等待，相当于单线程

### 2. 同步方法
synchronized还可以放在方法声明中，表示整个方法为同步方法
1. 继承Thread方式
   ```java
   public synchronized void show (String name){
      //
   }
   ```
   此时同步监视器为当前类`this`  

2. 实现接口方式
   ```java
   private static synchronized void show(){//同步监视器：当前类.class
      //
   }
   ```
   方法为静态方法，此时同步监视器为当前类本身  
   
### 线程死锁问题
1. 死锁
   *  不同的线程分别占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁
   *  出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续
2. 解决方法
   * 专门的算法、原则
   *  尽量减少同步资源的定义
   *  尽量避免嵌套同步

### 3. Lock(锁)
1. 步骤
   1. 实例化ReentrantLock
   2. 调用lock()方法，手动启动同步
   3. 调用unlock()方法，结束同步
2. 代码示例
   ```java
   private ReenTrantLock lock= new ReenTrantLock()
   //在run中调用
   ···
   lock.lock;
   // 同步内容
   lock.unlock;
   ```
3. 注意事项
   1. 对于继承方式，lock要声明为静态
   2. 使用中：Lock>同步代码块>同步方法

## 7.5 线程的通信
1. 涉及到的方法
   1. `wait()`: 当前线程进入阻塞状态并释放同步监视器
   2. `notify()`：唤醒被wait的线程，如果有多个则唤醒优先级高的
   3. `notifyAll()`：唤醒所有被wait的线程
2. 说明
   1. 上述三个方法必须使用中同步代码块或同步方法中
   2. 三个方法的调用者，必须是同步监视器
   3. 三个方法定义在Object类中
3. `sleep()`与`wait()`
   1. 都会使当前线程进入阻塞状态
   2. 声明位置不同，`sleep`声明在`Thread`类中
   3. 调用位置不同，`sleep`可以在任何需要的地方调用，`wait`必须使用中同步代码块或同步方法中
   4. 都用在同步代码块或方法中时，`sleep`不会释放监视器(锁)，`wait`会释放

## 7.6 JDK5.0 新增线程创建方式
### 1. 实现`Callable`接口
1. 步骤
   1. 创建`Callable`接口的实现类
   2. 实现 `call`方法，将此线程需要的操作声明在其中
   3.  创建实现类的对象
   4.  将对象作为参数传入FutureTask构造器中，创建`FutureTask`类的对象
   5.  将`FutureTask`类的对象作为参数创建Thread类的对象，并调用`start`方法
   6.  `FutureTask对象.get()`方法可以获取call方法的返回值
2. 说明
   1. call方法有返回值
   2. call方法可以抛出异常，并被外面的操作捕获
   3. Callable支持泛型
3. 代码示例
   ```java
   public class ThreadNew {
       public static void main(String[] args) {
           NumThread numThread = new NumThread();

           FutureTask futureTask =  new FutureTask(numThread);

           new Thread(futureTask).start();

           try{
               Object sum = futureTask.get();
               System.out.println(sum);
           }catch (InterruptedException e){
               e.printStackTrace();
           }catch (ExecutionException e){
               e.printStackTrace();
           }
       }

   }
   // 自定义实现类
   class NumThread implements Callable{
       @Override
       public Object call() throws Exception {
           // 线程内容，可以有返回值
       }
   }
   ```

### 2. 使用线程池
1. 使用
   ```java
   public class ThreadPool {

       public static void main(String[] args) {
           //1. 提供指定线程数量的线程池
           ExecutorService service = Executors.newFixedThreadPool(10);
           ThreadPoolExecutor service1 = (ThreadPoolExecutor) service;
           //设置线程池的属性
   //        ExecutorService 是接口，ThreadPoolExecutor是其实现类，service为其对象
   //        service1.setCorePoolSize(15);
   //        service1.setKeepAliveTime();


           //2.执行指定的线程的操作。需要提供实现Runnable接口或Callable接口实现类的对象
           service.execute(new NumberThread());//适用于Runnable
           service.execute(new NumberThread1());//适用于Runnable

   //        service.submit(Callable callable);//适用于Callable
           //3.关闭连接池
           service.shutdown();
       }
   }

   class NumberThread implements Runnable{

       @Override
       public void run() {
           // 线程内容
       }
   }

   class NumberThread1 implements Runnable{

       @Override
       public void run() {
           // 线程内容
       }
   }
   ```
2. 优点
   1. 提高响应速度（减少了创建新线程的时间）
   2. 降低资源消耗（重复利用线程池中线程，不需要每次都创建）
   3. 便于线程管理
      * corePoolSize：核心池的大小
      * maximumPoolSize：最大线程数
      * keepAliveTime：线程没有任务时最多保持多长时间后会终止


