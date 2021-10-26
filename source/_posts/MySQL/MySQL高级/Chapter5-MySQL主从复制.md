---
title: Chapter5 MySQL主从复制
excerpt: MySQL主从复制原理、配置
tags:
  - mysql
categories:
  - MySQL
  - MySQL高级
banner_img: /img/post/banner/cat.png
index_img: /img/post/mysql_logo.svg
category: MySQL/MySQL高级
abbrlink: ea6c32d9
date: 2021-02-21 00:48:55
updated: 2021-02-21 02:43:21
subtitle:
---
## 5.1 概述

### 5.1.1 MySQL的主从复制

1. 将主数据库的DDL 和 DML 操作通过二进制日志传到从库服务器中，然后在从库上对这些日志重新执行（也叫重做），从而使得从库和主库的数据保持同步。

2. MySQL支持一台主库同时向多台从库进行复制， 从库同时也可以作为其他从服务器的主库，实现链状复制

### 5.1.2 主从复制原理

1. 示意
   ![主从复制](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/MySQL/5-1-2.png)
2. 步骤
    * Master 主库在事务提交时，会把数据变更作为时间 Events 记录在二进制日志文件 Binlog 中。
    * 主库推送二进制日志文件 Binlog 中的日志事件到从库的中继日志 Relay Log 。
    * slave重做中继日志中的事件，将改变反映它自己的数据

### 5.1.3 基本原则

1. 每个slave只有一个master
2. 每个slave只能有一个唯一的服务器ID
3. 每个master可以有多个slave

### 5.1.4 优点

1. 主库出现问题，可以快速切换到从库提供服务。
2. 可以在从库上执行查询操作，从主库中更新，实现读写分离，降低主库的访问压力。
3. 可以在从库中执行备份，以避免备份期间影响主库的服务。

## 5.2 搭建步骤

### 5.2.1 主机搭建

1. 修改 /etc/my.cnf

    ```
    #mysql 服务ID,保证整个集群环境中唯一
    server-id=1

    #mysql binlog 日志的存储路径和文件名
    log-bin=/var/lib/mysql/mysqlbin

    #错误日志,默认已经开启
    #log-err

    #mysql的安装目录
    #basedir

    #mysql的临时目录
    #tmpdir

    #mysql的数据存放目录
    #datadir

    #是否只读,1 代表只读, 0 代表读写
    read-only=0

    #忽略的数据, 指不需要同步的数据库
    binlog-ignore-db=mysql

    #指定同步的数据库
    #binlog-do-db=db01
    ```
2. 重启mysql服务

    ```sh
    systemctl restart mysqld.servic
    ```

3. 创建同步数据的账户，并且进行授权操作

    ```sql
    # 修改密码规则
    SET GLOBAL validate_password_policy=0;
    SET GLOBAL validate_password_length=4; #最小长度

    # 创建并授权（用户名、密码自己起）
    GRANT REPLICATION SLAVE ON *.* TO 'itcast'@'从机IP' IDENTIFIED BY 'itcast';
    flush privileges;
    ```

4. 查看master状态

    ```sql
    SHOW MASTER STATUS;
    ```
    * File : 从哪个日志文件开始推送日志文件
    * Position ： 从哪个位置开始推送日志
    * Binlog_Ignore_DB : 指定不需要同步的数据库

### 5.2.2 从机搭建

1. 修改配置文件

    ```
    #mysql服务端ID,唯一
    server-id=2
    ##指定binlog日志
    log-bin=/var/lib/mysql/mysqlbin
    ```

2. 重启mysql服务

    ```sh
    systemctl restart mysqld.servic
    ```

3. 执行命令

    ```sql
    CHANGE MASTER TO MASTER_HOST = '主机IP', MASTER_USER ='itcast', MASTER_PASSWORD ='itcast', MASTER_LOG_FILE ='mysqlbin.000001', MASTER_LOG_POS =450;
    ```
    * 指定当前从库对应的主库的IP地址，用户名，密码，从哪个日志文件开始的那个位置开始同步推送日志
    * MASTER_LOG_FILE、MASTER_LOG_POS需根据主机状态设置

4. 开启同步操作

    ```sql
    START SLAVE;
    SHOW SLAVE STATUS\G;
    ```

5. 停止命令

    ```sql
    STOP SLAVE; 
    ```

### 5.2.3 验证

1. 在主库中创建数据库，创建表，并插入数据 ：

    ```sql
    CREATE DATABASE db01;
    USE db01;
    CREATE TABLE user
    (
        id   INT(11)     NOT NULL AUTO_INCREMENT,
        name VARCHAR(50) NOT NULL,
        sex  VARCHAR(1),
        PRIMARY KEY (id)
    ) ENGINE = innodb
    DEFAULT CHARSET = utf8;
    INSERT INTO user(id, name, sex) VALUES (NULL, 'Tom', '1');
    INSERT INTO user(id, name, sex) VALUES (NULL, 'Trigger', '0');
    INSERT INTO user(id, name, sex) VALUES (NULL, 'Dawn', '1');
    ```

2. 在从库中查询数据，进行验证

    ```sql
    SHOW DATABASES;
    USE db01;
    SELECT * FROM user;
    ```



