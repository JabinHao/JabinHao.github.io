---
title: Chapter6 Redis主从复制
excerpt: 一主二仆、薪火相传、反客为主、哨兵模式
tags:
  - redis
categories:
  - Redis
banner_img: /img/post/banner/kangni.png
index_img: /img/post/redis_logo.png
category: Redis
abbrlink: 36cc3fdd
date: 2021-02-03 23:29:16
updated: 2021-02-04 01:55:24
subtitle:
---
## 6.1 简介

### 6.1.1 定义

主机数据更新后根据配置和策略，自动同步到从机的master/slave机制， Master以写为主， Slave以读为主

### 6.1.2 用处

1. 读写分离，性能扩展
2. 容灾快速恢复

### 6.1.3 三种方式

1. 一主二仆
2. 薪火相传
3. 反客为主

## 6.2 一主二从

### 6.2.1 原理

1. 配从(库)不配主(库)
2. 配从(库): slaveof 主库IP 主库端口
3. 主写从读、读写分离
4. 从连前后同
5. 主断从待命、从断重新连

### 6.2.2 搭建步骤

1. 一台服务器模拟三台主机：
   * 将redis.conf 拷贝三份，名字分别是， redis6379.conf， redis6380.conf， redis6381.conf
   * 修改三个文件的port端口， pid文件名(pidfile)，日志文件名(logfile)， rdb文件名(dbfilename)
   * 分别使用三个配置文件启动三个 redis 服务器
2. 查询主从信息： info replication
   * 三台都是master
3. 6379 写操作

    ```
    set k1 v1
    ```

4. 设置主从关系
    * 6380和6381上执行:
        
        ```
        127.0.0.1:6380> slaveof 127.0.0.1 6379
        127.0.0.1:6381> slaveof 127.0.0.1 6379
        ```
    * 方式二：在配置文件中加入

        ```
        slaveof 127.0.0.1 6379
        ```

### 6.2.3 几个概念

1. 全量复制：6380、6381把6379之前的内容也复制了过来

    ```
    127.0.0.1:6380> get k1
    "v1"
    ```
2. 增量复制：6379此时进行写操作，6380、6381都会复制下来

    ```
    127.0.0.1:6379> set k2 v2
    OK
    ```
    ```
    127.0.0.1:6380> get k2
    "v2"
    ```
    ```
    127.0.0.1:6381> get k2
    "v2"
    ```
3. 主写从读、读写分离： 从机写报错

    ```
    127.0.0.1:6381> set k3 v3
    (error) READONLY You can't write against a read only replica.
    ```

4. 主机宕机：从机待命
   
   主机shutdown
    ```
    127.0.0.1:6379> shutdown
    ```
    从机待命
    ```
    127.0.0.1:6380> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6379
    master_link_status:down
    ....
    ```
5. 主机宕机后恢复：一切正常

6. 从机宕机：必须重新连接， 除非写进配置文件


## 6.3 薪火相传

### 6.3.1 背景

1. 一主多仆master服务器压力大
2. 上一个slave可以作为下一个slave的master
3. 中途变更转向：会清除之前的数据，重新拷贝最新的

### 6.3.2 操作

1. 一台主机配多台从机，一台从机再配多台从机，从而实现了庞大的集群架构。同时也减轻了一台主机的压力
2. 缺点是增加了服务器间的延迟
3. 6381作为6380的从机，6380作为6379的从机\

    ```
    127.0.0.1:6381> slaveof 127.0.0.1 6380
    OK
    ```

### 6.3.3 概念

1. 中间机6380：身份还是从机

    ```
    127.0.0.1:6380> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6379
    ...
    ```
2. 主机写，从机逐次更新

    ```
    127.0.0.1:6379> set k4 v4
    OK
    ```
    ```
    127.0.0.1:6380> get k4
    "v4"
    ```
    ```
    127.0.0.1:6381> get k4
    "v4"
    ```

## 6.4 反客为主

### 6.4.1 操作

1. 主机宕机

    ```
    127.0.0.1:6379> shutdown
    ```
2. 从机反客为主

    ```
    127.0.0.1:6380> slaveof no one
    OK
    ```
3. 其它从机转换主机

    ```
    127.0.0.1:6381> slaveof 127.0.0.1 6380
    OK 
    ```

### 6.4.2 概念

1. 主机恢复：已与它无关


## 6.5 哨兵模式(sentinel)

### 6.5.1 简介

1. 哨兵模式是反客为主的自动版，后台监控主机是否故障，故障则根据投票自动将从库转为主库
2. 一组 sentinel 可以同时监控多个 master

### 6.5.2 搭建

1. 按之前的一主二仆方式启动 三台redis
2. 新建 sentionel.conf 文件, 内容：

    ```
    sentinel monitor mymaster 127.0.0.1 6379 1
    ```
    * mymaster：给主服务器起的别名
    * 127.0.0.1 6379 表示： 指定监控主机的 ip 地址， port 端口，
    * 1：得票数多于 1 时需要切换主从关系
    * 如果设置密码了，还需要设置密码：`sentinel auth-pass mymaster psw`
3. 启动哨兵

    ```
    /usr/local/redis/bin/redis-sentinel sentinel.conf
    ```

### 6.5.3 操作

1. 主机宕机

    ```
    127.0.0.1:6379> shutdown
    ```
    哨兵监控到，投票后某台从机成为主机
    ```
    127.0.0.1:6381> info replication
    # Replication
    role:master
    connected_slaves:1
    ```
    其它从机转向新主机
    ```
    127.0.0.1:6380> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6381
    ```
2. 主机重新上线

    ```
    /usr/local/redis/bin/redis-server redis6379.conf
    /usr/local/redis/bin/redis-cli -p 6379
    ```
    哨兵检测到后，该服务器成为从机
    ```
    127.0.0.1:6379> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6381
    master_link_status:up
    ```

### 6.5.4 总结

1. 查看主从复制关系命令： info replication
2. 设置主从关系命令： slaveof 主机 ip 主机 port
3. 开启哨兵模式命令： ./redis-sentinel sentinel.conf
4. 主从复制原则：开始是全量复制，之后是增量复制
5. 哨兵模式三大任务：监控，提醒，自动故障迁移


