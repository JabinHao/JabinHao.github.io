---
title: Chapter7 Servlet(下)
excerpt: HttpServletRequest类和HttpServletResponse类的使用
tags:
  - java
  - JavaWeb
  - servlet
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: a207d578
date: 2020-12-01 16:17:43
updated: 2020-12-01 16:17:43
subtitle:
---
## 7.5 `HttpServletRequest` 类
### 7.5.1 概述
* 请求进入 Tomcat 服务器， Tomcat 服务器就会把请求过来的 HTTP 协议信息解析好封装到 Request 对象中。
* Request 对象传递到 service 方法（ doGet 和 doPost） 中，可以通过 HttpServletRequest 对象， 获取到所有请求的信息。

### 7.5.2 常用方法
1. 方法
   * getRequestURI() 获取请求的资源路径
   * getRequestURL() 获取请求的统一资源定位符（绝对路径） 
   * getRemoteHost() 获取客户端的 ip 地址
   * getHeader() 获取请求头
   * getParameter() 获取请求的参数
   * getParameterValues() 获取请求的参数（多个值的时候使用）
   * getMethod() 获取请求的方式 GET 或 POST
   * setAttribute(key, value); 设置域数据\
   * getAttribute(key); 获取域数据
   * getRequestDispatcher() 获取请求转发对象

2. 示例1
   ```java
   // RequestAPIServlet.java
    public class RequestAPIServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            // 1. getRequestURI() 获取请求的资源路径
            System.out.println("URI: "+ req.getRequestURI());

            // 2. getRequestURL() 获取请求的统一资源定位符（绝对路径）
            System.out.println("URL: "+ req.getRequestURL());

            // 3. getRemoteHost() 获取客户端的 ip 地址
            System.out.println("客户端地址："+ req.getRemoteHost());

            // 4. 获取请求头
            System.out.println("请求头User-Agent："+ req.getHeader("User-Agent"));

            // 5. 获取请求方式
            System.out.println("请求方式："+req.getMethod());
        }
    }
   ```
   web.xml
   ```xml
    <servlet>
        <servlet-name>RequestAPIServlet</servlet-name>
        <servlet-class>com.atguigu.servlet.RequestAPIServlet</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>RequestAPIServlet</servlet-name>
        <url-pattern>/requestAPIServlet</url-pattern>
    </servlet-mapping>
   ```
   控制台输出：
   ```
   URI: /07_servlet/requestAPIServlet
   URL: http://localhost:8080/07_servlet/requestAPIServlet
   客户端地址：0:0:0:0:0:0:0:1
   请求头User-Agent：Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.66 Safari/537.36
   请求方式：GET
   ```

3. 示例2：获取请求参数
   * ParameterServlet.java
        ```java
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            String username = request.getParameter("username");
            String password = request.getParameter("password");
            String[] hobby = request.getParameterValues("hobby");

            System.out.println("用户名："+username);
            System.out.println("密码："+password);
            System.out.println("兴趣爱好："+ Arrays.asList(hobby));
        }
        ```
    * web.xml：正常配置
    * form.xml
    ```html
    <body>
    <form action="http://localhost:8080/07_servlet/parameterServlet" method="get">
        用户名：<input type="text" name="username"> <br/>
        密码：<input type="password" name="password"><br>
        兴趣爱好：<input type="checkbox" name="hobby" value="cpp">C++
        <input type="checkbox" name="hobby" value="java"/>Java
        <input type="checkbox" name="hobby" value="js"/>JavaScript <br/>
        <input type="submit">
    </form>
    </body>
    ```

4. doGet与doPost中的乱码问题
   * 描述：如果用户名使用中文，则可能出现乱码问题
   * doGet
      ```java
      // 获取请求参数
      String username = req.getParameter("username");
      //1 先以 iso8859-1 进行编码
      //2 再以 utf-8 进行解码
      username = new String(username.getBytes("iso-8859-1"), "UTF-8");
      ```
   * doPost
      ```java
      request.setCharacterEncoding("UTF-8");
      // 要在调用getParameter()方法之前调用该方法
      ```

### 7.5.3 请求的转发
1. 过程
   * 客户端发送请求给servlet程序A
   * 程序A查看参数、添加参数等操作
   * 程序A将请求转发给Servlet程序B
   * 程序B处理后发送回复给客户端
2. 代码
   ```java
   // Servlet1.java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        System.out.println("Servlet1中查看参数："+username);

        request.setAttribute("key1","Servlet1印记");

        // 请求转发必须要以斜杠打头， / 斜杠表示地址为： http://ip:port/工程名/ , 映射到 IDEA 代码的 web 目录
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/servlet2");
        requestDispatcher.forward(request, response);
    }

   ```
   ```java
   //Servlet2.java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        System.out.println("Servlet2中查看参数："+username);
        // 检查Servlet1是否处理过请求
        Object key1 = request.getAttribute("key1");
        System.out.println("Servlet1添加的参数："+key1);
        // 处理自己的业务
        System.out.println("Servlet2自己的业务");

    }
   ```
3. 请求转发的特点
   * 浏览器地址没有变化
   * 这两个是一次请求
   * 共享Request域中数据
   * 可以转发到WEB-INF目录下
   * 不可以访问工程以外的资源

### 7.5.4 base标签
1. 简介
   * 请求转发时，往回跳转路径会出问题
   * base标签可以设置当前页面中所有相对路径工作时，参照哪个路径进行跳转
   * 通常放在head标签的title标签下方
2. 使用举例：
    * 从index.html跳转到a/b/c.html，再跳转回来
    * 通过请求转发跳到c再调回来会出问题
    ```html
    <!--web/a/b/c.html-->
    <body>
        这是a下的b下的c.html页面 <br/>
        <a href="../../index.html">跳回首页</a>
    </body>
    ```
    ```html
    <!--web/index.html-->
    <body>
    这是web下的index.html <br/>
    <a href="a/b/c.html">跳转到a/b/c.html</a><br/>
    <a href="http://localhost:8080/07_servlet/forwardc">请求转发：a/b/c.html</a>
    </body>
    ```
    ```java
    // ForwardC.java
    public class ForwardC extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            System.out.println("经过了ForwardC程序转发");
            req.getRequestDispatcher("/a/b/c.html").forward(req,resp);
        }
    }
    ```
    ```xml
    <!--web.xml-->
    <!-- 例行处理即可 -->
    ```
    * 此时会出问题：
      * 开始路径："http://localhost:8080/07_servlet/index.html"
      * 请求转发时路径为 "http://localhost:8080/07_servlet/forwardc" 
      * 跳回时去除两级目录："http://localhost:8080/index.html"页面不存在
    ```html
    <!-- c.html -->
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <base href="http://localhost:8080/07_servlet/a/b/">
    </head>
    ```

### 7.5.5 Web中的路径
1. 开发中通常使用：
   * 绝对路径
   * base+相对路径
2. web 中 / 斜杠的不同意义
   * 在 web 中斜杠 是一种绝对路径。`/`如果被浏览器解析， 得到的地址是： `http://ip:port/`
   * 斜杠如果被服务器解析， 得到的地址是： `http://ip:port/工程路径`
   * 特殊情况： `response.sendRediect(“/”);` 把斜杠发送给浏览器解析,得到 `http://ip:port/`

## 7.6 `HttpServletResponse`类
### 7.6.1 `HttpServletResponse` 类的作用
1. Tomcat 服务器接收到请求后，会创建一个 Response 对象传递给 Servlet 程序去使用 
2. HttpServletRequest 表示请求过来的信息， HttpServletResponse 表示所有响应的信息
3. 可以通过 HttpServletResponse 对象来设置返回给客户端的信息

### 7.6.2 两个输出流
1. 字节流与字符流
   * 字节流 getOutputStream(); 常用于下载（传递二进制数据）
   * 字符流 getWriter(); 常用于回传字符串（常用）
2. 说明
   * 两个流同时只能使用一个

3. 往客户端回传数据
   ```java
    // ResponseIOServlet.java
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer = resp.getWriter();
        writer.write("止于至善");
    }
   ```
4. 乱码问题 
   * 回传字符串为中文时容易出现乱码
   * 解决方案一：
     * 通过 `resp.setCharacterEncoding("utf-8");` 将服务器字符集设置为UFT-8
     * 通过响应头， 设置浏览器也使用 UTF-8 字符集：`resp.setHeader("Content-Type", "text/html; charset=UTF-8");`
      ```java
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.setCharacterEncoding("utf-8");
          resp.setHeader("Content-Type", "text/html; charset=UTF-8");
          PrintWriter writer = resp.getWriter();
          writer.write("止于至善");
      }
      ```
    * 解决方案二
      * `resp.setContentType("text/html; charset=UTF-8");`
      * 可以同时设置服务器和客户端都使用 UTF-8 字符集， 还设置了响应头
      * 此方法一定要在获取流对象之前调用才有效

### 7.6.3 请求重定向
1. 概念
   * 指客户端给服务器发请求时，服务器返回一个重定向指令，告诉浏览器地址已经变了，麻烦使用新的URL再重新发送新请求
   * 服务器返回相应状态码302和Location响应头
2. 特点
   * 浏览器地址会发生变化
   * 是两次请求
   * 不共享Request域中的数据
   * 不能访问WEB-INF中的资源
   * 可以访问工程外的资源
3. 两种实现方案
   * 方案一：
      ```java
      // 设置状态码
      resp.setStatus(302);
      //设置响应头，说明新地址
      resp.setHeader("Location","http://localhost:8080");
      ```
   * 方案二
      ```java
      resp.sendRedirect("http://localhost:8080");
      ```
4. 代码示例
   ```java
   // Response1.java
   protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
      System.out.println("Response1 访问");
       /*// 设置状态码
      resp.setStatus(302);
      //设置响应头，说明新地址
      resp.setHeader("Location","http://localhost:8080/07_servlet/response2");*/
      resp.sendRedirect("http://localhost:8080/07_servlet/response2");
    }
   ```

   ```java
   // Response2.java
   protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
      resp.getWriter().write("Response2's result");
   }
   ```


















