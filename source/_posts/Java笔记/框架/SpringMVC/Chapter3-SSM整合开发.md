---
title: Chapter3 SSM整合开发
excerpt: 摘要
tags:
  - java
  - ssm
categories:
  - Java笔记
  - SpringMVC
banner_img: /img/dog.png
index_img: /img/post/springmvc_logo.png
abbrlink: 7abcca27
date: 2020-12-19 23:41:15
updated: 2020-12-23 15:32:25
subtitle:
---
## 3.1 SSM简介
### 3.1.1 SSM编程
1. SSM即 SpringMVC + Spring + MyBatis 
   * SpringMVC:视图层，界面层，负责接收请求，显示处理结果的。
   * Spring：业务层，管理service，dao，工具类对象的。
   * MyBatis：持久层， 访问数据库的
2. SSM 整合的实现方式可分为两种：
   * 基于 XML 配置方式
   * 基于注解方式

### 3.1.2 三者关系
1. SSM整合也叫做SSI (IBatis也就是mybatis的前身)， 整合中有两个容器。
   * SpringMVC容器， 管理Controller控制器对象
   * Spring容器，管理Service，Dao,工具类对象
2. 配置文件
   * 把Controller还有web开发的相关对象交给springmvc容器， 这些web用的对象写在springmvc配置文件中
   * 把service，dao对象定义在spring的配置文件中，让spring管理这些对象。
3. 关系
   * springmvc容器是spring容器的子容器， 类似java中的继承。 子可以访问父的内容
   * 在子容器中的Controller可以访问父容器中的Service对象， 就可以实现controller使用service对象

## 3.2 搭建 SSM 开发环境
### 3.2.1 新建项目
1. 新建maven项目
2. 加入依赖
   * springmvc，spring，mybatis三个框架的依赖
   * jackson依赖，
   * mysql驱动、druid连接池
   * jsp，servlet
3. 插件
4. pom.xml:
    ```xml
    <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <build>
    <resources>
        <resource>
        <directory>src/main/java</directory><!--所在的目录-->
        <includes><!--包括目录下的.properties,.xml 文件都会扫描到-->
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
        </plugin>
    </plugins>
    </build>

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

    <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>javax.servlet.jsp-api</artifactId>
        <version>2.3.3</version>
        <scope>provided</scope>
    </dependency>

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

    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.3</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.5</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.21</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.3</version>
    </dependency>

    </dependencies>
    ```

### 3.2.2 配置 web.xml
1. 内容
   * 注册 ContextLoaderListener 监听器
   * 注册字符集过滤器
   * 配置中央调度器
2. 示例
    ```xml
    <!--注册springmvc的核心对象DispatcherServlet(中央调度器)-->
    <servlet>
    <servlet-name>ssm</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--自定义springmvc读取的配置文件的位置-->
    <init-param>
        <!--springmvc的配置文件的位置的属性-->
        <param-name>contextConfigLocation</param-name>
        <!--指定自定义文件的位置-->
        <param-value>classpath:conf/dispatcherServlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
    <servlet-name>ssm</servlet-name>
    <url-pattern>*.do</url-pattern>
    </servlet-mapping>

    <!--注册spring的监听器-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:conf/applicationContext.xml</param-value>
    </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

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

### 3.2.3 spring、springmvc、mybatis配置文件
1. spring配置文件applicationContext.xml
   ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

        <!-- spring配置文件：声明service、dao、工具类等对象 -->

        <!--声明数据源，连接数据库-->
        <context:property-placeholder location="classpath:conf/jdbc.properties"/>
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </bean>

        <!--SqlSessionFactoryBean创建SqlSessionFactory-->
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <property name="configLocation" value="classpath:conf/mybatis.xml"/>
        </bean>

        <!--声明mybatis的扫描器，创建dao对象-->
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
            <property name="basePackage" value="com.powernode.dao"/>
        </bean>

        <!--声明service的注解@Service所在的包名位置-->
        <context:component-scan base-package="com.powernode.service"/>

        <!--事务配置：注解或aspectj-->

    </beans>
   ```
2. 数据库配置信息jdbc.properties
   ```
    jdbc.url=jdbc:mysql://localhost:3306/ssm?serverTimezone=UTC&characterEncoding=utf8&useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
    jdbc.username=root
    jdbc.password=123456
   ```
   
3. springmvc配置文件 dispatcherServlet.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.alibaba.com/schema/stat"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.alibaba.com/schema/stat http://www.alibaba.com/schema/stat.xsd">

        <!-- springmvc配置文件，声明controller和其它web相关的对象 -->
        <context:component-scan base-package="com.powernode.controller"/>

        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/jsp/"/>
            <property name="suffix" value=".jsp"/>
        </bean>

        <!-- 注解驱动 -->
        <!--注意这个有多个，不要选错了：http://www.springframework.org/schema/mvc-->
        <mvc:annotation-driven/>
    </beans>
    ```

4. mybatis配置文件 mybatis.xml
    ```xml
    <configuration>
        <!--settings: 控制mybatis全局行为-->
        <settings>
            <!--日志-->
            <setting name="logImpl" value="STDOUT_LOGGING"/>
        </settings>
        
        <!--设置别名-->   
        <typeAliases>
            <!--name: 实体类所在的包名-->
            <package name="com.powernode.domain"/>
        </typeAliases>

        <!--sql mapper（sql映射文件）位置-->
        <mappers>
            <!--name: 包名-->
            <package name="com.powernode.dao"/>

            <!--也可使用resource-->
    <!--        <mapper resource="com/powernode/dao/StudentDao.xml"/>-->
        </mappers>

    </configuration>
    ```

## 3.3 SSM整合注解开发示例
### 3.3.1 准备工作
1. 搭建开发环境：见上一节
2. 项目结构
   ```
    +--- pom.xml
    +--- src
    |   +--- main
    |   |   +--- java
    |   |   |   +--- com
    |   |   |   |   +--- powernode
    |   |   |   |   |   +--- controller
    |   |   |   |   |   +--- dao
    |   |   |   |   |   +--- domain
    |   |   |   |   |   +--- service
    |   |   +--- resources
    |   |   |   +--- conf
    |   |   |   |   +--- applicationContext.xml
    |   |   |   |   +--- dispatcherServlet.xml
    |   |   |   |   +--- jdbc.properties
    |   |   |   |   +--- mybatis.xml
    |   |   +--- webapp
    |   |   |   +--- WEB-INF
    |   |   |   |   +--- applicationContext.xml
    |   |   |   |   +--- jsp
    |   |   |   |   +--- log4j.xml
    |   |   |   |   +--- view
    |   |   |   |   +--- web.xml
   ```

3. 新建数据库表student
    ```mysql
    create database springmvc;
    use springmvc;
    create table student(
        id int not null auto_increment,
        name varchar(80),
        age int,
        primary key (id)
    );
    ```

### 3.3.2 类与接口
1. 实体类Student
   ```java
    public class Student {
        private Integer id;
        private String name;
        private Integer age;

        public void setId(Integer id) {
            this.id = id;
        }

        public void setName(String name) {
            this.name = name;
        }

        public void setAge(Integer age) {
            this.age = age;
        }

        public Integer getId() {
            return id;
        }

        public String getName() {
            return name;
        }

        public Integer getAge() {
            return age;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
   ```
2. StudentDao接口及mapper文件StudentDao.xml
   ```java
    public interface StudentDao {

        int insertStudent(Student student);
        List<Student> selectStudents();
    }
   ```
   ```xml
    <mapper namespace="com.powernode.dao.StudentDao">
        <select id="selectStudents" resultType="com.powernode.domain.Student">
            select id, name, age from student order by id;
        </select>
        <insert id="insertStudent">
            insert into student(name,age) values (#{name},#{age})
        </insert>
    </mapper>
   ```
3. StudentService接口
   ```java
    public interface StudentService {

        int addStudent(Student student);
        List<Student> findStudents();
    }
   ```
4. 处理器 StudentController类
   ```java
    @Controller
    @RequestMapping("/student")
    public class StudentController {

        @Resource
        private StudentService service;

        // 注册学生
        @RequestMapping("/addStudent.do")
        public ModelAndView addStudent(Student student) {
            ModelAndView mv = new ModelAndView();
            String tips = "注册失败";
            // 调用service处理student
            int nums = service.addStudent(student);
            if (nums>0){
                // 注册成功
                tips = "学生：" + student.getName() + "注册成功";
            }
            mv.addObject("tips",tips);
            mv.setViewName("result");
            return mv;
        }
    }
   ```
5. service接口及实现类
   ```java
    public interface StudentService {

        int addStudent(Student student);
        List<Student> findStudents();
    }
   ```
   ```java
    @Service
    public class StudentServiceImpl implements StudentService {

        // 引用类型自动注入@Autowired，@Resource
        @Resource
        private StudentDao studentDao;

        @Override
        public int addStudent(Student student) {
            int nums = studentDao.insertStudent(student);
            return nums;
        }

        @Override
        public List<Student> findStudents() {
            return studentDao.selectStudents();
        }
    }
   ```

### 3.3.3 页面文件
1. index.jsp
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>首页</title>
    </head>
    <body>
        <div align="center">
            <p>SSM整合的案例</p>
            <img src="img/horse.png" height="200"/>
            <table>
                <tr>
                    <td><a href="addStudent.jsp">注册学生</a></td>
                </tr>
                <tr>
                    <td>浏览学生</td>
                </tr>
            </table>
        </div>
    </body>
    </html>
    ```

2. addStudent.jsp
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>注册学生</title>
    </head>
    <body>
        <div align="center">
            <form action="student/addStudent.do" method="post">
                <table>
                    <tr>
                        <td>姓名：</td>
                        <td><input type="text" name="name"></td>
                    </tr>
                    <tr>
                        <td>年龄：</td>
                        <td><input type="text" name="age"></td>
                    </tr>
                    <tr>
                        <td>&nbsp;&nbsp;&nbsp;</td>
                        <td><input type="submit" name="注册"></td>
                    </tr>
                </table>
            </form>
        </div>
    </body>
    </html>
    ```

3. result.jsp
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>result</title>
    </head>
    <body>
    result.jsp 结果页面：注册结果： ${tips}
    </body>
    </html>
    ```

### 3.3.4 补充说明
1. 项目目前结构
    ```
    +--- pom.xml
    +--- src
    |   +--- main
    |   |   +--- java
    |   |   |   +--- com
    |   |   |   |   +--- powernode
    |   |   |   |   |   +--- controller
    |   |   |   |   |   |   +--- StudentController.java
    |   |   |   |   |   +--- dao
    |   |   |   |   |   |   +--- StudentDao.java
    |   |   |   |   |   |   +--- StudentDao.xml
    |   |   |   |   |   +--- domain
    |   |   |   |   |   |   +--- Student.java
    |   |   |   |   |   +--- service
    |   |   |   |   |   |   +--- impl
    |   |   |   |   |   |   |   +--- StudentServiceImpl.java
    |   |   |   |   |   |   +--- StudentService.java
    |   |   +--- resources
    |   |   |   +--- conf
    |   |   |   |   +--- applicationContext.xml
    |   |   |   |   +--- dispatcherServlet.xml
    |   |   |   |   +--- jdbc.properties
    |   |   |   |   +--- mybatis.xml
    |   |   +--- webapp
    |   |   |   +--- addStudent.jsp
    |   |   |   +--- img
    |   |   |   |   +--- horse.png
    |   |   |   +--- index.jsp
    |   |   |   +--- WEB-INF
    |   |   |   |   +--- applicationContext.xml
    |   |   |   |   +--- jsp
    |   |   |   |   |   +--- result.jsp
    |   |   |   |   +--- log4j.xml
    |   |   |   |   +--- web.xml
    ```

2. 报错
   * 错误
        ```
        Client does not support authentication protocol requested by server
        ```
   * 解决:在mysql控制台执行命令
        ```mysql
        alter user 'root'@'localhost' identified with mysql_native_password by 'hjp356908';
        ```

### 3.3.5 添加其他功能
1. 添加查询功能，通过ajax请求完成
2. 页面
   * index.jsp
        ```html
        <tr>
            <td><a href="listStudent.jsp">浏览学生</a></td>
        </tr>
        ```

   * listStudent.jsp
        ```html
        <head>
            <title>查找学生ajax</title>
            <script type="text/javascript" src="js/jquery-3.4.1.js"></script>
            <script type="text/javascript">
                $(function (){
                    //当前页面dom对象加载后，执行loadStudentData（）
                    loadStudentData();
                    $("#btnLoader").click(function (){
                        loadStudentData();
                    })
                })
                function loadStudentData(){
                    $.ajax({
                        url:"student/queryStudent.do",
                        type:"get",
                        dataType:"json",
                        success:function (data){
                            // alert("data="+data);
                            // 清除旧数据
                            $("#info").html("");
                            //显示数据
                            $.each(data,function(i,n){
                                $("#info").append("<tr>")
                                    .append("<td>"+n.id+"<td>")
                                    .append("<td>"+n.name+"<td>")
                                    .append("<td>"+n.age+"<td>")
                                    .append("<tr>")
                            })
                        }
                    })
                }
            </script>
        </head>
        <body>
            <div align="center">
                <table>
                    <thead>
                    <tr>
                        <td>学号</td>
                        <td>姓名</td>
                        <td>年龄</td>
                    </tr>
                    </thead>
                    <tbody id="info">

                    </tbody>
                </table>
                <input type="button" id="btnLoader" value="查询数据">
            </div>
        </body>
        ```
3. Controller
   ```java
    // 处理查询，相应ajax
    @RequestMapping("/queryStudent.do")
    @ResponseBody
    public List<Student> queryStudent() {
        // 参数检查，简单处理数据
        System.out.println("调用查询");
        List<Student> students = service.findStudents();
        System.out.println(students);
        return students;
    }
   ```


