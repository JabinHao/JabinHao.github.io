---
title: Java中的锁
excerpt: synchronized锁的底层原理，锁优化，乐观锁与悲观锁
tags:
  - jvm
categories:
  - Java笔记
  - JVM
banner_img: /img/banner/limbo.jpg
index_img: /img/index/pacman.png
category: Java笔记/JVM
abbrlink: 8c3bebad
date: 2021-09-04 23:33:15
updated: 2021-09-05 21:05:36
subtitle:
---

### 2.1 说明
1. synchronized 特点
    * 非公平锁
    * 可重入锁
    * 互斥锁

2. 对象头中的信息
    * 是否为偏向锁：1bit
    * 锁标识：2bit

3. 64位虚拟机对象头

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/20210902031945.png)
    * 指针指向的是monitor对象，也称为管程或监视器锁
    * 每个对象都存在着一个monitor与之关联
  
4. 在Java虚拟机(HotSpot)中，monitor由ObjetMonitor实现(c++)：
   
    ```cpp
    ObjectMonitor() {
        _header       = NULL;
        _count        = 0;
        _waiters      = 0,       // 等待中的线程数
        _recursions   = 0;       // 线程重入次数
        _object       = NULL;    // 存储该 monitor 的对象
        _owner        = NULL;    // 指向拥有该 monitor 的线程
        _WaitSet      = NULL;    // 等待线程 双向循环链表_WaitSet 指向第一个节点
        _WaitSetLock  = 0 ;
        _Responsible  = NULL ;
        _succ         = NULL ;
        _cxq          = NULL ;   // 多线程竞争锁时的单向链表
        FreeNext      = NULL ;
        _EntryList    = NULL ;   // _owner 从该双向循环链表中唤醒线程，
        _SpinFreq     = 0 ;
        _SpinClock    = 0 ;
        OwnerIsThread = 0 ;
        _previous_owner_tid = 0; // 前一个拥有此监视器的线程 ID
    }
    ```
 
5. synchronized 锁状态
    * 无锁
    * 偏向锁
    * 轻量级锁
    * 重量级锁

### 2.2 底层原理

1. 同步代码块
    * 显示指令：monitorenter和monitorexit
2. 同步方法
    * 隐式，即无需通过字节码指令来控制的，它实现在方法调用和返回操作之中。
    * JVM通过方法常量池中的方法表结构中的 `ACC_SYNCHRONIZED` 访问标志区分一个方法是否同步方法
3. 可重入：_recursions记录次数

### 2.3 JDK1.6 对锁的优化

1. 效率问题
    * 监视器锁（monitor）是依赖于底层的操作系统的Mutex Lock来实现的，而操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，早期的synchronized效率低
    * JDK1.6优化
        * 锁粗化(Lock Coarsening)
        * 锁消除(Lock Elimination)
        * 轻量级锁(Lightweight Locking)
        * 偏向锁(Biased Locking)
        * 适应性自旋(Adaptive Spinning)
2. 几个概念
    * 锁粗化(Lock Coarsening)：也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。 
    * 锁消除(Lock Elimination)：通过运行时JIT编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本地Stack上进行对象空间的分配(hotpot采用标量替换)，还可以减少Heap上的垃圾收集开销。

3. 自旋锁
   * 是一种获取锁的机制，轻量级锁升级重量级锁过程时采用了此机制
   * 通过自旋（即循环）避免线程环循的开销
   * 自选次数是固定的，默认为10，可通过 `-XX:PreBlockSpin` 设置

4. 自适应自旋锁
   * JDK1.6引入
   * 线程的自旋时间不在固定，而是根据上一个持有该锁的线程的自旋时间以及状态来确定

### 2.4 四种状态
1. 偏向锁
   * 加锁
        * 检查 mark word 的线程 id 。
        * 如果为空则设置 CAS 替换当前线程 id。如果替换成功则获取锁成功，如果失败则撤销偏向锁。
        * 如果不为空则检查 线程 id为是否为本线程。如果是则获取锁成功，如果失败则撤销偏向锁。
   * 撤销：需要等待全局安全点
        * 暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态
        * 撤销偏向锁恢复到无锁（标志位为 01）或轻量级锁（标志位为 00）的状态
        * 唤醒线程

2. 轻量级锁：多个线程竞争偏向锁导致偏向锁升级为轻量级锁
   * 加锁
        * JVM 在当前线程的栈帧中创建 Lock Reocrd，并将对象头中的 Mark Word 复制到 Lock Reocrd 中。（Displaced Mark Word）
        * 线程尝试使用 CAS 将对象头中的 Mark Word 替换为指向 Lock Reocrd 的指针。如果成功则获得锁，锁标志位变为00。如果失败则先检查对象的 Mark Word 是否指向当前线程的栈帧如果是则说明已经获取锁，否则说明其它线程竞争锁则膨胀为重量级锁
    * 撤销
        * 使用 CAS 操作将 Mark Word 还原
        * 如果第 1 步执行成功则释放完成
        * 如果第 1 步执行失败则膨胀为重量级锁

3. 重量级锁


4. 对比

锁 | 优点 | 缺点 | 使用场景 
:-|:-|:-|:-
偏向锁 | 加锁和解锁不需要CAS操作，没有额外的性能消耗，和执行非同步方法相比仅存在纳秒级的差距 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗 | 适用于只有一个线程访问同步快的场景 
轻量级锁 | 竞争的线程不会阻塞，提高了响应速度 | 如线程成始终得不到锁竞争的线程，使用自旋会消耗CPU性能 | 追求响应时间，同步快执行速度非常快 
重量级锁 | 线程竞争不适用自旋，不会消耗CPU | 线程阻塞，响应时间缓慢，在多线程下，频繁的获取释放锁，会带来巨大的性能消耗 | 追求吞吐量，同步快执行速度较长

### 2.5 乐观锁与悲观锁

1. 概念  
   乐观锁与悲观锁是一种广义上的概念，体现了看待线程同步的不同角度。在Java和数据库中都有此概念对应的实际应用。
2. 悲观锁
    * 对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改
    * Java中，synchronized关键字和Lock的实现类都是悲观锁。
3. 乐观锁
    * 乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。
    * 乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法，Java原子类中的递增操作就通过CAS自旋实现的。


### 2.6 线程中断
1. 退出方法
   * 设置退出标志，使线程正常退出，也就是当run()方法完成后线程终止
   * 使用interrupt()方法中断线程
   * 使用stop方法强行终止线程（已过时，不推荐使用）

2. `interrupt()` 相关方法

    ```java
    //中断线程（实例方法）
    public void Thread.interrupt();

    //判断线程是否被中断（实例方法）
    public boolean Thread.isInterrupted();

    //判断是否被中断并清除当前中断状态（静态方法）
    public static boolean Thread.interrupted()
    ```

3. 过程
   * 线程调用该方法后并不会立刻打断一个正在运行的线程，而是修改这个线程的中断状态码（interrupt status）
   * 当线程被sleep(),wait(),join()阻塞时，会抛出 `InterruptedException` 异常打断线程,抛出之后,该线程的打断标志复原

