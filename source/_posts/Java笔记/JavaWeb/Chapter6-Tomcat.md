---
title: Chapter6 Tomcat
excerpt: 摘要
tags:
  - java
  - JavaWeb
  - Tomcat
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 4baf78ba
date: 2020-11-29 21:48:22
updated: 2020-11-29 21:56:22
subtitle:
---
## 6.1 概述
### 6.1.1 JavaWeb 的概念
1. JavaWeb
   * JavaWeb 是指， 所有通过 Java 语言编写可以通过浏览器访问的程序的总称， 叫 JavaWeb。
   * JavaWeb 是基于请求和响应来开发的
2. 请求与响应
   * 请求是指客户端给服务器发送数据， 叫请求 Request
   * 响应是指服务器给客户端回传数据， 叫响应 Response
   * 请求和响应是成对出现的， 有请求就有响应

### 6.1.2 Web 资源的分类
1. web 资源按实现的技术和呈现的效果的不同， 又分为静态资源和动态资源两种。
* 静态资源： html、 css、 js、 txt、 mp4 视频 , jpg 图片
* 动态资源： jsp 页面、 Servlet 程序

### 6.1.3 常用的 Web 服务器

* Tomcat： 由 Apache 组织提供的一种 Web 服务器， 提供对 jsp 和 Servlet 的支持。 它是一种轻量级的 javaWeb 容器（服务器） ， 也是当前应用最广的 JavaWeb 服务器（免费） 
* Jboss： 是一个遵从 JavaEE 规范的、 开放源代码的、 纯 Java 的 EJB 服务器， 它支持所有的 JavaEE 规范（免费）
* GlassFish： 由 Oracle 公司开发的一款 JavaWeb 服务器， 是一款强健的商业服务器， 达到产品级质量（应用很少）
* Resin： 是 CAUCHO 公司的产品， 是一个非常流行的服务器， 对 servlet 和 JSP 提供了良好的支持，性能也比较优良， resin 自身采用 JAVA 语言开发（收费，应用比较多） 
* WebLogic： 是 Oracle 公司的产品， 是目前应用最广泛的 Web 服务器，支持 JavaEE 规范，
而且不断的完善以适应新的开发要求， 适合大型项目（收费，用的不多 适合大公司） 

### 6.1.4 Tomcat 服务器和 Servlet 版本的对应关系

* 企业使用的jdk版本多为 7 和 8 
* Servlet 程序从 2.5 版本是现在世面使用最多的版本（xml 配置）
* Servlet3.0 以后为注解版本
* 后面以 2.5 版本为主线讲解 Servlet 程序

   Tomcat版本|Servlet/JSP版本|JavaEE版本/JDK版本
   :-:|:-:|:-:|:-:
   4.1|2.3/1.2|1.3|
   5.0|2.4/2.0|1.4|
   5.5/6.0|2.5/2.1|5.0
   7.0|3.0/2.2|6.0
   8.0|3.1/2.3|7.0

## 6.2 Tomcat的使用
### 6.2.1 安装
1. 到[Apache官网](https://tomcat.apache.org/)下载相应版本
2. 安装包文件目录：
   * bin 专门用来存放 Tomcat 服务器的可执行程序
   * conf 专门用来存放 Tocmat 服务器的配置文件
   * lib 专门用来存放 Tomcat 服务器的 jar 包
   * logs 专门用来存放 Tomcat 服务器运行时输出的日记信息
   * temp 专门用来存放 Tomcdat 运行时产生的临时数据
   * webapps 专门用来存放部署的 Web 工程。
   * work 是 Tomcat 工作时的目录， 用来存放 Tomcat 运行时 jsp 翻译为 Servlet 的源码， 和 Session 钝化的目录。

### 6.2.2 使用
1. 启动
    * 手动启动：到bin目录下点击startup.bat
    * 命令行启动：
      * cmd或powershell进入bin目录
      * 运行命令：`catalina run`
2. 停止
   * 控制台 ctrl+c
   * 双击bin下的shutdown.bat
3. 修改 Tomcat 端口号
   * Tomcat 默认的端口号是： 8080
   * conf 目录下的 server.xml 配置文件 
        ```xml
        <Connector port="8080" protocol="HTTP/1.1"connectionTimeout="20000"redirectPort="8443" />
        ```
        修改8080为想要的值（1-65535）
4. 问题
   * 控制台乱码
     * 编码问题，默认 UTF-8，而Windows默认GBK
     * 解决方法：打开conf目录下logging.properties文件，找到 `java.util.logging.ConsoleHandler.encoding = UTF-8`，将`UTF-8`改为`GBK`
   * 无法启动
     * 可能是没有配置JAVA_HOME 
     * 使用命令行方法会提示问题

### 6.2.3 部署
1. 方法一  
   * 把 web 工程的目录拷贝到 Tomcat 的 webapps 目录下
   * 浏览器通过`http://ip:port/工程名/目录下/文件名`地址访问
2. 方法二
   * 在 `conf/Catalina/localhost`目录下新建配置文件 store.xml
   * 文件配置内容：
        ```xml
        <!-- Context 表示一个工程上下文
        path 表示工程的访问路径:/abc
        docBase 表示你的工程目录在哪里
        -->
        <!-- 示例： -->
        <Context path="/bookstore" docBase="F:\book" />
        ```
        其中book为项目目录
   * 浏览器访问：`http://ip:port/abc/`，这里为 `http://localhost:8080/bookstore/`

3. 两种方法区别
   * 方法一（手托 html 页面到浏览器）使用的是 `file://` 协议
   * 方法二使用的是 `http` 协议

4. ROOT 的工程的访问
   * 在浏览器访问地址为：`http://ip:port/`时，访问的时root工程
   * root工程在 `webapps/ROOT` 目录下
5. 默认 index.html 页面
   * 浏览器访问地址为：`http://ip:port/工程名/`时，默认访问 index.html 页面

## 6.3 IDEA与Tomcat
### 6.3.1 IDEA 整合 Tomcat 服务器
1. 创建项目
2. 选择Run->Edit Configuration-> +号->Tomcat Server-Local
3. Application server右侧CONFIGURE找到Tomcat的目录（bin目录上一级）确定即可

### 6.3.2 IDEA 中动态 web 工程的操作
1. 首先创建工程或模块
2. 整合Tomcat服务器
3. 工程或模块名上右键->add framwork support->选中Web Application->默认勾选创建web.xml
4. 配置Tomcat（Edit Configuration）
   * 第二个选项卡Deployment->右边的加号->选择Artifact
   * Application context表示虚拟路径，建议改成项目或模块名
5. 启动即可

