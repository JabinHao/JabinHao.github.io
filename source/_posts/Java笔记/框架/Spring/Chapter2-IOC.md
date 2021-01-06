---
title: Chapter2 IOC
excerpt: 摘要
tags:
  - java
  - IOC
  - spring
categories:
  - Java笔记
  - Spring
banner_img: /img/dog.png
index_img: /img/post/spring-framework_logo.png
abbrlink: 336b991c
date: 2020-12-08 21:21:24
updated: 2020-12-08 21:21:24
subtitle:
---
## 2.1 IOC概述
### 2.1.1 概念及原理
1. 什么是IOC
   * 控制反转，把对象创建和对象之间的调用过程交给Spring进行管理
   * 使用IOC是为了降低耦合度
2. 依赖注入与控制反转
   * 依赖注入（Dependency Injection，DI）和控制反转含义相同，它们是从两个角度描述的同一个概念。
   * 当某个 Java 实例需要另一个 Java 实例时，传统的方法是由调用者创建被调用者的实例（例如，使用 new 关键字获得被调用者实例），而使用 Spring 框架后，被调用者的实例不再由调用者创建，而是由 Spring 容器创建，这称为控制反转。、
   * Spring 容器在创建被调用者的实例时，会自动将调用者需要的对象实例注入给调用者，这样，调用者通过 Spring 容器获得被调用者实例，这称为依赖注入

3. IOC底层原理
   * 关键词：xml解析、工厂模式、反射
   * 工程模式降低耦合度：
     * 新建一个工厂类，提供静态方法用于创建对象，其他类只需要调用该方法即可
     * 缺点：与工厂类仍有耦合
   * IOC原理：
     * xml文件配置创建的对象
     * 工厂类通过反射创建对象
     * 类路径改变时，只需要修改xml配置文件即可，进一步降低了耦合度
        ```java
        //工厂类
        class UserFactory {
            public static UserDao getDao(){
                String classValue = class属性值; // xml解析得到类的全路径
                Class clazz = Class.forName(classValue); // 通过反射创建对象
                return (UserDao)clazz.newInstance();
            }
        }
        ```

### 2.1.2 `BeanFactory`接口
   * IOC思想基于 IOC 容器完成， IOC容器底层就是对象工厂
   * Spring 提供 IOC 容器实现两种方式：（两个接口）
     * `BeanFactory`： 
       * IOC 容器基本实现，是 Spring 内部的使用接口，不提供开发人员进行使用
       * 加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象
     * `ApplicationContext`： 
       * `BeanFactory` 接口的子接口，提供更多更强大的功能，一般由开发人员进行使用
       * 加载配置文件时候就会创建对象
   * `ApplicationContext` 接口实现类
      * `FileSystemXmlApplicationContext`：参数需要填绝对路径（盘符开始）
      * `ClassPathXmlApplicationContext`：参数填配置文件的相对路径（src下）
   * `BeanFactory`接口有一个配置子接口：`ConfigurableApplicationContext`

### 2.1.3 Bean管理
1. 什么是Bean管理
   * Bean管理指两个操作
   * Spring创建对象
   * Spring注入属性
2. Bean管理操作的两种方式
   * 基于xml配置文件的方式
   * 基于注解的方式

## 2.2 基于xml的Bean操作
### 2.2.1 基本步骤
1. 创建对象
   * 在spring配置文件中，使用bean标签，添加对应属性即可创建对象
   * bean标签常用属性
     * id：类的唯一标识（别名）
     * class：类全路径（相对路径）
   * 默认执行无参构造器
2. 注入属性
   * DI：依赖注入，即注入属性，IOC的一种具体实现
   * 注入方式一：通过类的`setter`方法设置
        ```xml
        <!--配置User对象创建-->
        <bean id="user" class="com.atguigu.spring5.User"/>
        <bean id="book" class="com.atguigu.spring5.Book">
            <!--property标签完成属性注入-->
            <property name="bnmae" value="亲密关系"></property>
            <property name="bauthor" value="罗兰·米勒"></property>
        </bean>
        ```
        ```java
        // Book.java
        public class Book {
            private String bnmae;
            private String bauthor;

            public void setBnmae(String bnmae) {
                this.bnmae = bnmae;
            }

            public void setBauthor(String bauthor) {
                this.bauthor = bauthor;
            }
        }
        ```
   * 注入方式二：有参构造器
        ```xml
        <bean id="orders" class="com.atguigu.spring5.Orders">
            <constructor-arg name="address" value="China"></constructor-arg>
            <constructor-arg name="oname" value="PC"></constructor-arg>
        </bean>
        ```
        > 也可使用`<constructor-arg index="0" value="PC"></constructor-arg>`的方式，index表示第几个参数
        ```java
        // Orders.java
        public class Orders {
            private String oname;
            private String address;

            public Orders(String oname, String address) {
                this.oname = oname;
                this.address = address;
            }
        }
        ```
   * p空间注入：可以简化基于 xml的配置方式
      1. 添加 p 名称空间在配置文件中
            ```xml
            <!-- 配置文件 -->
            <?xml version="1.0" encoding="UTF-8"?>
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:p="http://www.springframework.org/schema/p"
                xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
            ```
      2.  进行属性注入，在 bean 标签里面进行操作
            ```xml
            <bean name="book" class="com.atguigu.spring5.Book" p:bnmae="亲密关系" p:bauthor="罗兰·米勒"></bean>
            ```
            
### 2.2.2 xml注入其他属性类型
1. 字面量
   * null值：通过null标签设置
      ```xml
      <property name="bnmae">
      <null/></property>
      ```
   * 特殊符号：
     * 使用`&lt;`、`&gt;`等转义符号
     * 使用 `CDATA`：
          ```xml
          <property name="bnmae">
              <value><!CDATA[<<编程珠玑>>]>
          </property>
          ```
2. 外部bean
   * 创建两个类Service和Dao
   * 在Service类里调用Dao里面的方法
   * 在Spring配置文件中配置
      ```java
      //UserDao接口实现类 UserDaoImpl.java
      public class UserDaoImpl implements UserDao {
          @Override
          public void update() {
              System.out.println("dao update");
          }
      }
      ```
      ```java
      //UserService.java
      public class UserService {

          // 创建UserDao类型属性，生成set方法
          private UserDao userDao;
          public void setUserDao(UserDao userDao) {
              this.userDao = userDao;
          }

          public void add() {
              System.out.println("service add");

              // 创建UserDao对象
              UserDao userDao = new UserDaoImpl();
              userDao.update();

          }
      }
      ```
      ```xml
      <!-- bean2.xml -->
      <bean id="userService" class="com.atguigu.spring5.service.UserService">
          <property name="userDao" ref="userDao"></property>
          <!--name属性：当前类中的属性名-->
          <!--ref属性：创建userDao对象的bean标签id值-->
      </bean>
      <bean id="userDao" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>
      ```
      ```java
      //Junit测试
      @Test
      public void test() {
          ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
          UserService userService = context.getBean("userService", UserService.class);

          System.out.println(userService);
          userService.add();
      }
      ```
3. 内部bean
   * 内部bean即在一个bean标签中嵌套其属性对象对于类的bean标签
   * 新建部门类`Dept`和员工类`Emp`，员工类有一个`Dept`对象属性代表其部门：
      ```java
      //Dept.java
      public class Dept {
          private String dname;

          public void setDname(String dname) {
              this.dname = dname;
          }

          @Override
          public String toString() {
              return "Dept{" +
                      "dname='" + dname + '\'' +
                      '}';
          }
      }
      ```
      ```java
      //Emp.java
      public class Emp {
          private String ename;
          private String gender;
          private Dept dept;

          public void setEname(String ename) {
              this.ename = ename;
          }

          public void setGender(String gender) {
              this.gender = gender;
          }

          public void setDept(Dept dept) {
              this.dept = dept;
          }
      }
      ```
      ```xml
      <bean id="emp" class="com.atguigu.spring5.bean.Emp">
          <property name="ename" value="rui"></property>
          <property name="gender" value="female"></property>
          <property name="dept">
              <bean id="dept" class="com.atguigu.spring5.bean.Dept">
                  <property name="dname" value="Financial Department"/>
              </bean>
          </property>
      </bean>
      ```

4. 级联赋值
   * 方式一：在外部bean中完成
      ```xml
      <bean id="emp" class="com.atguigu.spring5.bean.Emp">
          <property name="ename" value="rui"></property>
          <property name="gender" value="female"></property>
          <property name="dept" ref="dept"></property>
      </bean>
      <bean id="dept" class="com.atguigu.spring5.bean.Dept">
          <property name="dname" value="外部bean"></property><!--多了这一行-->
      </bean>
      ```
   * 方式二：需要在Emp中设置dept的get方法
      ```xml
      <bean id="emp" class="com.atguigu.spring5.bean.Emp">
          <property name="ename" value="rui"></property>
          <property name="gender" value="female"></property>
          <property name="dept" ref="dept"></property>
          <property name="dept.dname" value="级联复制"></property>
      </bean>
      <bean id="dept" class="com.atguigu.spring5.bean.Dept">
          <property name="dname" value="外部bean"></property><!--这一行可有可无-->
      </bean>
      ```

### 2.2.3 注入集合属性
1. 基本数据类型集合
   * 数组、List、Map、Set
   * 步骤
     * 创建类，定义数组、List等类型属性，生成对应的set方法
        ```java
        //Stu.java
        public class Stu {
            private String[] courses;
            private List<String> list;
            private Map<String,String> map;
            private Set<String> set;
            public void setCourses(String[] courses) {
                this.courses = courses;
            }

            public void setList(List<String> list) {
                this.list = list;
            }

            public void setMap(Map<String, String> map) {
                this.map = map;
            }

            public void setSet(Set<String> set) {
                this.set = set;
            }
        }
        ```
     * 在Spring配置文件中进行配置，使用相应的array、list等标签
        ```xml
        <!-- bean1.xml -->
        <bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">
            <property name="courses">
                <array>
                    <value>java课程</value>
                    <value>数据库</value>
                </array>
            </property>
            <property name="list">
                <list>
                    <value>rui</value>
                    <value>lei</value>
                </list>
            </property>
            <property name="map">
                <map>
                    <entry key="JAVA" value="java"></entry>
                    <entry key="PHP" value="php"></entry>
                </map>
            </property>
            <property name="set">
                <set>
                    <value>MySQL</value>
                    <value>Redis</value>
                </set>
            </property>
        </bean>
        ```
2. 在集合中设置对象类型
   * 创建对象集合属性，并设置相应的set方法
        ```java
        // Stu.java
        private List<Course> courseList;
        public void setCourseList(List<Course> courseList) {
            this.courseList = courseList;
        }
        ```
   * 在配置文件中配置，创建多个对象，使用ref标签注入
        ```xml
        <!-- bean1.xml -->
        <bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">
        <!-- 此处省略n行 -->
            <property name="courseList">
                <list>
                    <ref bean="course1"></ref>
                    <ref bean="course2"></ref>
                </list>
            </property>
        </bean>

        <!--  创建多个course对象  -->
        <bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
            <property name="cname" value="Spring5"></property>
        </bean>
        <bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
            <property name="cname" value="MyBatis"></property>
        </bean>
        ```
3. 把集合注入部分提取出来(便于复用)
   * 在spring配置文件中引入名字空间util
        ```xml
        <!-- bean2.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:util="http://www.springframework.org/schema/util"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-beans.xsd">
        ```
   * 使用util标签完成list集合注入提取
        ```xml
        <!-- bean2.xml -->
        <!--  提取list集合类型属性注入  -->
        <util:list id="bookList">
            <value>数据结构</value>
            <value>算法导论</value>
            <value>深入理解JVM虚拟机</value>
        </util:list>
        <!--  使用  -->
        <bean id="book" class="com.atguigu.spring5.collectiontype.Book">
            <property name="list" ref="bookList"></property>
        </bean>
        ```
   * Book.java类：
        ```jav
        public class Book {
            private List<String> list;

            public void setList(List<String> list) {
                this.list = list;
            }
        }
        ```

## 2.3 Bean管理 —— FactoryBean
### 2.3.1 说明
1. Spring有两种类型的Bean，普通bean和工厂bean
2. 普通bean：在配置文件中定义的bean类型（class属性）即为返回类型
3. 工厂bean：在配置文件中定义的bean类型可以和返回的类型不同

### 2.3.2 工厂bean的使用
1. 步骤
   * 创建类，让该类作为工厂bean，实现接口FactoryBean
   * 实现接口方法，在方法中定义返回值类型
2. 示例
   * 工厂类：
        ```java
        // Mybean.java
        public class MyBean implements FactoryBean<Course> {

            // 定义返回类型
            @Override
            public Course getObject() throws Exception {
                Course course = new Course();
                course.setCname("abc");
                return course;
            }

            @Override
            public Class<?> getObjectType() {
                return null;
            }

            @Override
            public boolean isSingleton() {
                return false;
            }
        }
        ```
   * 配置文件
        ```xml
        <!-- bean.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

            <bean id="myBean" class="com.atguigu.spring5.factorybean.MyBean"></bean>
        </beans>
        ```
   * 测试类
        ```java
        // Junit
        @Test
        public void testCollection3() {
            ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
            Course course = context.getBean("myBean", Course.class);
            System.out.println(course);
        }
        ```

## 2.4 Bean的作用域与生命周期
### 2.4.1 Bean作用域
1. Spring中可以设置bean是单实例还是多实例，默认是单实例
2. 如何设置
   * bean标签的`scope`属性
   * `singleton`：单实例对象
   * `prototype`：多实例对象
    ```xml
    <bean id="book" class="com.atguigu.spring5.collectiontype.Book" scope="prototype">
        <property name="list" ref="bookList"></property>
    </bean>
    ```
3. 区别
   * 设置 `scope` 值是 `singleton` 时候，加载 spring 配置文件时候就会创建单实例对象
   * 设置 scope 值是 `prototype` 时候，在调用 `getBean` 方法时才创建多实例对象
4. scope属性其他值
   * request
   * session

### 2.4.2 bean的生命周期
1. 生命周期指从对象创建到销毁的过程
2. bean的生命周期
   * 通过构造器创建bean示例（无参构造器）
   * 为bean的属性设置值以及对其他bean的引用（调用set方法）
   * 调用bean的初始化方法（需要进行配置，init-method属性）
   * 使用bean
   * 当容器关闭时，调用bean的销毁方法（需要配置 destory-method属性）
3. 示例
   * 类：Orders.java
        ```java
        public class Orders {
            private String oname;

            public Orders() {
                System.out.println("1.无参构造器");
            }

            public void setOname(String oname) {
                this.oname = oname;
                System.out.println("2.set方法设置属性值");
            }

            // 初始化方法
            public void initMethod() {
                System.out.println("3.初始化方法");
            }

            // 销毁方法
            public void destoryMethod(){
                System.out.println("5.销毁方法");
            }
        }
        ```
   * 配置文件 bean4.xml
        ```java
        <bean id="orders" class="com.atguigu.spring5.bean.Orders"
            init-method="initMethod" destroy-method="destoryMethod">
            <property name="oname" value="phone"></property>
        </bean>
        ```
   * Junit测试
        ```java
        @Test
        public void testBean() {
            ApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
            Orders orders = context.getBean("orders", Orders.class);
            ((ClassPathXmlApplicationContext)context).close();
        }
        ```

4. bean的后置处理器（bean生命周期共有七步）
   * 通过构造器创建bean示例（无参构造器）
   * 为bean的属性设置值以及对其他bean的引用（调用`set`方法）
   * 将bean示例传递到bean后置处理器的方法 `postProcessBeforeInitialization`
   * 调用bean的初始化方法（需要进行配置，`init-method`属性）
   * 将bean示例传递到bean后置处理器的方法 `postProcessAfterInitialization`
   * 使用bean
   * 当容器关闭时，调用bean的销毁方法（需要配置 `destory-method` 属性）

5. 添加后置处理器
   * 创建 `BeanPostProcessor` 接口的实现类
   * 在实现类中重写两个方法
   * 在配置文件中配置该类
   * 自动为每个bean添加了两个后置处理器方法
    ```java
    // MyBeanPost.java
    public class MyBeanPost implements BeanPostProcessor {
        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("初始化之前执行的方法");
            return bean ;
        }

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("初始化之后执行的方法");
            return bean;
        }
    }
    ```
    ```xml
    <!-- bean4.xml -->
    <bean id="orders" class="com.atguigu.spring5.bean.Orders"
          init-method="initMethod" destroy-method="destoryMethod">
        <property name="oname" value="phone"></property>
    </bean>
    <bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
    ```

## 2.5 xml自动装配
1. 自动装配：根据指定装配规则（属性名称或者属性类型）， Spring 自动将匹配的属性值进行注入
2. 如何装配
   * bean标签的autowire属性，配置自动装配
   * byName：根据属性名称注入，要求注入的bean的id值与当前bean的属性值相同
   * byType：根据属性类型注入（即根据对象属性的类名），当同一个类有多个bean时会报错
3. 示例
   * 部门类Dept
        ```java
        public class Dept {
            @Override
            public String toString() {
                return "Dept{}";
            }
        }
        ```
   * 员工类Emp
        ```java
        public class Emp {
            private Dept dept;

            public void setDept(Dept dept) {
                this.dept = dept;
            }

            @Override
            public String toString() {
                return "Emp{" +
                        "dept=" + dept +
                        '}';
            }
        }
        ```
   * 配置文件
        ```xml
        <bean id="emp" class="com.atguigu.spring5.autowire.Emp"  autowire="byName">
        </bean>
        <bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
        ```

## 2.6 外部属性文件
### 2.6.1 含义
1. 从外部的属性文件中导入配置（.properties）
2. 便于修改

### 2.6.2 示例 —— 引入数据库配置
1. 直接配置数据库信息
   * 配置德鲁伊连接池 —— 导入jar包
   * 直接配置
        ```xml
        <!--  引入外部属性文件  -->
        <context:property-placeholder location="classpath:jdbc.properties"/>
        <!--  配置连接池  -->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${prop.driverClass}"/>
            <property name="url" value="${prop.url}"/>
            <property name="username" value="${prop.userName}"/>
            <property name="password" value="${prop.password}"/>
        </bean>
        ```
2. 引入外部文件配置
   * 创建属性文件（.properties），填写数据库信息
        ```
        // jdbc.properties
        prop.driverClass=com.mysql.jdbc.Driver
        prop.url=jdbc:mysql://localhost:3306/userDb
        prop.userName=root
        prop.password=root
        ```
   * 将属性文件引入spring配置文件
        * 引入context名称空间
            ```xml
            <beans xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:context="http://www.springframework.org/schema/context"
                xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">            
            ```
        * 使用标签引入外部属性文件
            ```xml
            <context:property-placeholder location="classpath:jdbc.properties"/>
            ```
        * 配置连接池
            ```xml
            <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
                <property name="driverClassName" value="${prop.driverClass}"/>
                <property name="url" value="${prop.url}"/>
                <property name="username" value="${prop.userName}"/>
                <property name="password" value="${prop.password}"/>
            </bean>
            ```

## 2.7 基于注解的bean管理
### 2.7.1 概述
1. 什么是注解
   * 注解是代码特殊标记，格式： @注解名称(属性名称=属性值, 属性名称=属性值..)
   * 注解可以用于类、属性、方法
   * 使用注解的目的：简化xml配置
2. spring针对bean管理操作中创建对象提供的注解
   * @Component
   * @Service
   * @Controller
   * @Repository

    实际应用中，四个注解功能相同，都可以用于创建对象

### 2.7.2 注解创建对象  
1. 步骤
   * 引入依赖：spring-aop-5.2.3.RELEASE.jar
   * 开启组件扫描（context名称空间），如果有多个包：
     * 可以用逗号隔开
     * 也可以扫描上层目录
   * 创建类，在类上添加注解：
     * value属性的值等价于bean标签中的id属性值
     * 可以不写value属性，默认为类名首字母小写
2. 示例
   * `UserService`类
        ```java
        @Component
        public class UserService {

            public void add() {
                System.out.println("service add ...........");
            }
        }
        ```
   * bean1.xml 配置文件
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

            <!--  开启组件扫描  -->
            <context:component-scan base-package="com.atguigu.spring5"></context:component-scan>
        </beans>
        ```
3. 组件扫描细节
   * `use-default-filters="false"` 表示现在不使用默认 filter，自己配置 filter
      * `context:include-filter`，设置扫描哪些内容
      * 示例：只扫描`Component`注解
        ```xml
        <context:component-scan base-package="com.atguigu.spring5" use-default-filters="false">
            <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
        </context:component-scan>
        ```
   * `context:exclude-filter`： 设置哪些内容不进行扫描
      * 示例：不扫描 `Component` 注解
        ```xml
        <context:component-scan base-package="com.atguigu.spring5">
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component"/>
        </context:component-scan>
        ```

### 2.7.3 基于注解的属性注入
1. @AutoWired：根据属性类型自动装配
   * 当前类以及属性对象对应的类添加创建对象注解
   * 在属性上添加AutoWired注解
    ```java
    //当前类UserService.java
    @Service
    public class UserService {

        // 定义dao类型属性
        @Autowired
        private UserDao userDao;
        public void add() {
            System.out.println("service add ...........");
            userDao.add();
        }
    }
    ```
    ```java
    //UserDaoImpl.java类
    @Repository
    public class UserDaoImpl implements UserDao{
        @Override
        public void add() {
            System.out.println("dao add......");
        }
    }
    ```
   
2. @Qualifier：根据名称进行注入，与`AutoWired`一起使用
   
   ```java
    //当前类UserService.java
    @Service
    public class UserService {

        // 定义dao类型属性
        @Autowired
        @Qualifier(value = "userDaoImpl1")
        private UserDao userDao;
        public void add() {
            System.out.println("service add ...........");
            userDao.add();
        }
    }
    ```
    ```java
    //UserDaoImpl.java类
    @Repository(name = "userDaoImpl1")
    public class UserDaoImpl implements UserDao{
        @Override
        public void add() {
            System.out.println("dao add......");
        }
    }
    ```

3. @Resource：可以根据类型注入，也可以根据名称注入
    * 根据类型注入
        ```java
        //UserService.java
        @Resource  //根据类型进行注入
        private UserDao userDao;
        ```
        
    * 根据名称进行注入
        ```java
        //UserService.java
        @Resource(name = "userDaoImpl1")  //根据名称进行注入
        private UserDao userDao;
        ```
    * Resource可能会报错，需要导包：`import javax.annotation.Resource;`

4. @Value：注入普通属性
    * ``
        ```java
        //UserService.java
        public class UserService {

            @Value(value = "abc")
            private String name;
        }
        ```

### 2.7.4 完全注解开发
1. 创建配置类，替代xml配置文件
    * 使用`@Configuration`注解
        ```java
        //pringConfig.java
        @Configuration  //作为配置类，替代xml配置文件
        @ComponentScan(basePackages = {"com.atguigu"})
        public class SpringConfig {

        }
        ```
2. 编写测试类
    ```java
    @Test
    public void testService2() {
        //加载配置类
        ApplicationContext context
                = new AnnotationConfigApplicationContext(SpringConfig.class);
        UserService userService = context.getBean("userService", UserService.class);
        System.out.println(userService);
        userService.add();
    }

    ```





