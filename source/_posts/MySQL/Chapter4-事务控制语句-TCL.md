---
title: Chapter4 事务控制语句(TCL)
excerpt: 事务控制语句、视图、变量、存储过程、函数、流程控制结构
tags:
  - mysql
  - java
categories:
  - mysql
banner_img: /img/road.png
index_img: /img/post/Mysql/mysql_logo.svg
abbrlink: 2ac0582e
date: 2020-11-12 23:50:13
updated: 2020-11-12 23:50:13
subtitle:
---
# Chapter4 事务控制语句(TCL)
TCL即 Transaction Control Language 事务控制语言
## 4.1 概念
1. 事务定义：  
   一个或一组sql语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行。
2. 事务的特性：ACID
   * 原子性：一个事务不可再分割，要么都执行要么都不执行
   * 一致性：一个事务执行会使数据从一个一致状态切换到另外一个一致状态
   * 隔离性：一个事务的执行不受其他事务的干扰
   * 持久性：一个事务一旦提交，则会永久的改变数据库的数据.
3. 分类
   * 隐式事务：  
      事务没有明显的开启和结束的标记，如`insert`、`update`、`delete`语句
   * 显式事务：  
      事务具有明显的开启和结束的标记（前提：必须先设置自动提交功能为禁用：`set autocommit=0;`）

## 4.2 事物的创建
1. 创建步骤
   ```sql
   # 步骤1：开启事务
   set autocommit=0;
   start transaction; # 可选
   # 步骤2：编写事务中的sql语句(select insert update delete)
   # 语句1;
   # 语句2;
   # ...
   # 步骤3：结束事务
   commit; # 提交事务
   ```
2. 其他命令
   * `rollback;`：回滚事务，用在提交前
   * `savepoint 节点名;` ：设置保存点，与回滚搭配使用
      ```sql
      ...
      SAVEPOINT a;#设置保存点
      ...
      ROLLBACK TO a;#回滚到保存点
      ```

## 4.3 事务的隔离
1. 事务的并发问题
   * 脏读
   * 不可重复读
   * 幻读
2. 事务的隔离级别
   * `read uncommitted`
   * `read committed`
   * `repeatable read`
   * `serializable`

   隔离级别|脏读|不可重复读|幻读
   :-:|:-:|:-:|:-:
   read uncommitted：|√		|√		|√
   read committed：  |×		|√		|√
   repeatable read： |×		|×	   |√
   serializable	   |×    |×    |×
3. 说明
   * mysql中默认 第三个隔离级别 repeatable read
   * oracle中默认第二个隔离级别 read committed
4. 常用命令
   * `select @@tx_isolation;`：查看隔离级别
   * `set session|global transaction isolation level 隔离级别;`：设置隔离级别

# Chapter5 视图
## 5.1 概念
1. 虚拟表，和普通表一样使用
2. mysql5.1版本出现的新特性，是通过表动态生成的数据
## 5.2 使用视图
1. 创建视图
   ```sql
   create view 视图名
   as
   查询语句;
   ```
2. 修改视图
   1. 方式一
      ```sql
      create or replace view  视图名
      as
      查询语句;
      ```
   2. 方式二
      ```sql
      alter view 视图名
      as 
      查询语句;
      ```
3. 删除视图
   ```sql
   drop view 视图名,视图名,...;
   ```
4. 查看视图
   ```sql
   DESC myv3;

   SHOW CREATE VIEW myv3;
   ```
5. 视图的更新
   1. 插入数据
      ```sql
      INSERT INTO 视图名 VALUES()
      ```
   2. 修改数据
      ```sql
      UPDATE 视图名 SET 列名=
      ```
   3. 删除数据
      ```sql
      DELETE FROM 视图名 WHERE 条件;
      ```
6. 不允许更新的视图
   1. 包含以下关键字的sql语句：分组函数、distinct、group  by、having、union或者union all
   2. 常量视图
   3. Select中包含子查询
   4. join（联结表）
   5. from一个不能更新的视图
   6. where子句的子查询引用了from子句中的表
7. `delete`与`truncate`
   * delete可以回滚
   * truncate不能回滚

# Chapter6 变量
## 6.1 分类
1. 系统变量：
   * 全局变量
   * 会话变量
2. 自定义变量：
	* 用户变量
	* 局部变量
3. 全局变量需要添加`global`关键字，会话变量需要添加`session`关键字，如果不写，默认会话级别

## 6.2 使用
### 6.2.1 系统变量
1. 常用命令
   1. 查看所有系统变量
      ```sql
      show global|session variables;
      ```
   2. 查看满足条件的部分系统变量
      ```sql
      show global|session variables like '%char%';
      ```
   3. 查看指定的系统变量的值
      ```sql
      select @@global|session.系统变量名;
      ```
   4. 为某个系统变量赋值
      * 方式一：
         ```sql
         set global|session 系统变量名=值;
         ```
      * 方式二：
         ```sql
         set @@global|session.系统变量名=值;
         ```

2. 全局变量  
   作用域：针对于所有会话（连接）有效，但不能跨重启
   * 查看所有全局变量
      ```sql
      SHOW GLOBAL VARIABLES;
   * 查看满足条件的部分系统变量
      ```sql
      SHOW GLOBAL VARIABLES LIKE '%char%';
   * 查看指定的系统变量的值
      ```sql
      SELECT @@global.autocommit;
   * 为某个系统变量赋值
      ```sql
      SET @@global.autocommit=0;
      SET GLOBAL autocommit=0;

3. 会话变量  
   作用域：针对于当前会话（连接）有效
   * 查看所有会话变量
      ```sql
      SHOW SESSION VARIABLES;
   * 查看满足条件的部分会话变量
      ```sql
      SHOW SESSION VARIABLES LIKE '%char%';
   * 查看指定的会话变量的值
      ```sql
      SELECT @@autocommit;
      SELECT @@session.tx_isolation;
   * 为某个会话变量赋值
      ```sql
      SET @@session.tx_isolation='read-uncommitted';
      SET SESSION tx_isolation='read-committed';
      ```

### 6.2.2 自定义变量
1. 使用步骤：
   1. 声明
   2. 赋值
   3. 使用（查看、比较、运算等）
2. 用户变量  
   作用域：针对于当前会话（连接）有效，作用域同于会话变量
   * 声明并初始化
      ```sql
      SET @变量名=值;
      SET @变量名:=值;
      SELECT @变量名:=值;

   * 赋值（更新变量的值）
      ```sql
      #方式一：
      SET @变量名=值;
      SET @变量名:=值;
      SELECT @变量名:=值;

      #方式二：
      SELECT 字段 INTO @变量名
      FROM 表;
      ```
   * 使用（查看变量的值）
      ```sql
      SELECT @变量名;
3. 局部变量    
   作用域：仅仅在定义它的begin end块中有效，应用在 begin end中的第一句话
   * 声明
      ```sql
      DECLARE 变量名 类型;
      DECLARE 变量名 类型 [DEFAULT 值];
      ```
   * 赋值（更新变量的值）
      ```sql
      #方式一：
         SET 局部变量名=值;
         SET 局部变量名:=值;
         SELECT 局部变量名:=值;
      #方式二：
         SELECT 字段 INTO 变量名
         FROM 表;
      ```
   * 使用（查看变量的值）
      ```sql
      SELECT 局部变量名;
      ```

# Chapter7 存储过程
## 7.1 概念
1. 定义  
   一组预先编译好的SQL语句的集合，理解成批处理语句
2. 优点  
   1. 提高代码的重用性
   2. 简化操作
   3. 减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率

## 7.2 使用
### 7.2.1 创建语法
1. 定义
   ```sql
   CREATE PROCEDURE 存储过程名(参数列表)
   BEGIN

      存储过程体（一组合法的SQL语句）
   END
   ```
2. 说明  
   1. 参数列表包含三部分  
      ```sql
      参数模式  参数名  参数类型
      # 举例：
      in stuname varchar(20)
      ```
   1. 参数模式：
      * `in`：该参数可以作为输入，也就是该参数需要调用方传入值
      * `out`：该参数可以作为输出，也就是该参数可以作为返回值
      * `inout`：该参数既可以作为输入又可以作为输出，也就是该参数既需要传入值，又可以返回值

   2. 如果存储过程体仅仅只有一句话，`begin` `end`可以省略  
   存储过程体中的每条sql语句的结尾要求必须加分号, 存储过程的结尾可以使用 `delimiter` 重新设置  
      ```sql
      delimiter 结束标记
      # 例：
      delimiter $
      ```
3. 调用
   ```sql
   CALL 存储过程名(实参列表);
   ```

### 7.2.2 删除与查看存储过程
1. 删除
   ```sql
   drop procedure 存储过程名;
   ```
2. 查看
   ```sql
   SHOW CREATE PROCEDURE  存储过程名;
   ```

### 7.2.3 代码示例
1. 根据输入的女神名，返回对应的男神名和魅力值
   ```sql
   CREATE PROCEDURE myp7(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20),OUT usercp INT) 
   BEGIN
      SELECT boys.boyname ,boys.usercp INTO boyname,usercp
      FROM boys 
      RIGHT JOIN
      beauty b ON b.boyfriend_id = boys.id
      WHERE b.name=beautyName ;
      
   END $
   ```
   ```sql
   #调用
   CALL myp7('小昭',@name,@cp) $
   SELECT @name,@cp $
   ```

2. 传入a和b两个值，最终a和b都翻倍并返回
   ```sql
   CREATE PROCEDURE myp8(INOUT a INT ,INOUT b INT)
   BEGIN
      SET a=a*2;
      SET b=b*2;
   END $

   #调用
   SET @m=10$
   SET @n=20$
   CALL myp8(@m,@n) $
   SELECT @m,@n $
   ```

# Chapter8 函数
## 8.1 概述
1. 与存储过程区别
   * 存储过程：可以有0个返回，也可以有多个返回，适合做批量插入、批量更新
   * 函数：有且仅有1 个返回，适合做处理数据后返回一个结果

## 8.2 语法
1. 创建语法
   ```sql
   CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型
   BEGIN
      函数体
   END
   ```
   * 参数列表：参数名 参数类型
   * 函数体：必须有return语句
   * 函数体中仅有一句话，则可以省略begin end
   * 使用 delimiter语句设置结束标记
2. 调用语法
   ```sql
   SELECT 函数名(参数列表)
   ```
3. 查看函数
   ```sql
   SHOW CREATE FUNCTION myf3;
   ```
4. 删除函数
   ```sql
   DROP FUNCTION myf3;
   ```

## 8.3 代码示例
1. 根据员工名，返回它的工资
   ```sql
   DELIMITER $
   CREATE FUNCTION myf2(empName VARCHAR(20)) RETURNS DOUBLE
   BEGIN
      SET @sal=0;#定义用户变量 
      SELECT salary INTO @sal   #赋值
      FROM employees
      WHERE last_name = empName;
      
      RETURN @sal;
   END $

   SELECT myf2('k_ing') $
   ```
2. 根据部门名，返回该部门的平均工资
   ```sql
   CREATE FUNCTION myf3(deptName VARCHAR(20)) RETURNS DOUBLE
   BEGIN
      DECLARE sal DOUBLE ;
      SELECT AVG(salary) INTO sal
      FROM employees e
      JOIN departments d ON e.department_id = d.department_id
      WHERE d.department_name=deptName;
      RETURN sal;
   END $

   SELECT myf3('IT')$
   ```

# Chapter9 流程控制结构
## 9.1 分支结构
1. `if` 函数  
   ```sql
   IF(表达式1, 表达式2,表达式3 )
   ```
   表达式1成立则执行2，否则执行3
2. case结构
   * 类似Java中的switch语句
      ```sql
      case 变量或表达式
      when 值1 then 语句1;
      when 值2 then 语句2;
      ...
      else 语句n;
      end 
      ```
   * 类似Java中的多重IF语句，区间判断
      ```sql
      case 
      when 条件1 then 语句1;
      when 条件2 then 语句2;
      ...
      else 语句n;
      end 
      ```
3. `if` 结构
   ```sql
   if 条件1 then 语句1;
   elseif 条件2 then 语句2;
   ....
   else 语句n;
   end if;
   ```
   * 功能：类似于多重if
   * 只能应用在`begin` `end` 中

## 9.2 循环结构
1. while
   ```sql
   [标签] while 循环条件 do
	循环体;
   end while [标签];
   ```
2. loop
   ```sql
   [标签:] loop
	循环体;
   end loop [标签];
   ```
   `loop`可以用来模拟简单的死循环
3. repeat
   ```sql
   [标签:] repeat
	   # 循环体;
   until 结束循环的条件
   end repeat [标签];
   ```
4. 循环控制：
   * `iterate`类似于 `continue`，继续，结束本次循环，继续下一次
   * `leave` 类似于 `break`，跳出当前所在的循环

