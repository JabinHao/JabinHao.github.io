---
title: Chapter5 Redis持久化
excerpt: 两种持久化策略：RDB 与 AOF
tags:
  - redis
categories:
  - Redis
banner_img: /img/post/banner/kangni.png
index_img: /img/post/redis_logo.png
category: Redis
abbrlink: '28927665'
date: 2021-02-03 21:58:56
updated: 2021-02-03 23:21:04
subtitle:
---
## 5.1 RDB

### 5.1.1 简介

1. redis 是内存数据库，它把数据存储在内存中，这样在加快读取速度的同时也对数据安全性产生了新的问题，即当 redis 所在服务器发生宕机后， redis 数据库里的所有数据将会全部丢失。为了解决这个问题， redis 提供了持久化功能——RDB 和 AOF（Append Only File） 

2. RDB（Redis DataBase） 是 Redis 默认的持久化方案。在指定的时间间隔内，执行指定次数的写操作，则会将内存中的数据写入到磁盘中。即在指定目录下生成一个 dump.rdb 文件。 Redis 重启会通过加载 dump.rdb 文件来恢复数据

### 5.1.2 RDB 原理

1. Redis会复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程， 来进行持久化
2. 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。
3. 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。
4. RDB的缺点是最后一次持久化后的数据可能丢失。

### 5.1.3 RDB保存文件

1. RDB保存的文件是dump.rdb文件 ,位置保存在Redis的启动目录。
2. Redis每次同步数据到磁盘都会生成一个dump.rdb文件，新的 dump.rdb 会覆盖旧的 dump.rdb 文件
3. 两种自动保存方式
   * 操作频率触发持久化策略
   * 通过 shutdown 关闭服务器
4. 手动保存
   * save命令
   *  save 指令会阻塞所有客户端


### 5.1.4 备份与恢复

1. 通过脚本将 Redis 产生的 dump.rdb 文件备份(cp dump.rdb dump_bak.rdb)
2. 每次启动 Redis 前，把备份的 dump.rdb文件替换到 Redis 相应的目录

## 5.2 AOF

### 5.2.1 简介

1. RDB 缺点
   * 虽然在fork时使用了写时复制技术，但数据量大时还是比较消耗性能
   * 周期性备份，容易丢失最后的数据

2. AOF
    * AOF(Append Only File)， Redis 默认不开启。它的出现是为了弥补RDB的不足（数据的不一致性）
    * AOF采用日志的形式来记录每个写操作，并追加到文件中。 
    * Redis 重启会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

### 5.2.2 AOF 原理

1. Redis以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)
2. 只许追加文件但不可以改写文件， redis启动之初会读取该文件重新构建数据

### 5.2.3 AOF 保存的文件

1. AOF 保存的文件是 appendonly.aof 文件，保存目录与 dump.rdb 相同
2. 开启 AOF 后，Redis 每次记录写操作都会往 appendonly.aof 文件追加新的日志内容

### 5.2.4 AOF 数据恢复

1. AOF 与 RDB 同时开启，系统默认取 AOF 的数据
2. 若 AOF 文件损坏， 可以通过命令redis-check-aof --fix appendonly.aof 进行修复，会把出现异常的部分往后所有写操作日志去掉

### 5.2.5 AOF 的重写

1. AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制
2. 当AOF文件的大小超过所设定的阈值时， Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集
3. AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据
4. 重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似
5. Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于65M时触发，也可以在配置文件中进行配置（`auto-aof-rewrite-min-size`）


## 5.3 总结

### 5.3.1 AOF 优点
1. 备份机制更稳健，丢失数据概率更低
2. 可读的日志文本，通过操作AOF，可以处理误操作

### 5.3.2 AOF 缺点

1. 比起 RDB 占用了更多的磁盘空间
2. 恢复备份速度慢
3. 每次读写都同步时，有一定性能压力
4. 存在个别 bug，造成无法恢复

### 5.3.3 两者选择

1. 关于 Redis 持久化的使用： 若只打算用 Redis 做缓存，可以关闭持久化
2. 若打算使用 Redis 的持久化， 建议 RDB 和 AOF 都开启
