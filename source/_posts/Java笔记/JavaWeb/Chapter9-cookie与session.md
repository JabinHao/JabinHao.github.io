---
title: Chapter9 cookie与session
excerpt: cookie的概念与使用与session（待补充）
tags:
  - java
  - JavaWeb
  - Cookie
  - Session
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: eaeff7ef
date: 2020-12-02 18:12:34
updated: 2020-12-02 18:12:34
subtitle:
---
## 9.1 Cookie
1. 什么是Cookie
    * Cookie 是服务器通知客户端保存键值对的一种技术。
    * 客户端有了 Cookie 后， 每次请求都发送给服务器。
    * 每个 Cookie 的大小不能超过 4kb
2. 如何创建Cookie
   * 浏览器向客户端发起请求
   * 服务器创建Cookie
   * 通知客户端保存Cookie
   * 通过响应头Set-Cookie通知客户端保存Cookie

    ```java
    protected void createCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //创建Cookie对象
        Cookie cookie = new Cookie("key1","value1");
        // 通知客户端保存Cookie
        resp.addCookie(cookie);
        resp.getWriter().write("Cookie创建成功");
    }
    ```
    ```html
    <li><a href="cookieServlet?action=createCookie" target="target">Cookie的创建</a></li>
    ```

3. 服务器获取Cookie
   * 客户端发起请求
   * 若有Cookie则通过请求头Cookie:key=value把Cookie发送给服务器
   * 服务器通过 `req.getCookies()`获取Cookie数组
    
    ```java
    protected void getCookie(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
        Cookie[] cookies = req.getCookies();
        for (Cookie cookie : cookies) {
            resp.getWriter().write("Cookie["+cookie.getName()+"="+cookie.getValue()+"]<br>");
        }

        Cookie iWantCookie = null;
        for (Cookie cookie : cookies) {
            if ("key2".equals(cookie.getName())) {
                iWantCookie = cookie;
                break;
            }
        }
        if (iWantCookie != null)
            resp.getWriter().write("找到了需要的Cookie");
    }
    ```
    ```html
    <li><a href="cookieServlet?action=getCookie" target="target">Cookie的获取</a></li>
    ```
4. Cookie值的修改
   * 方案一
     * 先创建一个要修改的同名（指的就是 key） 的 Cookie 对象
     * 创建时同时赋于新的 Cookie 值。
     * 调用 response.addCookie( Cookie );

      ```java
      Cookie cookie = new Cookie("key1","newValue1");
      resp.addCookie(cookie);
      ```
    * 方案二
      * 先查找到需要修改的 Cookie 对象
      * 调用 setValue()方法赋于新的 Cookie 值。
      * 调用 response.addCookie()通知客户端保存修改

      ```java
      Cookie cookie = CookieUtils.findCookie("key2", req.getCookies());
      if (cookie != null) {
      // 调用 setValue()方法赋于新的 Cookie 值。
      cookie.setValue("newValue2");
      // 调用 response.addCookie()通知客户端保存修改
      resp.addCookie(cookie);
      }
      ```
    
    * cookie值不能包含符号、中文等，需要Base64编码

5. 浏览器查看Cookie
   * 谷歌：F12->Application->Cookies
   * 火狐：F12->存储->Cookie

6. Cookie生命控制
   * Cookie 的生命控制指的是如何管理 Cookie 什么时候被销毁（删除）
   * 通过setMaxAge()来设置
     * 正数， 表示在指定的秒数后过期
     * 负数， 表示浏览器一关， Cookie 就会被删除（默认值是-1）
     * 零， 表示马上删除 Cookie
   * 马上删除一个Cookie：
      ```java
      Cookie cookie = CookieUtils.findCookie("key2", req.getCookies());
      if (cookie != null) {
      cookie.setMaxAge(0); 
      resp.addCookie(cookie);
      }
      ```

7. Cookie 有效路径 Path 的设置
   * Cookie 的 path 属性可以有效的过滤哪些 Cookie 可以发送给服务器。 哪些不发。
   * path 属性是通过请求的地址来进行有效的过滤。
   * 通过serPath()来设置：`cookie.setPath( req.getContextPath() + "/abc" ); `

   ```java
   // 假设有两个Cookie
   CookieA path=/工程路径
   CookieB path=/工程路径/abc

   //请求地址如下：
   http://ip:port/工程路径/a.html
      CookieA 发送
      CookieB 不发送

   http://ip:port/工程路径/abc/a.html
      CookieA 发送
      CookieB 发送
   ```

## 9.2 Session会话
### 9.2.1 什么是 Session 
1. Session 是一个接口（HttpSession） 。
2. Session 是会话，是用来维护一个客户端和服务器之间关联的一种技术。
3. 每个客户端都有自己的一个 Session 会话。
4. Session 会话中，我们经常用来保存用户登录之后的信息。


### 9.2.2 Session的创建与获取
1. 创建和获取 Session的 API 是一样的
2. `request.getSession()`
   * 第一次调用是： 创建 Session 会话
   * 之后调用都是： 获取前面创建好的 Session 会话对象
3. `isNew()` 判断到底是不是刚创建出来的（新的）
   * `true` 表示刚创建
   * `false` 表示获取之前创建
4. `getId()`: 得到 Session会话的id值，每个会话都有一个唯一的ID值。


### 9.2.3 Session 域数据的存取
1. 通过setAttribute()方法保存数据，getAttribute()读取数据
2. 示例
   ```java
   //保存
   protected void setAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException,IOException {
      req.getSession().setAttribute("key1", "value1");
      resp.getWriter().write("已经往 Session 中保存了数据");
   }
   ```
   ```java
   // 获取
   protected void getAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException,IOException {
      Object attribute = req.getSession().getAttribute("key1");
      resp.getWriter().write("从 Session 中获取出 key1 的数据是：" + attribute);
   }
   ```

### 9.2.4 Session 生命周期控制
1. `public void setMaxInactiveInterval(int interval)` 设置 Session 的超时时间（ 以秒为单位），超过指定的时长，Session就会被销毁
   * 正数：Session 的超时时长。
   * 负数：永不超时（极少使用）
2. 修改默认超时时长
   * Session 默认的超时时间长为 30 分钟
   * Tomcat 服务器的配置文件 web.xml中配置了当前 Tomcat 服务器下所有的Session超时配置默认时长：
      ```xml
      <session-config>
        <session-timeout>20</session-timeout>
      </session-config>
      ```
3. 修改个别 Session 的超时时长时使用 `setMaxInactiveInterval` 方法
4. `public int getMaxInactiveInterval()` 获取 Session 的超时时间
5. `public void invalidate()` 让当前 Session 会话马上超时无效
6. Session的超时指客户端两次请求的最大间隔时长

### 9.2.5 浏览器和 Session 之间关联的技术内幕
Session 技术， 底层其实是基于 Cookie 技术来实现的

