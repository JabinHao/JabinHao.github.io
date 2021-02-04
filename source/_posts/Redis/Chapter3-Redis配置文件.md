---
title: Chapter3 Redis配置文件
excerpt: Redis常用配置：文件保存位置、安全设置、RDB和AOF配置等
tags:
  - redis
categories:
  - Redis
banner_img: /img/post/banner/Sasha.png
index_img: /img/post/redis_logo.png
category: Redis
abbrlink: e7c43d18
date: 2021-02-03 14:53:01
updated: 2021-02-03 23:05:48
subtitle:
---
## 3.1 位置

### 3.1.1 默认位置

1. 默认放在安装目录下（/opt/software/redis-6.0.10/redis.conf）
2. 启动时不指定则加载安装目录下的配置及文件

### 3.1.2 其它位置

1. 习惯上在 /uer/local/redis下复制一份，通过该配置文件启动
2. 相关可执行文件在 /uer/local/redis/bin 下

## 3.2 相关配置

### 3.2.1 include

1. 引入配置
2. 只需要在配置文件中配置需要的部分，其他部分通过include引入

### 3.2.2 网络相关配置 NETWORK

1. bind： 绑定 IP 地址，其它机器可以通过此 IP 访问 Redis，默认绑定 127.0.0.1， 也可以修改为本机的 IP 地址。
2. port：配置 Redis 占用的端口，默认是 6379
3. protected-mode：保护模式
4. 远程访问：
   * bind改为 0.0.0.0
   * 注释掉bind，并将保护模式关闭（no）
5. tcp-backlog：
   * 请求到达后至接受进程处理前的队列
   * backlog 队列总和=未完成三次握手队列 + 已完成三次握手队列
6. timeout：空闲客户端维持多久关闭，0为永不关闭



### 3.2.3 常规配置 GENERAL

1. daemonize：是非为后台进程（即后台启动）
2. pidfile：存放 pid 文件的位置，每个实例会产生一个pid文件
3. loglevel：日志级别，开发阶段可以设置成 debug，生产阶段通常设置为 notice 或者 warning
4. logfile：指定日志文件名， 如果不指定， Redis 只进行标准输出。 
   * 要保证日志文件所在的目录必须存在，文件可以不存在。 
   * 要在 redis 启动时指定所使用的配置文件，否则配置不起作用
5. databases： 配置 Redis 数据库的个数，默认是 16 个

### 3.2.4 安全设置

1. requirepass： 配置 Redis 的访问密码。默认不配置密码，即访问不需要密码验证。 
2. 设置密码需要开启安全模式： protected-mode=yes 
3. 使用密码登录客户端： redis-cli -h ip -p 6379 -a pwd
4. 可以通过客户端命令行设置密码，只在内存中起作用

    ```
    127.0.0.1:6379> config get requirepass
    1) "requirepass"
    2) ""
    127.0.0.1:6379> config set requirepass "123456"
    OK
    ```
    退出客户端后，再次连接需要输入密码才能执行命令
    ```
    127.0.0.1:6379> auth 123456
    OK
    ```
    重启服务器后密码失效

### 3.2.5 客户端相关 CLIENTS

1. maxclients：最大客户端连接数
2. maxmemory：设置Redis可以使用的内存量
   * 达到上限时，Redis会试图移除内部数据，移除规则通过 maxmemory-policy 指定
   * 若无法根据规则移除或设置了不允许移除，则会报错
3. maxmemory-policy：移除规则
   * volatile-lru 
   * allkeys-lru 
   * volatile-lfu 
   * allkeys-lfu 
   * volatile-random 
   * allkeys-random 
   * volatile-ttl 
   * noeviction

4. maxmemory-samples：设置样本数量
   * LRU算法和TTL算法都是估算值，可以设置样本大小
   * 一般 3-7，越小越不准确，性能消耗也越小


### 3.2.6 RDB

1. save seconds changes
   * 快照触发条件
   * 要禁用 Redis 的持久化功能，则把所有的 save 配置都注释掉

2. stop-writes-on-bgsave-error yes
   * 当redis无法写入到磁盘，直接关闭redis的写操作
   * 可以保证内存数据和磁盘数据的一致性

3. rdbcompression yes
   * 进行rdb保存时，将文件压缩
   * 采用 LZF 算法进行压缩

4. rdbchecksum yes
   * 在存储快照以后， 还可以让 Redis 使用 CRC64 算法来进行数据校验
   * 会消耗一定的性能

5. dbfilename：Redis 持久化数据生成的文件名，默认是 dump.rdb，也可以自己配置
6. dir： Redis 持久化数据生成文件保存的目录，默认是./即 redis 的启动目录，也可以自己配置

### 3.2.7 AOF

1. appendonly： 配置是否开启 AOF， yes 表示开启， no 表示关闭。默认是 no
2. appendfilename： AOF 保存文件名
3. appendfsync： AOF 异步持久化策略
   * always：同步持久化，每次发生数据变化会立刻写入到磁盘中。性能较差但数据完整性比较好（慢，安全）
   * everysec：出厂默认推荐，每秒异步记录一次（默认值）
   * no：不即时同步，由操作系统决定何时同步
4. no-appendfsync-on-rewrite： 重写时是否可以运用 appendsync，默认 no，可以保证数据的安全
5. auto-aof-rewrite-percentage： 设置重写的基准百分比
6. auto-aof-rewrite-min-size：设置重写的基准值


