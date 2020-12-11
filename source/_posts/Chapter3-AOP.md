---
title: Chapter3 AOP
excerpt: 摘要
tags:
  - java
  - spring
  - AOP
categories:
  - Java笔记
  - Spring
banner_img: /img/dog.png
index_img: /img/post/spring-framework_logo.png
abbrlink: 84a6a0ca
date: 2020-12-11 00:26:38
updated: 2020-12-11 20:51:00
subtitle:
---
## 3.1 理解AOP
### 3.1.1 什么是AOP
1. 面向切面编程（方面）， 利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
2. 通俗描述：不通过修改源代码方式，在主干功能里面添加新功能

### 3.1.2 AOP底层原理
1. AOP底层使用JDK动态代理
   * 有接口，使用JDK动态代理
      * 接口与实现类
        ```java
        interface UserDao{
            public void login();
        }
        ```
        ```java
        class UserDaoImpl implements UserDao() {
            public void login() {
                // 实现登录过程
            }
        }
     * 创建实现类的代理对象，增强类的方法 
   * 没有接口，使用CGLIB动态代理
     * 类（本来的类）
        ```java
        class User() {
            public add() {
                //.....
            }
        }
        ```
     * 创建其子类的代理对象
  
2. JDK动态代理示例
   * 所需类及方法
     * `java.lang.reflect.Proxy`
     * `newProxyInstance`静态方法，参数：
       * 类加载器
       * 增强方法所在的类实现的接口，支持多个接口
       * `InvocationHandler` 接口实现类，在里面创建代理对象，实现增强方法
   * 代码
      * 创建被代理的类及其接口
        ```java
        // Interface：UserDao.java
        public interface UserDao {

            public int add(int a, int b);

            public Spring update(Spring id);
        }
        ```
        ```java
        // 实现类 UserDaoImpl.java
        public class UserDaoImpl implements UserDao{
            @Override
            public int add(int a, int b) {
                System.out.println("add方法执行了");
                return a+b;
            }

            @Override
            public Spring update(Spring id) {
                return id;
            }
        }
        ```
     * 创建一个实现接口`InvocationHandler`的类，它必须实现`invoke`方法，以完成代理的具体操作。
        ```java
        // UserDaoProxy.java
        class UserDaoProxy implements InvocationHandler {

            private Object obj; //被代理的对象
            //有参构造器
            public UserDaoProxy(Object obj) {
                this.obj = obj;
            }

            // 增强的逻辑
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                // 方法之前
                System.out.println("方法之前之前执行..."+method.getName()+":传递的参数..."+ Arrays.toString(args));

                // 被增强的方法执行
                Object res = method.invoke(obj,args);

                // 方法之后
                System.out.println("方法执行之后....");

                return res;
            }
        }
        ```
     * 通过`Proxy`的静态方法`newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)` 创建一个UserDao接口代理
        ```java
        //创建接口实现类代理对象
        UserDaoImpl userDao = new UserDaoImpl();
        //得到代理对象
        UserDao dao = (UserDao) Proxy.newProxyInstance(UserDaoImpl.class.getClassLoader(), userDao.getClass().getInterfaces() , new UserDaoProxy(userDao));
        //通过代理对象调用原来的方法
        dao.add(1+2);
        ```

### 3.1.3 AOP 术语
1. 连接点  
   类里面可以被增强的方法，称为连接点
2. 切入点   
   实际中真正被增强的方法，称为切入点
3. 通知（增强）
   * 实际增强的逻辑部分称为通知
   * 通知的类型
     * 前置通知
     * 后置通知
     * 环绕通知
     * 异常通知
     * 最终通知（类似于try-catch-finally中的finally）
4. 切面  
   切面指的是一个过程，即把通知应用到切入点的过程

## 3.2 AOP 操作
### 3.2.1 准备工作
1. Spring框架一般基于AspectJ实现AOP操作，AspectJ并非AOP的一部分，但一般两者一起使用
2. 基于AspectJ实现AOP操作的方式
   * 基于xml配置文件实现
   * 基于注解方式实现（实际使用）
3. 引入相关依赖
4. 切入点表达式
   * 作用：生命对哪个类的哪个方法进行增强
   * 语法结构 `execution(权限修饰符 返回类型 全类名 方法(参数))`
     * `execution(* com.atguigu.dao.BookDao.add(..))` // *表示任意权限
     * `execution(* com.atguigu.dao.BookDao.*(..))` // 增强所有方法
     * `execution(* com.atguigu.dao.* *(..))` // dao包下所有类中的所有方法
### 3.2.2 AspectJ注解方式
1. 创建要被增强的类
   ```java
    public class User {

        public void add() {
            System.out.println("add....");
        }
    }
   ```
2. 创建增强类，并编写增强逻辑
   ```java
    public class UserProxy {

        // 前置通知
        public void before() {
            System.out.println("before.........");
        }
    }
   ```
3. 进行通知的配置
   * 在spring配置文件中，开启注解扫描
        ```xml
        <!-- bean1.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xmlns:aop="http://www.springframework.org/schema/aop"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                                http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

            <!-- 开启注解扫描 -->
            <context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>
        </beans>
        ```
   * 使用注解创建User和UserProxy对象
        ```java
        @Component
        public class UserProxy {
            ...
        ```
        ```java
        @Component
        public class User {
            ...
        ```
   * 在增强类上面添加注解 `@Aspect`
        ```java
        @Component
        @Aspect // 生成代理对象
        public class UserProxy {
            ...
        ```
   * 在spring配置文件中开启生成代理对象
        ```xml
        <!-- bean1.xml -->
        <!-- 开启Aspect生成代理对象 -->
        <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
        ```
4. 配置不同类型的通知 
    ```java
    //UserProxy.java
    //前置通知
    //@Before注解表示作为前置通知
    @Before(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void before() {
        System.out.println("before.........");
    }

    //后置通知（返回通知）
    @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterReturning() {
        System.out.println("afterReturning.........");
    }

    //最终通知
    @After(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void after() {
        System.out.println("after.........");
    }

    //异常通知
    @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterThrowing() {
        System.out.println("afterThrowing.........");
    }

    //环绕通知
    @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前.........");

        //被增强的方法执行
        proceedingJoinPoint.proceed();

        System.out.println("环绕之后.........");
    }
    ```

5. 相同的切入点抽取
   * 上面的几个增强方法后的(value="")相同，可以抽取出来
   * 自定义一个方法，加上注解`@Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")`
   * 把通知的注解值改为(value="自定义的方法名")
    ```java
    public class UserProxy {

        //相同切入点抽取
        @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
        public void pointdemo() {

        }

        //前置通知
        //@Before注解表示作为前置通知
        @Before(value = "pointdemo()")
        public void before() {
            System.out.println("before.........");
        }
        ....
    ```

6. 设置增强类优先级
   * 当有多个增强类对同一个方法进行增强时，可以设置优先级
   * 使用注解@Order(数字)，数字越小优先级越高
    ```java
    @Component
    @Aspect  //生成代理对象
    @Order(3)
    public class UserProxy {
        ...
    ```
7. 完全使用注解开发
   * 创建配置类
   * 使用注解 `@EnableAspectJAutoProxy`
    ```java
    @Configuration
    @ComponentScan(basePackages = {"com.atguigu"})
    @EnableAspectJAutoProxy(proxyTargetClass = true)
    public class ConfigAop {
    }
    ```

### 3.2.3 AspectJ xml配置文件方式
1. 创建增强类与被增强类
    ```java
    //Book.java
    public class Book {
        public void buy() {
            System.out.println("buy.............");
        }
    }
    ```
    ```java
    //BookProxy
    public class BookProxy {
        public void before() {
            System.out.println("before.........");
        }
    }
    ```

2. 在配置文件中创建两个对象
   ```xml
    <!--创建对象-->
    <bean id="book" class="com.atguigu.spring5.aopxml.Book"></bean>
    <bean id="bookProxy" class="com.atguigu.spring5.aopxml.BookProxy"></bean>
   ```
3. 在配置文件中配置切入点
    ```xml
    <!--配置 aop 增强-->
    <aop:config>
    <!--切入点-->
    <aop:pointcut id="p" expression="execution(*
    com.atguigu.spring5.aopxml.Book.buy(..))"/>
    <!--配置切面-->
    <aop:aspect ref="bookProxy">
    <!--增强作用在具体的方法上-->
    <aop:before method="before" pointcut-ref="p"/>
    </aop:aspect>
    </aop:config>
    ```

