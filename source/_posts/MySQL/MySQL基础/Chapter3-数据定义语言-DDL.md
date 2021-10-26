---
title: Chapter3 数据定义语言(DDL)
excerpt: 库和表的管理（增删改）、常见数据类型、常见约束
tags:
  - mysql
categories:
  - MySQL
  - MySQL基础
banner_img: /img/road.png
index_img: /img/post/Mysql/mysql_logo.svg
abbrlink: a526b75
date: 2020-11-12 23:44:33
updated: 2020-11-12 23:44:33
subtitle:
---
## 3.1 库和表的管理
### 3.1.1 创建库
```sql
CREATE DATABASE IF NOT EXISTS books;
```
### 3.1.2 修改库
1. 修改库名（已废弃）
   ```sql
   RENAME DATABASE books TO 新库名;
   ```
2. 更改字符集
   ```sql
   ALTER DATABASE books CHARACTER SET gbk;
   ```

### 3.1.3 库的删除
```sql
DROP DATABASE IF EXISTS books;
```

### 3.1.4 表的创建
1. 语法
   ```sql
   CREATE TABLE book(
       列名 类型 [（长度）（约束）],
       列名 类型 [（长度）（约束）],
       ...
   )
   ```
2. 示例
   ```sql
   CREATE TABLE author(
       id INT,
       au_name VARCHAR(20),
       nation VARCHAR(10)
   )
   ```

### 3.1.5 表的修改
1. 修改列名
   ```sql
   ALTER TABLE 表名 CHANGE COLUMN 列名 新列名 类型;
   ```
2. 修改列的类型或约束
   ```sql
   ALTER TABLE 表名 MODIFY COLUMN 列名 类型;
   ```
3. 添加新列
   ```sql
   ALTER TABLE 表名 ADD COLUMN 列名 类型;
   ```
4. 删除列
   ```sql
   ALTER TABLE 表名 DROP COLUMN 列名;
   ```
5. 修改表名
   ```sql
   ALTER TABLE 表名 RENAME TO 新表名
   ```

### 3.1.6 表的删除
1. 命令
   ```SQL
   DROP TABLE IF EXISTS 表名;
   ```

### 3.1.7 表的复制
1. 仅仅复制表的结构
   ```sql
   CREATE TABLE 新表名 LIKE 原表名
   ```
2. 复制表的结构+数据
   ```sql
   CREATE TABLE 新表名
   SELECT * FROM 原表;
   ```
3. 复制部分数据
   ```SQL
   CREATE TABLE 新表名
   SELECT 字段1, 字段2, ...
   FROM 原表
   WHERE 条件;
   ```
4. 复制部分字段（列）
   ```SQL
   CREATE TABLE 新表名
   SELECT 字段1, 字段2, ...
   FROM 原表
   WHERE 0;
   ```

## 3.2 常见数据类型
### 3.2.1 整型
1. 分类
   ```sql
      tinyint、smallint、mediumint、int/integer、bigint
   ```
2. 特点
   1. 默认有符号，无符号用 unsigned定义：`INT UNSIGNED`
   2. 插入数值超出范围会报错（老版本会插入临界值）
   3. 若不设置长度则会有默认长度，代表显示的宽度，必须搭配`zerofill`使用(不够使用0填充)
      ```sql
      INT(7) ZEROFILL,
      ```

### 3.2.2 浮点型
1. 分类
   1. 浮点型
      * float(M,D)
      * double(M,D)
   2. 定点型
      * dec(M,D)
      * decimal(M,D)
2. 特点
   1. M与D
      * M：整数位数+小数位数
      * 小数位数
   2. M与D可省略
      * decimal：M=10，D=0
      * 浮点型：根据输入数值

### 3.2.3 字符型
1. 分类
   1. 短文本
      * char：固定长度
      * varchar：可变长度
   2. 长文本
      * text
      * blob
2. 使用
   1. 长度
      * char(M), M可省略，默认为1
      * varchar(M), M不能省略
   2. 其他
      * binary、varbinary：用于保存较短的二进制
      * enum：保存枚举
      * set：保存集合

### 3.2.4 日期型
1. 分类
   1. date：日期
   2. time：时间
   3. year：年
   4. datetime：日期+时间
   5. timestamp：日期+时间
2. 特点
   1. datetime
      * 8字节
      * 1000-9999
      * 不受时区影响
   2. timestamp
      * 4字节
      * 1970-2038
      * 受时区影响

## 3.3 常见约束
### 3.3.1 分类
1. `NOT NULL`：非空
2. `DEFAULT`：默认，保证有默认值
3. `PRIMARY KEY`：主键，具有唯一性，非空
4. `UNIQUE`：唯一，可以为空
5. `CHECK`：检查约束(mysql不支持)
6. `FOREIGN`：外键，限制两个表的关系，保证该字段值来自主表关联列

### 3.3.2 使用
1. 添加时机
   * 创建表时
   * 修改表时
2. 分类
   * 列级约束：六个皆可，但外键约束没有效果
   * 表级约束：除了非空、默认
3. 主键与唯一键  
   类型|唯一性|能否为空|能否多个|允许组合
   :-:|:-:|:-:|:-:|:-:
   主键|是|否|否|允许, 不推荐
   唯一键|是|是|是|允许, 不推荐
4. 外键
   * 在从表设置外键关系
   * 从表列与主表关联列类型要求一致或兼容
   * 主表关联列必须是KEY(一般是主键或唯一)
   * 插入数据时，先插入主表再插入从表，删除数据时，先删从表再删主表

### 3.3.3 代码示例
1. 创建表时
   ```sql
   # 1. 列级约束
   CREATE TABLE student(
      id INT PRIMARY KEY, # 主键
      stuName VARCHAR(20) NOT NULL, # 非空
      gender CHAR(1) CHECK(gender='男' OR gender='女'), # 检查
      seat INT UNIQUE, # 唯一
      age INT DEFAULT 18 # 默认
   )
   ```
   ```sql
   CREATE TABLE student(
      id INT,
      stuName VARCHAR(20),
      gender CHAR(1),
      seat INT,
      age INT,
      majorid INT,

      CONSTRAINT pk PRIMARY KEY(id), # 主键
      CONSTRAINT uq UNIQUE(seat), #唯一键
      CONSTRAINT ck CHECK gender='男' OR gender='女'), # 检查
      CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id), # 外键
   );

   # pk uq等为键名，可以省略
   # 主键一般命名为 fk_从表_主表
   ```

2. 修改表时添加约束
   * 列级约束
      ```sql
      ALTER TABLE 表名 MODIFY COLUMN 列名 类型 约束
      ```
   * 表级约束
      ```sql
      ALTER TABLE 表名 ADD [CONSTRAINT 键名] 约束
      ```
3. 修改表时删除约束
   ```sql
   # 1. 删除非空约束
   ALTER TABLE 表名 MODIFY COLUMN 列名 类型;
   # 2. 删除默认约束
   ALTER TABLE 表名 MODIFY COLUMN 列名 类型;
   # 3. 删除主键
   ALTER TABLE 表名 DROP PRIMARY KEY;
   # 4. 删除唯一
   ALTER TABLE 表名 DROP INDEX 键名;
   # 5. 删除外键
   ALTER TABLE DROP FOREIGN KEY 外键名;
   ```

### 3.3.4 标识列
1. 说明  
   也叫自增长列，系统提供默认序列值
2. 特点
   * 必须是一个可以且只有一个
   * 标识列只能是数值型
   * 步长可以通过`SET auto_increment_increment = N设置`，起始值可以通过手动插入设置
   * 插入数据时，标识列可以为NULL、数值或直接省略