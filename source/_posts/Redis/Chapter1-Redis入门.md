---
title: Chapter1 Redis入门
excerpt: NoSQL简介、linux下Redis的安装与配置
tags:
  - redis
categories:
  - Redis
banner_img: /img/post/banner/Sasha.jpg
index_img: /img/post/redis_logo.png
category: Redis
abbrlink: 290657f5
date: 2021-02-02 20:14:45
updated: 2021-02-02 22:15:54
subtitle:
---
## 1.1 NoSQL

### 1.1.1 数据库应用的演变历程

1. 单机数据库时代 
2. Memcached 时代
3. 读写分离时代
4. 分表分库时代(集群)
5. nosql 时代

### 1.1.2 NoSQL 数据库

1. NoSQL = Not Only SQL(不仅仅是 SQL) ，泛指 non-relational(非关系型数据库)。
2. NoSQL 数据库的一个显著特点就是去掉了关系数据库的关系型特性，数据之间一旦没有关系，使得扩展性、读写性能都大大提高。

### 1.1.3 Redis 简介

1. 定义
   * Remote Dictionary Server(远程字典服务器)
   * 用 C 语言编写的、 开源的、基于内存运行并支持持久化的、高性能的 NoSQL 数据库.也是当前热门的 NoSQL 数据库之一

2. 特点
    * **支持数据持久化**：  
    Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
    * **支持多种数据结构**：  
    Redis 不仅仅支持简单的 key-value 类型的数据，同时还提供 list， set， zset， hash 等数据结构的存储。
    * **支持数据备份**：  
    Redis 支持数据的备份，即 master-slave 模式的数据备份
3. 单线程+多路IO复用

## 1.2 安装启动

### 1.2.1 下载解压

1. 下载

    ```sh
    # centos7
    # 安装wget
    # yum install wget
    # 下载到/opt/software目录下
    wget https://download.redis.io/releases/redis-6.0.10.tar.gz
    ```

2. 解压

    ```sh
    tar -zxvf redis-6.0.10.tar.gz
    ```

### 1.2.2 安装

1. 安装gcc、make

    ```sh
    yum -y install gcc automake autoconf libtool make
    yum install -y gcc-c++
    ```

2. 编译准备：修改Makefile文件

    ```sh
    cd redis-6.0.10
    vi src/Makefile
    # 修改存放可执行文件的位置：
    # PREFIX?=/usr/local/redis
    ```
3. 编译
    ```sh
    make # redis目录下执行
    ```
3. 报错
   
    ```bash
    sudo yum install centos-release-scl
    sudo yum install devtoolset-7-gcc*
    scl enable devtoolset-7 bash
    make
    make install
    ```

    仍然报错可以尝试执行
    ```sh
    make disclean
    ```

4. 配置

    ```sh
    cp /opt/software/redis-6.0.10/redis.conf /usr/local/redis/
    mkdir /var/redis
    ```
5. 修改配置文件

    ```sh
    vim /usr/local/redis/redis.conf
    # 修改以下三个地方
    # daemonize yes # 后台启动
    # logfile "/var/redis/redis.log" # 日志文件位置
    # dir /usr/local/redis    
    ```

### 1.2.3 启动

1. 启动Redis

    ```sh
    # 启动redis服务器
    /usr/local/redis/bin/redis-server /usr/local/redis/redis.conf 

    # 启动redis客户端
    /usr/local/redis/bin/redis-cli
    ```

2. 远程访问redis
   
    redis服务器：
    ```sh
    # 修改conf中的bind 为0.0.0.0
    vi /usr/local/redis/redis.conf
    ```
    远程客户端：
    ```
    #客户端连接
    redis-cli -h ip地址 -p 端口号
    ```
    端口号默认6379（Merz）

### 1.2.4 关闭

1. 退出客户端

    ```sh
    exit
    ```

2. 关闭服务器

    ```sh
    #客户端中执行
    shutdown
    # bash中执行
    redis-cli shundown
    # 指定端口关闭
    redis-cli -p 6379 shundown 
    ```
    也可以通过杀死进程关闭
    ```sh
    ps -ef | grep redis # 可以获取到redis-server的pid
    # root     16617 16286  0 12:59 pts/0    00:00:00 grep --color=auto redis
    # root     26720     1  0 Feb01 ?        00:04:27 /usr/local/redis/bin/redis-server 127.0.0.1:6379
    kill -9 pid # 这里的pid是26720
    ```

## 1.3 使用

### 1.3.1 常用系统命令

1. 测试连接(客户端)

    ```sh
    ping
    # 返回pong表示连接正常
    ```

2. 查看性能

    ```sh
    /usr/local/redis/bin/redis-benchmark
    ```

3. 选择库

    ```sh
    select 1 # 选择1号库，客户端命令 
    ```
    * 默认16个库，从0开始，默认初始库为0
    * 在 redis.conf 文件中 `databases 16` 可以修改

4. 查看 redis 服务器的统计信息

    ```sh
    info server # 客户端命令
    ```

5. 查看当前数据库中 key 的数目

    ```sh
    dbsize
    ```

6. 获取配置值

    ```sh
    config get *     # 获取所有配置
    config get port  # 获取端口号
    config get databases # 获取库个数
    ```

### 1.3.2 常用操作命令

1. 清空数据库
   
    ```sh
    flushdb  # 清空当前库
    flushall # 清空所有库
    ```

2. 查看当前数据库中有哪些 key
   
    ```sh
    keys *
    ```

3. 判断某个键是否存在

    ```
    exists key
    ```

4. 查看键的类型

    ```
    type key
    ```

5. 删除某个键

    ```
    del key
    ```

6. 设置过期时间，单位为秒

    ```
    expire key seconds
    ```

7. 查看还有多久过期

    ```
    ttl key
    ```
    * -1：永不过期
    * -2：已过期



