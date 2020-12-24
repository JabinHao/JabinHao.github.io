---
title: Chapter3 MyBatis框架Dao代理
excerpt: dao动态代理、dao接口方法参数与返回值、mybatis实现模糊查询
tags:
  - java
  - mybatis
categories:
  - Java笔记
  - MyBatis
banner_img: /img/dog.png
index_img: /img/post/mybatis_logo.jpg
abbrlink: 9b499888
date: 2020-12-17 02:11:27
updated: 2020-12-17 02:11:27
subtitle:
---
## 3.1 Dao 代理实现 CURD
### 3.1.1 步骤
上一章末尾我们通过mybatis动态代理实现了CURD操作，具体步骤：
1. 创建Student类
   ```java
    public class Student {
        private Integer id;
        private String name;
        private String email;
        private Integer age;

        // 创建相应的getter、setter方法，为节省篇幅此处省略

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
2. 创建dao接口`StudentDao`和相应的接口映射文件
   ```java
    // StudentDao.java
    public interface StudentDao {
        List<Student> selectStudents();
        int insertStudent(Student student);
    }
   ```
   ```xml
    <!-- StudentDao.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd"><!--指定约束文件-->
    <mapper namespace="com.powernode.dao.StudentDao">
        <select id="selectStudents" resultType="com.powernode.domain.Student">
            select * from student order by age
        </select>
        <insert id="insertStudent">
            insert into student values (#{id},#{name},#{email},#{age})
        </insert>
    </mapper>
   ```
3. 创建工具类`MyBatisUtils`封装`SqlSession`对象创建过程
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
4. 在test中使用：
   * 通过`getMapper`获取代理对象（即接口实现类对象）
   * 通过代理对象完成CURD操作
   * 代码：
        ```java
        @Test
        public void testSelectStudents() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class); //获取接口实现类
            List<Student> students = dao.selectStudents();
            for (Student student : students) {
                System.out.println(student);
            }
            sqlSession.close();
        }
        ```
5. 文件目录
   ```
    +--- src
    |   +--- main
    |   |   +--- java
    |   |   |   +--- com
    |   |   |   |   +--- powernode
    |   |   |   |   |   +--- App.java
    |   |   |   |   |   +--- dao
    |   |   |   |   |   |   +--- StudentDao.java
    |   |   |   |   |   |   +--- StudentDao.xml
    |   |   |   |   |   +--- domain
    |   |   |   |   |   |   +--- Student.java
    |   |   |   |   |   +--- utils
    |   |   |   |   |   |   +--- MyBatisUtils.java
    |   |   +--- resources
    |   |   |   +--- mybatis.xml
    |   +--- test
    |   |   +--- java
    |   |   |   +--- com
    |   |   |   |   +--- powernode
    |   |   |   |   |   +--- AppTest.java
    +--- pom.xml
   ```

### 3.1.2 原理
* jdk的动态代理
* 暂时跳过

## 3.2 深入理解参数（Dao接口方法的参数）
### 3.2.1 parameterType
1. 映射文件
    ```xml
    <select id="selectStudentById" parameterType="java.lang.Integer" resultType="com.powernode.domain.Student">
        select * from student where id=#{id}
    </select>
    ```
2. `parameterType`: 
   * 接口中方法参数的类型， 类型的完全限定名或别名
   * mybatis为各种类型起了别名，在文档中可以找到
   * 可以省略，MyBatis可以通过反射得到。
3. MyBatis 传递参数：从 java 代码中把参数传递到 mapper.xml （接口映射文件）

### 3.2.2 一个简单类型的参数
1. mybatis把java基本数据类型和String都叫做简单类型
2. 在映射文件中获取 ：`#{任意字符}`，与方法中的形参名无关
3. 原理
   * 使用`#{}`之后， mybatis执行sql是使用的jdbc中的PreparedStatement对象，由mybatis执行下面的代码：
        ```java
        // 1. mybatis创建Connection ， PreparedStatement对象
            String sql="select id,name, email,age from student where id=?";
            PreparedStatement pst = conn.preparedStatement(sql);
            pst.setInt(1,1001);

        // 2. 执行sql封装为resultType="com.bjpowernode.domain.Student"这个对象
            ResultSet rs = ps.executeQuery();
            Student student = null;
            while(rs.next()){
               //从数据库取表的一行数据， 存到一个java对象属性中
               student = new Student();
               student.setId(rs.getInt("id));
               student.setName(rs.getString("name"));
               student.setEmail(rs.getString("email"));
               student.setAge(rs.getInt("age"));
            }

           return student;  //给了dao方法调用的返回值
        ```

### 3.2.3 多个参数
1. 使用 `@Param`
   * 当Dao接口方法多个参数，可以使用命名参数。在方法形参前面加入`@Param(“自定义参数名”)`，mapper 文件使用 `#{自定义参数名}`获取参数值。
   * 接口
        ```java
        List<Student> selectMultiParam(@Param("myname") String name, @Param("myage") Integer age);
        ```
   * 映射文件
        ```xml
        <select id="selectMultiParam" resultType="com.powernode.domain.Student">
            select * from student where name = #{myname} or age = #{myage}
        </select>
        ```
   * 测试：
        ```java
        @Test
        public void testSelectMultiParam() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            List<Student> students = dao.selectMultiParam("Hermione", 12);
            sqlSession.close();
        }
        ```

2. 使用对象
   * 使用 java 对象传递参数， java 的属性值就是 sql 需要的参数值。 每一个属性就是一个参数。
   * 语法格式： `#{ property,javaType=java 中数据类型名,jdbcType=数据类型名称 }`
   * javaType, jdbcType 的类型 MyBatis 可以检测出来，一般不需要设置。 常用格式 `#{ property }`
   * 示例
     * vo.QueryParam类
        ```java
        public class QueryParam {
            private String paranName;
            private Integer paramAge;

            public void setParanName(String paranName) {
                this.paranName = paranName;
            }

            public void setParamAge(Integer paramAge) {
                this.paramAge = paramAge;
            }

            public String getParanName() {
                return paranName;
            }

            public Integer getParamAge() {
                return paramAge;
            }
        }
        ```
     * 接口
        ```java
        List<Student> selectMultiObject(QueryParam param);
        ```
     * mapper
        ```xml
        <select id="selectMultiObject" resultType="com.powernode.vo.QueryParam">
            select * from student where name=#{paramName} or age=#{paramAge}
        </select>
        ```
     * test同上

3. 根据位置
   * 参数位置从 0 开始， 引用参数语法 `#{ arg 位置 }` ， 第一个参数是`#{arg0}`, 第二个是`#{arg1}`
   * mybatis-3.3 版本和之前的版本使用`#{0}`,`#{1}`方式， 从 mybatis3.4 开始使用`#{arg0}`方式
   * 接口
        ```java
        List<Student> selectByNameAndAge(String name,int age);
        ```
   * mapper
        ```xml
        <select id="selectByNameAndAge" resultType="com.bjpowernode.domain.Student">
            select * from student where name=#{arg0} or age =#{arg1}
        </select>
        ```

4. 使用 `Map`
   * Map 集合可以存储多个值， 使用Map向 mapper 文件一次传入多个参数。
   * Map 集合使用 `String` 的 key，Object 类型的值存储参数，mapper 文件使用 `# { key }` 引用参数值
   * 示例
     * 接口
        ```java
        List<Student> selectMultiMap(Map<String, Object> map);
        ```
     * mapper
        ```xml
        <select id="selectMultiMap" resultType="com.powernode.domain.Student">
            select * from student where name=#{myname} or id=#{myid}
        </select>
        ```
     * test
        ```
        @Test
        public void testSelectMultiMap() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            Map<String,Object> map = new HashMap<>();
            map.put("myname","Harry Potter");
            map.put("myid",1002);
            List<Student> students = dao.selectMultiMap(map);
            sqlSession.close();
        }
        ```
   * 缺点：key在使用时定义，随意性很强，可读性很差

### 3.2.4 `#`与`$`
1. `#`：占位符
   * 告诉 mybatis 使用实际的参数值代替。
   * 使用 PrepareStatement 对象执行 sql 语句时，用`#{…}`代替sql 语句的`“?”`。 
   * 这样做更安全，更迅速，通常也是首选做法
2. `$`：字符串替换，
   * 告诉 mybatis 使用`$`包含的`“字符串”`替换所在位置。
   * 使用 Statement 把 sql 语句和`${}`的内容连接起来。
   * 主要用在替换表名，列名，不同列排序等操作。
3. 示例
   * `#`
     * mapper: `select id,name, email,age from student where id=#{studentId}`
     * 结果： `select id,name, email,age from student where id=?` 
   * `$`
     * mapper：`select id,name, email,age from student where id=${studentId}`
     * `$` 的结果：`select id,name, email,age from student where id=1001`
4. `$`存在安全隐患
   * `$`是直接替换，使用时若在后面添加了其他内容，会有安全隐患
   * 示例
        ```xml
        <select id="selectUse$" resultType="com.powernode.domain.Student">
            select * from student where name=${myname}
        </select>
        ```
        ```java
        @Test
        public void testSelectUse$(){
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            List<Student> students = dao.selectUse$("'Hermione';drop table student");//注意加了单引号
            for(Student stu: students){
                System.out.println("student="+stu);
            }
            sqlSession.close();
        }
        ```
        执行时会把表删掉
5. 区别
   * `#` 使用`?`在sql语句中做占位的， 使用 PreparedStatement 执行sql，效率高
   * `#` 能够避免sql注入，更安全。
   * `$` 不使用占位符，是字符串连接方式，使用Statement对象执行sql，效率低
   * `$` 有sql注入的风险，缺乏安全性。
   * `$` 可以替换表名或者列名：
     * mapper
        ```xml
        <select id="selectUse$Order" resultType="com.bjpowernode.domain.Student">
            select * from student order by ${colName}
        </select>
        ```
     * test
        ```java
        @Test
        public void testSelectUse$Order(){
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            List<Student> students = dao.selectUse$Order("age"); //修改参数则可以按不同的列修改排序
            for(Student stu: students){
                System.out.println("学生="+stu);
            }
            sqlSession.close();
        }
        ```

## 3.3 封装 MyBatis 输出结果
### 3.3.1 `resultType` 属性
1. 说明
   * mapper子标签属性，用于规定执行 sql 得到 ResultSet 转换的类型
   * 使用类型的完全限定名或别名
   * **如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身**。
   * resultType 和 resultMap，不能同时使用
2. 自定义别名
   * 可以给自定义的类创建别名
   * 在mybatis配置文件中，使用`<typeAliass>`标签定义,标签定义在configuration标签下
     * 方式一：`<typeAlias>`子标签，`typeAlias`属性为全类名，`alias`属性为别名
        ```xml
        <typeAliases>
            <typeAlias type="com.powernode.vo.ShortStudent" alias="sstu"/>
            <typeAlias type="com.powernode.domain.Student" alias="stu"/>
        </typeAliases>
        ```
        此
     * 方式二：使用`<package>`标签，name属性为包名，则自动将包下所有类的类名作为其别名
        ```xml
        <typeAliases>
            <package name="com.powernode.domain"/>
        </typeAliases>
        ```
3. 返回自定义类型
   * mybatis默认将返回结果中的值赋给类的同名属性
   * 可以将返回结果赋给其他类（非student类）
   * 创建`ShortStudent`类：
        ```java
        public class ShortStudent {
            private Integer id;
            private String name;

            public String getName() {
                return name;
            }

            public Integer getId() {
                return id;
            }

            public void setId(Integer id) {
                this.id = id;
            }

            public void setName(String name) {
                this.name = name;
            }

            @Override
            public String toString() {
                return "ShortStudent{" +
                        "id=" + id +
                        ", name='" + name + '\'' +
                        '}';
            }
        }
        ```
   * 接口中定义方法
        ```java
        ShortStudent selectShortStudent(@Param("id") Integer id);
        ```
   * mapper：
        ```xml
        <select id="selectShortStudent" resultType="sstu">
            select * from student where id = #{id}
        </select>
        ```
   * mybatis配置文件自定义别名（见上面）

4. 返回Map
   * sql 的查询结果作为 Map 的 key 和 value。推荐使用 Map<Object,Object>。
   * 只能返回一条记录，多了会报错
   * 示例：
     * 接口
        ```java
        Map<Object,Object> selectReturnMap(@Param("name") String name);
        ```
     * mapper
        ```xml
        <select id="selectReturnMap" resultType="java.util.HashMap">
            select * from student where name=#{name}
        </select>
        ```
     * test
        ```java
        @Test
        public void testSelectReturnMap() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            Map<Object, Object> result = dao.selectReturnMap("Hermione");
            System.out.println(result);
        }
        ```

### 3.3.2 `resultMap`
1. 说明
   * 结果映射，自定义 sql 的结果和 java 对象属性的映射关系
   * 可以让列名和对象属性名不同
2. 使用
   * 先在mapper中定义 resultMap,指定列名和属性的对应关系
     * 主键列，使用id子标签
     * 非主键列，使用result子标签
     * 子标签两个属性，column代表数据库中列名，property代表对象属性名
   * 在`<select>`中把 resultType 替换为 resultMap
3. 示例：使用resultMap将name和email对调
   * Dao接口
        ```java
        List<Student> selectAllStudent();
        ```
   * mapper
        ```xml
        <!--  使用resultMap  -->
        <resultMap id="studentMap" type="com.powernode.domain.Student"><!--id属性是唯一标识，后面select调用时使用-->
            <!--自定义列名和对象属性之间的关系-->
            <!--主键列，使用id标签-->
            <id column="id" property="id"/>
            <!--非主键列，使用result-->
            <result column="name" property="email"/>
            <result column="email" property="name"/>
        </resultMap>
        <select id="selectAllStudent" resultMap="studentMap">
            select * from student;
        </select>
        ```
   * test
        ```java
        @Test
        public void testSelectAllStudent() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            List<Student> students = dao.selectAllStudent();
            students.forEach(stu->System.out.println(stu));
        }
        ```
   * 结果：
        ```
        Student{id=1001, name='potter@jk.com', email='Harry Potter', age=13}
        Student{id=1002, name='granger@jk.com', email='Hermione', age=14}
        Student{id=1003, name='weasley@jk.com', email='Ron', age=13}
        Student{id=1004, name='Weasley@jk.com', email='Ginny', age=12}
        Student{id=1005, name='Malfoy@jk.com', email='Draco', age=14}
        ```

### 3.3.3 实体类属性名和列名不同的处理方式
1. 使用 `resultMap`
2. 使用列别名和 `<resultType>`
   * mapper
        ```xml
        <select id="selectDiffColproperty" resultType="com.powernode.domain.Student">
            select id, name as email, email as name, age from student;
        </select>
        ```

## 3.4 模糊查询 like
### 3.4.1 实现方式一
1. 在java 代码中给查询数据加上`%`
2. 示例
   * dao
        ```java
        List<Student> selectLikeOne(@Param("sql") String sql);
        ```
   * mapper
        ```xml
        <select id="selectLikeOne" resultType="Student">
            select * from student where email like #{sql}
        </select>
        ```
   * test
        ```java
        @Test
        public void testSelectLikeOne() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            String sql = "%er%";
            List<Student> students = dao.selectLikeOne(sql);
            students.forEach(stu->System.out.println(stu));
        }
        ```

### 3.4.2 实现方式二
1. 在 mapper 文件 sql 语句的条件位置加上 `%`
2. 示例
    * dao
        ```java
        List<Student> selectLikeTwo(@Param("sql") String sql);
        ```
    * mapper
        ```xml
        <select id="selectLikeTwo" resultType="Student">
            select * from student where email like "%" #{sql} "%"  <!--注意此处的空格-->
        </select>
        ```
    * test
        ```java
        @Test
        public void testSelectLikeTwo() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            String sql = "er";
            List<Student> students = dao.selectLikeTwo(sql);
            students.forEach(stu->System.out.println(stu));
        }
        ```


