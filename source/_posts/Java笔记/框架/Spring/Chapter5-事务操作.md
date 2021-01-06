---
title: Chapter5 事务操作
excerpt: spring5的事务操作，注解开发、xml配置文件开发以及纯注解开发
tags:
  - java
  - spring
categories:
  - Java笔记
  - Spring
banner_img: /img/dog.png
index_img: /img/post/spring-framework_logo.png
abbrlink: 9b32c473
date: 2020-12-11 23:45:30
updated: 2020-12-30 18:19:07
subtitle:
---
## 5.1 事务
### 5.1.1 事务简介
1. 什么是事务  
   * 事务是数据库中的概念
   * 事务是数据库操作最基本单元，逻辑上一组操作，要么都成功，如果有一个失败所有操作都失败
   * 事务的四个特性：ACID（原子性、一致性、隔离性、持久性）
2. 使用场景
   * 当数据库操作涉及到多个表，或者是多个sql语句的insert，update，delete，需要保证这些语句都是成功才能完成我的功能，或者都失败，保证操作是符合要求的
3. jdbc,mybatis,hibernate各自的事务处理
   * jdbc处理事务： Connection conn ; conn.commit(); conn.rollback();
   * mybatis处理事务： SqlSession.commit();  SqlSession.rollback();
   * hibernate处理事务： Session.commit(); Session.rollback();

4. 问题
   * 就是多种数据库的访问技术，有不同的事务处理的机制，对象，方法。
   * spring提供一种处理事务的统一模型， 能使用统一步骤，方式完成多种不同数据库访问技术的事务处理

### 5.1.2 Spring事务管理
1. 说明
   * 事务原本是数据库中的概念，在 Dao 层。但spring中，一般将事务提升到业务层，即 Service 层，这样做是为了能够使用事务的特性来管理具体的业务。
   * spring处理事务的模型，使用的步骤都是固定的。只需把事务使用的信息提供给spring
2. spring事务操作方式
   * 编程式事务管理：手动编写程序，较为繁琐一般不用
   * 声明式事务管理，使用注解或xml配置文件（常用），**底层使用AOP原理**

### 5.1.3 环境搭建
1. 创建数据库表
   * 新建数据库spring5和表t_account
      ```mysql
      CREATE DATABASE `spring5` CHARACTER SET utf8;
      USE spring5;
      DROP TABLE IF EXISTS t_account;
      (
          id           VARCHAR(20) NOT NULL ,  # 主键
          username     VARCHAR(50) ,           # 姓名
          money        INT,                    # 存款
          PRIMARY KEY (id)                     # 设置主键
      );
      ```
   * 新建MySQL用户并授权
      ```mysql
      CREATE USER 'Jacob' @'%' IDENTIFIED BY 'jacob12015229'; # 创建用户
      GRANT ALL PRIVILEGES ON spring5.* TO 'Jacob'@'%'; # 授权
      ```

2. 创建maven项目
   * 新建空的Maven项目，JDK版本选择1.8
   * pom.xml:
        ```xml
        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
        </properties>


        <dependencies>
            <!--单元测试-->
            <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.7.0</version>
            <scope>test</scope>
            </dependency>
            <!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
            <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.5.RELEASE</version>
            <scope>test</scope>
            </dependency>
            <!--spring核心ioc-->
            <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.5.RELEASE</version>
            </dependency>
            <!--做spring事务用到的-->
            <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.2.5.RELEASE</version>
            </dependency>
            <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.5.RELEASE</version>
            </dependency>
            <!--mybatis依赖-->
            <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.1</version>
            </dependency>
            <!--mybatis和spring集成的依赖-->
            <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.1</version>
            </dependency>
            <!--mysql驱动-->
            <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
            <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.21</version>
            </dependency>
            <!--阿里公司的数据库连接池-->
            <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
            <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.3</version>
            </dependency>
        </dependencies>

        <build>
            <!--目的是把src/main/java目录中的xml文件包含到输出结果中。输出到classes目录中-->
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
            <!--指定jdk的版本-->
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
        ```
3. 创建类和相关接口
   * 先看一下项目结构
        ```
        src
        ├─main
        │  ├─java
        │  │  └─com
        │  │      └─atguigu
        │  │          ├─dao
        │  │          ├─domain
        │  │          └─service
        │  └─resources
        └─test
            └─java
                └─com
                    └─atguigu
        ```
   * dao下新建UserDao和mapper文件UserDao.xml
        ```java
        public interface UserDao {

            public void addMoney(@Param("name") String name, @Param("money") Integer money);
            public void reduceMoney(@Param("name") String name, @Param("money") Integer money);
        }
        ```
        ```xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE mapper
                PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-mapper.dtd"><!--指定约束文件-->
        <mapper namespace="com.atguigu.dao.UserDao">
            <update id="addMoney">
                update t_account set money=money+#{money} where username=#{name}
            </update>
            <update id="reduceMoney">
                update t_account set money=money-#{money} where username=#{name}
            </update>
        </mapper>
        ```
   * service下新建UserService
        ```java
        @Service
        public class UserService {

            // 注入dao
            @Autowired
            private UserDao userDao;

            // 转账
            public void accountMoney(String n1, String n2, Integer money) {
                userDao.addMoney(n1,money);
                userDao.reduceMoney(n2,money);
            }
        }
        ```
4. 配置spring和mybatis
    * resources下新建applicationContext.xml文件
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xmlns:tx="http://www.springframework.org/schema/tx"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
                                http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

            <!-- 组件扫描 -->
            <context:component-scan base-package="com.atguigu"></context:component-scan>

            <context:property-placeholder location="classpath:jdbc.properties" />
            <!--声明数据源DataSource, 作用是连接数据库的-->
            <bean id="myDataSource" class="com.alibaba.druid.pool.DruidDataSource"
                init-method="init" destroy-method="close">
                <!--set注入给DruidDataSource提供连接数据库信息 -->
                <!--    使用属性配置文件中的数据，语法 ${key} -->
                <property name="url" value="${jdbc.url}" /><!--setUrl()-->
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}" />
                <property name="maxActive" value="${jdbc.max}" />
            </bean>

            <!--mybatis-->
            <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
                <!--set注入，把数据库连接池付给了dataSource属性-->
                <property name="dataSource" ref="myDataSource" />
                <property name="configLocation" value="classpath:mybatis.xml" />
            </bean>
            <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
                <!--指定SqlSessionFactory对象的id-->
                <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
                <property name="basePackage" value="com.atguigu.dao"/>
            </bean>
        </beans>
        ```
    * resources下新建mybatis.xml文件
        ```xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>

            <!--设置别名-->
            <typeAliases>
                <package name="com.atguigu.domain"/>
            </typeAliases>
            <!-- sql mapper(sql映射文件)的位置-->
            <mappers>
                <package name="com.atguigu.dao"/>
            </mappers>
        </configuration>
        ```
    * resources下新建jdbc.properties文件（这里用的是mysql8.0）
        ```
        jdbc.url=jdbc:mysql://localhost:3306/spring5?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true
        jdbc.username=Jacob
        jdbc.password=jacob12015229
        jdbc.driver=com.mysql.cj.jdbc.Driver
        jdbc.max=30
        ```
5. 测试
   * 使用spring5+junit5的方式
   * 在test/java/com/atguigu下新建测试类：
        ```java
        package com.atguigu;

        import com.atguigu.service.UserService;
        import org.junit.jupiter.api.Test;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

        @SpringJUnitConfig(locations = "classpath:applicationContext.xml")
        public class TestAccount {

            @Autowired
            private UserService userService;

            @Test
            public void testAccount(){
                userService.accountMoney("rui","mei",100);
            }
        }
        ```
6. 模拟编程式的事务处理
   * 可以在UserService类中手动进行事务处理
   * 代码：
        ```java
        // 事务操作
        public void accountMoney(String n1, String n2, Integer money) {
            try {
                // 1. 开启事务
                // 2. 进行事务操作
                userDao.addMoney(n1,money);
                // 模拟异常
                int i = money/0;
                userDao.reduceMoney(n2,money);
                // 3. 无异常，提交事务
            } catch (Exception e) {
                // 4. 有异常，事务回滚
                System.out.println("rollback");
            }
        ```

## 5.2 事务管理API
### 5.2.1 事务管理器接口
1. Spring提供 `PlatformTransactionManager` 接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类
2. 常用的实现类
   * `DataSourceTransactionManager`：使用 JDBC 或 MyBatis 进行数据库操作时使用
   * `HibernateTransactionManager`：使用 Hibernate 进行持久化数据时使用。
3. spring回滚方式  
   * Spring 事务的默认回滚方式是： 发生运行时异常和 error 时回滚，发生受查(编译)异常时提交。
   * 对于受查异常，程序员也可以手工设置其回滚方式。
4. java中的错误与异常
   * Throwable 类是 Java 语言中所有错误或异常的超类。只有当对象是此类(或其子类之一)的实例时， 才能通过 Java 虚拟机或者 Java 的 throw 语句抛出。
   * Error 是程序在运行过程中出现的无法处理的错误，比如 OutOfMemoryError、ThreadDeath、 NoSuchMethodError 等。当这些错误发生时，程序是无法处理（捕获或抛出）的， JVM 一般会终止线程
   * 程序在编译和运行时出现的另一类错误称之为异常，分为运行时异常与受查异常
     * 运行时异常，是 RuntimeException 类或其子类， 即只有在运行时才出现的异常。如，NullPointerException、 ArrayIndexOutOfBoundsException、 IllegalArgumentException 等均属于运行时异常。这些异常由 JVM 抛出，在编译时不要求必须处理（捕获或抛出）
     * 受查异常，也叫编译时异常，即在代码编写时要求必须捕获或抛出的异常，若不处理，则无法通过编译。如 SQLException， ClassNotFoundException， IOException
     * RuntimeException 及其子类以外的异常，均属于受查异常。当然，用户自定义的 Exception的子类，即用户自定义的异常也属受查异常，**所以自己手动创建抛出的异常不会使spring事务回滚**

        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/java/spring/5-2-1.png)

### 5.2.2 事务定义接口
1. `TransactionDefinition` 接口
   * 事务定义接口 `TransactionDefinition` 中定义了事务描述相关的三类常量：事务隔离级别、事务传播行为、事务默认超时时限，及对它们的操作
   * 这些常量用于在`Transactional`中设置事务的属性
2. 多事务操作的问题
   * 脏读：一个未提交事务读取到另一个未提交事务的数据（第二个事务回滚则会脏读）
   * 不可重复读：一个未提交事务读取到另一提交事务修改数据
   * 虚读/幻读：一个未提交事务读取到另一提交事务添加数据
3. 隔离级别常量
   * DEFAULT： 采用 DB 默认的事务隔离级别。 MySql 的默认为 REPEATABLE_READ； Oracle默认为 READ_COMMITTED。
   *  READ_UNCOMMITTED： 读未提交。未解决任何并发问题。
   *  READ_COMMITTED： 读已提交。解决脏读，存在不可重复读与幻读。
   *  REPEATABLE_READ： 可重复读。解决脏读、不可重复读，存在幻读
   *  SERIALIZABLE： 串行化。不存在并发问题。

        &nbsp;|脏读|不可重复读|幻读
        :-:|:-:|:-:|:-:
        READ_UNCOMMITTED|有|有|有
        READ_COMMITTED|无|有|有
        REPEATABLE_READ|无|无|有
        SERIALIZABLE|无|无|无

4. 事务传播行为常量
   * `PROPAGATION_REQUIRED`：指定的方法必须在事务内执行。若当前存在事务，就加入到当前事务中；若当前没有事务，则创建一个新事务。
   * `PROPAGATION_REQUIRES_NEW`：总是新建一个事务，若当前存在事务，就将当前事务挂起，直到新事务执行完毕。
   * `PROPAGATION_SUPPORTS`：指定的方法支持当前事务，但若当前没有事务，也可以以非事务方式执行。
   * `PROPAGATION_MANDATORY`
   * `PROPAGATION_NESTED`
   * `PROPAGATION_NEVER`
   * `PROPAGATION_NOT_SUPPORTED`
5. 默认事务超时时限
   * 常量 `TIMEOUT_DEFAULT` 定义了事务底层默认的超时时限， sql 语句的执行时长。
   * 事务的超时时限起作用的条件比较多，且超时的时间计算点较复杂，一般使用默认值即可


## 5.3 使用注解的事务管理
### 5.3.1 `@Transactional` 注解
1. 通过 `@Transactional` 注解方式， 可将事务织入到相应 public 方法中，实现事务管理。
2. `@Transactional` 注解可以加到service类上，表示类中所有方法都添加事务，也可以添加到具体的方法上
3. 属性
   * `propagation`： 用于设置事务传播属性。该属性类型为 `Propagation` 枚举，默认值为 `Propagation.REQUIRED`
   * `isolation`： 用于设置事务的隔离级别。该属性类型为 `Isolation` 枚举，默认值为 `Isolation.DEFAULT`
   * `readOnly`： 用于设置该方法对数据库的操作是否是只读的。该属性为 `boolean`，默认值为 `false`。
   * `timeout`： 用于设置本操作与数据库连接的超时时限。单位为秒，类型为 int，默认值为-1，即没有时限。
   * `rollbackFor`： 指定需要回滚的异常类。类型为 `Class[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组
   * `rollbackForClassName`： 指定需要回滚的异常类类名。类型为 `String[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组。
   * `noRollbackFor`： 指定不需要回滚的异常类。类型为 `Class[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组。
   * `noRollbackForClassName`： 指定不需要回滚的异常类类名。类型为 `String[]`，默认值为空数组。当然，若只有一个异常类时，可以不使用数组
### 5.3.2 使用步骤
1. 在applicationContext.xml文件中配置spring事务管理器
    ```xml
    <!--使用spring的事务处理器-->
    <!--声明事务处理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--连接的数据库，指定数据源-->
        <property name="dataSource" ref="myDataSource"/>
    </bean>
    ```
2. 在中配置文件，开启事务注解
    ```xml
    <!--开启事务注解驱动-->
        <tx:annotation-driven transaction-manager="transactionManager"/>
    ```
3. 在service类或其方法上添加事务注解 `@Transactional`
    ```java
    @Service
    @Transactional
    public class UserService {
    ...
    ```
4. 测试
   * 在service中添加异常
      ```java
      // 转账
      public void accountMoney(String n1, String n2, Integer money) {
          userDao.addMoney(n1,money);
          int i = money/0;
          userDao.reduceMoney(n2,money);
      }
      ```
   * 执行测试方法，刷新数据库表会发现两人的存款金额位发生变化。

## 5.4 使用配置文件的事务管理
### 5.4.1 说明
1. 缺点：每个目标类都需要配置事务代理。当目标类较多，配置文件会变得非常臃肿
2. 需要引入AspectJ的依赖：
    ```xml
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-aspects -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>5.3.2</version>
    </dependency>
    ```

### 5.4.2 步骤
1. 修改配置文件applicationContext.xml
    ```xml
    <!--使用spring的事务处理器-->
    <!--声明事务处理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--连接的数据库，指定数据源-->
        <property name="dataSource" ref="myDataSource"/>
    </bean>

    <!--配置通知-->
    <tx:advice id="txadvice" transaction-manager="transactionManager">
        <!--配置事务参数-->
        <tx:attributes>
            <!--指定在哪种规则的方法上添加事务-->
            <tx:method name="accountMoney" propagation="REQUIRED"/>
            <!--<tx:method name="account*"/>&lt;!&ndash;表示给所有account开头的方法添加事务&ndash;&gt;-->
        </tx:attributes>
    </tx:advice>
    <!--配置切入点和切面-->
    <aop:config>
        <!--配置切入点-->
        <aop:pointcut id="pt" expression="execution(* com.atguigu.service.UserService.*(..))"/>
        <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>
    </aop:config>
    ```
2. 删除掉service中的 `Transactional`注解
    ```java
    @Service
    // @Transactional
    public class UserService {

        // 注入dao
        @Autowired
        private UserDao userDao;

        // 转账
        public void accountMoney(String n1, String n2, Integer money) {
            userDao.addMoney(n1,money);
            int i = money/0;
            userDao.reduceMoney(n2,money);
        }
    }
    ```
3. 测试
    ```java
    @SpringJUnitConfig(locations = "classpath:applicationContext.xml")
    public class TestAccount {

        @Autowired
        private UserService userService;

        @Test
        public void testAccount() {
            userService.accountMoney("rui","mei",100);

        }
    }
    ```
4. 注意事项
   * 在基于xml的声明式事务中，配置文件中事务属性的`tx:method`是必须配置的，如果没有配置则事务对这个方法不生效（即使配置了切入点表达式）
   * `tx:method`中可以通过`rollback-for`属性配置回滚的异常，默认运行时异常，建议编译时异常+运行时异常

## 5.5 完全注解开发
### 5.5.1 说明
1. 完全注解开发是指无需spring、mybatis配置文件，完全使用注解来开发
2. 需要新建一个配置类，代替配置文件，并删除原来的xml配置文件

### 5.5.2 步骤
1. 新建配置类TxConfig（com.atguigu.config包下）
    ```java
    @Configuration //配置类
    @ComponentScan(basePackages = "com.atguigu") // 组件扫描
    @EnableTransactionManagement //开启事务
    @PropertySource("classpath:jdbc.properties") // 读取数据库连接信息
    @MapperScan("com.atguigu.dao") // 扫描mapper映射文件
    public class TxConfig {
        // 1. 读取连接连接信息
        @Value("${jdbc.url}")
        private String url;
        @Value("${jdbc.username}")
        private String username;
        @Value("${jdbc.password}")
        private String password;
        @Value("${jdbc.max}")
        private String maxActive;
        @Value("${jdbc.driver}")
        private String driverClassName;

        // 2. 创建数据库连接池
        @Bean
        public DruidDataSource getDruidDataSource() {
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setUrl(url);
            dataSource.setUsername(username);
            dataSource.setPassword(password);
            dataSource.setMaxActive(Integer.parseInt(maxActive));
            dataSource.setDriverClassName(driverClassName);
            System.out.println("数据库连接池");
            return dataSource ;
        }

        // 3. 创建mybatis的SqlSessionFactoryBean对象
        @Bean
        public SqlSessionFactoryBean getSqlSessionFactoryBean(DataSource dataSource) throws IOException {
            SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
            sqlSessionFactoryBean.setDataSource(dataSource);
            sqlSessionFactoryBean.setTypeAliasesPackage("com.atguigu.domain"); // 设置别名
            return sqlSessionFactoryBean;
        }

        // 4. 创建事务管理器
        @Bean
        public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
            DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
            transactionManager.setDataSource(dataSource);
            return transactionManager;
        }
    }
    ```
2. 测试
    ```java
    @SpringJUnitConfig(TxConfig.class)
    public class TestAccount {

        @Autowired
        private UserService userService;

        @Test
        public void testAccount() {
            userService.accountMoney("rui","mei",100);

        }
    }
    ```



