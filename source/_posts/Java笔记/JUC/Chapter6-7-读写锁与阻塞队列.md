---
title: Chapter6-7 读写锁与阻塞队列
excerpt: 读写锁ReadWriteLock、阻塞队列BlockingQueue
tags:
  - java
  - juc
categories:
  - Java笔记
  - JUC
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 2ada5ae
date: 2020-11-21 23:35:44
updated: 2020-11-21 23:35:44
subtitle:
---
## 6. 读写锁 `ReadWriteLock`
1. 说明
   * `ReadWriteLock` 接口位于 `java.util.concurrent.locks` 包中
   * `ReentrantReadWriteLock` 是其实现类
2. 接口方法
   * `Lock readLock()`：读锁，允许多个线程同时读取
   * `Lock writeLock()`：写锁，保证原子性
3. 代码示例
   ```java
    public class ReadWriteLockDemo {
        public static void main(String[] args) {
            MyCache myCache = new MyCache();
            for (int i = 1; i <= 5; i++) {
                final int tempInt = i;
                new Thread(()->{myCache.put(String.valueOf(tempInt),String.valueOf(tempInt));},String.valueOf(i)).start();
            }
            for (int i = 1; i <= 5; i++) {
                final int tempInt = i;
                new Thread(()->{myCache.get(String.valueOf(tempInt));},String.valueOf(i)).start();
            }
        }
    }

    class MyCache {
        private volatile Map<String,Object> map = new HashMap<>();
        private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

        public void put(String key, Object value) {
            readWriteLock.writeLock().lock();
            try {
                System.out.println(Thread.currentThread().getName()+"\t-----写入数据"+key);
                try {
                    TimeUnit.MILLISECONDS.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                map.put(key,value);
                System.out.println(Thread.currentThread().getName()+"\t-----写入完成");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                readWriteLock.writeLock().unlock();
            }
        }

        public void get(String key){
            readWriteLock.readLock().lock();
            try {
                System.out.println(Thread.currentThread().getName()+"\t 读取数据");
                try {
                    TimeUnit.MILLISECONDS.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Object result = map.get(key);
                System.out.println(Thread.currentThread().getName()+"\t 读取完成"+result);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                readWriteLock.readLock().unlock();
            }
        }
    }
   ```

## 7. 阻塞队列
### 7.1 概述
1. `BlockingQueue`接口
   * `Collection`的三个子接口：`List`、`Set`、`Queue`
   * `Queue`接口子接口：`BlockingQueue`，该子接口有七个实现类
2. 七个实现类：
   * `ArrayBlockingQueue`：数组实现的有界阻塞队列
   * `LinkedBlockingQueue`：由链表结构组成的有界队列
   * `PriorityBlockingQueue`：支持线程优先级排序的无界队列
   * `DelayQueue`：实现PriorityBlockingQueue实现延迟获取的无界队列
   * `SynchronousQueue`：不存储元素的阻塞队列，每一个put操作必须等待take操作
   * `LinkedTransferQueue`：个由链表结构组成的无界阻塞队列
   * `LinkedBlockingDeq ue	`：由链表结构组成的双向阻塞队

### 7.2  `BlockingQueue` 核心方法
1. 方法
    方法\处理方式|抛出异常|返回特殊值|一直阻塞|超时退出
    :-:|:-|:-:|:-:|:-:
    插入|add(e)|offer(e)|put(e)|offer(e,time,unit)
    移除|remove()|poll()|take()|poll(time,unit)
    检查|element()|peek()|-|-
    * 异常：是指当阻塞队列满时候，再往队列里插入元素，会抛出IllegalStateException("Queue full")异常。当队列为空时，从队列里获取元素时会抛出NoSuchElementException异常 
    * 返回特殊值：插入方法会返回是否成功，成功则返回true。移除方法，则是从队列里拿出一个元素，如果没有则返回null
    * 一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里take元素，队列也会阻塞消费者线程，直到队列可用。
    * 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出

2. 代码示例
   ```java
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);

        /*System.out.println(blockingQueue.add("a")); //true
        System.out.println(blockingQueue.add("b"));//true
        System.out.println(blockingQueue.add("c"));//true
    //        System.out.println(blockingQueue.add("d"));//异常IllegalStateException("Queue full")
        */
        /*System.out.println(blockingQueue.remove()); //先进先出，返回a
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
    //        System.out.println(blockingQueue.remove());// 异常NoSuchElementException
        */
        /*System.out.println(blockingQueue.offer("a")); //true
        System.out.println(blockingQueue.offer("b")); //true
        System.out.println(blockingQueue.offer("c")); //true
        System.out.println(blockingQueue.offer("d")); //false
        */
        /*System.out.println(blockingQueue.poll()); //a
        System.out.println(blockingQueue.poll()); //b
        System.out.println(blockingQueue.poll()); //c
        System.out.println(blockingQueue.poll()); // null
        */
        /*blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
    //        blockingQueue.put("d"); //阻塞
        */
        /*System.out.println(blockingQueue.take()); //a
        System.out.println(blockingQueue.take()); //b
        System.out.println(blockingQueue.take()); //c
    //        System.out.println(blockingQueue.take()); //阻塞
        */
        System.out.println(blockingQueue.offer("a")); //true
        System.out.println(blockingQueue.offer("b")); //true
        System.out.println(blockingQueue.offer("c")); //true
        blockingQueue.offer("d",3L, TimeUnit.SECONDS);//等待三秒后退出
    }
   ```