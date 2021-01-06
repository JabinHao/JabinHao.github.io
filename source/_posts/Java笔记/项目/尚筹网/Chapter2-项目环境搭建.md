---
title: Chapter2 项目环境搭建
excerpt: 摘要
tags:
  - java
  - ssm
categories:
  - Java笔记
  - 项目
  - 尚筹网
banner_img: /img/post/banner/mandao.png
index_img: /img/post/ssm.png
abbrlink: 82ed269d
date: 2020-12-27 16:30:35
updated: 2021-01-03 00:27:53
subtitle:
---
## 2.1 创建工程
### 2.1.1 项目架构

### 2.1.2 创建空工程
1. 新建一个空项目，命名为 atcrowdfunding
2. 新建三个 Maven Module，不选择模板(即不勾选 Create from archetype)
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-1-2_1.png)
   * atcrowdfunding01-admin-parent
   * atcrowdfunding05-common-util
   * atcrowdfunding06-common-reverse
3. 新建三个 Maven Module，不选择模板，Parent选择atcrowdfunding01-admin-parent
   * atcrowdfunding02-admin-webui
   * atcrowdfunding03-admin-component
   * atcrowdfunding04-admin-entity
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-1-2.png)
4. 选择打包方式
   * atcrowdfunding01-admin-parent下的pom.xml
        ```xml
        <groupId>com.atguigu.crowd</groupId>
        <artifactId>atcrowdfunding01-admin-parent</artifactId>
        <packaging>pom</packaging>    <!--打包方式为pom-->
        <version>1.0-SNAPSHOT</version>

        ```
   * atcrowdfunding02-admin-webui下的pom.xml
        ```xml
        <modelVersion>4.0.0</modelVersion>
        <packaging>war</packaging>
        ```

   * 其它Module下的pom.xml
        ```xml
        <modelVersion>4.0.0</modelVersion>
        <packaging>jar</packaging>
        ```

### 2.1.3 建立工程之间的依赖关系
1. 依赖关系：
   * webui 依赖 component
   * component 依赖 entity
   * component 依赖 util
2. 建立依赖关系
   * atcrowdfunding02-admin-webui下的pom.xml文件：

        ```xml
        <dependencies>
            <dependency>
                <groupId>com.atguigu.crowd</groupId>
                <artifactId>atcrowdfunding03-admin-component</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
        </dependencies>
        ```
   * atcrowdfunding03-admin-component下的pom.xml文件

        ```xml
        <dependencies>
            <dependency>
                <groupId>com.atguigu.crowd</groupId>
                <artifactId>atcrowdfunding04-admin-entity</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>com.atguigu.crowd</groupId>
                <artifactId>atcrowdfunding05-common-util</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
        </dependencies>
        ```

## 2.2 创建数据库和数据库表
### 2.2.1 

### 2.2.2 创建
1. 新建数据库
   ```mysql
   CREATE DATABASE `project_crowd` CHARACTER SET utf8;
   ```
2. 新建表
   ```mysql
   USE project_crowd;
   DROP TABLE IF EXISTS t_admin;
   CREATE TABLE t_admin
   (
        id            INT NOT NULL AUTO_INCREMENT,  # 主键
        login         VARCHAR(255) NOT NULL,        # 登录账号
        user_pswd     CHAR(32) NOT NULL,            # 登录密码
        user_name     VARCHAR(255) NOT NULL,        # 昵称
        email         VARCHAR(255) NOT NULL,        # 邮箱
        create_time   CHAR(19),                     # 创建时间
        PRIMARY KEY (id)                            # 设置主键
   );
   ```

3. 新建mysql用户
   ```mysql
   CREATE USER 'Jacob' @'%' IDENTIFIED BY 'jacob12015229'; # 创建用户
   GRANT ALL PRIVILEGES ON project_crowd.* TO 'Jacob'@'%'; # 授权
   ```

## 2.3 基于Maven的MyBatis逆向工程
### 2.3.1 说明

### 2.3.2 配置
1. 逆向工程为 atcrowdfunding06-common-reverse模块
2. 配置Maven（pom.xml）
    ```xml
    <dependencies>
        <!--mybatis核心依赖-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.3</version>
        </dependency>
    </dependencies>

    <build>
        <!--构建过程中用到的插件-->
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.4.0</version>
                <configuration>
                    <configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <!--插件的依赖-->
                <dependencies>
                    <!--逆向工程核心依赖-->
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.4.0</version>
                    </dependency>
                    <!--数据库连接池-->
                    <dependency>
                        <groupId>com.mchange</groupId>
                        <artifactId>c3p0</artifactId>
                        <version>0.9.5.5</version>
                    </dependency>
                    <!--mysql驱动-->
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.21</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
    ```
3. 在Resources目录下新建文件 generatorConfig.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>

    <!DOCTYPE generatorConfiguration
            PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
            "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
    <generatorConfiguration>
        <!-- mybatis-generator:generate -->
        <context id="atguiguTables" targetRuntime="MyBatis3">
            <commentGenerator>
                <!--  是否去除自动生成的注释 true:是;false:否 -->
                <property name="suppressAllComments" value="true" />
            </commentGenerator>
            <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
            <jdbcConnection
                    driverClass="com.mysql.cj.jdbc.Driver"
                    connectionURL="jdbc:mysql://localhost:3306/project_crowd?serverTimezone=UTC"
                    userId="Jacob"
                    password="jacob12015229">
            </jdbcConnection>
            <!--  默认 false，把 JDBC DECIMAL  和 NUMERIC  类型解析为 Integer，为 true 时把
            JDBC DECIMAL
            和 NUMERIC  类型解析为 java.math.BigDecimal -->
            <javaTypeResolver>
                <property name="forceBigDecimals" value="false" />
            </javaTypeResolver>
            <!-- targetProject:生成 Entity 类的路径 -->
            <javaModelGenerator targetProject=".\src\main\java"
                                targetPackage="com.atguigu.crowd.entity">
                <!-- enableSubPackages:是否让 schema 作为包的后缀 -->
                <property name="enableSubPackages" value="false" />
                <!--  从数据库返回的值被清理前后的空格 -->
                <property name="trimStrings" value="true" />
            </javaModelGenerator>
            <!-- targetProject:XxxMapper.xml 映射文件生成的路径 -->
            <sqlMapGenerator targetProject=".\src\main\java"
                            targetPackage="com.atguigu.crowd.mapper">
                <!-- enableSubPackages:是否让 schema 作为包的后缀 -->
                <property name="enableSubPackages" value="false" />
            </sqlMapGenerator>
            <!-- targetPackage：Mapper 接口生成的位置 -->
            <javaClientGenerator type="XMLMAPPER"
                                targetProject=".\src\main\java"
                                targetPackage="com.atguigu.crowd.mapper">
                <!-- enableSubPackages:是否让 schema 作为包的后缀 -->
                <property name="enableSubPackages" value="false" />
            </javaClientGenerator>
            <!--  数据库表名字和我们的 entity 类对应的映射指定 -->
            <table tableName="t_admin" domainObjectName="Admin" />
        </context>
    </generatorConfiguration>
    ```


### 2.3.3 构建
1. 执行命令
   * 执行的Maven命令为 `mybatis-generator:generate -e`
   * IDEA可以直接图形化构建：
      * 右键pom.xml->Maven->Reload project
      * 点击 右侧Maven工具栏-> atcrowdfunding06-common-reverse -> Plugins -> mybatis-generator -> mybatis-generator:generate 
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-3-3.png)
   * 成功后IDEA会帮我们创建相应的类的mapper文件：
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-3-3_2.png)

2. 为Admin类添加构造器和toString方法
    ```java
    public Admin() {
    }

    public Admin(Integer id, String login, String userPswd, String userName, String email, String createTime) {
        this.id = id;
        this.login = login;
        this.userPswd = userPswd;
        this.userName = userName;
        this.email = email;
        this.createTime = createTime;
    }

    @Override
    public String toString() {
        return "Admin{" +
                "id=" + id +
                ", login='" + login + '\'' +
                ", userPswd='" + userPswd + '\'' +
                ", userName='" + userName + '\'' +
                ", email='" + email + '\'' +
                ", createTime='" + createTime + '\'' +
                '}';
    }
    ```
3. 资源归位
   * 在模块atcrowdfunding02-admin-webui的resources目录下新建 mybatis/mapper文件夹
   * 将刚刚生成的AdminMapper.xml移动到该文件夹下
   * 将Admin和AdminExample类移动到atcrowdfunding04-admin-entity模块下
   * 将AdminMapper类移动到atcrowdfunding03-admin-component模块下
3. 解决报错问题
   * AdminMapper会报错，找不到 `Param` 注解
   * 在atcrowdfunding03-admin-component模块的pom.xml文件中加入mybatis的依赖：
        ```xml
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.3</version>
        </dependency>
        ```
4. 项目结构(简略)：
   ```
    +--- atcrowdfunding01-admin-parent
    |   +--- atcrowdfunding02-admin-webui
    |   |   +--- pom.xml
    |   |   +--- src
    |   |   |   +--- main
    |   |   |   |   +--- java
    |   |   |   |   +--- resources
    |   |   |   |   |   +--- mybatis
    |   |   |   |   |   |   +--- mapper
    |   |   |   |   |   |   |   +--- AdminMapper.xml
    |   |   |   +--- test
    |   |   |   |   +--- java
    |   +--- atcrowdfunding03-admin-component
    |   |   +--- pom.xml
    |   |   +--- src
    |   |   |   +--- main
    |   |   |   |   +--- java
    |   |   |   |   |   +--- crowd
    |   |   |   |   |   |   +--- mapper
    |   |   |   |   |   |   |   +--- AdminMapper.java
    |   |   |   |   +--- resources
    |   |   |   +--- test
    |   |   |   |   +--- java
    |   +--- atcrowdfunding04-admin-entity
    |   |   +--- atcrowdfunding04-admin-entity.iml
    |   |   +--- pom.xml
    |   |   +--- src
    |   |   |   +--- main
    |   |   |   |   +--- java
    |   |   |   |   |   +--- crowd
    |   |   |   |   |   |   +--- entity
    |   |   |   |   |   |   |   +--- Admin.java
    |   |   |   |   |   |   |   +--- AdminExample.java
    |   |   |   |   +--- resources
    |   |   |   +--- test
    |   |   |   |   +--- java
    |   +--- pom.xml
    |   +--- src
    |   |   +--- main
    |   |   |   +--- java
    |   |   |   +--- resources
    |   |   +--- test
    |   |   |   +--- java
   ```

## 2.4 父工程依赖管理
1. 在atcrowdfunding01-admin-parent模块下的pom.xml中引入相应的依赖和版本声明
   * 在properties标签中定义统一的版本依赖
   * dependencyManagement标签
     * 常用于多模块环境下定义一个top module来专门管理公共依赖
     * 若dependencies里的dependency自己没有声明version，那么maven就会到depenManagement 里去找
     * 若 dependencies 中的 dependency 声明了version，则 dependencyManagement 中的声明无效
     * dependencyManagement 只是声明依赖的版本号，该依赖不会引入，因此子项目需要显示声明所需要引入的依赖，若不声明则不引入
2. 版本声明-pom.xml：
    ```xml
    <properties>
        <!-- 声明属性， 对 Spring 的版本进行统一管理 -->
        <!-- spring.version是别名，随便起，但要跟dependences标签中一致 -->
        <spring.version>5.2.11.RELEASE</spring.version>
        <!-- 声明属性， 对 SpringSecurity 的版本进行统一管理 -->
        <spring.security.version>5.4.2</spring.security.version>
    </properties>
    ```
3. 依赖-pom.xml：
    ```xml
    <dependencyManagement>
        <dependencies>
            <!-- Spring 依赖 -->
            <!-- https://mvnrepository.com/artifact/org.springframework/spring-orm -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-orm</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjweaver</artifactId>
                <version>1.9.6</version>
                <scope>runtime</scope>
            </dependency>
            <!-- https://mvnrepository.com/artifact/cglib/cglib -->
            <dependency>
                <groupId>cglib</groupId>
                <artifactId>cglib</artifactId>
                <version>3.3.0</version>
            </dependency>
            <!-- 数据库依赖 -->
            <!-- MySQL 驱动 -->
            <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.21</version>
            </dependency>
            <!-- 数据源 -->
            <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.2.3</version>
            </dependency>
            <!-- MyBatis -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>3.5.3</version>
            </dependency>
            <!-- MyBatis 与 Spring 整合 -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>2.0.5</version>
            </dependency>
            <!-- MyBatis 分页插件 -->
            <dependency>
                <groupId>com.github.pagehelper</groupId>
                <artifactId>pagehelper</artifactId>
                <version>5.2.0</version>
            </dependency>
            <!-- 日志 -->
            <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>2.0.0-alpha1</version>
            </dependency>
            <!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.3.0-alpha5</version>
                <scope>test</scope>
            </dependency>
            <!-- 其他日志框架的中间转换包 -->
            <!-- https://mvnrepository.com/artifact/org.slf4j/jcl-over-slf4j -->
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>jcl-over-slf4j</artifactId>
                <version>2.0.0-alpha1</version>
            </dependency>
            <!-- https://mvnrepository.com/artifact/org.slf4j/jul-to-slf4j -->
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>jul-to-slf4j</artifactId>
                <version>2.0.0-alpha1</version>
            </dependency>

            <!-- Spring 进行 JSON 数据转换依赖 -->
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-core</artifactId>
                <version>2.11.3</version>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.11.3</version>
            </dependency>
            <!-- JSTL 标签库 -->
            <!-- https://mvnrepository.com/artifact/javax.servlet.jsp.jstl/jstl-api -->
            <dependency>
                <groupId>javax.servlet.jsp.jstl</groupId>
                <artifactId>jstl-api</artifactId>
                <version>1.2</version>
            </dependency>
            <!-- junit 测试 -->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
                <scope>test</scope>
            </dependency>
            <!-- 引入 Servlet 容器中相关依赖 -->
            <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>javax.servlet-api</artifactId>
                <version>3.1.0</version>
                <scope>provided</scope>
            </dependency>
            <!-- JSP 页面使用的依赖 -->
            <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
            <dependency>
                <groupId>javax.servlet.jsp</groupId>
                <artifactId>javax.servlet.jsp-api</artifactId>
                <version>2.3.3</version>
                <scope>provided</scope>
            </dependency>
            <!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
            <dependency>
                <groupId>com.google.code.gson</groupId>
                <artifactId>gson</artifactId>
                <version>2.8.6</version>
            </dependency>
            <!-- SpringSecurity 对 Web 应用进行权限管理 -->
            <!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-web -->
            <dependency>
                <groupId>org.springframework.security</groupId>
                <artifactId>spring-security-web</artifactId>
                <version>5.4.2</version>
            </dependency>
            <!-- SpringSecurity 配置 -->
            <!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-config -->
            <dependency>
                <groupId>org.springframework.security</groupId>
                <artifactId>spring-security-config</artifactId>
                <version>5.4.2</version>
            </dependency>
            <!-- SpringSecurity 标签库 -->
            <!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-taglibs -->
            <dependency>
                <groupId>org.springframework.security</groupId>
                <artifactId>spring-security-taglibs</artifactId>
                <version>5.4.2</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

## 2.5 Spring 整合 MyBatis
### 2.5.1 整合思路
![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-5-1.png)

### 2.5.2 具体配置
### 2.5.2 具体配置一
1. 子工程中加入依赖
   * 子工程选择component：webui依赖component，因此可以使用component的依赖（依赖传递）
   * junit和spring-test依赖生命周期设置为test，不能传递，因此各项目使用时要在自己的pom.xml文件总引入
   * component模块的pom.xml:
        ```xml
        <dependencies>
            <dependency>
                <groupId>com.atguigu.crowd</groupId>
                <artifactId>atcrowdfunding04-admin-entity</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>com.atguigu.crowd</groupId>
                <artifactId>atcrowdfunding05-common-util</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
            <!--Junit测试-->
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter-api</artifactId>
                <scope>test</scope>
            </dependency>
            <!-- Spring 依赖 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-orm</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <scope>test</scope>
            </dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjweaver</artifactId>
            </dependency>
            <dependency>
                <groupId>cglib</groupId>
                <artifactId>cglib</artifactId>
            </dependency>
            <!-- 数据库依赖 -->
            <!-- MySQL 驱动 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>
            <!-- 数据源 -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
            </dependency>
            <!-- MyBatis -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
            </dependency>
            <!-- MyBatis 与 Spring 整合 -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
            </dependency>
            <!-- MyBatis 分页插件 -->
            <dependency>
                <groupId>com.github.pagehelper</groupId>
                <artifactId>pagehelper</artifactId>
            </dependency>
            <!-- Spring 进行 JSON 数据转换依赖 -->
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-core</artifactId>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
            </dependency>
            <!-- JSTL 标签库 -->
            <dependency>
                <groupId>javax.servlet.jsp.jstl</groupId>
                <artifactId>jstl-api</artifactId>
            </dependency>
            <dependency>
                <groupId>com.google.code.gson</groupId>
                <artifactId>gson</artifactId>
            </dependency>
        </dependencies>
        ```
2. 配置数据库连接池
   * mysql8.0以后的数据库连接池，必须在url中指定时区
   * 在webui模块下的resources目录下新建 jdbc.perperties文件
        ```
        jdbc.url=jdbc:mysql://localhost:3306/project_crowd?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true
        jdbc.username=Jacob
        jdbc.password=jacob12015229
        jdbc.driver=com.mysql.cj.jdbc.Driver
        ```
3. mybatis配置文件
   * 在webui模块下的resources/mybatis目录下新建mybatis-config.xml文件
   * mybatis-config.xml
        ```xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>

        </configuration>
        ```

4. spring-mybatis整合配置文件
   * 在webui模块下的resources目录下新建 spring-persist-mybatis.xml文件
   * 在文件中配置数据库连接池：
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

            <!--加载外部属性文件-->
            <context:property-placeholder location="classpath:jdbc.properties"/>
            <!--配置数据源-->
            <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close" init-method="init">
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="driverClassName" value="${jdbc.driver}"/>
            </bean>
        </beans>
        ```
5. 测试是否配置成功
   * 这里使用Spring5和Junit5整合的方式，需要在webui的pom.xml文件中加入junit和spring-test依赖：
        ```xml
        <dependencies>
            <dependency>
                <groupId>com.atguigu.crowd</groupId>
                <artifactId>atcrowdfunding03-admin-component</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
            <!-- junit 测试 -->
            <dependency>
                <groupId>org.junit.jupiter</groupId>
                <artifactId>junit-jupiter-api</artifactId>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
        ```
   * 在webui模块的test/java目录下新建包com.atguigu.crowd
   * 在新建的包中新建测试类CrowdTest：
        ```java
        package com.atguigu.crowd;

        import org.junit.jupiter.api.Test;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.test.context.ContextConfiguration;
        import org.springframework.test.context.junit.jupiter.SpringJUnitConfig;

        import javax.sql.DataSource;
        import java.sql.Connection;
        import java.sql.SQLException;

        // 这里用junit5进行测试
        @SpringJUnitConfig(locations = "classpath:spring-persist-mybatis.xml")
        public class CrowdTest {
            @Autowired
            private DataSource dataSource;

            @Test
            public void  testConnection() throws SQLException{
                Connection connection = dataSource.getConnection();
                System.out.println(connection);
            }
        }
        ```
   * 运行测试方法，能输出类似以下内容说明成功：
   * 如果出现错误 “Java：不支持发行版本5”，则需要配置一下版本（maven默认的jdk为1.5，太旧了所以出错）
     * 首先在项目结构中确保Project的module使用的是相同版本的jdk（我用的1.8）
     * 第二步：在parent模块中的pom.xml文件里通过properties标签里设置默认java：
        ```xml
        <properties>
            <!-- 声明属性， 对 Spring 的版本进行统一管理 -->
            <spring.version>5.2.11.RELEASE</spring.version>
            <!-- 声明属性， 对 SpringSecurity 的版本进行统一管理 -->
            <spring.security.version>5.4.2</spring.security.version>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
        </properties>
        ```

### 2.5.3 具体配置二
1. 配置 SqlSessionFactoryBean（还是spring-persist-mybatis.xml文件）

    ```xml
    <!--配置SqlSessionFactoryBean整合MyBatis-->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--指定mybatis全局配置文件位置-->
        <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
        <!--指定mapper配置文件位置-->
        <property name="mapperLocations" value="classpath:mybatis/mapper/*Mapper.xml"/>
        <!--装配数据源：引用前面的dataSource数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    ```
    这里尚硅谷的视频里有错误，mybatis-config.xml被老师拿到resources目录下了，我没有动，所以不太一样

2. 配置mybatis扫描器

    ```xml
    <!--配置mybatis的扫描器，扫描mapper接口所在的包，创建dao对象-->
    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.atguigu.crowd.mapper"/>
    </bean>
    ```

3. 测试（CrowdTest）
    ```java
    @Autowired
    private AdminMapper adminMapper;

    @Test
    public void testInsertAdmin(){
        Admin admin;
        admin = new Admin(null,"Rachel","123123","rui","rui@qq.com",null);
        int count = adminMapper.insert(admin);
        System.out.println(count);
    }
    ```
    成功后会在数据库表中插入一条信息

## 2.6 日志系统
### 2.6.1 spring日志框架

### 2.6.2 具体操作
在很多情况下项目中会自带日志框架，比如Tomcat中自带了Commons Logging，spring4为原生jcl:commons-logging.jar
一些开源组件可能直接采用了jul 进行日志输出，为保证日志的统一配置管理，需将其迁移到slf4j 日志框架上
1. 原生（jcl+jul）
   * spring5重写了jcl，默认使用jul实现
   * 运行之前的 `testInsertAdmin`方法，控制台输出

        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-6-2_1.png)

2. 使用slf4j
   * 只加入slf4j的依赖，slf4j只是接口，因此此时没有具体实现，也不打印日志
   * 运行之前的 `testInsertAdmin`方法，控制台输出
        
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-6-2_2.png)

3. 使用slf4j+logback
   * slf4j和logback-classic的依赖
        ```xml
        <!-- 日志 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </dependency>
        ```

   * 同样运行之前的测试方法：
        * spring4
        ![spring4 slf4j](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-6-2_5.png)
        * spring5
        ![spring5 slf4j](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-6-2_4.png)

4. 更换框架的日志系统
   * 尚硅谷的课程中使用的spring4，需要更换框架系统将jcl桥连到slf4j，实现所有日志均由slf4j输出
   * 本项目用的是spring5，操作不太一样:
   * **注意**：spring4和spring5是不一样的，spring5重写了jcl：spring-jcl，即spring5实际上自己实现ljcl，spring-jcl不再像原生jcl那样只是一个接口（门面了），不过spring-jcl内部也是使用其它技术来记录日志的，他有一个getClassLoader()方法来选择：
     * LOG4J：其实是log4j2，如果有log4j2的依赖则直接使用log4j2来打印日志
     * SLF4J_LAL：slf4j（SPI），自动桥连到slf4j上，但slf4j也只是一个“门面”，需要其它实现：
       * 无实现：不打印
       * logback：最佳，无需中间转换组件，只需两者依赖，也是目前流行做法
       * log4j：需要额外的依赖 `slf4j-log4j12`
       * jul：需要额外的依赖：`slf4j-jdk14`
       * 其它：见slf4j官网图片
       * 此时可以通过各种依赖将框架里其它组件的日志桥连到slf4j上：`jul-to-slf4j`、`log4j-over-slf4j`
     * SLF4J（slf4j API）
     * jul：如果都没有就是用默认的jul
   * spring4中原生jcl日志工具优先级：
     * 当前factory中名叫 `org.apache.commons.logging.Log` 配置属性的值
     * 寻找系统中属性中名叫 `org.apache.commons.logging.Log` 的值
     * 如果应用程序的classpath中有log4j,则使用相关的包装(wrapper)类(Log4JLogger)
     * 如果应用程序运行在jdk1.4的系统中，使用相关的包装类(Jdk14Logger)
     * 使用简易日志包装类(SimpleLog)

5. logback 配置文件
   * logback可以通过配置文件logback.xml实现对日志配置输出。
   * logback.xml一般放在/src/main/resource/文件夹下
   * logback.xml内容：
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <configuration debug="true">
            <!-- 指定日志输出的位置 -->
            <appender name="STDOUT"
                    class="ch.qos.logback.core.ConsoleAppender">
                <encoder>
                    <!-- 日志输出的格式 -->
                    <!-- 按照顺序分别是： 时间、 日志级别、 线程名称、 打印日志的类、 日志主体
        内容、 换行 -->
                    <pattern>[%d{HH:mm:ss.SSS}] [%-5level] [%thread] [%logger]
                        [%msg]%n
                    </pattern>
                </encoder>
            </appender>
            <!-- 设置全局日志级别。 日志级别按顺序分别是： DEBUG、 INFO、 WARN、 ERROR -->
            <!-- 指定任何一个日志级别都只打印当前级别和后面级别的日志。 -->
            <root level="INFO">
                <!-- 指定打印日志的 appender， 这里通过“STDOUT”引用了前面配置的 appender -->
                <appender-ref ref="STDOUT"/>
            </root>
            <!-- 根据特殊需求指定局部日志级别 -->
            <logger name="com.atguigu.crowd.mapper" level="DEBUG"/>
        </configuration>
        ```
   * 执行测试方法查看输出效果：

        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-6-2_6.png)


## 2.7 声明式事务
### 2.7.1 说明
1. 这里使用基于xml配置文件的方式配置事务管理



### 2.7.2 操作
1. 加入依赖（webui模块）
    ```xml
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
    </dependency>
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
    </dependency>
    ```
2. 创建spring配置文件 spring-persist-tx.xml（webui模块resources目录下）
    ```xml
    <!--1. 配置自动扫描的包：主要是为了把Service扫描到IOC容器中-->
    <context:component-scan base-package="com.atguigu.crowd.service"/>

    <!--2. 配置事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--装配数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--3. 配置AOP-->
    <aop:config>
        <!--考虑到后面整合SpringSecurity，避免把UserDetailService加入事务控制。让切入点表达式定位到ServiceImpl而不是Service-->
        <aop:pointcut id="txPointcut" expression="execution(* *..*ServiceImpl.*(..))"/>
        <!--将切入点表达式跟事务通知关联起来-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>

    <!--4. 配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!--配置事务属性-->
        <tx:attributes>
            <!--查询方法：配置只读属性，让数据库知道这是一个查询操作，能够进行一定优化-->
            <!--service中一般查询方法一般以get、find等开头-->
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="query*" read-only="true"/>
            <tx:method name="count*" read-only="true"/>

            <!--增删改-->
            <tx:method name="save*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
            <tx:method name="update*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
            <tx:method name="remove*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
            <tx:method name="batch*" propagation="REQUIRES_NEW" rollback-for="java.lang.Exception"/>
        </tx:attributes>
    </tx:advice>
    ```

3. 测试
   * 新建service接口及实现方法（component模块下）
        ```java
        // com/atguigu/crowd/service/api/AdminService.java

        public interface AdminService {

            void saveAdmin(Admin admin);
        }
        ```
        ```java
        // com/atguigu/crowd/service/impl/AdminServiceImpl.java

        @Service
        public class AdminServiceImpl implements AdminService {

            @Autowired
            private AdminMapper adminMapper;

            @Override
            public void saveAdmin(Admin admin) {
                adminMapper.insert(admin);
                // throw new RuntimeException();
            }
        }
        ```
   * 测试类
        ```java
        @SpringJUnitConfig(locations = {"classpath:spring-persist-tx.xml", "classpath:spring-persist-mybatis.xml"})
        public class CrowdTest {

            @Autowired
            private AdminService adminService;

            @Test
            public void testTx(){
                Admin admin = new Admin(null, "mei Lee", "123456", "mei", "mei@qq.com", new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
                adminService.saveAdmin(admin);
            }
        ```
4. 注意事项
   * 在基于xml的声明式事务中，配置文件中事务属性的tx:method是必须配置的，如果没有配置则事务对这个方法不生效

## 2.8 表述层工作机制
### 2.8.1 表述层工作机制
1. 项目启动过程
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-8-1_1.png)
2. 访问过程
   
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-8-1_2.png)

### 2.8.2 环境搭建
1. web工程
   * 在webui模块中添加web框架（右键 -> Add Framework Support）
   * 将web目录重命名为webapp，移动到main目录下
2. 修改web.xml文件
   * ContextLoaderListener
        ```xml
        <!--ContextLoaderListener-->
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-persist-*.xml</param-value>
        </context-param>
        <!-- Bootstraps the root web application context before servlet initialization -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        ```

   * CharacterEncodingFilter
        ```xml
        <filter>
            <filter-name>characterEncodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <init-param>
                <param-name>encoding</param-name>
                <param-value>UTF-8</param-value>
            </init-param>
            <init-param>
                <param-name>forceRequestEncoding</param-name>
                <param-value>true</param-value>
            </init-param>
            <init-param>
                <param-name>forceResponseEncoding</param-name>
                <param-value>true</param-value>
            </init-param>
        </filter>
        <!--这个Filter执行顺序要在所有其它Filter前面-->
        <filter-mapping>
            <filter-name>characterEncodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
        ```

   * HiddenHttpMethodFilter  
    遵循 RESTFUL 风格将 POST 请求转换为 PUT 请求、 DELETE 请求时使用。省略不配

   * DispatcherServlet
        ```xml
        <!-- 配置 SpringMVC 的前端控制器 -->
        <!-- The front controller of this Spring Web application, responsible for handling all application
        requests -->
        <servlet>
            <servlet-name>springDispatcherServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:spring-web-mvc.xml</param-value>
            </init-param>
            <!-- 让 DispatcherServlet 在 Web 应用启动时创建对象、 初始化 -->
            <!-- 默认情况： Servlet 在第一次请求的时候创建对象、 初始化 -->
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <!-- DispatcherServlet 映射的 URL 地址 -->
            <servlet-name>springDispatcherServlet</servlet-name>
            <url-pattern>/</url-pattern><!--拦截所有请求-->
            <!--方式二
            <url-pattern>*.html</url-pattern>
            <url-pattern>*.json</url-pattern>
                优点：伪静态效果，给黑客入侵增加难度、有利于SEO优化
                缺点：不符合RESTFUL风格、请求扩展名不匹配时会406错误
            -->
        </servlet-mapping>
        ```

3. springmvc配置
   * webui模块下的resources目录下新建spring-web-mvc.xml
        ```xml
        <!--自动扫描-->
        <context:component-scan base-package="com.atguigu.crowd.mvc"/>

        <!--配置springmvc注解驱动-->
        <mvc:annotation-driven/>

        <!--配置视图解析器-->
        <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/"/>
            <property name="suffix" value=".jsp"/>
        </bean>
        ```
   * component模块的crowd包下新建mvc包，mvc下新建config、handler、interceptor三个包

### 2.8.3 base标签
1. 在jsp页面中使用base标签，从而简化连接，避免回退异常
2. 在index.jsp的head部分加入base标签：
    ```html
    <head>
    <title>测试页面</title>
        <base href="http://${pageContext.request.serverName}:${pageContext.request.serverPort}${pageContext.request.contextPath}/">
    </head>
    ```
3. 说明
   * base 标签必须在所有“带具体路径” 的标签的前面
   * serverName 部分 EL 表达式和 serverPort 部分 EL 表达式之间必须写“:”
   * **serverPort 部分 EL 表达式和 contextPath 部分 EL 表达式之间绝对不能写“/”**
     * contextPath 部分 EL 表达式本身就是“/” 开头
     * 如果多写一个“/” 会干扰 Cookie 的工作机制
   * serverPort 部分 EL 表达式后面必须写“/


### 2.8.4 测试
1. webui模块中加入依赖
    ```xml
    <!-- 引入 Servlet 容器中相关依赖 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- JSP 页面使用的依赖 -->
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>javax.servlet.jsp-api</artifactId>
        <scope>provided</scope>
    </dependency>
    ```
   * 注意范围都是provided，因为这两个包tomcat服务器已经提供了，打包的时候不用打包进去

2. component模块handler包下新建TestHandler类
    ```java
    @Controller
    public class TestHandler {

        @Autowired
        private AdminService adminService;

        @RequestMapping("/test/ssm.html")
        public String testSSM(ModelMap modelMap){

            List<Admin> adminList = adminService.getAll();
            modelMap.addAttribute("adminList",adminList);

            return "target";
        }
    }
    ```
3. webui模块WEB-INF目录下新建target.jsp
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>AdminList</title>
    </head>
    <body>

        <h1>Success</h1>
        ${requestScope.adminList}

    </body>
    </html>
    ```
4. service：
   * AdminService接口
        ```java
        List<Admin> getAll();
        ```

   * AdminServiceImpl
        ```java
        @Override
        public List<Admin> getAll() {
            return adminMapper.selectByExample(new AdminExample());
        }
        ``

5. 启动
   * 配置Tomcat服务器
   * 启动，浏览器自动访问index页面

## 2.9 SpringMVC下的Ajax请求
### 2.9.1 说明
1. 普通请求与Ajax请求
   * 普通请求： 后端处理完成后返回页面， 浏览器使用使用页面替换整个窗口中的内容
   * Ajax 请求： 后端处理完成后通常返回 JSON 数据， jQuery 代码使用 JSON 数据对页面局部更新
2. 流程  
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-9-1_1.png)
3. 环境
   * `@ResponseBody`和`@RequestBody`注解需要jackon依赖：
        ```xml
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.11.3</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.11.3</version>
        </dependency>
        ```
   * jQuery：在webui模块的webapp目录下新建js目录，将jQuery放到该目录下
   * index页面中添加jQuery（head部分）
        ```html
        <script type="text/javascript" src="jQuery/jquery-3.5.1.js"></script>
        ```
   * 如果springmvc的配置文件中url-pattern使用/则服务器会找不到js，需要配置：
        ```xml
        <!--配置springmvc注解驱动-->
        <mvc:annotation-driven/>
        <mvc:resources mapping="/js/**" location="/js/"/>
        ```

### 2.9.2 Ajax发送数组 — 方式一
1. index.jsp
    ```html
    <html>
    <head>
        <title>测试页面</title>
        <script type="text/javascript" src="js/jquery-3.5.1.js"></script>
        <base href="http://${pageContext.request.serverName}:${pageContext.request.serverPort}${pageContext.request.contextPath}/">
        <script type="text/javascript">
        $(function (){
            $("#btn01").click(function (){
                $.ajax({
                    "url": "send/array1.do",
                    "type": "post",
                    "data": {
                    "array":[5,8,12]
                    },
                    "dataType": "text",
                    "success": function (response){
                    alert(response);
                    },
                    "error": function (response){
                    alert(response);
                    }
                });
            });
        })
        </script>
    </head>
    <body>
        <button id="btn01">Send text</button>
    </body>
    </html>
    ```

2. TestHandler
    ```java
    @ResponseBody
    @RequestMapping("/send/array1.do")
    public String testReceiveArrayOne(@RequestParam("array[]") List<Integer> array){
        // 接收参数时需要在参数名后面加[]
        for (Integer number : array){
            System.out.println(number);
        }
        return "success";
    }
    ```

### 2.9.3  Ajax发送数组 — 方式二
1. index.jsp
    ```html
    <script type="text/javascript">
      $(function (){
        $("#btn01").click(function (){
          // 准备好要发送到服务端的数组
          var array = [5,8,12];

          // 将JSON数组转换为JSON字符串
          var requestBody = JSON.stringify(array);
          $.ajax({
            "url": "send/array2.do",
            "type": "post",
            "data": requestBody,
            contentType: "application/json;character=UTF-8",
            "dataType": "text",
            "success": function (response){
              alert(response);
            },
            "error": function (response){
              alert(response);
            }
          });
        });
      })
    </script>
    ```

2. TestHandler
    ```java
    private final Logger logger = LoggerFactory.getLogger(TestHandler.class);

    @ResponseBody
    @RequestMapping("/send/array2.do")
    public String testReceiveArrayTwo(@RequestBody List<Integer> array){

        for (Integer number : array){
            logger.info("number="+number); //注意是 org.slf4j.Logger，不是jul中的Logger
        }
        return "success";
    }
    ```

### 2.9.4 返回复杂对象
1. 创建相关类（entity模块）
   * Address
        ```java
        public class Address {

            private String province;
            private String city;
            private String street;

            public Address() {
            }
            public Address(String province, String city, String street) {
                this.province = province;
                this.city = city;
                this.street = street;
            }
            public String getProvince() { return province; }

            public String getCity() { return city; }

            public String getStreet() { return street; }

            public void setProvince(String province) { this.province = province; }

            public void setCity(String city) { this.city = city; }

            public void setStreet(String street) { this.street = street; }

            @Override
            public String toString() {
                return "Address{" +
                        "province='" + province + '\'' +
                        ", city='" + city + '\'' +
                        ", street='" + street + '\'' +
                        '}';
            }
        }
        ```
   * Subject
        ```java
        public class Subject {
            private String subName;
            private Integer subScore;

            // 构造器、getter、setter、toString
        }
        ```
   * Student
        ```java
        public class Student {

            private Integer stuId;
            private String stuName;
            private Address address;
            private List<Subject> subjectList;
            private Map<String,String> map;

            // 构造器、getter、setter、toString
        }
        ```
2. TestHandler
    ```java
    @ResponseBody
    @RequestMapping("/send/compose/object.do")
    public String testReceiveComplicatedObject(@RequestBody Student student){
        logger.info(student.toString());
        return "success";
    }
    ```

3. index
    ```js
    $("#btn03").click(function (){
        // 准备要发送的数据
        var student = {
        stuId: 5,
        stuNmae: "tom",
        address: {
            province: "江苏",
            city: "南京",
            street: "秣陵街道"
        },
        subjectList: [
            {
            subName: "java",
            subScore: 100
            },
            {
            subName: "c++",
            subScore: 98
            }
        ],
        map:{
            key1: "value1",
            key2: "value2"
        }
        };
        // 将JSON对象转换为JSON字符串
        var requestBody = JSON.stringify(student);

        // 发送Ajax请求
        $.ajax({
        url: "send/compose/object.do",
        type: "post",
        data: requestBody,
        contentType: "application/json;character=UTF-8",
        dataType: "text",
        success: function (resp){
            alert(resp);
        },
        error: function (resp) {
            alert(resp)
        }
        })
    })
    ```
    ```html
    <!-- body中添加按钮 -->
    <button id="btn03">Send Object</button>
    ```

### 2.9.5 规范Ajax返回值
1. util模块下新建类（atcrowdfunding05-common-util/src/main/java/com/atguigu/crowd/util/ResultEntity.java）
    ```java
    public class ResultEntity<T> {

        public static final String SUCCESS = "SUCCESS";
        public static final String FAILED = "FAILED";
        // 用来封装当前请求处理的结果是成功还是失败
        private String result;

        // 请求处理失败时返回的错误消息
        private String message;

        // 要返回的数据
        private T data;

        /*
        * 请求处理成功且不需要返回数据
        * @param message
        * @return
        */
        public static <Type> ResultEntity<Type> successWithoutData(){
            return new ResultEntity<Type>(SUCCESS, null, null);
        }

        /*
        * 请求处理成功且需要返回数据
        * @param data
        * @return
        */
        public static <Type> ResultEntity<Type> successWithData(Type data){
            return new ResultEntity<Type>(SUCCESS, null, data);
        }

        /*
        * 请求处理失败后使用的工具方法
        * @param message
        * @return
        */
        public static <Type> ResultEntity<Type> failed(String message) {
            return new ResultEntity<Type>(FAILED, message, null);
        }   
        // 构造器、getter、setter、toString
    ```

2. 修改TestHandler
    ```java
    @ResponseBody
    @RequestMapping("/send/compose/object.do")
    public ResultEntity<Student> testReceiveComplicatedObject(@RequestBody Student student){
        logger.info(student.toString());
        return ResultEntity.successWithData(student);
    }
    ```

3. 修改index
    ```js
    $.ajax({
        url: "send/compose/object.do",
        type: "post",
        data: requestBody,
        contentType: "application/json;character=UTF-8",
        dataType: "json",
        success: function (resp){
            console.log(resp)
        },
        error: function (resp) {
            console.log(resp)
        }
    })
    ```

## 2.10 异常映射
### 2.10.1 说明
1. 两种异常映射
   * SpringMVC可以通过配置mvc:view-controller直接解析到视图页面，对于这种情况下出现的异常要使用基于xml的异常映射机制
        ```xml
        <mvc:view-controller path="/login.html" view-name="login" />
        ```
   * 对于经过Controller的异常则需要通过基于注解的异常映射
   * 为了处理所有异常，我们需要把两种机制都配好
2. 异常映射工作机制
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/2-10-1.png)

### 2.10.2 基于xml的异常映射
1.  在springmvc的配置文件中进行配置：
    ```xml
    <!--配置基于xml的异常映射-->
    <bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <!-- 配置异常类型和具体视图页面对应关系 -->
        <property name="exceptionMappings">
            <props>
                <!-- key属性指定异常全类名，标签体中指定对应的视图(前后缀拼接) -->
                <prop key="java.lang.Exception">system-error</prop>
            </props>
        </property>
    </bean>
    ```

2.  创建错误视图：在WEB-INF目录下新建system-error.jsp
    ```html
    <head>
        <title>出错了....</title>
    </head>
    <body>

        <h1>出错了！</h1>
        <!-- 从请求域取出Exception对象，再进一步访问message属性就能显示错误消息 -->
        ${ requestScope.exception.message }
    </body>
    ```

3.  测试
    ```java
    @RequestMapping("/test/ssm.html")
    public String testSSM(ModelMap modelMap){

        List<Admin> adminList = adminService.getAll();
        modelMap.addAttribute("adminList",adminList);
        System.out.println(10/0);

        return "target";
    }
    ```
    * 此时会接管所有异常（包括注解的）

### 2.10.3 基于注解的异常映射
1. util模块加入servlet相关依赖
    ```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    ```

2. 创建工具类CrowdUtil（util模块）
    ```java
    public class CrowdUtil {

        /*
        * 判断当前请求是否为Ajax请求
        * @param request 请求对象
        * @return
        *      true：当前请求为Ajax请求
        *      false：当前请求不是Ajax请求
        */

        public static boolean judgeRequestType(HttpServletRequest request) {

            // 1. 获取请求头
            String acceptHeader = request.getHeader("Accept");
            String xRequestHeader = request.getHeader("X-Request-With");

            // 2. 判断
            return (acceptHeader != null && acceptHeader.contains("application/json"))
                    ||
                    (xRequestHeader != null && xRequestHeader.equals("XMLHttpRequest"));
        }

    }
    ```

3. 测试
    ```java
    @ResponseBody
    @RequestMapping("/send/compose/object.do")
    public ResultEntity<Student> testReceiveComplicatedObject(@RequestBody Student student, HttpServletRequest request){

        boolean judgeResult = CrowdUtil.judgeRequestType(request);
        logger.info("judgeResult = "+judgeResult);

        logger.info(student.toString());
        return ResultEntity.successWithData(student);
    }

    @RequestMapping("/test/ssm.html")
    public String testSSM(ModelMap modelMap, HttpServletRequest request){

        boolean judgeResult = CrowdUtil.judgeRequestType(request);
         logger.info("judgeResult = "+judgeResult);

        List<Admin> adminList = adminService.getAll();
        modelMap.addAttribute("adminList",adminList);
        return "target";
    }
    ```

   * 启动服务器，点击第一个链接，控制台打印 `judgeResult = false`
   * 点击第二个链接，控制台打印 `judgeResult = true`

4. 创建异常处理类(component模块config包下)
    ```java
    @ControllerAdvice
    public class CrowdExceptionResolver {

        // 空指针异常
        @ExceptionHandler(value = NullPointerException.class)
        public ModelAndView resolveNullPointerException(NullPointerException exception, HttpServletRequest request, HttpServletResponse response) throws IOException {

            String viewName =  "system-error";
            return commonResolve(viewName,exception,request,response);
        }

        // 创建通用方法
        private ModelAndView commonResolve(String viewName, Exception exception, HttpServletRequest request, HttpServletResponse response ) throws IOException {
            // 1. 判断当前请求类型
            boolean judgeResult = CrowdUtil.judgeRequestType(request);
            // 2. 如果为Ajax请求
            if (judgeResult) {
                // 3. 创建 ResultEntity 对象
                ResultEntity<Object> resultEntity = ResultEntity.failed(exception.getMessage());
                // 4. 创建Gson对象
                Gson gson = new Gson();
                // 5. 将ResultEntity对象转换为JSON字符串
                String json = gson.toJson(resultEntity);
                // 6. 将JSON字符作为响应体返回给浏览器
                response.getWriter().write(json);
                // 7. 上面已经通过原生response对象返回了响应，因此不再提供ModelAndView对象
                return null;
            }
            // 8. 如果不是Ajax请求，则创建ModelAndView对象
            ModelAndView modelAndView = new ModelAndView();
            // 9. 将Exception对象存入模型
            modelAndView.addObject(CrowdConstant.ATTR_NAME_EXCEPTION, exception);
            // 10. 设置对应的视图名称
            modelAndView.setViewName(viewName);
            return modelAndView;
        }
    }
    ```

5. 声明一个类，用于管理常量（util模块下）
    ```java
    package com.atguigu.crowd.constant;

    public class CrowdConstant {

        public static final String ATTR_NAME_EXCEPTION = "exception";
        public static final String MESSAGE_LOGIN_FAILED = "抱歉！，账号密码错误，请重新输入！";
        public static final String MESSAGE_LOGIN_ACCT_ALREADY_IN_USE = "账号已被占用！";
        public static final String MESSAGE_ACCESS_FORBIDDEN = "请登录后访问！";
    }
    ```

## 2.11 前端页面
### 2.11.1 静态资源
将静态资源放到webui模块下的webapp目录下

### 2.11.2 创建后台管理员登录页面 
1. 创建admin-login.jsp文件
    ```html
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="keys" content="">
        <meta name="author" content="">
        <base href="http://${pageContext.request.serverName}:${pageContext.request.serverPort}${pageContext.request.contextPath}/">
        <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="css/font-awesome.min.css">
        <link rel="stylesheet" href="css/login.css">
        <script src="jquery/jquery-2.1.1.min.js"></script>
        <script src="bootstrap/js/bootstrap.min.js"></script>
        <style>

        </style>
    </head>
    <body>
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
        <div class="container">
            <div class="navbar-header">
                <div><a class="navbar-brand" href="index.html" style="font-size:32px;">尚筹网-创意产品众筹平台</a></div>
            </div>
        </div>
    </nav>

    <div class="container">

        <form action="admin/to/login" method="post" class="form-signin" role="form">
            <h2 class="form-signin-heading"><i class="glyphicon glyphicon-log-in"></i>管理员登录</h2>
            <div class="form-group has-success has-feedback">
                <input type="text" class="form-control" id="inputSuccess4" placeholder="请输入登录账号" autofocus>
                <span class="glyphicon glyphicon-user form-control-feedback"></span>
            </div>
            <div class="form-group has-success has-feedback">
                <input type="text" class="form-control" id="inputSuccess5" placeholder="请输入登录密码" style="margin-top:10px;">
                <span class="glyphicon glyphicon-lock form-control-feedback"></span>
            </div>
            <button type="submit" class="btn btn-lg btn-success btn-block">登录</button>
        </form>
    </div>
    </body>
    ```
2. mvc配置文件
   * 配置视图
        ```xml
        <!--配置view-controller，直接把请求地址和视图名称关联起来，从而无需写handler方法-->
        <mvc:view-controller path="/admin/to/login.do" view-name="admin-login"/>
        ```

   * 如果url-pattern为 `/` 则需要配置
   * 使用 `<mvc:default-servlet-handler/>`或 `mvc:resources`标签
        ```xml
        <mvc:default-servlet-handler/>
        ```
3. 启动服务器，访问 `http://localhost:8080/atcrowdfunding02_admin_webui_war/admin/to/login.do`，如果为黑白页面则重启IDEA

### 2.11.3 layer
1. layer是一个 Web 弹层组件
2. 引入：
   * 将layer文件夹复制到webapp目录下
   * 在index头部加入（jquery之后）
        ```html
        <script type="text/javascript" src="layer/layer.js"></script>
        ```
3. 使用
   * index.jsp head
        ```js
        $("#btn04").click(function (){
          layer.msg("layer的弹框");
        });
        ```

   * index.jsp body
        ```html
        <button id="btn04">layer</button>
        ```

### 2.11.4 修饰system-error页面
1. 根据admin-login页面对system-error页面进行修改
   
    ```html
    <!DOCTYPE html>
    <html lang="zh-CN">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="keys" content="">
        <meta name="author" content="">
        <base href="http://${pageContext.request.serverName}:${pageContext.request.serverPort}${pageContext.request.contextPath}/">
        <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="css/font-awesome.min.css">
        <link rel="stylesheet" href="css/login.css">
        <script src="jquery/jquery-2.1.1.min.js"></script>
        <script src="bootstrap/js/bootstrap.min.js"></script>
        <script type="text/javascript">
            $(function (){
                $("button").click(function (){
                    // 相当于浏览器的后退按钮
                    window.history.back();
                });
            });
        </script>
        <style>

        </style>
    </head>
    <body>
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
        <div class="container">
            <div class="navbar-header">
                <div><a class="navbar-brand" href="index.html" style="font-size:32px;">尚筹网-创意产品众筹平台</a></div>
            </div>
        </div>
    </nav>

    <div class="container">
        <h2 class="form-signin-heading" style="text-align: center;"><i class="glyphicon glyphicon-log-in"></i>尚筹网系统消息</h2>
        <!--
            requestScope对应的是存放request域数据的Map
            requestScope.exception相当于request.getAttribute("exception")
            requestScope.exception.message相当于exception.getMessage()
        -->
        <h3 style="text-align: center;"> ${ requestScope.exception.message } </h3>
        <button style="width: 150px;margin: 50px auto 0px auto;" type="submit" class="btn btn-lg btn-success btn-block">返回上一步</button>

    </div>
    </body>
    </html>
    ```










