---
title: Chapter10 Filter过滤器
excerpt: 摘要
tags:
  - java
  - JavaWeb
  - Filter
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 4155797c
date: 2020-12-03 16:28:55
updated: 2020-12-03 16:28:55
subtitle:
---
## 10.1 Filter概述
1. 什么是过滤器
   * Filter 过滤器是 JavaWeb 的三大组件之一。 三大组件分别是： Servlet 程序、 Listener 监听器、 Filter 过滤器
   * Filter 过滤器是 JavaEE 的规范。 也就是接口
   * Filter 过滤器的作用是： 拦截请求， 过滤响应。
2. 拦截常见应用场景：
   * 权限检查
   * 日记操作
   * 事务管理

## 10.2 Filter的使用
1. 使用步骤：
   * 编写一个类去实现 Filter 接口
   * 实现过滤方法 doFilter()
   * 到 web.xml 中去配置 Filter 的拦截路径
2. 示例
   ```java
   // AdminFilter.java
    public class AdminFilter implements Filter {
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

            HttpSession session = httpServletRequest.getSession();
            Object username = session.getAttribute("username");
            // 如果为null则未登录
            if (username == null) {
                servletRequest.getRequestDispatcher("/login.jsp").forward(servletRequest,servletResponse);
                return;
            }else {
            // 允许访问目标资源
            filterChain.doFilter(servletRequest,servletResponse);
            }
        }
        ...
    }
   ```
   ```java
   // LoginServlet.java
    public class LoginServlet extends HttpServlet {
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            response.setContentType("text/html; charset=UTF-8");//解决乱码问题
            String username = request.getParameter("username");
            String password = request.getParameter("password");

            if ("JabinHao".equals(username)&&"hjp356908".equals(password)) {
                request.getSession().setAttribute("username",username);
                response.getWriter().write("登录成功Successful");
            }else {
                request.getRequestDispatcher("/login.jsp").forward(request,response);
            }
        }
    }
   ```
   ```xml
   <!-- web.xml -->
    <!--filter标签用于配置一个Filter过滤器-->
    <filter>
        <filter-name>AdminFilter</filter-name> <!--别名-->
        <filter-class>com.atguigu.filter.AdminFilter</filter-class> <!--全类名-->
    </filter>

    <!--    filter-mapping标签配置Filter过滤器的拦截路径-->
    <filter-mapping>
        <filter-name>AdminFilter</filter-name>  <!--为哪个类配置拦截路径-->
        <url-pattern>/admin/*</url-pattern> <!--地址为http://ip:host/工程路径/ ,映射到IDEA为web目录下-->
    </filter-mapping>

    <servlet>
        <servlet-name>LoginServlet</servlet-name>
        <servlet-class>com.atguigu.servlet.LoginServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LoginServlet</servlet-name>
        <url-pattern>/loginServlet</url-pattern>
    </servlet-mapping>
   ```
   ```jsp
   <!--login.jsp-->
   <body>
    这是登陆页面：login.jsp <br>
    <form action="http://localhost:8080/10_servlet/loginServlet" method="get">
        用户名：<input type="text" name="username"/><br>
        密码： <input type="password" name="password"/> <br>
        <input type="submit"/>
    </form>
    </body>
   ```

## 10.3 Filter的生命周期
### 10.3.1 Filter的生命周期包含几个方法：
1. 构造器方法
2. init初始化方法
3. doFilter过滤方法
4. destroy销毁方法

### 10.3.2 各方法调用时间
1. 构造器方法与初始化方法：web工程启动的时候
2. doFilter方法：每次拦截到请求时执行
3. destroy方法：停止web工程时执行

## 10.4 `FilterConfig`类
1. 说明
   * `FilterConfig` 类是 Filter 过滤器的配置文件类。
   * Tomcat 每次创建 Filter 的时候， 也会同时创建一个 `FilterConfig` 类， 这里包含了 Filter 配置文件的配置信息。
2. `FilterConfig` 类的作用：获取 filter 过滤器的配置内容
   * 获取 Filter 的名称 `filter-name` 的内容
   * 获取在 Filter 中配置的 `init-param` 初始化参数
   * 获取 `ServletContext` 对象
3. 示例
   ```java
    public void init(FilterConfig filterConfig) throws ServletException {
        // 1. 获取名称
        System.out.println(filterConfig.getFilterName());
        // 2. 获取初始化参数
        System.out.println(filterConfig.getInitParameter("username"));
        System.out.println(filterConfig.getInitParameter("url"));
        // 3. 获取ServletContext对象
        System.out.println(filterConfig.getServletContext());
    }
   ```
   ```xml
   <!--web.xml -->
    <filter>
        <filter-name>AdminFilter</filter-name> <!--别名-->
        <filter-class>com.atguigu.filter.AdminFilter</filter-class> <!--全类名-->
        <init-param><!--初始化参数-->
            <param-name>username</param-name>
            <param-value>root</param-value>
        </init-param>
        <init-param>
            <param-name>url</param-name>
            <param-value>jdbc:mysql://localhost3306/test</param-value>
        </init-param>
    </filter>
   ```

## 10.4 `FilterChain` 过滤器链
1. 说明
   * `FilterChain`就是过滤器链，即多个过滤器如何一起工作
   * `FileChain` 是一个接口，内置 `doFilter()` 方法
2. `FileChain.doFilter()`的作用
   * 执行下一个Filter过滤器（如果有）
   * 执行目标资源（如果没有Filter）
   * 多个Filter的执行顺序由在web.xml中配置的顺序决定
3. 多个Filter过滤器执行特点
   * 所有Filter和目标资源默认执行在同一个线程中
   * 多个Filter共同执行时，使用同一个Request对象

## 10.5 Filter的拦截路径

1. 精确匹配
   * `<url-pattern>/target.jsp</url-pattern>`
   * 精确匹配到 `http://ip:port/工程路径/target.jsp`
2. 目录匹配
   * `<url-pattern>/admin/*</url-pattern>`
   * 匹配 `http://ip:port/工程路径/admin/` 下的所有内容
3. 后缀名匹配
   * `<url-pattern>*.html</url-pattern>`
   * 请求地址必须以.html结尾才会被匹配到

