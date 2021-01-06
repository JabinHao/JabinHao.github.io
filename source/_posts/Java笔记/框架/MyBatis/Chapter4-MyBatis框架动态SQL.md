---
title: Chapter4 MyBatis框架动态SQL
excerpt: 动态sql的使用，if、where、foreach标签，代码片段等
tags:
  - java
  - mybatis
categories:
  - Java笔记
  - MyBatis
banner_img: /img/dog.png
index_img: /img/post/mybatis_logo.jpg
abbrlink: 748b7526
date: 2020-12-17 23:31:14
updated: 2020-12-18 17:25:50
subtitle:
---
## 4.1 概述
### 4.1.1 什么是动态SQL
1. 动态 SQL是指通过 MyBatis 提供的各种标签对条件作出判断以实现动态拼接 SQL 语句
2. 常用的动态 SQL 标签有`<if>`、 `<where>`、 `<choose/>`、 `<foreach>`等
3. 动态 SQL，主要用于解决查询条件不确定的情况

### 4.1.2 环境准备
1. 步骤
   * 创建maven项目，引入mybatis、mysql驱动依赖
   * 创建实体类Student、StudentDao接口、接口映射文件、mybatis配置文件
   * 创建mysql数据库，新建表student
2. 注意事项
   * 要使用动态sql，dao接口方法的参数必须是对象
   * mapper 的动态 SQL 中，大于号、小于号、大于等于、小于等于等应进行转义
   * 符号表：

        符号|含义|转义符
        :-:|:-:|:-:
        <|小于|`&lt;`
        &gt;|大于|`&gt;`
        &gt;=|大于等于|`&gt;=`
        <=|小于等于|`&lt;=`

## 4.2 动态SQL — `if`
### 4.2.1 if标签
1. if 标签作为 select 等标签的子标签使用
2. test 属性值为判断语句

### 4.2.2 使用实例
1. 说明：
   * 找到邮箱为weasley@jk.com且年龄大于12岁的学生（即Ron）
   * where后面加上 1=1，这样每个if标签都可以加and或者or标签（如果第一个不加and而第一个条件不满足直接执行后面的if，就变成了 `where and age > #{age}`，会报错）
2. 代码
   * mapper
        ```xml
        <select id="selectStudentIf" resultType="com.powernode.domain.Student">
            select * from student
            where 1=1
            <if test="email != null and email != '' ">
                and email = #{email}
            </if>
            <if test="age>0">
                and age > #{age}
            </if>
        </select>
        ```
   * test
        ```java
        @Test
        public void testSelectLikeTwo() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            Student student = new Student();
            student.setAge(12);
            student.setEmail("weasley@jk.com");
            List<Student> students = dao.selectStudentIf(student);
            students.forEach(stu->System.out.println(stu));
            sqlSession.close();
        }
        ```

## 4.3 动态SQL — `<where>`
1. 说明
   * 前面提到if标签需要在where后面加一个恒等条件，但当查询量很大时，会影响效率
   * 使用where标签可以避免这种问题
   * where标签作为if的上一级标签使用，第一个if语句可以不加and或or，但后面的要加
2. 实例
   * mapper
        ```xml
        <select id="selectStudentWhere" resultType="com.powernode.domain.Student">
            select * from student
            <where>
                <if test="email != null and email != '' ">
                    email = #{email}
                </if>
                <if test="age>0">
                    and age > #{age}
                </if>
            </where>
        </select>
        ```

   * test
        ```java
        @Test
        public void testSelectStudentWhere() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            Student student = new Student();
            student.setAge(12);
            student.setEmail("weasley@jk.com");
            List<Student> students = dao.selectStudentWhere(student);
            students.forEach(stu->System.out.println(stu));
            sqlSession.close();
        }
        ```

## 4.4 动态SQL — `foreach`
### 4.4.1 说明
1. foreach 标签  
   * `<foreach>`标签用于实现对于数组与集合的遍历
   * 语法：`<foreach collection=" " open=" " close="" item=" " separator=" ">`
2. 属性
   * collect：集合类型，array或list
   * open：要转换成sql语句的开始符号，如 `(`
   * cloae：结束符号
   * item：自定义，表示数组或集合的成员
   * separator：分隔符
3. 案例一：遍历简单类型的集合
   * mapper
        ```xml
        <select id="selectStudentForeachOne" resultType="com.powernode.domain.Student">
            select * from student where id in
            <foreach collection="list" item="id" open="(" close=")" separator=",">
                #{id}
            </foreach>
        </select>
        ```
   * test
        ```java
        @Test
        public void testSelectStudentForeachOne() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            List<Integer> idList =new ArrayList<>();
            idList.add(1001);
            idList.add(1002);
            idList.add(1003);
            List<Student> students = dao.selectStudentForeachOne(idList);
            students.forEach(stu->System.out.println(stu));
            sqlSession.close();
        }
        ```
4. 案例二：遍历对象集合
   * mapper
        ```xml
        <select id="selectStudentForeachTwo" resultType="com.powernode.domain.Student">
            select * from student where id in
            <foreach collection="list" item="stu" open="(" close=")" separator=",">
                #{stu.id}
            </foreach>
        </select>
        ```

   * test
        ```java
        @Test
        public void testSelectStudentForeachTwo() {
            SqlSession sqlSession = MyBatisUtils.getSqlSession();
            StudentDao dao = sqlSession.getMapper(StudentDao.class);
            List<Student> student = new ArrayList<>();
            Student stu = new Student();
            stu.setId(1001);
            student.add(stu);
            stu = new Student();
            stu.setId(1004);
            student.add(stu);
            List<Student> students = dao.selectStudentForeachTwo(student);
            students.forEach(s->System.out.println(s));
            sqlSession.close();
        }
        ```

## 4.5 动态SQL — 代码片段
### 4.5.1 说明
1. 定义  
   `<sql/>`标签用于定义 SQL 片段，以便其它 SQL 标签复用
2. 使用
   * 语法规则
     * 创建： `<sql id="">  </sql>`
     * 使用：`<include refid="sql标签的id"/>`
   * 使用 SQL 片段，需要使用`<include/>`标签
   * `<include>`子标签可以放在动态 SQL的任何位置

### 4.5.2 示例
1. mapper
   ```xml
   <!-- 创建 -->
    <sql id="studentSql">
        select * from student
    </sql>

    <!-- 使用 -->
    <select id="selectStudentForeachOne" resultType="com.powernode.domain.Student">
        <include refid="studentSql"/> where id in
        <foreach collection="list" item="id" open="(" close=")" separator=",">
            #{id}
        </foreach>
    </select>
   ```


