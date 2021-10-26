---
title: Chapter1 前言
excerpt: CentOS7安装MySQL5.7、MySQL配置、MySQL架构
tags:
  - mysql
categories:
  - MySQL
  - MySQL高级
banner_img: /img/post/banner/cat.png
index_img: /img/post/mysql_logo.svg
category: MySQL/MySQL高级
abbrlink: ac9f6c57
date: 2021-02-05 21:24:08
updated: 2021-02-05 23:59:33
subtitle:
---
## 1.1 准备工作

### 1.1.1 安装

在centos7上安装mysql5.7


### 1.1.2 配置

1. 查看安装目录

    ```sh
    ps -ef|grep mysql
    ```
    结果：
    ```
    mysql     3233     1  0 21:19 ?        00:00:00 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
    root      3269  1089  0 21:19 pts/0    00:00:00 mysql -uroot -p
    root      3528  3343  0 21:23 pts/1    00:00:00 grep --color=auto mysql
    ```

2. 配置文件位置

    路径| 解释|备注
    :-:|:-:|:-:
    /var/lib/mysql/ |mysql数据库文件存放位置| 文件名与数据库同名
    /usr/share/mysql|配置文件|常规配置文件：/etc/my.cnf
    /usr/bin|相关命令目录|mysqladmin、mysqldump等命令


## 1.2 MySQL架构

### 1.2.1 逻辑架构

1. **连接层**  
   最上层是一些客户端和链接服务，包含本地sock 通信和大多数基于客户端/服务端工具实现的类似于 TCP/IP的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限
2. **服务层**  
   第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如 过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定表的查询的顺序，是否利用索引等， 最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能
3. **引擎层**
   * MyISAM
   * InnoDB
   * ...
4. **存储层**  
    数据存储层， 主要是将数据存储在文件系统之上，并完成与存储引擎的交互

### 1.2.2 存储引擎

1. 查看命令

    ```sql
    show engines;   # 查看所有引擎
    show variables like '%storage_engine%'; # 查看当前引擎
    ```

2. 存储引擎对比

    对比项|MyISAM|InnoDB
    :-:|:-:|:-:
    主外键|不支持|支持
    事务|不支持|支持
    行表锁|表锁|行锁（适合高并发）
    缓存|只缓存索引|缓存索引和真实数据
    表空间|小|大
    关注点|性能|事务
    默认安装|Y|Y


## 1.3 MySQL 日志

### 1.3.1 错误日志

1. 简介
    * 错误日志是 MySQL 中最重要的日志之一，它记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。
    * 当数据库出现任何故障导致无法正常使用时，可以首先查看此日志。
    * 错误日志默认开启 ， 默认存放目录为 mysql 的数据目录（var/lib/mysql）, 默认的日志文件名为hostname.err（hostname是主机名）。
2. 查看日志位置

    ```sql
    SHOW VARIABLES LIKE 'log_error%';
    ```

3. 查看日志内容(shell)

    ```sh
    tail -f /var/log/mysqld.log
    ```

### 1.3.2 二进制日志

1. 概述
    * 二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，但是不包括数据查询语句。
    * 二进制日志，默认情况下是没有开启的，需要到MySQL的配置文件中开启，并配置MySQL日志的格式。

2. 在配置文件中配置（/etc/my.cnf）

    ```
    # 配置开启binlog日志， 日志的文件前缀为 mysqlbin -----> 生成的文件名如 :
    # mysqlbin.000001,mysqlbin.000002
    log_bin=mysqlbin

    # 配置二进制日志的格式
    binlog_format=STATEMENT
    ```

3. 日志格式
    * STATEMENT：
        * 日志文件中记录的都是SQL语句（statement），每一条对数据进行修改的SQL都会记录在日志文件中
        * 通过Mysql提供的mysqlbinlog工具，可以清晰的查看到每条语句的文本。
        * 主从复制的时候，从库（slave）会将日志解析为原文本，并在从库重新执行一次。
    * ROW
        * 该日志格式在日志文件中记录的是每一行的数据变更，而不是记录SQL语句。
        * 例如，执行SQL语句 ： `update tb_book set status='1'` , 由于是对全表进行更新，也就是每一行记录都会发生变更，ROW 格式的日志中会记录每一行的数据变更。
    * MIXED
        * 这是目前MySQL默认的日志格式，即混合了STATEMENT 和 ROW两种格式。默认情况下采用STATEMENT，但是在一些特殊情况下采用ROW来进行记录。
        * MIXED 格式能尽量利用两种模式的优点，而避开他们的缺点。

4. 查看
   
    * 日志以二进制方式存储，不能直接读取，需要用mysqlbinlog工具来查看
        ```sh
        mysqlbinlog log-file-name；
        ```
    * /var/lib/mysql/mysqlbin.index中存储着日志文件列表
    * 示例
        ```sh
        # 查看 STATEMENT 日志
        mysqlbinlog mysqlbing.000001；

        # 查看 ROW 日志
        mysqlbinlog -vv mysqlbin.000002
        ```

5. 日志删除
    * 全部删除
        ```sql
        RESET MASTER;
        ```
    * 删除指定编号之前的日志
        ```sql
        PURGE MASTER LOGS TO 'mysqlbin.******';
        ```
    * 删除指定时间前的日志
        ```sql
        PURGE MASTER LOGS BEFORE 'yyyy-mm-dd hh24:mi:ss';
        ```

    * 设置过期天数：/etc/my.cnf

        ```
        --expire_logs_days = 过期天数
        ```

### 1.3.3 查询日志

1. 询日志中记录了客户端的所有操作语句，而二进制日志不包含查询数据的SQL语句。
2. 默认情况下，查询日志是未开启的，在配置文件中开启 ：

    ```
    #该选项用来开启查询日志 ， 可选值 0 或者 1， 0 代表关闭， 1 代表开启
    general_log=1
    #设置日志的文件名 ， 如果没有指定， 默认的文件名为 host_name.log
    general_log_file=file_name
    ```

### 1.3.4 慢查询日志

1. 简介
    * 慢查询日志记录了所有执行时间超过参数 long_query_time 设置值并且扫描记录数不小于min_examined_row_limit 的所有的SQL语句的日志。
    * long_query_time 默认为 10 秒，最小为 0， 精度可以到微秒。

2. 配置及使用：见第三章


