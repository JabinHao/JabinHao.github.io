---
title: Chapter6 Spring5 新特性
excerpt: 摘要
tags:
  - java
  - spring
categories:
  - Java笔记
  - Spring
banner_img: /img/dog.png
index_img: /img/post/spring-framework_logo.png
date: 2020-12-11 23:45:46
updated: 2020-12-11 23:45:46
subtitle:
---
## 6.1 简单的新功能
### 6.1.1 整合日志框架


### 6.1.2 Nullable注解



### 6.1.3 整合Junit单元测试框架
1. Junit4
    * 首先引入`Junit`依赖，导入：`import org.junit.Test`
    * 使用`@RunWith`注解引入单元测试框架
    * 使用`@ContextConfiguration`注解加载配置文件
    * 示例
        ```java
        @RunWith(SpringJUnit4ClassRunner.class)
        @ContextConfiguration("classpath:bean1.xml")
        public class JTest4 {

            @Autowired
            private UserService userService;

            @Test
            public void test(){
                //...
            }
        }
        ```
2. JUnit5
    * 引入JUnit5的jar包，导入：`import org.junit.jupiter.api.Test`
    * 使用@ExtendWith注解引入单元测试框架
    * 使用`@ContextConfiguration`注解加载配置文件
    * 可以使用`@SpringJUnitConfig`复合注解代替上面两个注解
    * 示例
        ```java
        @SpringJUnitConfig(location = "classpath:bean1.xml")
        public class JTest4 {

            @Autowired
            private UserService userService;

            @Test
            public void test(){
                //...
            }
        }
        ```

## 6.2 WebFlux











