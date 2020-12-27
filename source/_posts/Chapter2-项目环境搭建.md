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
date: 2020-12-28 02:39:48
updated: 2020-12-28 02:39:48
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
        <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
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
                    <!-- https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core -->
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.4.0</version>
                    </dependency>
                    <!--数据库连接池-->
                    <!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
                    <dependency>
                        <groupId>com.mchange</groupId>
                        <artifactId>c3p0</artifactId>
                        <version>0.9.5.5</version>
                    </dependency>
                    <!--mysql驱动-->
                    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
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
### 2.5.1 

### 2.5.2 具体配置





