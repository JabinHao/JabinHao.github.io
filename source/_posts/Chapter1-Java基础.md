---
title: Chapter1 Java基础
excerpt: 常用DOS命令、Java概述及基本数据类型等
tags:
  - java
categories:
  - Java笔记
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: ac680975
date: 2020-09-23 23:15:29
updated: 2020-10-21 01:21:29
subtitle:
---
## 1. 常用DOS命令
* dir：ls
* md：mkdir
* rd：rm -r
* cd：cd
* cd..：cd ..
* cd\：cd \
* del：rm
* exit：exit
* echo：echo
* e: ：进入E盘

## 2. Java概述
### 1. 注释
* 单行注释：//
* 多行注释：/* 内容 */
* 文档注释：/** 内容 */  
文档注释可以被javadoc解析，生成一套说明文档（网页形式）：
```sh
javadoc -d 目录名 -author -version xxx.java
```
### 2. 文件与类
一个.java文件中：
* 可以有多个类
* 只能有一个 `public class`
* 文件名与 (`public`) 类同名

### 3. 输入输出
```java
System.out.println(); //输出后换行
System.out.print();   //输出不换行
```
## 3. Java基础
### 3.1 关键字与保留字
1. 关键字(keyword)
2. 保留字(reserved word)  
`goto, const`
### 3.2 标识符(Identifier)
#### **1. 规则：**
* 以字母、下划线、$开头
* 由字母、数字、下划线、$组成
* 不能包含空格
#### **2. 规范：**
* 包名：多单词组成时全部小写 xxyyzz
* 类名、接口名多单词组成时首字母全部大写 AaBbCc
* 变量名、方法名：第一个单词小写，其它首字母大写：`goodEnough`
* 常量：所有字母都大写，多单词下划线连接：AA_BB_CC
### 3.3 变量
#### 1. 规则
   *  先声明后使用
   *  有作用域
#### 2. 数据类型
   1. 基本数据类型：
      *   数值：
          *   整型：`byte、 short、 int、 long`
          *   浮点型：`float、 double`
      *   字符型：`char`
      *   布尔型：`boolean`
   2. 引用数据类型：
      * 类 `class`
      * 接口 `interface`
      * 数组 `array`  
  
##### 2.1 整型
类型|大小|表数范围
:-:|:-:|:-:|
byte|1字节|-128~127
short|2字节|-$2^{15}$~$2^{15}$-1
int|4字节|-$2^{31}$~$2^{31}$-1
long|8字节|-$2^{63}$~$2^{63}$-1
* **声明long型变量时，必须以 “l” 或 “L”结 尾**
* 通常使用 `int` 型
##### 2.2 浮点型
* float：单精度，7位有效数字，**以 “f” 或 “F” 结尾**
* double：双精度，**常用此**

类型|大小|表数范围
:-:|:-:|:-:|
float|4字节|-3.403E38~3.403E38
double|8字节|-1.798E308~1.798E308
##### 2.3 字符型
规则：
* 只能包含一个字符，可以是中文
* 转义字符
* 可以与字符串相加
* Unicode：char c = `'\u0043'`
##### 2.4 布尔类型
* `true、false`
* 定义：`boolean isMarried = true;`
##### 2.5 基本数据类型间运算
* 自动类型提升
  * `char + int = int`
  * `char + char = int`
  * 常量默认`int`
* 强制类型转换  
  `int q = (int)c`

#### 2.6 `String`类型
* 属于引用数据类型, 字符串
* 使用双引号 ""
* 长度无限制，可以为空
* String可以和所有(8种)基本数据类型相加, 效果为连接

#### 3. 进制
* 二进制(binary) 0b或0B开头
* 八进制(octal) 0开头
* 十六进制(hex) 0x或0X开头, 0-9+A-F
### 3.4 运算符
#### 1. 算术运算符
* +、-、*、/
* %：取模
* ++a
* a++
* --a
* a--
* +：连接符
#### 2. 赋值运算符的
=、+=、-=、*=、/=、%=
#### 3. 比较运算符
* ==
* !=
* <
* \>
* <=
* \>=
* instanceof：检查是否是类的对象
#### 4. 逻辑运算符
* &
* |
* !
* &&：短路与，前者假则不执行后者
* ||
* ^：a、b不同时为真
#### 5. 位运算符
* \>>：右移，即除以$2^x$
* <<：左移，即乘以$2^x$
* \>>>：无符号右移
* &：两数字二进制位与操作
* |：两数字二进制位或操作
* ^：
* ~：二进制码取反(包括符号位)
#### 6. 三元运算符
`(条件表达式)?表达式一:表达式二`
### 3.5 程序流程控制
#### 1. if else
```java
if(){

}
else if(){

}
else{

}
```
#### 2. 输入
* 导包
* 实例化Scanner
* 调用相关方法
```java
import java.util.Scanner;

class ScannerTest{
   public static void main(String[] args){
      Scanner scan = new Scanner(System.in);
      int num = scan.nextInt();
   }
}
```
#### 3. switch - case
1. 格式
```java
switch(){
case 1:
   语句；
   break;
case 2:
   语句;
   break;
···
default:
   执行语句;
}
```
2. 结构表达式只能为：  
   `byte、short、char、int、枚举类型、String`
#### 4. 循环结构
1. `for`循环
```java
for(;;){

}
```
2. `while` 循环
3. `do while`循环
```java
do{

}while();
```
#### 5. `break` 与 `continue`
1. 带标签的 `break` 与 `continue`
```java
label:for(;;){
   ···
   for(;;){
      ···
      if(){
         ···
         break label;
      }
   }
}
```

