---
title: Chapter2 MyBatis简单使用
excerpt: mybatis主要接口和类、mybatis使用传统Dao开发示例、mybatis动态代理简单介绍
tags:
  - java
  - mybatis
categories:
  - Java笔记
  - MyBatis
banner_img: /img/dog.png
index_img: /img/post/mybatis_logo.jpg
abbrlink: 4ddf6c39
date: 2020-12-17 00:28:34
updated: 2020-12-17 02:11:27
subtitle:
---
## 2.1 MyBatis 对象分析
### 2.1.1 主要类
1. `Resources`类
   * 资源类，用于读取配置文件，提供很多静态方法加载并解析资源文件，返回IO流对象
   * `InputStream in = Resources.getResourceAsStream(config);`
2. `SqlSessionFactoryBuilder`类
   * 创建`SqlSessionFactory`对象
   * 使用：
        ```java
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        ```
3. `SqlSessionFactory` 接口
   * `SqlSessionFactory` 接口对象是一个重量级对象（系统开销大），一个项目只需一个该对象
   * 实现类：`DefaultSqlSessionFactory`
   * 通过 `openSession()` 方法创建 `SqlSession` 对象
     * `openSession(true)`：创建一个有自动提交功能的 `SqlSession`
     * `openSession(false)`：创建一个非自动提交功能的 `SqlSession`，需手动提交
     * 默认为 `false`
   * `qlSession sqlSession = factory.openSession();`
4. `SqlSession` 接口
   * 定义了一系列操作数据的方法，用于数据库操作
   * 实现类：`DefaultSqlSession`
   * 线程不安全，需要在方法内部使用，执行sql语句前通过`openSession()`方法获取`SqlSession`对象，执行完后调用自己的`close()`方法关闭会话

### 2.1.2 创建工具类
1. 说明
   * `SqlSessionFactoryBuilder`、`SqlSessionFactory`等的创建使用过程存在大量重复代码
   * 可以创建工具类把过程封装起来
2. 工具类 MyBatisUtils.java
   ```java
    public class MyBatisUtils {

        private static SqlSessionFactory factory = null;
        static {
            String config = "mybatis.xml";
            try {
                InputStream in = Resources.getResourceAsStream(config);
                factory = new SqlSessionFactoryBuilder().build(in);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        // 获取SqlSession的方法
        public static SqlSession getSqlSession(){
            SqlSession sqlSession = null;
            if (factory != null) {
                sqlSession = factory.openSession(); // 非自动提交事务
            }
            return sqlSession;
        }
    }
   ```
3. 使用：
   ```java
    public static void main( String[] args ) throws IOException {
        // 获取SqlSession对象
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        // 指定要执行的sql语句的标识:sql映射文件中的namespace+"."+标签id值
        String sqlId = "com.powernode.dao.StudentDao"+"."+"selectStudents";
        // 执行sql语句，通过sqlId找到语句
        List<Student> studentList = sqlSession.selectList(sqlId);
        // 输出结果
        studentList.forEach(stu->System.out.println(stu));
        // 关闭SqlSession对象
        sqlSession.close();
    }
   ```
   
## 2.2 MyBatis 使用传统 Dao 开发方式
### 2.2.1 开发流程
1. 创建Student类
2. 创建Dao接口、接口映射文件
3. 创建Dao接口实现类并实现接口中的方法
   * StudentDaoImpl.java
        ```java
        public class StudentDaoImpl implements StudentDao{
            @Override
            public List<Student> selectStudents() {
                SqlSession sqlSession = MyBatisUtils.getSqlSession();
                String sqlId = "com.powernode.dao.StudentDao.selectStudents";
                List<Student> students = sqlSession.selectList(sqlId);
                sqlSession.close();
                return students;
            }

            @Override
            public int insertStudent(Student student) {
                SqlSession sqlSession = MyBatisUtils.getSqlSession();
                String sqlId = "com.powernode.dao.StudentDao.insertStudent";
                int nums = sqlSession.insert(sqlId,student);
                sqlSession.commit();
                sqlSession.close();
                return nums;
            }
        }
        ```
4. 编写工具类MybatisUtils
5. 在测试文件中使用
   ```java
    @Test
    public void testInsertStudent() {
        StudentDao dao = new StudentDaoImpl();
        Student student = new Student();
        student.setId(1005);
        student.setName("Draco");
        student.setEmail("Malfoy@jk.com");
        student.setAge(14);
        int nums = dao.insertStudent(student);
        System.out.println(nums);
    }
   ```

### 2.2.2 传统开发方式优化
1. 问题  
   实现类的实现方法是格式化的，可以抽取出来
2. 实现类与映射文件对应关系
   ```java
    @Test
    public void testSelectStudents() {
        StudentDao dao = new StudentDaoImpl();
        List<Student> students = dao.selectStudents();
        for (Student student : students) {
            System.out.println(student);
        }
    }
   ```
   * dao对象：`mapper`标签的`namespace`可以获取全类名
   * 方法名称：mapper子标签的id对应实现类的方法名称
   * 通过返回值也可确定要调用的sqlSession的相应方法（增删改查）
     * List：SqlSession.selectList()
     * int：根据mapper的子标签名是insert还是update
3. mybatis动态代理
   * 由上，mybatis可以直接根据dao的方法调用，获取执行sql语句的信息
   * mybatis根据dao接口，创建实现类，并创建对象，完成SqlSession方法调用，访问数据库。

4. 使用
   * Student类
   * StudentDao接口、接口映射文件
   * MyBatisUtils工具类
   * 使用：
        ```java
        @Test
        public void testSelectStudents() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class); //获取接口实现类
            List<Student> students = dao.selectStudents();
            for (Student student : students) {
                System.out.println(student);
            }
        }
        ```
    * 使用了mybatis的动态代理机制：
      * 相对于入门案例，无需直接调用sqlSession中的方法
      * 相对于传统Dao开发，无需创建接口实现类



