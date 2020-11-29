---
title: Chapter2 数组
excerpt: 数组基础
tags:
  - java
categories:
  - Java笔记
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 6e0b8153
date: 2020-09-29 16:35:55
updated: 2020-10-21 01:23:55
subtitle:
---
## 2.1 数组概述
* 引用数据类型
* 内存中连续的空间
* 长度不能修改
## 2.2 一维数组
### 1. 声明与初始化
```java
// 1. 声明
int[] num; 

//2. 初始化
// 静态初始化
num = new int[]{1001,1002,1003,1004};
// 动态初始化
String[] names = new String[5];
```
默认初始化值：
* 整型: 0
* 浮点型: 0.0
* 字符: 0
* boolean: `false`
* String: `null`
### 2. 内存解析
1. 局部变量：栈(stack)
2. new-对象、数组：堆(heap)
3. 常量池、静态域：方法区
## 2.3 多维数组
### 1. 声明与初始化
```java
// 静态初始化
int[][] arr = new int[][]{{1,2,3},{1,2},{2,5}};
// 动态初始化1
String[][] arr2 = new String[3][2]
//动态初始化2
String[][] arr2 = new String[3][]//arr2[i]=null
```
### 2. 数组长度
```java
arr.length;
arr[1].length;
```
### 3. 数组常见算法
1. 冒泡排序
2. 快速排序
## 2.4 Arrays工具类
1. `boolean equals(array1,array2)`  
判断两个数组是否相等, `boolean`
2. `String toString(array)`
```java
Arrays.toString(array1);
```
3. `void fill(array, int b)`  
将指定值填充的数组中（即将数组中元素替换为该值）
4. `void sort(array)`  
对数组进行排序
5. `int binarySearch(array, int key)`  
寻找元素，返回索引位置，未找到则为负
## 2.5 数组常见异常
1. 角标越界异常
2. 空指针异常
