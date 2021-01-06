---
title: Chapter1 Spring框架概述
excerpt: Spring框架概述，版本为Spring5x
tags:
  - java
categories:
  - Java笔记
  - Spring
banner_img: /img/dog.png
index_img: /img/post/spring-framework_logo.png
abbrlink: 1f7fa6d1
date: 2020-12-06 21:48:55
updated: 2020-12-06 21:48:55
subtitle:
---
1. 什么是Spring  
   * Spring 是轻量级的开源的 JavaEE 框架
   * 可以解决企业应用开发的复杂性
   * 在Spring Framework基础上，还有Spring Boot、Spring Cloud、Spring Data、Spring Security等一系列项目
   * 本章内容为Spring Framework

2. Spring核心部分
   * IOC：控制反转，把创建对象过程交给 Spring 进行管理
   * Aop：面向切面，不修改源代码进行功能增强
3. Spring的特点
   * 方便解耦，简化开发
   * Aop 编程支持
   * 方便程序测试
   * 方便和其他框架进行整合
   * 方便进行事务操作
   * 降低 API 开发难度
4. Spring 的使用（开发中一般直接配置）
   * 需要下载包
   * 新建java项目，导入相关jar包
   * 创建配置文件（`xml`文件）
   * 创建一个类`User`用于新建对象
   * 新建一个类用于测试
5. 简单的示例
   * bean1.xml
      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
         <!--配置User对象创建-->
         <bean id="user" class="com.atguigu.spring5.User"/>
      </beans>
      ```
   * User.java
      ```java
      public class User {
         public void add(){
            System.out.println("add....");
         }
      }
      ```
   * test
      ```java
      public void testAdd(){
         // 1. 加载配置文件
         ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
         // 2. 获取配置创建的对象
         User user = context.getBean("user",User.class);

         System.out.println(user);
         user.add();
      }
      ```

