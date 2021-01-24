---
title: Chapter11 — 前台环境搭建
excerpt: 前台基础环境搭建：SpringBoot + SpringCloud，Eureak、Zuul、openfeign、MySQL、Redis
tags:
  - java
  - ssm
categories:
  - Java笔记
  - 项目
  - 尚筹网
banner_img: /img/post/banner/mandao.png
index_img: /img/post/ssm.png
category: Java笔记/项目/尚筹网
abbrlink: 4f869e7
date: 2021-01-22 02:31:04
updated: 2021-01-23 03:46:07
subtitle:
---
## 11.1 技术路线
### 11.1.1 架构
1. 尚硅谷官方架构 

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/11-1-1.png)

2. 技术更迭
   * 2020年后，SpringCloud 发生了重大改变
   * SpringCloud 2020.0.0 版本中已经移除了之前的很多模块

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/11-1-1_2.png)  

3. 个人改动
   * 尽量跟项目中的一致
   * 由于使用的springboot、springcloud版本较新，部分模块配置可能有所不同

### 11.1.2 Moudles
1. 父工程atcrowdfunding07-member-parent，仅用于版本控制，使用SpringBoot 2.3.8.RELEASE + SpringCloud Hoxton.SR9
2. 子工程
   * 注册中心： atcrowdfunding08-member-eureka
   * 实体类模块： atcrowdfunding09-member-entity
   * MySQL 数据服务： atcrowdfunding10-member-mysql-provider
   * Redis 数据服务： atcrowdfunding11-member-redis-provider
   * 会员中心： atcrowdfunding12-member-authentication-consumer
   * 项目维护： atcrowdfunding13-member-project-consumer
   * 订单维护： atcrowdfunding14-member-order-consumer
   * 支付功能： atcrowdfunding15-member-pay-consumer
   * 网关： atcrowdfunding16-member-zuul
   * API 模块： atcrowdfunding17-member-api

## 11.2 基础环境搭建
### 11.2.1 新建工程
1. 根据上面列出的Modules新建模块
2. atcrowdfunding07-member-parent为父模块，剩下的均继承自该模块
3. 父工程依赖
    ```xml
    <groupId>com.atguigu.crowd</groupId>
    <artifactId>atcrowdfunding07-member-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>atcrowdfunding08-member-eureka</module>
        <module>atcrowdfunding09-member-entity</module>
        <module>atcrowdfunding10-member-mysql-provider</module>
        <module>atcrowdfunding11-member-redis-provider</module>
        <module>atcrowdfunding12-member-authentication-consumer</module>
        <module>atcrowdfunding13-member-project-consumer</module>
        <module>atcrowdfunding14-member-order-consumer</module>
        <module>atcrowdfunding15-member-pay-consumer</module>
        <module>atcrowdfunding16-member-zuul</module>
        <module>atcrowdfunding17-member-api</module>
    </modules>
    <packaging>pom</packaging>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- SpringCloud需要使用的依赖 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR9</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- SpringBoot需要使用的依赖 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.8.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- MyBatis相关依赖 -->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.1.3</version>
            </dependency>
            <!-- 德鲁伊连接池 -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.2.4</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

### 11.2.2 搭建环境约定
1. 包名：新创建的包都作为 com.atguigu.crowd 的子包
2. 主启动类类名：模块名后半部分+Main，如：EurekaMain
3. 端口号
   * atcrowdfunding08-member-eureka 1000
   * atcrowdfunding10-member-mysql-provider 2000
   * atcrowdfunding11-member-redis-provider 3000
   * atcrowdfunding12-member-authentication-consumer 4000
   * atcrowdfunding13-member-project-consumer 5000
   * atcrowdfunding14-member-order-consumer 7000
   * atcrowdfunding15-member-pay-consumer 8000
   * atcrowdfunding16-member-zuul 80

### 11.2.3 eureka 工程
1. 依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    ```

2. 主启动类
    ```java
    @EnableEurekaServer
    @SpringBootApplication
    public class EurekaMain {

        public static void main(String[] args) {
            SpringApplication.run(EurekaMain.class, args);
        }
    }
    ```
3. 配置文件 application.yml
    ```yml
    spring:
    application:
        name: atguigu-crowd-eureka

    eureka:
    instance:
        hostname: localhost
    client:
        register-with-eureka: false
        fetch-registry: false
        service-url:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
    ```

### 11.2.4 entity 工程
1. 结构
   * VO：View Object 视图对象
     * 用途 1： 接收浏览器发送过来的数据
     * 用途 2： 把数据发送给浏览器去显示
   * PO：Persistent Object 持久化对象
     * 用途 1： 将数据封装到 PO 对象存入数据库  
     * 用途 2： 将数据库数据查询出来存入 PO 对象
     * PO 对象是和数据库表对应， 一个数据库表对应一个 PO 对象
   * DO：Data Object 数据对象
     * 用途 1： 从 Redis 查询得到数据封装为 DO 对象
     * 用途 2： 从 ElasticSearch 查询得到数据封装为 DO 对象
     * 用途 3： 从 Solr 查询得到数据封装为 DO 对象
     * ……
     * 从中间件或其他第三方接口查询到的数据封装为 DO 对象
   * DTO：Data Transfer Object 数据传输对象
     * 用途 1： 从 Consumer 发送数据到 Provider
     * 用途 2： Provider 返回数据给 Consumer
2. 交互
   
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/11-2-4.png)
   * 使用 org.springframework.beans.BeanUtils.copyProperties(Object, Object)在不同实体类之间复制属性
   * 例如：MemberVO -> 复制属性→MemberPO

3. 创建包
* com.atguigu.crowd.entity.po
* com.atguigu.crowd.entity.vo

4. lombok
   * 使用前需要加入依赖，安装 lombok 插件
   * 作用：让我们在开发时不必编写 getXxx()、 setXxx()、 有参构造器、 无参构造器等等这样具备固定模式的代码
   * 原理：根据注解确定要生成的代码， 然后将要生成的代码侵入到字节码文件中

        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/11-2-4_2.png)
   * 注解
      * @Data： 每一个字段都加入 getXxx()、 setXxx()方法
      * @NoArgsConstructor： 无参构造器
      * @AllArgsConstructor： 全部字段都包括的构造器
      * @EqualsAndHashCode： equals 和 hashCode 方法
      * @Getter
        * 加到类名上： 所有字段都加入 getXxx()方法
        * 加到字段上： 当前字段加入 getXxx()方法
     * @Setter
       * 类： 所有字段都加入 setXxx()方法
       * 字段： 当前字段加入 setXxx()方法
     * @Value：自动生成全参构造函数、Getter方法、equals方法、hashCode法、toString方法
     * @Slf4j：等同于以下语句，即可以直接在类中使用 `log.info()` 等
        ```java
        private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
        ```

### 11.2.5 MySQL 工程基础环境
1. 依赖
    ```xml
    <dependencies>
        <!-- MyBatis相关依赖 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!-- 德鲁伊连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>com.atguigu.crowd</groupId>
            <artifactId>atcrowdfunding09-member-entity</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.atguigu.crowd</groupId>
            <artifactId>atcrowdfunding05-common-util</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    ```
2. 主启动类
    ```java
    @MapperScan("com.atguigu.crowd.mapper")
    @SpringBootApplication
    public class MysqlProviderMain {

        public static void main(String[] args) {
            SpringApplication.run(MysqlProviderMain.class, args);
        }
    }
    ```
3. 配置文件
    ```yml
    server:
    port: 2000

    spring:
    application:
        name: atguigu-crowd-mysql
    datasource:
        name: mydb
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://127.0.0.1:3306/project_crowd?serverTimezone=UTC
        username: Jacob
        password: jacob12015229
        driver-class-name: com.mysql.cj.jdbc.Driver

    eureka:
    client:
        service-url:
        defaultZone: http://localhost:1000/eureka

    mybatis:
    mapper-locations: classpath*:/mybatis/mapper/*Mapper.xml

    logging:
    level:
        com.atguigu.crowd.mapper: debug
        com.atguigu.crowd.test: debug
    ```
4. 测试类：test目录下新建 com/atguigu/crowd/test/MybatisTest.java
    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class MybatisTest {

        @Autowired
        private DataSource dataSource;

        @Autowired
        private MemberPOMapper memberPOMapper;

        @Test
        public void testMapper() {

            BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
            String source = "123123";
            String encode = passwordEncoder.encode(source);

            MemberPO memberPO = new MemberPO(null, "jack", encode, "杰克", "jack@qq.com", 1, 1, "杰克", "123123", 2);

            memberPOMapper.insert(memberPO);
        }

        @Test
        public void testConnection() throws SQLException {

            Connection connection = dataSource.getConnection();

            log.debug(connection.toString());
        }
    }
    ```
5. 建表
    ```mysql
    CREATE TABLE t_member(
        id              INT(11) NOT NULL AUTO_INCREMENT,
        loginacct       VARCHAR(255) NOT NULL,
        userpswd        CHAR(200) NOT NULL ,
        username        VARCHAR(255),
        email           VARCHAR(255),
        authstatus      INT(4) COMMENT '实名认证状态 0 - 未实名认证，1 - 实名认证申请中，2 - 已实名认证',
        usertype        INT(4) COMMENT '0 - 个人，1 - 企业',
        realname        VARCHAR(255),
        cardnum         VARCHAR(255),
        accttype        INT(4) COMMENT '0 - 企业，1 - 个体，2 - 个人，3 - 政府',
        PRIMARY KEY (id)
    );
    ```
6. 逆向工程
   * 修改 reverse 工程的 generatorConfig.xml
        ```xml
        <!--  数据库表名字和我们的 entity 类对应的映射指定 -->
        <table tableName="t_member" domainObjectName="MemberPO" />
        ```
   * atcrowdfunding06-common-reverse -> Plugins -> mybatis-generator -> mybatis-generator:generate
7. 资源归位
   * MemberPO、MemberPOExample 移动到 entity 模块 po 包下
   * MemberPOMapper 类移动到 provider 模块 mapper 包下
   * mapper 文件移动到provider模块 resources/mybatis/mapper 目录下

### 11.2.6 MySQL 工程对外暴露服务
1. 思路  
   
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/11-2-6.png)
2. api工程添加依赖
    ```xml
    <dependency>
        <groupId>com.atguigu.crowd</groupId>
        <artifactId>atcrowdfunding05-common-util</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>com.atguigu.crowd</groupId>
        <artifactId>atcrowdfunding09-member-entity</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    ```
3. api 工程新建接口
    ```java
    package com.atguigu.crowd.api;

    import com.atguigu.crowd.entity.po.MemberPO;
    import com.atguigu.crowd.util.ResultEntity;
    import org.springframework.cloud.openfeign.FeignClient;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestParam;

    @FeignClient("atguigu-crowd-mysql")
    public interface MySQLRemoteService {

        @RequestMapping("get/memberpo/by/login/acct/remote")
        ResultEntity<MemberPO> getMemberPOByLoginAcctRemote(@RequestParam("loginacct") String loginacct);
    }
    ```
4. 新建三个类（接口）
   
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/11-2-6_2.png)
5. MemberService
    ```java
    public interface MemberService {
        MemberPO getMemberPOByLoginAcct(String loginacct);
    }
    ```
6. MemberServiceImpl
    ```java
    @Transactional(readOnly = true)
    @Service
    public class MemberServiceImpl implements MemberService {

        @Autowired
        private MemberPOMapper memberPOMapper;

        @Override
        public MemberPO getMemberPOByLoginAcct(String loginacct) {

            MemberPOExample example = new MemberPOExample();

            MemberPOExample.Criteria criteria = example.createCriteria();

            criteria.andLoginacctEqualTo(loginacct);

            List<MemberPO> list = memberPOMapper.selectByExample(example);

            return list.get(0);
        }
    }
    ```
7. MemberProviderHandler
    ```java
    @RestController
    public class MemberProviderHandler {

        @Autowired
        private MemberService memberService;

        @RequestMapping("/get/memberpo/by/login/acct/remote")
        ResultEntity<MemberPO> getMemberPOByLoginAcctRemote(@RequestParam("loginacct") String loginacct){

            MemberPO memberPO = null;
            try {
                // 1.调用本地Service完成查询
                memberPO = memberService.getMemberPOByLoginAcct(loginacct);

                // 2.成功则返回结果
                return ResultEntity.successWithData(memberPO);
            } catch (Exception e) {
                e.printStackTrace();

                // 3. 异常则返回异常信息
                return ResultEntity.failed(e.getMessage());
            }
        }
    }
    ```

### 11.2.7 Redis 工程基础环境
1. 依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>com.atguigu.crowd</groupId>
        <artifactId>atcrowdfunding09-member-entity</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>com.atguigu.crowd</groupId>
        <artifactId>atcrowdfunding05-common-util</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    ```
2. 主启动类
    ```java
    @SpringBootApplication
    public class RedisProviderMain {

        public static void main(String[] args) {
            
            SpringApplication.run(RedisProviderMain.class, args);
            
        }
    }
    ```
3. 测试类
    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    @Slf4j
    public class RedisTest {

        @Autowired
        private StringRedisTemplate redisTemplate;

        @Test
        public void testSet() {

            ValueOperations<String, String> operations = redisTemplate.opsForValue();

            operations.set("apple", "red");
        }

    }
    ```
4. 配置文件
    ```yml
    server:
    port: 3000

    spring:
    application:
        name: atguigu-crowd-redis
    redis:
        host: 127.0.0.1

    eureka:
    client:
        service-url:
        defaultZone: http://localhost:1000/eureka
    ```

### 11.2.8 Redis 工程对外暴露服务
1. api模块新建接口
    ```java
    @FeignClient("atguigu-crowd-redis")
    public interface RedisRemoteService {

        @RequestMapping("/set/redis/key/value/remote")
        ResultEntity<String> setRedisKeyValueRemote(@RequestParam("key") String key,
                                                    @RequestParam("value") String value);

        @RequestMapping("/set/redis/key/value/remote/with/timeout")
        ResultEntity<String> setRedisKeyValueRemoteWithTimeout(@RequestParam("key") String key,
                                                            @RequestParam("value") String value,
                                                            @RequestParam("time") long time,
                                                            @RequestParam("timeUnix")TimeUnit timeUnit);

        @RequestMapping("/get/redis/string/value/by/key")
        ResultEntity<String> getRedisStringValueByKey(@RequestParam("key") String key);

        @RequestMapping("/remove/redis/key/remote")
        ResultEntity<String> removeRedisKeyRemote(@RequestParam("key") String key);

    }
    ```

2. reids 模块handler
    ```java
    @RestController
    public class RedisHandler {

        @Autowired
        private StringRedisTemplate redisTemplate;

        @RequestMapping("/set/redis/key/value/remote")
        ResultEntity<String> setRedisKeyValueRemote(@RequestParam("key") String key,
                                                    @RequestParam("value") String value){

            try {
                ValueOperations<String, String> operations = redisTemplate.opsForValue();
                operations.set(key, value);

                return ResultEntity.successWithoutData();
            } catch (Exception e) {
                e.printStackTrace();

                return ResultEntity.failed(e.getMessage());
            }
        }

        @RequestMapping("/set/redis/key/value/remote/with/timeout")
        ResultEntity<String> setRedisKeyValueRemoteWithTimeout(@RequestParam("key") String key,
                                                            @RequestParam("value") String value,
                                                            @RequestParam("time") long time,
                                                            @RequestParam("timeUnit") TimeUnit timeUnit){

            try {
                ValueOperations<String, String> operations = redisTemplate.opsForValue();
                operations.set(key, value, time, timeUnit);

                return ResultEntity.successWithoutData();
            } catch (Exception e) {
                e.printStackTrace();

                return ResultEntity.failed(e.getMessage());
            }

        }

        @RequestMapping("/get/redis/string/value/by/key")
        ResultEntity<String> getRedisStringValueByKey(@RequestParam("key") String key){

            try {
                ValueOperations<String, String> operations = redisTemplate.opsForValue();
                String value = operations.get(key);

                return ResultEntity.successWithData(value);
            } catch (Exception e) {
                e.printStackTrace();

                return ResultEntity.failed(e.getMessage());
            }
        }

        @RequestMapping("/remove/redis/key/remote")
        ResultEntity<String> removeRedisKeyRemote(@RequestParam("key") String key){

            try {
                redisTemplate.delete(key);

                return ResultEntity.successWithoutData();

            } catch (Exception e) {
                e.printStackTrace();

                return ResultEntity.failed(e.getMessage());
            }

        }

    }
    ```

### 11.2.9 认证工程显示首页
1. 依赖
    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
        <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.atguigu.crowd</groupId>
            <artifactId>atcrowdfunding17-member-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    ```
2. 主启动类
    ```java
    @SpringBootApplication
    @EnableEurekaClient
    public class AuthenticationConsumerMain {

        public static void main(String[] args) {
            
            SpringApplication.run(AuthenticationConsumerMain.class, args);
        }
    }
    ```
3. 配置文件
    ```yml
    server:
    port: 4000

    spring:
    application:
        name: atguigu-crowd-auth
    thymeleaf:
        prefix: classpath:/templates/
        suffix: .html

    eureka:
    client:
        service-url:
        defaultZone: http://localhost:1000/eureka
    ```
4. resources目录下新建文件夹
   * static
   * templates
5. handler
    ```java
    @Controller
    public class PortalHandler {

        @RequestMapping("/")
        public String showPortalPage() {

            // 节省时间，省略加载数据部分

            return "portal";
        }
    }
    ```
6. 加入静态资源：复制到static目录下
7. 新建portal.html 页面（templates目录下）
    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UFT-8">
        <meta content="IE=edge" http-equiv="X-UA-Compatible">
        <meta content="width=device-width, initial-scale=1" name="viewport">
        <meta content="" name="description">
        <meta content="" name="author">
        <base th:href="@{/}"/>
        <link href="bootstrap/css/bootstrap.min.css" rel="stylesheet">
        <link href="css/font-awesome.min.css" rel="stylesheet">
        <link href="css/carousel.css" rel="stylesheet">
        <style>
            h3 {
                font-weight: bold;
            }

            #footer {
                padding: 15px 0;
                background: #fff;
                border-top: 1px solid #ddd;
                text-align: center;
            }

            #topcontrol {
                color: #fff;
                z-index: 99;
                width: 30px;
                height: 30px;
                font-size: 20px;
                background: #222;
                position: relative;
                right: 14px !important;
                bottom: 11px !important;
                border-radius: 3px !important;
            }

            #topcontrol:after {
                /*top: -2px;*/
                left: 8.5px;
                content: "\f106";
                position: absolute;
                text-align: center;
                font-family: FontAwesome;
            }

            #topcontrol:hover {
                color: #fff;
                background: #18ba9b;
                -webkit-transition: all 0.3s ease-in-out;
                -moz-transition: all 0.3s ease-in-out;
                -o-transition: all 0.3s ease-in-out;
                transition: all 0.3s ease-in-out;
            }

            /* 侧栏导航 */
            .sideBox {
                padding: 10px;
                height: 220px;
                background: #fff;
                margin-bottom: 10px;
                overflow: hidden;
            }

            .sideBox .hd {
                height: 30px;
                line-height: 30px;
                background: #f60;
                padding: 0 10px;
                text-align: center;
                overflow: hidden;
            }

            .sideBox .hd .more {
                color: #fff;
            }

            .sideBox .hd h3 span {
                font-weight: bold;
                font-size: 14px;
                color: #fff;
            }

            .sideBox .bd {
                padding: 5px 0 0;
            }

            #sideMenu .bd li {
                margin-bottom: 2px;
                height: 30px;
                line-height: 30px;
                text-align: center;
                overflow: hidden;
            }

            #sideMenu .bd li a {
                display: block;
                background: #EAE6DD;
            }

            #sideMenu .bd li a:hover {
                background: #D5CFBF;
            }

            /* 列表页 */
            #mainBox {
                margin-bottom: 10px;
                padding: 10px;
                background: #fff;
                overflow: hidden;
            }

            #mainBox .mHd {
                border-bottom: 2px solid #09c;
                height: 30px;
                line-height: 30px;
            }

            #mainBox .mHd h3 {
                display: initial;
                *display: inline;
                zoom: 1;
                padding: 0 15px;
                background: #09c;
                color: #fff;
            }

            #mainBox .mHd h3 span {
                color: #fff;
                font-size: 14px;
                font-weight: bold;
            }

            #mainBox .path {
                float: right;
            }

            /* 位置导航 */
            .path {
                height: 30px;
                line-height: 30px;
                padding-left: 10px;
            }

            .path a, .path span {
                margin: 0 5px;
            }

            /* 文章列表 */
            .newsList {
                padding: 10px;
                text-align: left;
            }

            .newsList li {
                background: url("../images/share/point.png") no-repeat 2px 14px;
                padding-left: 10px;
                height: 30px;
                line-height: 30px;
            }

            .newsList li a {
                display: inline-block;
                *display: inline;
                zoom: 1;
                font-size: 14px;
            }

            .newsList li .date {
                float: right;
                color: #999;
            }

            .newsList li.split {
                margin-bottom: 10px;
                padding-bottom: 10px;
                border-bottom: 1px dotted #ddd;
                height: 0px;
                line-height: 0px;
                overflow: hidden;
            }

            /* 通用带图片的信息列表_普通式 */
            .picList {
                padding: 10px;
                text-align: left;
            }

            .picList li {
                margin: 0 5px;
                height: 190px;
            }

            h3.break {
                font-size: 16px;
                display: block;
                white-space: nowrap;
                word-wrap: normal;
                overflow: hidden;
                text-overflow: ellipsis;
            }

            h3.break > a {
                text-decoration: none;
            }

        </style>
    </head>
    <body>
    <div class="navbar-wrapper">
        <div class="container">
            <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
                <div class="container">
                    <div class="navbar-header">
                        <a class="navbar-brand" href="index.html" style="font-size:32px;">尚筹网-创意产品众筹平台</a>
                    </div>
                    <div class="navbar-collapse collapse" id="navbar" style="float:right;">
                        <ul class="nav navbar-nav navbar-right">
                            <li><a href="login.html">登录</a></li>
                            <li><a href="reg.html">注册</a></li>
                            <li><a>|</a></li>
                            <li><a href="admin-login.html">管理员入口</a></li>
                        </ul>
                    </div>
                </div>
            </nav>

        </div>
    </div>


    <!-- Carousel
    ================================================== -->
    <div class="carousel slide" data-ride="carousel" id="myCarousel">
        <!-- Indicators -->
        <ol class="carousel-indicators">
            <li class="" data-slide-to="0" data-target="#myCarousel"></li>
            <li class="active" data-slide-to="1" data-target="#myCarousel"></li>
            <li class="" data-slide-to="2" data-target="#myCarousel"></li>
        </ol>
        <div class="carousel-inner" role="listbox">
            <div class="item" onclick="window.location.href='project.html'" style="cursor:pointer;">
                <img alt="First slide" src="img/carousel-1.jpg">
            </div>
            <div class="item active" onclick="window.location.href='project.html'" style="cursor:pointer;">
                <img alt="Second slide" src="img/carousel-2.jpg">
            </div>
            <div class="item" onclick="window.location.href='project.html'" style="cursor:pointer;">
                <img alt="Third slide" src="img/carousel-3.jpg">
            </div>
        </div>
        <a class="left carousel-control" data-slide="prev" href="#myCarousel" role="button">
            <span class="glyphicon glyphicon-chevron-left"></span>
            <span class="sr-only">Previous</span>
        </a>
        <a class="right carousel-control" data-slide="next" href="#myCarousel" role="button">
            <span class="glyphicon glyphicon-chevron-right"></span>
            <span class="sr-only">Next</span>
        </a>
    </div><!-- /.carousel -->


    <!-- Marketing messaging and featurettes
    ================================================== -->
    <!-- Wrap the rest of the page in another container to center all the content. -->

    <div class="container marketing">

        <!-- Three columns of text below the carousel -->
        <div class="row">
            <div class="col-lg-4">
                <img alt="Generic placeholder image" class="img-circle" src="img/p1.jpg"
                    style="width: 140px; height: 140px;">
                <h2>智能高清监控机器人</h2>
                <p>可爱的造型，摄像安防远程互联的全能设计，让你随时随地守护您的家人，陪伴你的生活。</p>
                <p><a class="btn btn-default" href="project.html" role="button">项目详情 »</a></p>
            </div><!-- /.col-lg-4 -->
            <div class="col-lg-4">
                <img alt="Generic placeholder image" class="img-circle" src="img/p2.jpg"
                    style="width: 140px; height: 140px;">
                <h2>NEOKA智能手环</h2>
                <p>要运动更要安全，这款、名为“蝶舞”的NEOKA-V9100智能运动手环为“安全运动而生”。</p>
                <p><a class="btn btn-default" href="project.html" role="button">项目详情 »</a></p>
            </div><!-- /.col-lg-4 -->
            <div class="col-lg-4">
                <img alt="Generic placeholder image" class="img-circle" src="img/p3.png"
                    style="width: 140px; height: 140px;">
                <h2>驱蚊扣</h2>
                <p>随处使用的驱蚊纽扣，<br>解决夏季蚊虫问题。</p>
                <p><a class="btn btn-default" href="project.html" role="button">项目详情 »</a></p>
            </div><!-- /.col-lg-4 -->
        </div><!-- /.row -->

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="box ui-draggable" id="mainBox">
                        <div class="mHd" style="">
                            <div class="path">
                                <a href="projects.html">更多...</a>
                            </div>
                            <h3>
                                科技 <small style="color:#FFF;">开启智慧未来</small>
                            </h3>
                        </div>
                        <div class="mBd" style="padding-top:10px;">
                            <div class="row">
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-1.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">活性富氢净水直饮机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-2.gif" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">酷驰触控龙头，智享厨房黑科技</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-3.png" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">小熊猫鱼眼全景安防摄像机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-4.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">一款精致的机械表</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="box ui-draggable" id="mainBox">
                        <div class="mHd" style="">
                            <div class="path">
                                <a href="projects.html">更多...</a>
                            </div>
                            <h3>
                                设计 <small style="color:#FFF;">创意改变生活</small>
                            </h3>
                        </div>
                        <div class="mBd" style="padding-top:10px;">
                            <div class="row">
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-5.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">活性富氢净水直饮机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-6.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">酷驰触控龙头，智享厨房黑科技</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-7.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">小熊猫鱼眼全景安防摄像机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-8.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">一款精致的机械表</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="box ui-draggable" id="mainBox">
                        <div class="mHd" style="">
                            <div class="path">
                                <a href="projects.html">更多...</a>
                            </div>
                            <h3>
                                农业 <small style="color:#FFF;">网络天下肥美</small>
                            </h3>
                        </div>
                        <div class="mBd" style="padding-top:10px;">
                            <div class="row">
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-9.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">活性富氢净水直饮机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-2.gif" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">酷驰触控龙头，智享厨房黑科技</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-3.png" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">小熊猫鱼眼全景安防摄像机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-4.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">一款精致的机械表</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="box ui-draggable" id="mainBox">
                        <div class="mHd" style="">
                            <div class="path">
                                <a href="projects.html">更多...</a>
                            </div>
                            <h3>
                                其他 <small style="color:#FFF;">发现更多惊喜</small>
                            </h3>
                        </div>
                        <div class="mBd" style="padding-top:10px;">
                            <div class="row">
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-1.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">活性富氢净水直饮机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-2.gif" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">酷驰触控龙头，智享厨房黑科技</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-3.png" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">小熊猫鱼眼全景安防摄像机</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-3">
                                    <div class="thumbnail">
                                        <img alt="300x200" src="img/product-4.jpg" style="cursor: pointer;">
                                        <div class="caption">
                                            <h3 class="break">
                                                <a href="project.html">一款精致的机械表</a>
                                            </h3>
                                            <p>
                                            </p>
                                            <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                        title="目标金额"></i> $20,000
                                            </div>
                                            <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                        title="截至日期"></i>
                                                2017-20-20
                                            </div>
                                            <p></p>
                                            <br>
                                            <div class="progress" style="margin-bottom: 4px;">
                                                <div aria-valuemax="100" aria-valuemin="0"
                                                    aria-valuenow="40" class="progress-bar progress-bar-success" role="progressbar"
                                                    style="width: 40%">
                                                    <span>40% </span>
                                                </div>
                                            </div>
                                            <div><span style="float:right;"><i
                                                    class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                    class="glyphicon glyphicon-user" title="支持人数"></i> 12345</span></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>

        <!-- FOOTER -->
        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div id="footer">
                        <div class="footerNav">
                            <a href="http://www.atguigu.com" rel="nofollow">关于我们</a> | <a href="http://www.atguigu.com"
                                                                                        rel="nofollow">服务条款</a>
                            | <a href="http://www.atguigu.com" rel="nofollow">免责声明</a> | <a href="http://www.atguigu.com"
                                                                                            rel="nofollow">网站地图</a>
                            | <a href="http://www.atguigu.com" rel="nofollow">联系我们</a>
                        </div>
                        <div class="copyRight">
                            Copyright ?2017-2017atguigu.com 版权所有
                        </div>
                    </div>

                </div>
            </div>
        </div>

    </div><!-- /.container -->


    <script src="jquery/jquery-2.1.1.min.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
    <script src="script/docs.min.js"></script>
    <script src="script/back-to-top.js"></script>
    <script>
        $(".thumbnail img").css("cursor", "pointer");
        $(".thumbnail img").click(function () {
            window.location.href = "project.html";
        });
    </script>

    <div id="topcontrol" style="position: fixed; bottom: 5px; right: 5px; opacity: 0; cursor: pointer;" title=""></div>
    </body>
    </html>
    ```

### 11.2.10 网关
1. 依赖
    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
    </dependencies>
    ```
2. 主启动类
    ```java
    @SpringBootApplication
    @EnableZuulProxy
    public class ZuulMain {

        public static void main(String[] args) {
            
            SpringApplication.run(ZuulMain.class, args);
        }
    }
    ```
3. 配置文件
    ```yml
    server:
    port: 80

    spring:
    application:
        name: atguigu-crowd-zuul

    eureka:
    client:
        service-url:
        defaultZone: http://localhost:1000/eureka

    zuul:
    routes:
        crowd-portal:
        service-id: atguigu-crowd-auth
        path: /**
    ignored-services: "*"
    sensitive-headers: "*"
    ```
4. 端口映射（可选）
   * 打开host文件：C:\Windows\System32\drivers\etc
   * 加入以下内容
        ```
        127.0.0.1    www.crowd.com
        ```






