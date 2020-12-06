---
title: IDEA相关
excerpt: IDEA使用过程中遇到的一些问题等
tags:
  - idea
categories:
  - Others
banner_img: /img/dog.png
index_img: /img/post/IDEA_logo.png
abbrlink: 659b75d3
date: 2020-12-06 20:55:43
updated: 2020-12-06 20:55:43
subtitle:
---
## 1. 配置



## 2. 常用快捷键
1. ctrl+alt+T：插入代码块
2. alt+insert：插入方法

## 3. 问题
1. junit无法在控制台输入
   * help->Edit Custom VM Options
   * 在最后添加`-Deditable.java.test.console=true`
   * 重启IDEA
2. 