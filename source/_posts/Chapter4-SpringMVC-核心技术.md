---
title: Chapter4 SpringMVC 核心技术
excerpt: 摘要
tags:
  - java
  - springmvc
categories:
  - Java笔记
  - SpringMVC
banner_img: /img/dog.png
index_img: /img/post/springmvc_logo.png
abbrlink: 3e7880f6
date: 2020-12-23 15:35:54
updated: 2020-12-23 15:35:54
subtitle:
---
## 4.1 请求重定向和转发
### 4.1.1 请求转发
1. `forward:`的使用
   * 处理器方法返回 ModelAndView时，在 `setViewName()`指定的视图前添加 `forward:`
   * 处理器方法返回 String（视图）时，在视图路径前面加入 `forward:` 
   * 用于屏蔽视图解析器，必须写出相对于项目根的路径
2. 示例
   * Controller
      ```java
      @RequestMapping("/doForward.do")
      public ModelAndView doForward() {
         ModelAndView mv = new ModelAndView();
         mv.addObject("tip","forward");
         mv.setViewName("forward:/hello.jsp");
         return mv;
      }
      ```
   * 视图解析器
      ```xml
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
         <!-- 前缀：视图文件路径 -->
         <property name="prefix" value="/WEB-INF/view/"/>
         <!-- 后缀：视图文件扩展名 -->
         <property name="suffix" value=".jsp"/>
      </bean>
      ```

   * hello.jsp
      ```html
      <body>
      hello页面：${tip}
      </body>
      ```

   * 文件结构
      ```
      +--- webapp
      |   +--- hello.jsp
      |   +--- index.jsp
      |   +--- img
      |   |   +--- girl.png
      |   +--- WEB-INF
      |   |   +--- view
      |   |   |   +--- show.jsp
      |   |   +--- web.xml
      ```

### 4.1.2 请求重定向
1. `redirect:`使用
   * 在处理器方法返回的视图字符串的前面添加 `redirect:`
   * 后面也需要指定视图完整路径
   * 不能访问WEB-INF下的资源
2. 示例
   * Controller
      ```java
      @RequestMapping("/doForward.do")
      public ModelAndView doForward() {
         ModelAndView mv = new ModelAndView();
         mv.addObject("tip","forward");
         mv.setViewName("forward:/hello.jsp");
         return mv;
      }
      ```
   * hello.jsp
      ```html
      <body>
      hello页面：${param.tip}
      </body>
      ```
3. 说明
   * 框架会把Model中的简单数据，转为string，作为重定向页面的请求参数使用，从而可以在两次请求之间传递参数
   * 在重定向页面，可以使用参数集合对象  `${param}` 获取请求参数值：`${param.tip}`

## 4.2 异常处理
### 4.2.1 说明
1. 传统异常处理
   * 传统方式使用try-catch来处理异常
   * 传统方法代码过于冗长不便于管理
2. SpringMVC 框架处理异常方式
   * 使用 `@ExceptionHandler` 注解处理异常
   * 体现了AOP思想，业务功能与非业务功能剥离
### 4.2.2 `@ExceptionHandler` 注解与 `@ControllerAdvice`注解
1. `@ExceptionHandler` 注解
   * 注解 `@ExceptionHandler` 可以将一个方法指定为异常处理方法
   * 可选属性 value，为一个 Class<?>数组(`xxx.class`)，用于指定该注解的方法所要处理的异常类
2. `@ControllerAdvice`注解
   * 控制器增强注解， 即给控制器对象增强功能
   * 使用该注解修饰的类中可以使用`@ExceptionHandler`
   * @ControllerAdvice 是使用@Component 注解修饰的，配置组件扫描`<context:component-scan>`，扫描到 `@ControllerAdvice` 所在的类路径(包名)， 创建对象
3. 注解使用方法
   * 可以直接在controller中定义异常处理方法，并使用`@ExceptionHandler`注解修饰（不常用）
   * 常用方法：定义专门的异常处理类，并使用`@ControllerAdvice`注解修饰，在该类中定义使用`@ExceptionHandler`注解修饰的异常处理方法

### 4.2.3 异常处理示例
1. 创建异常类
   * 父类 `MyUserException`
      ```java
      public class MyUserException extends Exception{
         public MyUserException() {
            super();
         }

         public MyUserException(String message) {
            super(message);
         }
      }
      ```

   * 子类 `NameException`
      ```java
      public class NameException extends MyUserException{
         public NameException() {
            super();
         }

         public NameException(String message) {
            super(message);
         }
      }
      ```
   * 子类 `AgeException`
      ```java
      public class AgeException extends MyUserException{
         public AgeException() {
            super();
         }

         public AgeException(String message) {
            super(message);
         }
      }
      ```
2. 修改 Controller 抛出异常
    ```java
    @RequestMapping(value = {"/exception.do"})
    public ModelAndView doTest(String name, String age) throws MyUserException {
        ModelAndView mv = new ModelAndView();
        //根据请求参数，抛出异常
        if (!"rui".equals(name)){
            throw new NameException("姓名不正确");
        }
        if (age==null||Integer.valueOf(age) > 25){
            throw new AgeException("年龄过大！");
        }
        mv.addObject("name",name);
        mv.addObject("age",age);
        mv.setViewName("show");
        System.out.println("成功");
        return mv;
    }
    ```
3. 定义全局异常处理类
   ```java
   @ControllerAdvice
   public class GlobalExceptionHandler {

      @ExceptionHandler(value = NameException.class)
      public ModelAndView doNameException(Exception exception){
         // 处理NameException的异常
         /*
               异常处理：
                  1. 记录异常，保存到数据库、日志文件：发生时间，发生的位置、错误内容
                  2. 发送通知，把异常信息通知给相关人员
                  3. 给用户友好的提示
            */
         ModelAndView mv = new ModelAndView();
         mv.addObject("msg","name must be rui");
         mv.addObject("e", exception);
         mv.setViewName("nameError");
         return mv;
      }

      @ExceptionHandler(value = AgeException.class)
      public ModelAndView doAgeException(Exception exception){
         // 处理 AgeException 的异常
         ModelAndView mv = new ModelAndView();
         mv.addObject("msg","age cannot be older than 25");
         mv.addObject("e", exception);
         mv.setViewName("ageError");
         return mv;
      }

      @ExceptionHandler()
      public ModelAndView doOtherException(Exception exception){
         // 处理其他可能的未知异常
         ModelAndView mv = new ModelAndView();
         mv.addObject("msg","unknown error");
         mv.addObject("e", exception);
         mv.setViewName("defaultError");
         return mv;
      }
   }
   ```
4. 修改配置文件
   ```xml
   <!-- springmvc.xml -->
      <!--  组件扫描  -->
   <context:component-scan base-package="com.powernode.handler"/>
   <!--  注解驱动  -->
   <mvc:annotation-driven/>
   ```
5. 创建异常响应页面
   * nameError.jsp
      ```html
      <head>
         <title>nameError</title>
      </head>
      <body>
    nameError.jsp <br>
    提示信息：${msg} <br>
    系统异常消息：${e.message}
</body>
```

   * ageError.jsp
      ```html
      <head>
         <title>ageError</title>
      </head>
      <body>
         ageError.jsp <br>
         提示信息：${msg} <br>
         系统异常消息：${e.message}
      </body>
      ```

   * defaultError.jsp
      ```html
      <head>
         <title>defaultError</title>
      </head>
      <body>
         defaultError.jsp <br>
         提示信息：${msg} <br>
         系统异常消息：${e.message}
      </body>
      ```
实现

## 4.3 拦截器
### 4.3.1 文字说明
1. 简介
   * 拦截器是springmvc中的一种对象，需要实现`HandlerInterceptor`接口
   * 与过滤器类似，功能侧重点不同：
      * 过滤器用于过滤参数请求，设置编码字符集等工作
      * 拦截器用于拦截用户请求，对请求做判断处理
   * 拦截器是全局的，可以对多个Controller做拦截，一个项目可以有多个拦截器
   * 拦截器使用场景
      * 用户登录处理
      * 权限检查
      * 记录日志
2. 使用步骤
   1. 定义 `HandlerInterceptor` 接口实现类
   2. 在springmvc配置文件中，声明拦截器，让框架知道拦截器的存在
3. 拦截器执行时间
   * 在请求处理之前，即控制器方法执行之前先被拦截
   * 在控制器方法执行之后也会执行拦截器
   * 在请求处理完成之后也会执行拦截器
4. `HandlerInterceptor` 接口方法
   * 预处理方法 `boolean preHandle(request, response, Object handler)`
      * `handler`：被拦截的控制器对象
      * 返回值：
         * `true`：表示请求通过，可以执行处理器方法，将`afterCompletion()`方法放入到一个专门的方法栈中等待执行
         * `false`：请求不通过，不执行控制器方法
      * 在控制器方法之前执行，可以获取请求的信息，验证请求是否符合要求
   * 后处理方法 `postHandle(request, response, Object handler, ModelAndView modelAndView)`
      * `handler`：被拦截的控制器对象
      * `modelAndView`：处理器方法的返回值
      * 在处理器方法之后执行，可以获取处理器方法返回值，修改其中的视图，影响最后的执行结果
   * 最后执行的方法 `afterCompletion(request, response, Object handler, Exception ex)`
      * `handler`：被拦截的控制器对象
      * `ex`：程序中发生的异常
      * 请求处理完成后执行（forward之后）
      * 一般用于资源回收工作，程序请求过程中创建的对象，在这里可以删除，清理内存空间

### 4.3.2 单个拦截器
1. 请求不通过：
   1. 创建 `HandlerInterceptor` 接口实现类
      ```java
      //拦截器类
      public class MyInterceptor implements HandlerInterceptor {
         @Override
         public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            System.out.println("拦截器的preHandle()");

            // 给浏览器一个返回结果
            request.getRequestDispatcher("/tips.jsp").forward(request,response);
            return false;
         }

         @Override
         public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            System.out.println("拦截器的postHandle()");
         }

         @Override
         public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            System.out.println("拦截器的的afterCompletion()");
         }
      }
      ```
   2. 修改配置文件
      ```xml
      <!--声明拦截器：可以有多个-->
      <mvc:interceptors>
         <!--第一个拦截器-->
         <mvc:interceptor>
            <!--指定拦截的请求uri地址-->
            <mvc:mapping path="/user/**"/> <!--拦截 'http://localhost:8080/项目名/user/' 开头的所有请求-->
            <!--声明拦截器对象-->
            <bean class="com.powernode.handler.MyInterceptor"/>
         </mvc:interceptor>
      </mvc:interceptors>
      ```

   3. webapp/tips.jsp
      ```html
      <body>
         tips.jsp  请求被拦截，不能执行
      </body>
      ```

2. 请求通过，添加数据
   * MyInterceptor.java
      ```java
      public class MyInterceptor implements HandlerInterceptor {
         @Override
         public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            System.out.println("拦截器的preHandle()");
            return true;
         }

         @Override
         public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            System.out.println("拦截器的postHandle()");
            if (modelAndView!=null){
                  modelAndView.addObject("mydate", new Date());
                  modelAndView.setViewName("other");
            }
         }

         @Override
         public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            System.out.println("拦截器的的afterCompletion()");
         }
      }
      ```
   * other.jsp
      ```html
      <body>
         <h3>show.jsp：</h3>
         <h3>姓名：${name}</h3><br/>
         <h3>年龄：${age}</h3>
         <h3>日期：${mydate}</h3>
      </body>
      ```

### 4.3.3 多个拦截器
1. 说明
   * 当有多个拦截器时，形成拦截器链，拦截器链的执行顺序，与其注册顺序一致
   * 当某一个拦截器的 `preHandle()`方法返回 true 并被执行到时，会向一个专门的方法栈中放入该拦截器的 `afterCompletion()`方法
   * 方法栈中的`afterCompletion()`方法一定会被执行

2. 示例
   * 定义多个拦截器（`HandlerInterceptor`接口实现类）
      ```java
      public class MyInterceptor2 implements HandlerInterceptor {
         @Override
         public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            System.out.println("拦截器2的preHandle()");
            return true;
         }

         @Override
         public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            System.out.println("拦截器2的postHandle()");
         }

         @Override
         public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            System.out.println("拦截器2的afterCompletion()");
         }
      }
      ```
   * 修改配置文件
      ```xml
      <!--声明拦截器：可以有多个，存在一个ArrayList中-->
      <mvc:interceptors>
         <!--第一个拦截器-->
         <mvc:interceptor>
               <!--指定拦截的请求uri地址-->
               <mvc:mapping path="/**"/>
               <!--声明拦截器对象-->
               <bean class="com.powernode.handler.MyInterceptor"/>
         </mvc:interceptor>
         <!--声明第二个拦截器-->
         <mvc:interceptor>
               <mvc:mapping path="/**"/>
               <bean class="com.powernode.handler.MyInterceptor2"/>
         </mvc:interceptor>
      </mvc:interceptors>
      ```

### 4.3.4 总结
1. 拦截器和过滤器的区别
   * 过滤器是servlet中的对象，  拦截器是框架中的对象
   * 过滤器实现Filter接口的对象， 拦截器是实现HandlerInterceptor
   * 过滤器是用来设置request，response的参数，属性的，侧重对数据过滤的。拦截器是用来验证请求的，能截断请求。
   * 过滤器是在拦截器之前先执行的。
   * 过滤器是tomcat服务器创建的对象拦截器是springmvc容器中创建的对象
   * 过滤器是一个执行时间点。拦截器有三个执行时间点
   * 过滤器可以处理jsp，js，html等等，拦截器是侧重拦截对Controller的对象。 如果你的请求不能被DispatcherServlet接收， 这个请求不会执行拦截器内容
   * 拦截器拦截普通类方法执行，过滤器过滤servlet请求响应

### 4.3.5 拦截器示例：权限拦截器
1. 创建相关模拟页面
   * login.jsp
      ```html
      <head>
         <title>登录页面</title>
      </head>
      <body>
         模拟登录
         <%
            session.setAttribute("name","rui");
         %>
      </body>
      ```
   * logout.jsp
      ```html
      <head>
         <title>登出</title>
      </head>
      <body>
         退出系统，从session中删除数据
         <%
            session.removeAttribute("name");
         %>
      </body>
      ```
   * tips.jsp
2. 修改拦截器对象
   ```java
   public class MyInterceptor implements HandlerInterceptor {

      // 验证登录的用户信息，正确返回true，否则返回false
      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
         System.out.println("拦截器1的preHandle()");
         String username = "";
         // 从session中获取name值
         Object attr = request.getSession().getAttribute("name");
         if (attr != null){
               username = (String)attr;
         }

         // 判断登录账户，是否符合要求
         if (!"rui".equals(username)){
               // 不符合要求，跳转到提示页面
               request.getRequestDispatcher("/tips.jsp").forward(request,response);
               return false;
         }
         return true;
      }
   }
   ```
3. 其他部分
   * Controller与之前相同
   * index.jsp也与之前相同，但拦截器获取的并不是该页面提交的数据，而是session中储存的模拟数据
4. 使用流程
   * 直接访问首页，点击提交按钮，此时session为空，请求不通过，跳转到tips页面
   * 访问login.jsp（直接输入地址访问），此时数据存入session，再次访问首页并提交，请求通过，来到show页面
   * 访问logout.jsp页面，session数据被删除，再次访问首页并提交，请求再次被拦截


