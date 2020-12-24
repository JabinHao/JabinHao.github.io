---
title: Chapter1 SpringMVC概述
excerpt: SpringMVC概述、入门案例
tags:
  - java
  - SpringMVC
categories:
  - Java笔记
  - SpringMVC
banner_img: /img/dog.png
index_img: /img/post/springmvc_logo.png
abbrlink: 838e3ec7
date: 2020-12-13 15:27:32
updated: 2020-12-13 15:27:32
subtitle:
---
## 1.1 SpringMVC 简介
SpringMVC 也叫 Spring web mvc。是 Spring 框架的一部分，是在 Spring3.0 后发布的。
## 1.2 SpringMVC 优点
1. 基于 MVC 架构功能分工明确。解耦合，
2. 容易理解，上手快；使用简单。SpringMVC 也是轻量级的， jar 很小。不依赖的特定的接口和类。
3. 作 为 Spring 框 架 一 部 分 ， 能 够 使 用 Spring 的 IoC 和 Aop 。 方 便 整 合Strtus,MyBatis,Hiberate,JPA 等其他框架。
4. SpringMVC 强化注解的使用，在控制器， Service， Dao 都可以使用注解。方便灵活。
   * 使用@Controller 创建处理器对象
   * @Service 创建业务对象， 
   * @Autowired 或者@Resource在控制器类中注入 Service, Service 类中注入 Dao。

## 1.3 简单案例
### 1.3.1 步骤
1. 新建空项目，项目中新建模块：web maven工程
2. 加入依赖
   * spring-webmvc依赖，间接把spring的依赖都加入到项目
   * jsp，servlet依赖
   * pom.xml:
        ```xml
        <dependencies>

            <!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
            <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
            </dependency>

            <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
            <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.9.RELEASE</version>
            </dependency>

            <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
            <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
            </dependency>

        </dependencies>
        ```
    * pom.xml文件报错的话，Maven里选择 Roload project
    * 出现错误：
      * 首先在项目结构中将moudle和project的jdk改为同一个版本（如果不是的话）
      * pom.xml中：
        ```xml
        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
        </properties>
        ```

3. 重点： 在web.xml中注册springmvc框架的核心对象 `DispatcherServlet`
   * `DispatcherServlet` 叫做中央调度器， 是一个servlet， 它的父类是继承 `HttpServlet`
   * `DispatcherServlet` 页叫做前端控制器（front controller）
   * `DispatcherServlet` 负责接收用户提交的请求， 调用其它的控制器对象，并把请求的处理结果显示给用户
   * web.xml:
        ```xml
        <!--  声明，注册springmvc的核心对象DispatcherServlet  -->
        <servlet>
            <servlet-name>springmvc</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

            <!-- 自定义springmvc读取的配置文件位置 -->
            <init-param>
                <!-- springmvc配置文件的位置属性 -->
                <param-name>contextConfigLocation</param-name>
                <!-- 自定义文件的位置，这里为main下的resources目录下 -->
                <param-value>classpath:springmvc.xml</param-value>
            </init-param>
            <!--  tomcat启动后，创建Servlet对象，数字代表顺序，越小越早（>=0的整数）  -->
            <load-on-startup>1</load-on-startup>
        </servlet>
        ```
   * 配置servlet-mapping，使用框架时，url-pattern有两种形式
     * 使用扩展名：`*.xxx`，xxx是自定义的扩展名，常用 `*.do`、`*.action`、`*.mvc` 等
     * 使用斜杠 “/”  

4. 创建一个发起请求的页面 index.jsp
    * webapp目录下
    * index.jsp:
        ```html
        <%@ page contentType="text/html;charset=UTF-8" language="java" %>
        <html>
        <head>
            <title>Title</title>
        </head>
        <body>
            <p>第一个springmvc项目</p>
            <p><a href="some.do">发起some.do的请求</a> </p>
        </body>
        </html>
        ```
5. 创建控制器(处理器)类
   * 在类的上面加入`@Controller`注解，创建对象，并放入到springmvc容器中
   * 在类中的方法上面加入`@RequestMapping`注解（请求映射）。
     * 称为处理器方法或控制器方法
     * 作用：为请求地址绑定方法
     * 属性（value）：String，表示请求的uri地址（some.do），唯一，使用时推荐以“/”开头
     * 位置：方法上面、类上面
     * 返回值：`ModelAndView`
     * src/main/java/com/powernode/controller/MyController.java：
        ```java
        @Controller
        public class MyController {
            // 处理用户提交的请求
            @RequestMapping(value = "/some.do")
            public ModelAndView doSome() {
                ModelAndView modelAndView = new ModelAndView();
                // 添加数据，框架最后会将数据存入request作用域
                modelAndView.addObject("msg","欢迎使用springmvc");
                modelAndView.addObject("fun","执行doSome方法");
                // 指定视图（请求转发）
                modelAndView.setViewName("/show.jsp");
                return modelAndView;
            }
        }
        ```

6. 创建一个作为结果的jsp，显示请求的处理结果。
   * webapp/show.jsp
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
        <h3>show.jsp</h3>
        <h3>msg数据：${msg}</h3><br/>
        <h3>fun数据：${fun}</h3>
    </body>
    </html>
    ```
7. 创建springmvc的配置文件（spring的配置文件一样）
   * 声明组件扫描器， 指定@Contorller注解所在的包名：创建配置文件 src/main/resources/springmvc.xml
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

            <!--  声明组件扫描器  -->
            <context:component-scan base-package="com.powernode.controller"/>
        </beans>
        ```
   * 声明视图解析器。帮助处理视图的：可以简化第五步处理器方法中指定视图的内容
     * 将show.jsp放到WEB-INF目录下以防止用户直接访问到
     * 在springmvc的配置文件springmvc.xml中配置视图解析器：
        ```xml
        <!-- 声明springmvc框架中的视图解析器，设置视图文件路径 -->
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <!-- 前缀：视图文件路径 -->
            <property name="prefix" value="/WEB-INF/view/"/>
            <!-- 后缀：视图文件扩展名 -->
            <property name="suffix" value=".jsp"/>
        </bean>
        ```
     * 修改控制器方法：
        ```java
        // 指定视图
        modelAndView.setViewName("show");
        ```

8. 其他内容
   * 控制器方法可以对应多个url：`@RequestMapping`注解值使用数组
   * 创建多个控制器方法：
     * 在控制器类中创建多个方法
     * 在index.jsp页面中配置访问标签即可
   * 示例：
        ```java
        @Controller
        public class MyController {
            @RequestMapping(value = {"/some.do", "/first.do"})
            public ModelAndView doSome() {
                ModelAndView modelAndView = new ModelAndView();
                modelAndView.addObject("msg","欢迎使用springmvc");
                modelAndView.addObject("fun","执行doSome方法");
                modelAndView.setViewName("show");
                return modelAndView;
            }

            @RequestMapping(value = {"/other.do", "/second.do"})
            public ModelAndView doOther(){
                ModelAndView modelAndView = new ModelAndView();
                modelAndView.addObject("msg","欢迎使用springmvc");
                modelAndView.addObject("fun","执行doOther方法");
                modelAndView.setViewName("show");
                return modelAndView;
            }
        }
        ```

## 1.4 SpringMVC 执行流程
### 1.4.1 简介
1. 说明  
   这部分建议学完springmvc框架后再回来看
2. 流程图
   
    ![图1-1](https://raw.githubusercontent.com/JabinHao/mihs/master/img/springmvc-1-1.png)

### 1.4.2 执行流程简单分析
1. 用户发起请求 **.do
2. `DispatcherServlet`接收请求，将请求交给处理器映射器，框架将找到的处理器对象放入处理器执行链中保存
   * 处理器映射器：springmvc框架中的对象，实现`HandlerMapping`接口的类叫做映射器（有多个）
   * 作用：根据请求，从springmvc容器对象中获取相应的处理器对象（`ctx.getBean(MyController controller = "some.do")`）
   * `HandlerExecutionChain`：处理器执行链，保存这=着处理器对象和项目中所有的拦截器（`List<HandlerInterceptor>`）
3. `DispatcherServlet`将处理器执行链中的处理器对象交给处理器适配器对象
   * 处理器适配器：springmvc框架中的对象，实现`HandlerAdapter`接口的类叫做适配器（有多个）
   * 作用：执行处理器方法（`MyController.doSome()`），得到结果并封装到一个对象 `ModelAndView` 中，并将其返回给处理器适配器
4. `DispatcherServlet`将上一步得到的`ModelAndView`交给视图解析器
   * 视图解析器：springmvc框架中的对象，实现了`ViewResoler`接口的类（有多个）
   * 作用：使用前缀、后缀组成视图完整路径，创建View对象
   * View是一个接口，用于表示视图（视图类：InternalResourceView）
5. `DispatcherServlet`调用视图(View)对象中的方法，将Model数据放入到request作用域，执行forward，请求接收


