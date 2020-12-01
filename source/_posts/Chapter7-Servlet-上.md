---
title: Chapter7 Servlet(上)
excerpt: 摘要
tags:
  - java
  - JavaWeb
  - servlet
  - http
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 3a4b154
date: 2020-11-29 22:00:24
updated: 2020-11-29 22:00:24
subtitle:
---
## 7.1 Servlet入门
### 7.1.1 什么是Servlet
1. Servlet 是 JavaEE 规范之一。 规范就是接口
2. Servlet 就 JavaWeb 三大组件之一。 三大组件分别是： Servlet 程序、 Filter 过滤器、 Listener 监听器。
3. Servlet 是运行在服务器上的一个 java 小程序， 它可以接收客户端发送过来的请求， 并响应数据给客户端

### 7.1.2 如何实现
1. 编写一个类去实现 Servlet 接口
2. 实现 service 方法， 处理请求， 并响应数据
3. 到 web.xml 中去配置 servlet 程序的访问地址
4. 代码示例
   * 新建一个web工程，添加servlet相关jar包：
     * 从tomcat的lib中将servlet-api.jar包拷贝到项目的web/lib文件夹下
     * 右键servlet-api.jar->Add As Library或在项目目录里添加
   * 新建HelloServlet类继承Servlet接口并重写相关方法
        ```java
        package com.atguigu.servlet;

        import javax.servlet.*;
        import java.io.IOException;

        public class HelloServlet implements Servlet{

            @Override
            public void init(ServletConfig servletConfig) throws ServletException {
            }

            @Override
            public ServletConfig getServletConfig() {
                return null;
            }

            @Override
            public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
                System.out.println("Hello Servlet被访问了");
            }

            @Override
            public String getServletInfo() {
                return null;
            }

            @Override
            public void destroy() {
            }
        }
        ```
   * 在web/WEB-INF/web.xml中添加配置
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
                version="4.0">

            <servlet>
                <servlet-name>helloServlet</servlet-name>
                <servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>
            </servlet>

            <servlet-mapping>
                <servlet-name>helloServlet</servlet-name>
                <url-pattern>/hello</url-pattern>
            </servlet-mapping>

        </web-app>
        ```
        * servlet标签：给 Tomcat 配置 Servlet 程序
          * servlet-name 标签 Servlet 程序起一个别名（一般是类名）
          * servlet-class 是 Servlet 程序的全类名
        * servlet-mapping标签：给 servlet 程序配置访问地址
          * servlet-name 标签的作用是告诉服务器， 当前配置的地址给哪个 Servlet 程序使用
          * url-pattern 标签：/hello 表示地址为： http://ip:port/工程路径/hello

### 7.1.3 Servlet 的生命周期

1. 执行 Servlet 构造器方法：第一次访问url的时候创建 Servlet 程序会调用
2. 执行 init 初始化方法：同上
3. 执行 service 方法：每次url访问都会调用
4. 执行 destroy 销毁方法：在 web 工程停止的时候调用

### 7.1.4 GET 和 POST 请求的分发处理
1. service方法可以获取页面返回的请求类型
   ```java
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("Hello Servlet被访问了");
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String method = httpServletRequest.getMethod();
    }
   ```
2. 可以写一个专门的方法应对不同的请求
   ```java
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("Hello Servlet被访问了");
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String method = httpServletRequest.getMethod();
        if (method.equals("GET") )
            doGet();
        else if (method.equals("POST"))
            doPost();
    }
    public void doGet(){
        System.out.println("get");
    }

    public void doPost(){
        System.out.println("post");
    }
   ```

### 7.1.5 通过继承 HttpServlet 实现 Servlet 程序
1. 一般在实际项目开发中， 都是使用继承 `HttpServlet` 类的方式去实现 `Servlet` 程序。
2. 步骤
   * 编写一个类去继承 `HttpServlet` 类
   * 根据业务需要重写 `doGet` 或 `doPost` 方法
   * 到 `web.xml` 中的配置 `Servlet` 程序的访问地址

### 7.1.6 使用 IDEA 创建 Servlet 程序
1. 包名上右键-> Create New Servlet->类名
2. web.xml中自动添加了servlet标签，只需再添加servlet-mapping标签
3. 修改相关html提交url即可

### 7.1.7 Servlet 类的继承体系
1. `Servlet`接口：定义了Servlet程序的访问规范
2. `GenericServlet`实现类：很多空实现
3. `HttpServlet`子类：实现了`service()`方法，并实现了分发处理
4. 自定义Servlet程序：继承`HttpServlet`类，根据业务需要重写`doGet`与`doPost`方法

## 7.2 `ServletConfig` 类
### 7.2.1 说明
1. `ServletConfig` 类是`Servlet` 程序的配置信息类。
2. `Servlet` 程序和 `ServletConfig` 对象都是由 `Tomcat` 负责创建， 我们负责使用。
3. `Servlet` 程序默认是第一次访问的时候创建， `ServletConfig` 是每个 `Servlet` 程序创建时， 就创建一个对应的 `ServletConfig` 对象。

### 7.2.2 `ServletConfig` 类的三大作用
1. 作用
   * 可以获取 Servlet 程序的别名 servlet-name 的值
   * 获取初始化参数 init-param
   * 获取 ServletContext 对象
2. 代码
   ```java
   // HelloServlet.java
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("2. 初始化");
        // 1. 获取程序别名
        System.out.println("HelloServlet程序的别名是："+servletConfig.getServletName());
        // 2. 获取初始化参数username
        System.out.println("初始化参数username是："+servletConfig.getInitParameter("username"));
        // 获取初始化参数url
        System.out.println("初始化参数url是："+servletConfig.getInitParameter("url"));
        //3. 获取ServletContext对象
        System.out.println(servletConfig.getServletContext());
    }

   ```
   web.xml:
   ```xml
    <servlet>
        <!--程序别名-->
        <servlet-name>HelloServlet</servlet-name>
        <!--全类名-->
        <servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>
        <!--初始化参数-->
        <init-param>
            <!--参数名-->
            <param-name>username</param-name>
            <!--参数值-->
            <param-value>root</param-value>
        </init-param>
        <init-param>
            <param-name>url</param-name>
            <param-value>jdbc:mysql://localhost:3306/test</param-value>
        </init-param>
    </servlet>
   ```
3. 注意：  
   * 继承 `HttpServlet` 类实现 `Servlet` 程序时，如果重写init方法，一定要在第一句写上`super.init();`
   * 因为 `HttpServlet` 类的`init`方法有一个保存Config的操作，不继承会丢失。

## 7.3 `ServletContext` 类
### 7.3.1 什么是 `ServletContext`?
1. `ServletContext`
   * `ServletContext` 是一个接口， 它表示 `Servlet` 上下文对象
   * 一个 `web` 工程， 只有一个 `ServletContext` 对象实例。
   * `ServletContext` 对象是一个域对象。
   * `ServletContext` 是在 web 工程部署启动的时候创建。 在 web 工程停止的时候销毁。

2.  域对象：可以像 Map 一样存取数据的对象，叫域对象。这里的域指的是存取数据的操作范围， 整个 web 工程    
  
    &nbsp;|存数据|取数据|删除
    :-:|:-:|:-:|:-:
    Map|put()|get()|remove
    域对象|setAttribute()|getAttribute()|removeAttribute()
<br/> 

### 7.3.2 `ServletContext` 类的四个作用
1. 作用
   * 获取 web.xml 中配置的上下文参数 context-param
   * 获取当前的工程路径， 格式: /工程路径
   * 获取工程部署后在服务器硬盘上的绝对路径
   * 像 Map 一样存取数据

2. 代码示例
   * 前三个作用
        ```java
        // ContextServlet
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            // 1. 获取web.xml中配置的上下文参数context-param
            ServletContext context = getServletConfig().getServletContext();

            String username = context.getInitParameter("username");
            System.out.println("context-param参数username的值是："+username);
            System.out.println("context-param参数password的值是："+ context.getInitParameter("password"));

            // 2. 获取当前的工程路径，格式为：/工程路径
            System.out.println("当前工程路径"+ context.getContextPath());

            // 3. 获取工程部署后在服务器硬盘上的绝对路径
            System.out.println("工程部署的路径是："+context.getRealPath("/"));
            System.out.println("工程下css目录的绝对路径是："+context.getRealPath("/css"));
            System.out.println("工程下imgs目录的绝对路径是："+context.getRealPath("/imgs/1.jpg"));
        }
        ```
        web.xml
        ```xml
            <!--上下文参数，属于整个web工程-->
        <context-param>
            <param-name>username</param-name>
            <param-value>context</param-value>
        </context-param>
        <context-param>
            <param-name>password</param-name>
            <param-value>root</param-value>
        </context-param>
        ....
            <servlet>
            <servlet-name>ContextServlet</servlet-name>
            <servlet-class>com.atguigu.servlet.ContextServlet</servlet-class>
        </servlet>
        ....
            <servlet-mapping>
            <servlet-name>ContextServlet</servlet-name>
            <url-pattern>/contextservlet</url-pattern>
        </servlet-mapping>
        ```

    * 存取数据：
        ```java
        // ContextServlet1
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            ServletContext context = getServletContext();

            context.setAttribute("key1", "value");
            System.out.println(context.getAttribute("key1"));
        }

        // ContextServlet2
            protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            ServletContext context = getServletContext();
            System.out.println("Context2 中获取域数据 key1 的值是:"+ context.getAttribute("key1"));
        }
        ```
        web.xml:
        ```xml
        <servlet>
            <servlet-name>ContextServlet1</servlet-name>
            <servlet-class>com.atguigu.servlet.ContextServlet1</servlet-class>
        </servlet>
        <servlet>
            <servlet-name>ContextServlet2</servlet-name>
            <servlet-class>com.atguigu.servlet.ContextServlet2</servlet-class>
        </servlet>
        ...
        <servlet-mapping>
            <servlet-name>ContextServlet1</servlet-name>
            <url-pattern>/contextservlet1</url-pattern>
        </servlet-mapping>

        <servlet-mapping>
            <servlet-name>ContextServlet2</servlet-name>
            <url-pattern>/contextservlet2</url-pattern>
        </servlet-mapping>
        ```
        * 加载contextservlet1前加载contextservlet2：得到null
        * 加载contextservlet1：value
        * 加载contextservlet1后加载contextservlet2：value

## 7.4 HTTP协议
### 7.4.1 什么是HTTP协议
1. 协议：协议是指双方， 或多方， 相互约定好， 大家都需要遵守的规则
2. HTTP协议
   * 所谓 HTTP 协议， 就是指， 客户端和服务器之间通信时， 发送的数据， 需要遵守的规则， 叫 HTTP 协议。
   * HTTP 协议中的数据又叫报文

### 7.4.2 请求HTTP的协议格式
1. 请求与相应
   * 客户端给服务器发送数据叫请求。
   * 服务器给客户端回传数据叫响应
2. `GET` 请求
   * 请求行
     * 请求的方式：GET
     * 请求的资源路径[+?+请求参数]
     * 请求的协议版本号：HTTP/1.1
   * 请求头
     * 由key:value 组成，不同的键值对代表不同含义
     * 示例
       * Accept：告诉服务器，客户端可以接收的数据类型
       * Accept-Language：客户端可以接收的语言类型zh_CN、en_US
       * User-Agent：浏览器信息
       * Accept-Encoding：客户端可以接收的数据编码格式
       * Host：请求的服务器ip和端口号
       * Connection：请求连接如何处理
         * Keep-Alive：回传数据后保持一小段时间的连接
         * Closed：马上关闭
3. `POST` 请求
   * 请求行
     * 请求的方式：GET
     * 请求的资源路径[+?+请求参数]
     * 请求的协议版本号：HTTP/1.1
   * 请求头：由 `key:value` 组成，不同的键值对代表不同含义
   * 空行
   * 请求体：发送给服务器的数据

4.  `GET` 请求与 `POST` 请求应用场景
   * GET 请求：
     * form 标签 method=get
     * a 标签
     * 标签引入 css
     * Script 标签引入 js 文件
     * img 标签引入图片
     * iframe 引入 html 页面
     * 在浏览器地址栏中输入地址后敲回车
   * POST请求：
     * form 标签 method=post

### 7.4.3 响应的 HTTP 协议格式
1. 响应行
   * 响应的协议和版本号
   * 响应状态码
   * 响应状态描述符
2. 响应头：`key : value` 
3. 空行
4. 响应体：回传给客户端的数据

### 7.4.4 常用的响应码说明
1. 200 表示请求成功
2. 302 表示请求重定向（明天讲）
3. 404 表示请求服务器已经收到了， 但是你要的数据不存在（请求地址错误）
4. 500 表示服务器已经收到请求， 但是服务器内部错误（代码错误）

### 7.4.5 MIME 类型说明
1. 概念
   * MIME 是 HTTP 协议中数据类型。
   * MIME 的英文全称是"Multipurpose Internet Mail Extensions" 多功能 Internet 邮件扩充服务。 MIME 类型的格式是“大类型/小类型” ， 并与某一种文件的扩展名相对应。
2. 常见的 MIME 类型
    文件| MIME 类型
    :-:|:-:
    超文本标记语言文本| .html , .htm text/html
    普通文本| .txt text/plain
    RTF 文本| .rtf application/rtf
    GIF 图形| .gif image/gif
    JPEG 图形| .jpeg,.jpg image/jpeg
    au 声音文件| .au audio/basicMIDI 
    音乐文件| mid,.midi audio/midi,audio/x-midi
    RealAudio 音乐文件| .ra, .ram audio/x-pn-realaudio
    MPEG 文件| .mpg,.mpeg video/mpeg
    AVI 文件| .avi video/x-msvideo
    GZIP 文件| .gz application/x-gzip
    TAR 文件| .tar application/x-tar

### 7.4.6 浏览器如何查看HTTP 协议
1. F12打开控制台
2. Network里面




