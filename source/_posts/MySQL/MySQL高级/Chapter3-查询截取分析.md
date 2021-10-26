---
title: Chapter3 查询截取分析
excerpt: 查询优化、慢查询日志分析、show profile
tags:
  - mysql
categories:
  - MySQL
  - MySQL高级
banner_img: /img/post/banner/cat.png
index_img: /img/post/mysql_logo.svg
category: MySQL/MySQL高级
abbrlink: 6fb1e01
date: 2021-02-08 02:11:29
updated: 2021-02-14 02:08:47
subtitle:
---
# 3.1 查询优化

## 3.1.1 小表驱动大表

1. 即用小的数据集驱动大的数据集，减少了连接次数

2. in与exists
    * in

        ```sql
        select * from A where id in (select id from B);
        # 相当于：
        for select id from B
        for select * from A where A.id=B.id
        ```
    * exists

        ```sql
        select * from A where exists (select 1 from B where B.id = A.id)
        # 等价于
        for select * from A
        for select * from B where B.id = A.id
        ```
    * 当B表数据集小于A表时用in优于exists，反之exists由于in

## 3.1.2 order by关键字优化

### 1. Index 与 FileSort

1. 建表

    ```sql
    CREATE TABLE tblA
    (
        age   INT,
        birth TIMESTAMP NOT NULL,
        name  VARCHAR(100) NOT NULL
    );

    INSERT INTO tblA(age, birth, name) VALUES (22, NOW(), 'Tom');
    INSERT INTO tblA(age, birth, name) VALUES (23, NOW(), 'Jerry');
    INSERT INTO tblA(age, birth, name) VALUES (24, NOW(), 'Luci');
    ```

2. 建索引

    ```sql
    CREATE INDEX idx_A_ageBirth ON tblA(age, birth);
    ```
3. 两种排序

    * 情形一：全部排序

        ```sql
        EXPLAIN SELECT * FROM tblA ORDER BY age;
        ```
        ```
        +-----------------------------+
        | Extra                       |
        +-----------------------------+
        | Using filesort              |
        +-----------------------------+
        ```

    * 情形二：序索引顺序

        ```
        EXPLAIN SELECT age, birth FROM tblA WHERE age > 20 ORDER BY age;
        EXPLAIN SELECT age, birth FROM tblA WHERE age = 20 ORDER BY birth;
        EXPLAIN SELECT age, birth FROM tblA WHERE age > 20 ORDER BY age,birth;
        EXPLAIN SELECT age, birth FROM tblA WHERE birth > '2016-01-28 00:00:00' ORDER BY age;
        ```
        ```
        +--------------------------+
        | Extra                    |
        +--------------------------+
        | Using where; Using index |
        +--------------------------+
        ```

    * 情形三：多字段

        ```sql
        EXPLAIN SELECT age, birth FROM tblA ORDER BY birth;
        EXPLAIN SELECT age, birth FROM tblA ORDER BY birth,age;
        EXPLAIN SELECT age, birth FROM tblA ORDER BY age ASC, birth DESC;
        EXPLAIN SELECT age, birth, name FROM tblA ORDER BY age;
        ```
        ```
        +------------------------------------------+
        | Extra                                    |
        +------------------------------------------+
        | Using index; Using filesort              |
        +------------------------------------------+
        ```

4. 使用 Index方式的情形
   * Order by 语句使用索引最左前列
   * 使用 WHERE 子句与 ORDER BY 子句条件列组合满足索引最左前列（即where使用索引的最左前缀定义为常量）
   * Order by 的字段都是升序，或者都是降序

### 2. Filesort 的优化

对于Filesort ， MySQL 有两种排序算法

1. 双路排序：MySQL4.1 之前，使用该方式排序
    * 首先从磁盘读取数据，然后在排序区sort buffer 中排序
    * 如果sort buffer不够，则在临时表 temporary table 中存储排序结果
    * 完成排序之后，再根据行指针回表读取记录，两次扫描磁盘，很耗时
2. 单路排序
    * 一次性取出满足条件的所有字段，然后在排序区 sort buffer 中排序后直接输出结果集。
    * 排序时内存开销较大，但是排序效率比两次扫描算法要高
    * 若数据量太大无法一次性取出则会导致多次I/O操作，反而降低了效率
3. 优化策略
    * MySQL 通过比较系统变量 max_length_for_sort_data 的大小和Query语句取出的字段总大小， 来判定是否那种排序算法，如果max_length_for_sort_data 更大，那么使用第二种优化之后的算法；否则使用第一种
    * 适当提高 sort_buffer_size 和 max_length_for_sort_data 系统变量，来增大排序区的大小，提高排序的效率

## 3.1.3 优化 group by 语句

### 1. 简介

1. GROUP BY 也同样会进行排序操作，先排序后分组
2. 可以执行order by null 禁止排序，避免排序结果的消耗

### 2. 案例

1. 无索引
    ```sql
    EXPLAIN SELECT age FROM tblB GROUP BY age;
    ```
    ```
    +---------------------------------+
    | Extra                           |
    +---------------------------------+
    | Using temporary; Using filesort |
    +---------------------------------+
    ```

2. 不排序

    ```sql
    EXPLAIN SELECT age FROM tblB GROUP BY age ORDER BY NULL;
    ```
    ```
    +-----------------+
    | Extra           |
    +-----------------+
    | Using temporary |
    +-----------------+
    ```

3. 有索引
    
    ```sql
    CREATE INDEX idx_tblB_ageBirth ON tblB(age, birth);
    EXPLAIN SELECT age FROM tblB GROUP BY age;
    ```
    ```
    +-------------+
    | Extra       |
    +-------------+
    | Using index |
    +-------------+
    ```

# 3.2 慢查询日志分析

## 3.2.1 说明

1. Mysql的查询讯日志是Mysql提供的一种日志记录，它用来记录在Mysql中响应时间超过阈值的语句
2. 具体指运行时间超过long_query_time值得SQL，则会被记录到慢查询日志中。long_query_time的默认为10，意识是运行10秒以上的语句
3. 默认Mysql没有开启慢查询，需要我们说动设置这个参数。如果不是调优需要，一般不建议开启该参数，会带来一定的性能影响

## 3.2.2 使用

1. 查询是否开启

    ```sql
    SHOW VARIABLES LIKE '%slow_query_log%';
    ```

2. 开启

    ```sql
    # 0关闭，1开启
    SET GLOBAL slow_query_log = 0|1;
    ```
    只对当前数据库生效，重启失效
3. 阈值时间

    ```sql
    # 查看，默认为10秒
    SHOW VARIABLES LIKE 'long_query_time';

    # 修改
    SET GLOBAL long_query_time = 3;
    ```
4. 查询

    ```sql
    # 查询超过4秒的慢SQL
    SELECT sleep(4);

    # 查看系统有多少条记录
    SHOW GLOBAL STATUS LIKE '%Slow_queries%';
    ```

5. 配置

    ```sql
    # my.cnf

    [mysqld]
    show_query_log = 1;
    show_query_log_file=/var/lib/mysql/mysql_slow.log
    log_query_time=3;
    log_output=FILE
    ```
## 3.2.3 日志分析工具

1. MySQL提供了日志分析工具mysqldumpslow
2. MySQL默认安装了该功能，在shell种使用
3. 查看帮助
   
    ```sh
    mysqldumpslow --help;
    ```
    ```
    Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

    Parse and summarize the MySQL slow query log. Options are

    --verbose    verbose
    --debug      debug
    --help       write this text to standard output

    -v           verbose
    -d           debug
    -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                    al: average lock time
                    ar: average rows sent
                    at: average query time
                    c: count
                    l: lock time
                    r: rows sent
                    t: query time
    -r           reverse the sort order (largest last instead of first)
    -t NUM       just show the top n queries
    -a           don't abstract all numbers to N and strings to 'S'
    -n NUM       abstract numbers with at least n digits within names
    -g PATTERN   grep: only consider stmts that include this string
    -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
                default is '*', i.e. match all
    -i NAME      name of server instance (if using mysql.server startup script)
    -l           don't subtract lock time from total time
    ```
    中文：
    ```
    s:是表示按照何种方式排序

    c:访问次数

    i：锁定时间

    r：返回记录

    t：查询时间

    al:平均锁定时间

    ar：平均返回记录数

    at：平均查询时间

    t：即为返回前面多少条数据

    g：后边搭配一个正则匹配模式，大小写不敏感
    ```
4. 常用

    ```sh
    mysqldumpslow -s r -t 10 /data/mysql/mysql-slow.log  //得到返回记录集最多的10个SQL
    mysqldumpslow -s c -t 10 /data/mysql/mysql-slow.log //得到访问次数最多的10个SQL 
    mysqldumpslow -s t -t 10 -g "left join" /data/mysql/mysql-slow.log  //得到按照时间排序的前10条里面含有做了连接的查询SQL
    mysqldumpslow -s r -t 10 /data/mysql/mysql-slow.log | more  //另外建议在使用这些命令时结合|和more使用，否则有可能出现爆屏情况
    ```

# 3.3 存储过程与函数

## 3.3.1 简介

1. 定义
   * 存储过程和函数是 事先经过编译并存储在数据库中的一段 SQL 语句的集合
   * 存储过程和函数的区别在于函数必须有返回值，而存储过程没有

2. 建表

    ```sql
    CREATE TABLE dept
    (
        id     INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
        deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
        dname  VARCHAR(20)        NOT NULL DEFAULT "",
        loc    VARCHAR(13)        NOT NULL DEFAULT ""
    ) ENGINE = innodb
    DEFAULT CHARSET = GBK;

    CREATE TABLE emp
    (
        id       INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
        empno    MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
        ename    VARCHAR(20)        NOT NULL DEFAULT "",
        job      VARCHAR(9)         NOT NULL DEFAULT "",
        mgr      MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
        hiredate DATE               NOT NULL,
        sal      DECIMAL(7, 2)      NOT NULL,
        comm     DECIMAL(7, 2)      NOT NULL,
        deptno   MEDIUMINT UNSIGNED NOT NULL DEFAULT 0
    ) ENGINE = INNODB
    DEFAULT CHARSET = GBK;
    ```
## 3.3.2 使用

1. 创建 

    ```sql
    CREATE PROCEDURE procedure_name ([proc_parameter[,...]])
    begin
        -- SQL语句
    end ;
    ```
    示例
    ```sql
    DELIMITER $

    CREATE PROCEDURE pro_test2()
    BEGIN
        SELECT 'Hello word';
    END $

    DELIMITER ;
    ```
3. 调用

    ```sql
    CALL procedure_name();
    ```
4. 查看

    ```sql
    # 查看db_name中所有存储过程
    SELECT name from mysql.proc WHERE db='db_name';

    # 查询存储过程中的状态信息
    SHOW PROCEDURE STATUS;

    # 
    SHOW CREATE  PROCEDURE db_name.pro_name \G;
    ```

5. 删除

    ```sql
    DROP PROCEDURE [IF EXISTS] sp_name ；
    ```

## 3.3.3 语法

### 1. 变量
* DECLARE：通过 DECLARE 可以定义一个局部变量，该变量的作用范围只能在 BEGIN…END 块中
    ```sql
    DECLARE var_name[,...] type [DEFAULT value]
    ```
* SET：直接赋值使用 SET
    ```sql
    SET var_name = expr [, var_name = expr] ...
    ```
* SELECT ... INTO 进行赋值
* 示例
    ```sql
    DELIMITER $
    CREATE PROCEDURE pro_test5()
    BEGIN
        DECLARE num int DEFAULT 9;
        DECLARE num2 int;
        SET num = num + 0;
        SELECT count(*) INTO num2 FROM city;
        SELECT num;
        SELECT concat('city表中记录数为：', num2);
    END$
    DELIMITER ;
    ```

### 2. 游标

1. 说明
    * 游标是用来存储查询结果集的数据类型
    * 在存储过程和函数中可以使用光标对结果集进行循环的处理
    * 光标的使用包括光标的声明、OPEN、FETCH 和 CLOSE    

2. 语法
   
    ```sql
    # 声明
    DECLARE cursor_name CURSOR FOR select_statement;

    # OPEN 游标
    OPEN cursor_name;

    # FETCH(指针前进，用于遍历)
    FETCH cursor_name INTO var_name [var_name....] ;

    # 关闭
    CLOSE cursor_name;
    ```

3. 建表

    ```sql
    CREATE TABLE emp
    (
        id     INT(11)     NOT NULL AUTO_INCREMENT,
        name   VARCHAR(50) NOT NULL COMMENT '姓名',
        age    INT(11) COMMENT '年龄',
        salary INT(11) COMMENT '薪水',
        PRIMARY KEY (`id`)
    ) ENGINE = innodb
    DEFAULT CHARSET = utf8;
    INSERT INTO emp(id, name, age, salary)
    VALUES (NULL, '金毛狮王', 55, 3800),
        (NULL, '白眉鹰王', 60, 4000),
        (NULL, '青翼蝠王', 38, 2800),
        (NULL, '紫衫龙王', 42, 1800);
    ```

4. 示例：循环遍历表内容

    ```sql
    DELIMITER $
    CREATE PROCEDURE pro_test7()
    BEGIN

        DECLARE e_id INT(11);
        DECLARE e_name VARCHAR(50);
        DECLARE e_age INT(11);
        DECLARE e_salary INT(11);
        DECLARE has_data INT DEFAULT 1;

        DECLARE emp_result CURSOR FOR SELECT * FROM emp;
        DECLARE EXIT HANDLER FOR NOT FOUND SET has_data=0;

        OPEN emp_result;

        REPEAT
            FETCH emp_result INTO e_id, e_name, e_age, e_salary;
            SELECT concat('id=',e_id,',name=',e_name,',age=',e_age,',salary=',e_salary);
            until has_data=0
        END REPEAT;

        CLOSE emp_result;
    END $
    DELIMITER ;
    ```

## 3.3.4 批量数据脚本

1. 创建随机函数

    ```sql
    DELIMITER $
    CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
    BEGIN
        DECLARE chart_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
        DECLARE return_str VARCHAR(255) DEFAULT '';
        DECLARE i INT DEFAULT 0;
        WHILE i < n DO
            SET return_str = concat(return_str, substring(chart_str, floor(1+rand()*52),1));
            SET i = i+1;
        END WHILE;
        RETURN return_str;
    END $
    DELIMITER ;
    ```
    ```sql
    DELIMITER $
    CREATE FUNCTION rand_num()
    RETURNS INT(5)
    BEGIN
        DECLARE i INT DEFAULT 0;
        SET i = floor(100+rand()*10);
        RETURN i;
    END $
    DELIMITER ;
    ```
2. 创建插入函数

    ```sql
    DELIMITER $
    CREATE PROCEDURE insert_emp(IN START INT(10), IN max_num INT(10))
    BEGIN
        DECLARE i INT DEFAULT 0;
        SET AUTOCOMMIT = 0;
        REPEAT
            SET i = i+1;
            INSERT INTO atguigu.emp(empno, ename, job, mgr, hiredate, sal, comm, deptno) VALUES ((START+i),rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
        UNTIL i = max_num
        END REPEAT;
        COMMIT;
    END $
    DELIMITER ;
    ```
    ```sql
    DELIMITER $
    CREATE PROCEDURE insert_dept(IN start INT(10), IN max_num INT(10))
    BEGIN
        DECLARE i INT DEFAULT 0;
        SET autocommit = 0;
        REPEAT
            SET i = i + 1;
            INSERT INTO dept(deptno, dname, loc) VALUES ((start + i), ran_string(10), ran_string(8));
        UNTIL i = max_num
            END REPEAT;
        COMMIT;
    END $
    DELIMITER ;
    ```
3. 插入

    ```
    CALL insert_dept(100,10);

    CALL insert_emp(100001,50000);
    ```

# 3.4 Show Profile

## 3.4.1 简介

1. Mysql从5.0.37版本开始增加了对 show profiles 和 show profile 语句的支持。show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了
2. 通过 have_profiling 参数，能够看到当前MySQL是否支持profile
    ```sql
    mysql> SELECT @@have_profiling;
    +------------------+
    | @@have_profiling |
    +------------------+
    | YES              |
    +------------------+
    ```

## 3.4.2 使用

1. 默认是关闭的，可以在session中开启
    ```sql
    set profiling=1;
    ```

2. 查看
    ```sql
    # 查看所有sql语句耗时
    SHOW PROFILES;

    # 查看具体的sql语句
    SHOW PROFILE FOR QUERY query_id;

    # 进一步选择all、cpu、block io 、context switch、page faults等明细类型
    SHOW PROFILE CPU FOR QUERY query_id;
    ```

3. 步骤

    * 开启功能
    * 执行SQL语句
    * SHOW PROFILES 查看结果
    * 诊断SQL，根据id进一步查看SQL问题：
        ```sql
        SHOW PROFILE CPU, BLOCK IO FOR QUERY query_id;
        ```


4. 常见问题

    * converting HEAP to MyISAM 查询结果太大，内存不够用，往磁盘上存数据了
    * creating tmp table 创建临时表
    * copying to tmp table on disk 把内存中的临时表复制到磁盘
    * locked

# 3.5 全局查询日志

1. 开启

    ```sql
    set global general_log = 1;
    set global log_output='TABLE';
    ```

2. 查看

    ```sql
    select * from mysql.general_log;
    ```







