---
title: Chapter5 MyBatis配置文件
excerpt: mybatis主配置文件的设置，包括数据源dataSource、事务、别名等，翻页工具PageHelper
tags:
  - java
  - mybatis
categories:
  - Java笔记
  - MyBatis
banner_img: /img/dog.png
index_img: /img/post/mybatis_logo.jpg
abbrlink: 4d3f287a
date: 2020-12-18 17:27:08
updated: 2020-12-18 23:42:17
subtitle:
---
## 5.1 主配置文件
1. mybatis.xml时项目的主配置文件
2. 文件结构
   * 开头声明约束文件
   * 使用 `configuration`标签作为根标签
   * 主要内容包括：定义别名、数据源、mapper文件

## 5.2 主要内容
### 5.2.1 数据源dataSource
1. 说明
   * java体系中，规定实现了javax.sql.DataSource接口的都是数据源
   * 使用dataSource标签实现连接池配置，语法：`<dataSource type=""></dataSource>`
2. type属性：指定数据源类型
   * POOLED：使用连接池，mybatis会创建PooledDataSource类
   * UPOOLED：不使用连接池，每次执行sql语句，先创建连接，执行sql后再关闭连接，通过UnPooledDataSource对象来管理Connection对象的使用
   * JNDI：java命名和目录服务，类似于windows注册表
3. 数据库连接信息配置
   * 将数据库连接信息放到单独的文件中，和mybatis主配置文件分开
   * 在resources目录下定义配置文件，名为 xxx.properties，数据格式为`key=value`
   * key一般使用`.` 分为多级：`jdbc.driver=`
4. dataSource 配置
   * 在主配置文件中，使用properties标签指定配置文件位置
   * 使用configuration标签指定数据源配置：
     * 使用property子标签指定具体信息
     * 使用 `${key}`的形式调用配置文件信息
5. 示例
   * jdbc.properties
        ```
        jdbc.driver=com.mysql.cj.jdbc.Driver
        jdbc.url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC
        jdbc.username=root
        jdbc.password=123456
        ```
   * mybatis.xml
        ```xml
        <configuration>
            <properties resource="jdbc.properties"/>
            <environments default="development">
                <environment id="development">
                    <transactionManager type="JDBC"/>
                    <dataSource type="POOLED">
                        <property name="driver" value="${jdbc.driver}"/>
                        <property name="url" value="${jdbc.url}"/>
                        <property name="username" value="${jdbc.username}"/>
                        <property name="password" value="${jdbc.password}"/>
                    </dataSource>
                </environment>
            </environments>
        ```

### 5.2.2 事务
1. 说明
   * Mybatis 框架是对 JDBC 的封装，所以 Mybatis 框架的事务控制方式，本身也是用 JDBC 的 Connection对象的 commit(), rollback() .
   * Connection 对象的 setAutoCommit()方法来设置事务提交方式的。自动提交和手工提交、
2. 使用transactionManager标签规定事务管理器
   * 格式：`<transactionManager type=" "/>`
   * type属性：
      * JDBC：使用 JDBC 的事务管理机制。即，通过 Connection 的 commit()方法提交，通过 rollback()方法回滚，默认手动提交事务
      * MANAGED：由容器来管理事务的整个生命周期（如 Spring 容器）
3. 自动提交事务
   * 通过`SqlSessionFactory`的`openSession`方法创建SqlSession
   * `openSession`可以传入属性，属性为true则自动提交事务

### 5.2.3 mappers
1. mappers标签用于指定映射文件位置，可以有多个映射文件
2. 有两种指定方式
   * mapper子标签：
     * 格式：`<mapper resource=" " />`
     * 用于指定单个文件
     * 示例：
        ```xml
        <mappers>
            <mapper resource="com/powernode/dao/StudentDao.xml"/>
        </mappers>
        ```

   * package子标签
     * 格式：`<package name=" "/>`
     * 指定包下的所有Dao接口
     * 要求 Dao 接口名称和 mapper 映射文件名称相同，且在同一个目录中
     * 示例：
        ```xml
        <mappers>
            <package name="com/powernode/dao"/>
        </mappers>
        ```

### 5.2.4 typeAliases
1. 类型别名
   * mybatis可以给类定义别名，主要使用在`<select resultType=”别名”>`中
   * 可以指定单个别名。也可以批量定义
2. 指定单个类型别名
   * 使用`typeAlias`子标签，格式：`<typeAlias type="" alias=""/>`
   * type属性指定类名，alias属性指定别名
   * 示例
        ```xml
        <typeAliases>
            <typeAlias type="com.powernode.domain.Student" alias="stu"/>
        </typeAliases>
        ```
2. 批量定义
   * 使用 `package` 子标签，格式 `<package name=""/>`
   * 属性指定包名，则此包下的类使用其类名作为包名（首字母大小写都可以）
   * 示例
        ```xml
        <typeAliases>
            <package name="com.powernode.domain"/>
            <package name="com.powernode.vo"/>
        </typeAliases>
        ```



## 5.3 PageHelper扩展
### 5.3.1 说明
1.  PageHelper是一个mybatis通用分页插件
2.  支持多种数据库语言，是一个开源项目（[github](https://github.com/pagehelper/Mybatis-PageHelper.git)）

### 5.3.2 使用
1. Maven配置文件中引入依赖
   ```xml
    <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>5.2.0</version>
    </dependency>
   ```
2. mybatis主配置文件中加入 plugin 配置
   * plugins在配置文件中的位置必须符合要求，否则会报错，顺序如下:
        ```
        properties?, settings?, 
        typeAliases?, typeHandlers?, 
        objectFactory?,objectWrapperFactory?, 
        plugins?, 
        environments?, databaseIdProvider?, mappers?
        ```
   * 代码：
        ```xml
        <plugins>
            <!-- com.github.pagehelper为PageHelper类所在包名 -->
            <plugin interceptor="com.github.pagehelper.PageInterceptor">
            </plugin>
        </plugins>
        ```
3. 使用
   * 查询语句之前调用 `PageHelper.startPage` 静态方法，紧跟在这个方法后的第一个 MyBatis 查询方法会被进行分页
   * 该方法有两个属性
      * pageName：第几页，从1开始
      * pageSize：每页有几行数据
   * 示例
        ```java
        @Test
        public void testSelectAll() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            PageHelper.startPage(2,3);
            List<Student> students = dao.selectAll();
            students.forEach(stu->System.out.println(stu));
            sqlSession.close();
        }
        ```

