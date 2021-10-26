---
title: Chapter4 MySQL锁机制
excerpt: MySQL行锁、表锁使用
tags:
  - mysql
categories:
  - MySQL
  - MySQL高级
banner_img: /img/post/banner/cat.png
index_img: /img/post/mysql_logo.svg
category: MySQL/MySQL高级
abbrlink: 30c53523
date: 2021-02-14 02:09:27
updated: 2021-02-20 23:58:38
subtitle:
---
##  4.1 概述

### 4.1.1 什么是锁

1. 锁是计算机协调多个进程或线程并发访问某一资源的机制（避免争抢）。
2. 锁冲突也是影响数据库并发访问性能的一个重要因素。

### 4.1.2 分类

1. 从对数据操作的粒度分 ：
    * 表锁：操作时，会锁定整个表。
    * 行锁：操作时，会锁定当前操作行。

2. 从对数据操作的类型分：
    * 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。
    * 写锁（排它锁）：当前操作没有完成之前，它会阻断其他写锁和读锁。

### 4.1.3 MySQL 锁

1. 不同的存储引擎支持不同的锁机制

    存储引擎 | 表级锁 | 行级锁 | 页面锁
    :-|:-|:-|:-
    MyISAM | 支持 | 不支持 | 不支持
    InnoDB | 支持 | 支持 | 不支持
    MEMORY | 支持 | 不支持 | 不支持
    BDB | 支持 | 不支持 | 支持

2. 归纳
   * 表级锁：偏向MyISAM 存储引擎，开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。
   * 行级锁：偏向InnoDB 存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。
   * 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

3. 使用场景
   * 表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web 应用；
   * 行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有并查询的应用，如一些在线事务处理（OLTP）系

## 4.2 MyISAM 表锁

### 4.2.1 语法

1. 加锁
   
    ```sql
    # 加读锁 ： 
    lock table table_name read;

    # 加写锁 ： 
    lock table table_name write；
    ```

2. 解锁

    ```sql
    unlock tables;
    ```

### 4.2.2 读锁案例

1. 建表

    ```sql
    CREATE DATABASE demo_03 DEFAULT CHARSET = utf8mb4;
    USE demo_03;
    CREATE TABLE `tb_book`
    (
        `id`           INT(11) AUTO_INCREMENT,
        `name`         VARCHAR(50) DEFAULT NULL,
        `publish_time` DATE        DEFAULT NULL,
        `status`       CHAR(1)     DEFAULT NULL,
        PRIMARY KEY (`id`)
    ) ENGINE = myisam
    DEFAULT CHARSET = utf8;
    INSERT INTO tb_book (id, name, publish_time, status) VALUES (NULL, 'java编程思想', '2088-08-01', '1');
    INSERT INTO tb_book (id, name, publish_time, status) VALUES (NULL, 'solr编程思想', '2088-08-08', '0');

    CREATE TABLE `tb_user`
    (
        `id`   INT(11) AUTO_INCREMENT,
        `name` VARCHAR(50) DEFAULT NULL,
        PRIMARY KEY (`id`)
    ) ENGINE = myisam
    DEFAULT CHARSET = utf8;
    INSERT INTO tb_user (id, name) VALUES (NULL, '令狐冲');
    INSERT INTO tb_user (id, name) VALUES (NULL, '田伯光');
    ```

2. 客户端一：

    * 加读锁
        ```sql
        lock table tb_book read;
        ```
    * 查询操作：可以正常进行
        ```sql
        select * from tb_book;
        ```
    * 写操作：报错
        ```sql
        mysql> insert into tb_book values(null,'Mysql高级','2088-01-01','1');
        ERROR 1099 (HY000): Table 'tb_book' was locked with a READ lock and can't be updated
        ```

    * 查询其他表：报错
        ```sql
        mysql> select * from tb_user;
        ERROR 1100 (HY000): Table 'tb_user' was not locked with LOCK TABLE
        ```

3. 客户端二：
    * 读操作：正常
        ```sql
        mysql> select * from tb_book;
        +----+------------------+--------------+--------+
        | id | name             | publish_time | status |
        +----+------------------+--------------+--------+
        |  1 | java编程思想     | 2088-08-01   | 1      |
        |  2 | solr编程思想     | 2088-08-08   | 0      |
        +----+------------------+--------------+--------+
        ```
    * 写操作：堵塞
        ```sql
        mysql> insert into tb_book values(null,'Mysql高级','2088-01-01','1');
        等待....
        ```

### 4.2.3 写锁案例

1. 客户端一
    * 加写锁
        ```sql
        lock table tb_book write ;
        ```
    * 读当前表：正常
        ```sql
        mysql> select * from tb_book;
        +----+------------------+--------------+--------+
        | id | name             | publish_time | status |
        +----+------------------+--------------+--------+
        |  1 | java编程思想     | 2088-08-01   | 1      |
        |  2 | solr编程思想     | 2088-08-08   | 0      |
        +----+------------------+--------------+--------+
        2 rows in set (0.00 sec)
        ```
    * 写当前表：正常
        ```sql
        mysql> insert into tb_book values(null,'Mysql高级','2088-01-01','1');
        Query OK, 1 row affected (0.00 sec)
        ```
    * 读写其他表：报错

2. 客户端二：
   * 读写锁住的表：阻塞
        ```sql
        mysql> select * from tb_book;
        等待...

        mysql> update tb_book set name = 'java编程思想（第二版）' where id = 1;
        等待....
        ```

### 4.2.4 总结

1. 自动加锁
    * MyISAM 在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁
    * 在执行更新操作（UPDATE、DELETE、INSERT 等）前，会自动给涉及的表加写锁
2. MySQL的两种表级锁
   * 表共享读锁
   * 表独享写锁

    锁类型|可否兼容|读锁|写锁
    :-|:-|:-|:-
    读锁|是|是|否
    写锁|是|否|否

3. 总结
    * 对MyISAM 表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；
    * 对MyISAM 表的写操作，则会阻塞其他用户对同一表的读和写操作；
    * 简而言之，就是读锁会阻塞写，但是不会阻塞读。而写锁，则既会阻塞读，又会阻塞写。

### 4.2.5 锁争用情况查询

1. 查看锁情况

    ```sql
    # 查看所有
    show open tables;

    # 查看具体数据库
    show open tables from db_name;
    ```

    * In_user : 表当前被查询使用的次数。如果该数为零，则表是打开的，但是当前没有被使用。
    * Name_locked：表名称是否被锁定。名称锁定用于取消表或对表进行重命名等操作。

2. 查看

    ```sql
    show status like 'Table_locks%';
    ```
    * Table_locks_immediate ： 指的是能够立即获得表级锁的次数，每立即获取锁，值加1。
    * Table_locks_waited ： 指的是不能立即获取表级锁而需要等待的次数，每等待一次，该值加1，此值高说明存在着较为严重的表级锁争用情况

## 4.3 InnoDB 行锁

### 4.3.1 行锁简介

1. 行锁特点：
   * 偏向InnoDB 存储引擎，开销大，加锁慢；
   * 会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。
2. InnoDB 与 MyISAM 的最大不同有两点：一是支持事务；二是 采用了行级锁。

### 4.3.2 事务回顾

1. 事务是由一组SQL语句组成的逻辑处理单元。
2. 事务具有以下4个特性，简称为事务ACID属性
    * 原子性（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全部成功，要么全部失败。
    * 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。
    * 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的 “独立” 环境下运行。
    * 持久性（Durable）：事务完成之后，对于数据的修改是永久的。

3. 并发事务处理带来的问题
    * 丢失更新（LostUpdate）：当两个或多个事务选择同一行，最初的事务修改的值，会被后面的事务修改的值覆盖。
    * 脏读（DirtyReads）：当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
    * 不可重复读（NonRepeatableReads）：一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现和以前读出的数据不一致。
    * 幻读（PhantomReads）：一个事务按照相同的查询条件重新读取以前查询过的数据，却发现其他事务插入了满足其查询条件的新数据。

4. 事务的隔离级别

    隔离级别 | 丢失更新 | 脏读 | 不可重复读 | 幻读
    :-|:-:|:-:|:-:|:-:
    Read uncommitted| × | √ | √ | √
    Read committed | × | × | √ | √
    Repeatable read（默认） | × | × | × | √
    Serializable | × | × | × | ×

5. 查看默认隔离级别

    ```sql
    show variables like 'tx_isolation';
    ```

### 4.3.3  InnoDB 的行锁模式

1. InnoDB 实现了以下两种类型的行锁：
    * 共享锁（S）：又称为读锁，简称S锁，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
    * 排他锁（X）：又称为写锁，简称X锁，排他锁就是不能与其他锁并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据就行读取和修改。
2. 自动加锁
    * 对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁(X)；
    * 对于普通SELECT语句，InnoDB不会加任何锁；

3. 语法

    ```sql
    # 共享锁(S)：
    SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE

    # 排他锁(X)：
    SELECT * FROM table_name WHERE ... FOR UPDATE
    ```

### 4.3.4 案例

1. 建表

    ```sql
    CREATE TABLE test_innodb_lock
    (
        id   INT(11),
        name VARCHAR(16),
        sex  VARCHAR(1)
    ) ENGINE = innodb
    DEFAULT CHARSET = utf8;
    INSERT INTO test_innodb_lock
    VALUES (1, '100', '1');
    INSERT INTO test_innodb_lock
    VALUES (3, '3', '1');
    INSERT INTO test_innodb_lock
    VALUES (4, '400', '0');
    INSERT INTO test_innodb_lock
    VALUES (5, '500', '1');
    INSERT INTO test_innodb_lock
    VALUES (6, '600', '0');
    INSERT INTO test_innodb_lock
    VALUES (7, '700', '0');
    INSERT INTO test_innodb_lock
    VALUES (8, '800', '1');
    INSERT INTO test_innodb_lock
    VALUES (9, '900', '1');
    INSERT INTO test_innodb_lock
    VALUES (1, '200', '0');

    create index idx_test_innodb_lock_id on test_innodb_lock(id);
    create index idx_test_innodb_lock_name on test_innodb_lock(name);
    ```

2. 行锁演示

    ![行锁演示](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/MySQL/4-3-4.png)


### 4.3.5 无索引行锁升级为表锁

1. 如果不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，实际效果跟表锁一样。
2. 示例
    ![无索引行锁升级为表锁](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/MySQL/4-3-4_2.png)

### 4.3.6 间隙锁危害

1. 说明
    * 当我们用范围条件，而不是使用相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据进行加锁； 
    * 对于键值在条件范围内但并不存在的记录，叫做 "间隙（GAP）" ， InnoDB也会对这个 "间隙" 加锁，这种锁机制就是所谓的 间隙锁（Next-Key锁）。
2. 案例
   * 上面的表中不存在id为2的数据，但当我们给 `1<id<5`的数据进行修改时，id=2也会被锁
   * 其它事务操作（即新增） id=2的数据会被阻塞
   * 示例：
        ![间隙锁](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/MySQL/4-3-6.png)


### 4.3.7 其它

1. 锁定一行

    ```sql
    begin;
    select * from table_name where ... for update;
    commit;
    ```

2. 查看锁争用情况

    ```sql
    show status like 'innodb_row_lock%';
    ```
    * Innodb_row_lock_current_waits: 当前正在等待锁定的数量
    * Innodb_row_lock_time: 从系统启动到现在锁定总时间长度
    * Innodb_row_lock_time_avg:每次等待所花平均时长
    * Innodb_row_lock_time_max:从系统启动到现在等待最长的一次所花的时间
    * Innodb_row_lock_waits: 系统启动后到现在总共等待的次数
    * 当等待的次数很高，而且每次等待的时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手制定优化计划。

## 4.4 总结

1. InnoDB存储引擎由于实现了行级锁定，虽整体并发处理能力方面远远优于MyISAM的表锁。

2. 当我们使用不当的时候，可能会让InnoDB的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

3. 优化建议：
    * 尽可能让所有数据检索都能通过索引来完成，避免无索引行锁升级为表锁。
    * 合理设计索引，尽量缩小锁的范围
    * 尽可能减少索引条件，及索引范围，避免间隙锁
    * 尽量控制事务大小，减少锁定资源量和时间长度
    * 尽可使用低级别事务隔离（但是需要业务层面满足需求）


