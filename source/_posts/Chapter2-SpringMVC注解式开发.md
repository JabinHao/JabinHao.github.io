---
title: Chapter2 SpringMVC注解式开发
excerpt: 控制器方法的参数与返回值、RequestMapping和RequestParam注解的使用、url-pattern的配置等
tags:
  - java
  - SpringMVC
categories:
  - Java笔记
  - SpringMVC
banner_img: /img/dog.png
index_img: /img/post/springmvc_logo.png
date: 2020-12-13 16:44:17
updated: 2020-12-13 16:44:17
subtitle:
---
## 2.1 @RequestMapping 定义请求规则
### 2.1.1 指定模块名称
1. `@RequestMapping`注解作用于处理器类上可以指定模块名称
2. `value`属性用于指定URI公共部分
3. 示例：
   ```java
    @Controller
    @RequestMapping("/test")
    public class MyController {
        @RequestMapping(value = {"/some.do", "/first.do"})
        public ModelAndView doSome() {
            // ...
            return modelAndView;
        }
    }
   ```
   * 此时访问地址为 “http://localhost:port/工程名/test/some.do”

### 2.1.2 定义请求提交的方式
1. `method`属性，定义请求提交的方式（用于处理器方法）
2. 属性值为 `RequestMethod` 枚举常量:
   * `RequestMethod.GET` 
   * `RequestMethod.POST`
3. 请求提交的方式满足该 method 属性时，才会执行该被注解方法
4. 不定义则任何请求都可以
5. 示例：
   * 处理器方法：
        ```java
        @RequestMapping(value = {"/some.do", "/first.do"}, method = RequestMethod.POST)
        public ModelAndView doSome() {
            ModelAndView modelAndView = new ModelAndView();
            //...
            return modelAndView;
        }
        ```
   * index.jsp
        ```html
        <body>
            <form action="user/some.do" method="post">
                <input type="submit" value="post请求some.do">
            </form>
        </body>
        ```

## 2.2 处理器方法的参数
处理器方法可以包含以下四类参数，这些参数会在系统调用时由系统自动赋值，即程序
员可在方法内直接使用:
* HttpServletRequest
* HttpServletResponse
* HttpSession
* 请求中所携带的请求参数

### 2.2.1 逐个参数接收
1. 页面提交的参数，控制器可以根据参数名直接接收
2. 框架会根据参数列表自动进行类型转换，不匹配则会失败
3. 示例：
   * index.jsp提交参数
        ```html
        <body>
            <p>提交参数给Controller</p>
            <form action="receive.do" method="post">
                姓名：<input type="text" name="name"><br>
                年龄：<input type="text" name="age"><br>
                <input type="submit" value="提交">
            </form>
        </body>
        ```
   * Controller接收参数
        ```java
        @RequestMapping(value = {"/receive.do"}, method = RequestMethod.POST)
        public ModelAndView doReceive(String name, Integer age) {
            // 使用Integer可以接收空值
            ModelAndView modelAndView = new ModelAndView();
            modelAndView.addObject("myname",name);
            modelAndView.addObject("myage",age);
            modelAndView.setViewName("show");
            return modelAndView;
        }
        ```
   * show.jsp呈现参数
        ```html
        <body>
            <h3>show.jsp：</h3>
            <h3>姓名：${myname}</h3><br/>
            <h3>年龄：${myage}</h3>
        </body>
        ```

### 2.2.2 乱码问题
1. 使用post提交参数时，中文会出现乱码
2. 使用过滤器解决
   * 自定义过滤器
   * 使用框架提供的 `CharacterEncodingFilter`
3. 示例：  
   修改web.xml：
   ```xml
    <!--  声明过滤器，解决乱码问题  -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--  设置过滤器参数  -->
        <!-- 字符编码 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <!-- 强制请求对象（HttpServletRequest）使用Encoding编码值 -->
        <init-param>
            <param-name>forceRequestEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
        <!-- 强制应答对象（HttpServletResponse）使用Encoding编码值 -->
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <!-- 强制所有请求先通过过滤器 -->
        <url-pattern>/*</url-pattern>
    </filter-mapping>
   ```

### 2.2.3 校正请求参数名 `@RequestParam`
1. 作用
   * 请求 URL 所携带的参数名称与处理方法中指定的参数名不相同时，指定请求参数名
   * 指定参数是否为必须参数
2. 属性
   * value：指定请求中的参数名
   * required：boolean型，默认为true，即请求中必须有此参数
3. 使用
   * 用在逐个参数接收的情况中
   * 用在在处理器方法参数前：`@RequestParam(value, required) 形参`
4. 示例
   * index.jsp
        ```html
        <p>请求参数名和处理器形参不同名</p>
        <form action="receiveparam.do" method="post">
            姓名：<input type="text" name="pname"><br>
            年龄：<input type="text" name="page"><br>
            <input type="submit" value="提交">
        </form>
        ```
   * Controller.java
        ```java
        @RequestMapping(value = {"/receiveparam.do"}, method = RequestMethod.POST)
        public ModelAndView doReceiveParam(@RequestParam(value = "pname",required = true) String name, @RequestParam(value = "page",required = false) Integer age) {
            ModelAndView modelAndView = new ModelAndView();
            modelAndView.addObject("myname",name);
            modelAndView.addObject("myage",age);
            modelAndView.setViewName("show");
            return modelAndView;
        }
        ```

### 2.2.4 使用对象接收参数
1. 说明
   * 请求参数较多时，逐个参数接收不方便，可以使用对象来接收
   * 将处理器方法的参数定义为一个对象，只要保证请求参数名与这个对象的属性同名即可（getter、setter方法）
2. 示例
   * Student.java
        ```java
        public class Student {
            private String pname;
            private Integer page;

            public String getPname() {return pname;}

            public Integer getPage() {return page;}

            public void setPname(String pname) {this.pname = pname;}

            public void setPage(Integer page) {this.page = page;}
        }
        ```
   * 控制器方法
        ```java
        @RequestMapping(value = {"/receiveparam.do"}, method = RequestMethod.POST)
        public ModelAndView doReceiveParam(Student student) {
            ModelAndView modelAndView = new ModelAndView();
            modelAndView.addObject("myname",student.getPname());
            modelAndView.addObject("myage",student.getPage());
            modelAndView.addObject("mystudent",student);
            modelAndView.setViewName("show");
            return modelAndView;
        }
        ```

## 2.3 处理器方法的返回值
使用@Controller 注解的处理器的处理器方法，其返回值常用的有四种类型：
* 第一种： ModelAndView
* 第二种： String
* 第三种：无返回值 void
* 第四种：返回自定义类型对象

### 2.3.1 返回 ModelAndView
1. 返回视图和数据，适合需要跳转到其它资源，且又要在跳转的资源间传递数据
2. 若只需跳转而不传递数据，或只是传递数据而并不向任何资源跳转，则会多余。

### 2.3.2 返回 String
1. 处理器方法返回的字符串可以指定逻辑视图名，通过视图解析器解析可以将其转换为物理地址
2. 示例
   * 返回逻辑视图名称
      * 控制器方法
        ```java
        @RequestMapping(value = {"/receiveString.do"}, method = RequestMethod.POST)
        public String doReturnView(String name, String age) {
            System.out.println("doReturnView, name="+name+"  age="+age);
            // 返回逻辑视图名称，需要配置视图解析器（springmvc.xml）
            // 框架对视图执行转发操作
            return "show";
        }
        ```
     * index.jsp
        ```html
        <body>
            <p>返回String表示视图名称</p>
            <form action="receiveString.do" method="post">
                姓名：<input type="text" name="name"><br>
                年龄：<input type="text" name="age"><br>
                <input type="submit" value="提交">
            </form>
        </body>
        ```
     * 可以在控制器方法中手动添加数据
        ```java
        @RequestMapping(value = {"/receiveString.do"}, method = RequestMethod.POST)
        public String doReturnView(HttpServletRequest request, String name, String age) {
            System.out.println("doReturnView, name="+name+"  age="+age);
            request.setAttribute("myname", name);
            request.setAttribute("myage", age);
            // 返回逻辑视图名称，需要配置视图解析器（springmvc.xml）
            // 框架对视图执行转发操作
            return "show";
        }
        ```
   * 返回完整视图路径
     * 控制器方法
        ```java
        return "/WEB-INF/view/show";
        ```
     * 此时不能配置视图解析器

### 2.3.3 返回 void
1. 使用场景
   * 对于处理器方法返回 void 的应用场景， AJAX 响应.
   * 若处理器对请求处理后，无需跳转到其它任何资源，此时可以让处理器方法返回 void
2. ajax示例
   * 引入jQuery.js
   * 加依赖
        ```xml
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
        <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.11.3</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
        <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.11.3</version>
        </dependency>
        ```
   * 控制器方法
        ```java
        @RequestMapping(value = {"/returnVoid-ajax.do"}, method = RequestMethod.POST)
        public void doReturnVoidAjax(HttpServletResponse response, String name, Integer age) throws IOException {
            System.out.println("doReturnVoidAjax, name="+name+"  age="+age);
            //处理ajax，使用json做数据的格式
            // service调用完成了，使用Student表示处理结果
            Student student = new Student();
            student.setPname(name);
            student.setPage(age);

            String json = "";
            // 把结果的对象转化为json格式的数据
            if (student != null) {
                ObjectMapper om  = new ObjectMapper();
                json = om.writeValueAsString(student);
                System.out.println("Student转换的json======"+json);
            }
            //输出数据，响应ajax的请求
            response.setContentType("application/json;character=utf-8");
            PrintWriter pw = response.getWriter();
            pw.println(json);
            pw.flush();
            pw.close();
        }
        ```
   * index.jsp
        ```html
        <head>
            <title>Title</title>
            <script type="text/javascript" src="js/jquery-3.4.1.js"></script>
            <script type="text/javascript">
                $(function (){
                    $("button").click(function (){
                        alert("button click")
                        $.ajax({
                            url: "returnVoid-ajax.do",
                            data:{
                                name: "rui",
                                age: 24
                            },
                            type:"post",
                            //dataType:"json",
                            success:function (resp){
                                alert(resp);
                            }
                        })
                    })
                })
            </script>
        </head>
        <body>
            <button id="btn">发起ajax请求</button>
        </body>
        ```

### 2.3.4 返回对象 Object
1. 说明
   * 处理器方法可以返回 Object 对象，这个 Object 可以是 Integer， String， 自定义对象，Map，List 等
   * 需要使用 `@ResponseBody` 注解， 将转换后的 JSON 数据放入到响应体中
2. 使用步骤
   * 引入依赖（上一节两个Jackson依赖）
   * 声明注解驱动：`<mvc:annotation-driven>`
   * 处理器方法加`@ResponseBody`注解
3. 内部原理（了解）
   * 注解驱动用于创建`HttpMessageConverter`接口(**七个实现类**)，负责将请求信息转换为一个对象（类型为 T），将对象（类型为 T）输出为响应信息
   * `HttpMessageConverter`接口定义了java转为json，xml等数据格式的方法。 这个接口有很多的实现类。这些实现类完成 java对象到json， java对象到xml，java对象到二进制数据的转换
   * 接口方法
      * `boolean canRead(Class<?> clazz,MediaType mediaType)`: 指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为clazz类型的对象 ，同时指定支持MIME类型(text/html,applaiction/json 等)
      * `boolean canWrite(Class<?> clazz,MediaType mediaType)`:指定转换器是否可将 clazz 类型的对象写到响应流中，响应流支持的媒体类型在 MediaType 中定义
      * `void write(T t,MediaType contnetType,HttpOutputMessgae outputMessage)`:将 T 类型的对象写到响应流中，同时指定相应的媒体类型为 contentType
   * 常用实现类
     * `MappingJackson2HttpMessageConverter`： 使用jackson工具库中的ObjectMapper实现java对象转为json字符串
     * `StringHttpMessageConverter` 负责读取字符串格式的数据和写出字符串格式的数据
4. 返回自定义类型对象
   * 处理器方法
        ```java
        @RequestMapping("/returnStudentJson.do")
        @ResponseBody //将返回对象转换为json，通过HttpServletResponse输出给浏览器
        public Student doStudentJsonObject(String name, Integer age) {
            // 调用service，获取请求结果数据
            Student student = new Student();
            student.setPname("mei");
            student.setPage(24);
            return student;
        }
        ```
   * index.jsp
        ```js
        url: "returnStudentJson.do", //只修改url即可
        ```
   * 处理流程
     1. 框架调用`ArrayList<HttpMessageConverter>`中每个类的 `canWrite()`方法，检查该实现类能否处理Student类型的数据（`MappingJackson2HttpMessageConverter`）
     2. 调用实现类的 `write()` 方法，将student对象转换为json（`Jackson.ObjectMapper`，默认编码utf-8）
     3. 框架调用 `@ResponseBody` 将转换后的数据输出给浏览器，ajax请求处理完成

5. 返回 List 集合
   * 处理器方法
        ```java
        @RequestMapping(value = "/returnStudentJsonArray.do")
        @ResponseBody //将返回对象转换为json，通过HttpServletResponse输出给浏览器
        public List<Student> doStudentJsonObjectArray(String name, Integer age) {
            List<Student> list = new ArrayList<>();
            Student student = new Student();
            student.setPname("mei");
            student.setPage(24);
            list.add(student);

            student = new Student();
            student.setPname("fei");
            student.setPage(23);
            list.add(student);
            return list;
        }
        ```
   * index.jsp
        ```js
        $(function (){
            $("button").click(function (){
                $.ajax({
                    url: "returnStudentJsonArray.do",
                    data:{
                        name: "rui",
                        age: 24
                    },
                    type:"post",
                    //dataType:"json",
                    success:function (resp){
                        $.each(resp,function (i,n){ //循环遍历
                            alert(n.pname+" "+n.page)
                        })
                    }
                })
            })
        })
        ```
        ajax得到的是一个数组
6. 返回字符串对象
   * 与2.3.2区分：加了`@ResponseBody`注解。代表返回的是数据
   * 框架调用 `StringHttpMessageConverter` 类来实现转换，默认编码格式为ISO-8859-1
   * 控制器方法：
        ```java
        @RequestMapping(value = "/returnObjectString.do", produces = "text/plain;charset=utf-8")
        // 指明编码方式防止中文乱码
        @ResponseBody
        public String doObjectString() {
            return "doObjectString返回字符串";
        }
        ```
   * index.jsp
        ```js
        $(function (){
            $("button").click(function (){
                $.ajax({
                    url: "returnObjectString.do",
                    type:"post",
                    dataType:"text",
                    success:function (resp){
                        alert("返回值："+resp)
                    }
                })
            })
        })
        ```

## 2.4 关于`<url-pattern/>`
### 2.4.1 配置详解
1. web.xml
   ```xml
    <!--  声明，注册springmvc的核心对象DispatcherServlet  -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!-- 自定义springmvc读取的配置文件位置 -->
        <init-param>
            <!-- springmvc配置文件的位置属性 -->
            <param-name>contextConfigLocation</param-name>
            <!-- 自定义文件的位置 -->
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!--  tomcat启动后，创建Servlet对象，数字代表顺序，越小越早（>=0的整数）  -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
   ```
2. 对于框架，可以使用扩展名的方式
   * 语法 *.xxx，如 *.do，*.action等，不能使用*.jsp
   * 此时可以处理静态资源：
        ```html
        <!-- index.jsp -->
        <body>
            <p>返回String表示视图名称</p>
            <form action="test.do" method="post">
                姓名：<input type="text" name="name"><br>
                年龄：<input type="text" name="age"><br>
                <input type="submit" value="提交">
            </form>
            <br>
            <img src="img/girl.png" alt="静态资源图片1，显示出错" height="400px">
            <img src="img/monster.jpg" alt="静态资源图片2，显示出错" height="400px">
        </body>
        ```
        可以显示图片，也能正常跳转
3. 也可以使用斜杠的方式 “/”：
    * 此时它会替代tomcat中的default
    * 这种情况下，所有的资源都会给DispatcherServlet处理，而静态方法没有相应的控制器对象，则无法访问（404）
    * 动态资源可以访问，因为有控制器对象，可以处理some.do请求
        ```xml
        <!-- web.xml -->
        <servlet-mapping>
            <servlet-name>springmvc</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
        ```
        ```html
        <!-- index.jsp -->
        同上
        ```
        图片不能显示，但可以跳转到show页面
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/springmvc-2-1.png)

### 2.4.2 静态资源访问
可以通过一些配置，使得斜杠方式可以访问静态资源
1. 方式一
   * 使用`<mvc:default-servlet-handler/>`
        ```xml
        <!-- springmvc.xml -->
        <!--  注解驱动  -->
        <mvc:annotation-driven/>
        <!--解决方式一：-->
        <mvc:default-servlet-handler/>
        ```
   * 原理：该标签会让框架创建控制器对象DefaultServletHttpRequestHandler，将接受的请求转发给tomcat的default这个Servlet
   * 需要注解驱动：default-servlet-handler和`@RequestMapping`注解有冲突，需要注解驱动解决
![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/springmvc-2-2.png)

2. 方式二
   * 使用标签`<mvc:resources/>`，Spring 定义了专门用于处理静态资源访问请求的处理器 `ResourceHttpRequestHandler`。
   * 属性
     * `mapping`：静态资源所在目录。 不能使用/WEB-INF/及其子目录
     * `location`：请求url，使用通配符 **，`例如/img/**` 表示以img开始的请求：img/girl.png等
   * 仍然需要加入注解驱动，否则不能访问动态资源
        ```xml
        <!-- springmvc.xml -->
        <mvc:annotation-driven/>
        <mvc:resources mapping="/img/**" location="/img"/>
        <mvc:resources mapping="/html/**" location="/html/"/>
        <mvc:resources mapping="/js/**" location="/js/"/>
        ```
3. 实际开发中的配置
   * 实际开发中一般把静态资源都放到static目录下
   * 配置：
        ```xml
        <!-- springmvc.xml -->
        <mvc:annotation-driven/>
        <mvc:resources mapping="/static/**" location="/static/"/>
        ```


