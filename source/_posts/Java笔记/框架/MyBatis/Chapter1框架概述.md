---
title: Chapter1 框架概述
excerpt: 什么是框架、MyBatis框架简介
tags:
  - java
  - mybatis
categories:
  - Java笔记
  - MyBatis
banner_img: /img/dog.png
index_img: /img/post/mybatis_logo.jpg
abbrlink: a685019b
date: 2020-12-15 23:49:49
updated: 2020-12-17 00:28:34
subtitle:
---
## 1.1 软件开发常用结构
### 1.1.1 三层架构
1. 架构简介
   * **界面层**（表示层，视图层）：接受用户的数据，显示请求的处理结果。
   * **业务逻辑层**：接收表示传递过来的数据，检查数据，计算业务逻辑，调用数据访问层获取数据。
   * **数据访问层**： 与数据库打交道。主要实现对数据的增、删、改、查。将存储在数据库中的数据提交给业务层，同时将业务层处理的数据保存到数据库
2. 对应的包
   * 界面层：controller包
   * 业务逻辑层：service包
   * 数据访问层：dao包
3. 三层交互  
   用户使用界面层--->业务逻辑层--->数据访问层（持久层）--->DB 数据库（mysql）
### 1.1.2 常用框架
   * 界面层：servlet--springmvc
   * 业务逻辑层：service--spring
   * 数据访问层：dao--mybatis

## 1.2 框架
### 1.2.1 定义
1. 框架（Framework）是整个或部分系统的可重用设计，表现为一组抽象构件及构件实例间交互的方法;另一种认为，框架是可被应用开发者定制的应用骨架、模板
2. 框架是半成品软件，是一组组件，供开发者使用完成需要的系统

### 1.2.2 框架优点
1. 解决了技术整合问题
2. 提高了开发效率

## 1.3 JDBC
缺点123...（车轱辘话）  
所以不用

## 1.4 MyBatis 框架概述
开源项目，原名ibatis
### 1.4.1 mybatis框架功能
1. 注册数据库的驱动，例如 Class.forName(“com.mysql.jdbc.Driver”)
2. 创建 JDBC 中必须使用的 Connection ， Statement， ResultSet 对象
3. 从 xml 中获取 sql，并执行 sql 语句，把结果转换 java 对象
4. 关闭资源

### 1.4.2 什么是mybatis
1. mybatis是一个sql映射框架，提供的数据库的操作能力，相当于增强的JDBC
2. 使用mybatis让开发人员专注与sql，不必关心Connection,Statement,ResultSet
  的创建，销毁等工作。

## 1.5 mybatis入门
### 1.5.1 准备工作

1. 创建mysql数据库和表

   ```mysql
    create database mybatis;
    use mybatis;
    create table student (
        id int not null,
        name varchar(80),
        email varchar(100),
        age int,
        primary key (id)
    );
    insert into student
    values
    (1001, 'Harry Potter', 'potter@jk.com',13),
    (1002,'Hermione','granger@jk.com',14),
    (1003,'Ron','weasley@jk.com',13);
    ```

2. 创建maven工程
   * 依赖
        ```xml
        <dependencies>
            <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
            </dependency>

            <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
            <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.5</version>
            </dependency>

            <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
            <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.19</version>
            </dependency>
        </dependencies>
        ```
    * 插件
        ```xml
        <build>
            <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                <include>**/*.xml</include>
                </includes>
            </resource>
            </resources>
        </build>
        ```
        不加这个编译的时候会忽略配置文件
3. 添加日志功能
    * 日志可以在控制台输出执行的 sql 语句和参数
    * mybatis.xml
        ```xml
        <configuration>
            <settings>
                <setting name="logImpl" value="STDOUT_LOGGING"/>
            </settings>
        ```
    * **必须加在configuration标签的最前面，否则会报错**

### 1.5.2 查询操作
1. 创建实体类及Dao接口
    * Student.java
        ```java
        public class Student {
            private Integer id;
            private String name;
            private String email;
            private Integer age;

            public Integer getId() {return id;}

            public String getName() {return name;}

            public String getEmail() {return email;}

            public Integer getAge() {return age;}

            public void setId(Integer id) {this.id = id;}

            public void setName(String name) {this.name = name;}

            public void setEmail(String email) { this.email = email;}

            public void setAge(Integer age) {  this.age = age;}

            @Override
            public String toString() {
                return "Student{" +
                        "id=" + id +
                        ", name='" + name + '\'' +
                        ", email='" + email + '\'' +
                        ", age=" + age +
                        '}';
            }
        }
        ```
    * Dao接口StudentDao.java
        ```java
        public interface StudentDao {

            // 查询所有学生数据
            public List<Student> selectStudents();
        }
        ```
    * 接口映射文件 StudentDao.xml
        * 与接口同一个包下且同名
        ```xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd"><!--指定约束文件-->
        <mapper namespace="com.powernode.dao.StudentDao">
        <select id="selectStudent" resultType="com.powernode.domain.Student">
            select * from student order by id
        </select>
        </mapper>
        ```
        * 约束文件：限制，检查在当前文件中出现的标签，属性必须符合mybatis的要求。
        * mapper标签：当前文件的根标签
        * namespace属性：命名空间，唯一值的， 可以是自定义的字符串。一般使用dao接口全类名
        * 使用特定的标签，表示数据库的特定操作
            * `<select>`: 表示执行查询，select语句
            * `<update>`: 表示更新数据库的操作， 就是在<update>标签中 写的是update sql语句
            * `<insert>`: 表示插入， 放的是insert语句
            * `<delete>`: 表示删除， 执行的delete语句
        * select标签：
            * `id`: 你要执行的sql语法的唯一标识，可以自定义，一般使用接口中的方法名。
            * `resultType`:结果类型，sql语句执行后得到ResultSet,遍历这个ResultSet得到java对象。值写的类型的全限定名称
2. mybatis主配置文件
    * resources目录下
    * mybatis.xml：
        ```xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>
            <environments default="development">
                <environment id="development">
                    <transactionManager type="JDBC"/>
                    <dataSource type="POOLED">
                        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"/>
                        <property name="username" value="root"/>
                        <property name="password" value="xxxxx"/>
                    </dataSource>
                </environment>
            </environments>
            <!--指定映射文件位置-->
            <mappers>
                <mapper resource="com/powernode/dao/StudentDao.xml"/>
            </mappers>
        </configuration>
        ```
    * 这里用的是mysql8.0，所有url比较复杂
3. 测试类
    ```java
    public static void main( String[] args ) throws IOException {
        // 访问mybatis读取student数据
        // 1. 定义mybatis主配置文件名称，从类路径根开始
        String config = "mybatis.xml";
        // 2. 读取文件
        InputStream in = Resources.getResourceAsStream(config);
        // 3. 创建SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        // 4. 创建SqlSessionFactory对象
        SqlSessionFactory factory = builder.build(in);
        // 5. 获取SqlSession对象
        SqlSession sqlSession = factory.openSession();
        // 6. 指定要执行的sql语句的标识:sql映射文件中的namespace+"."+标签id值
        String sqlId = "com.powernode.dao.StudentDao"+"."+"selectStudents";
        // 7. 执行sql语句，通过sqlId找到语句
        List<Student> studentList = sqlSession.selectList(sqlId);
        // 8. 输出结果
        studentList.forEach(stu->System.out.println(stu));
        // 9. 关闭对象
        sqlSession.close();
    }
    ```

### 1.5.3 插入操作
1. 接口、接口映射文件
    * StudentDao.java
        ```java
        public int insertStudent(Student student);
        ```
    * StudentDao.xml
        ```xml
        <insert id="insertStudent" >
            insert into student values(#{id},#{name},#{email},#{age})
        </insert>
        ```
2. 测试类
    * Test
        ```java
        @Test
        public void insertTest() throws IOException {
            // 访问mybatis读取student数据
            // 1. 定义mybatis主配置文件名称，从类路径根开始
            String config = "mybatis.xml";
            // 2. 读取文件
            InputStream in = Resources.getResourceAsStream(config);
            // 3. 创建SqlSessionFactoryBuilder对象
            SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
            // 4. 创建SqlSessionFactory对象
            SqlSessionFactory factory = builder.build(in);
            // 5. 获取SqlSession对象
            SqlSession sqlSession = factory.openSession();
            // 6. 指定要执行的sql语句的标识:sql映射文件中的namespace+"."+标签id值
            String sqlId = "com.powernode.dao.StudentDao"+"."+"insertStudent";
            // 7. 执行sql语句，通过sqlId找到语句
            Student student = new Student();
            student.setAge(14);
            student.setEmail("Weasley@jk.com");
            student.setId(1004);
            student.setName("Ginny");
            int nums = sqlSession.insert(sqlId,student);
            sqlSession.commit(); // 需要提交事务
            // 8. 输出结果
            System.out.println("执行insert的结果："+nums);
            // 9. 关闭对象
            sqlSession.close();
        }
        ```

### 1.5.4 其他操作
1. 更新及删除操作流程同上：
    * 在接口中增加相应方法
    * 在映射文件中加入相关语句
    * 调用sqlSession的相关方法
2. 示例
    * StudentDao.java
        ```java
        int deleteStudent(int id);//删
        int updateStudent(Student student); //改
        ```
    * StudentDao.xml
        ```xml
        <update id="updateStudent">
            update student set age = #{age} where id=#{id}
        </update>
        <delete id="deleteStudent">
            delete from student where id=#{studentId}
        </delete>
        ```
    * Test
        ```java
        int rows = session.delete(sqlId,id); //删除
        int rows = session.update(sqlId,student);// 插入
        ```


