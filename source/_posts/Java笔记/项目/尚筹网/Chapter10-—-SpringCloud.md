---
title: Chapter10 — SpringCloud
excerpt: SpringCloud环境搭建、Eureka注册中心、Feign、Hystrix、Zuul
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
abbrlink: 8c421584
date: 2021-01-15 22:03:13
updated: 2021-01-21 23:18:02
subtitle:
---
## 10.1 环境搭建
### 10.1.1 新建空的Maven工程作为父工程
1. 依赖管理
    ```xml
    <groupId>com.atguigu</groupId>
    <artifactId>my-spring-clound-parent</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>spring-cloud-common</module>
        <module>spring-cloud-provider</module>
        <module>spring-cloud-consumer</module>
    </modules>

    <properties>
        <maven.compiler.source>15</maven.compiler.source>
        <maven.compiler.target>15</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- 导入 SpringCloud 需要使用的依赖信息 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2020.0.0</version>
                <type>pom</type>
                <!-- import 依赖范围表示将 spring-cloud-dependencies 包中的依赖信息导入 -->
                <scope>import</scope>
            </dependency>
            <!-- 导入 SpringBoot 需要使用的依赖信息 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.4.1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

2. 需要install一下子工程才能使用spring-root:run
   * 右侧工具栏打开Maven
   * 父工程 -> Lifecycle -> package, install

### 10.1.2 新建子工程 common
1. 依赖
    ```xml
    <parent>
        <artifactId>my-spring-clound-parent</artifactId>
        <groupId>com.atguigu</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-common</artifactId>

    <properties>
        <maven.compiler.source>15</maven.compiler.source>
        <maven.compiler.target>15</maven.compiler.target>
    </properties>
    ```

2. 新建 Employee 类
    ```java
    package com.atguigu.spring.cloud.entity;

    public class Employee {

        private Integer empId;
        private String empName;
        private Double empSalary;

        // 构造器、getter、setter、tostring
    }
    ```

### 10.1.3 新建子工程 provider
1. 依赖
    ```xml
    <parent>
        <artifactId>my-spring-clound-parent</artifactId>
        <groupId>com.atguigu</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-provider</artifactId>

    <properties>
        <maven.compiler.source>15</maven.compiler.source>
        <maven.compiler.target>15</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.atguigu</groupId>
            <artifactId>spring-cloud-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```
2. 主启动类
    ```java
    @SpringBootApplication
    public class AtguiguMainClass {

        public static void main(String[] args) {
            SpringApplication.run(AtguiguMainClass.class,args);
        }
    }
    ```
3. handler
    ```java
    package com.atguigu.spring.cloud.handler;

    import com.atguigu.spring.cloud.entity.Employee;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    @RestController
    public class EmployeeHandler {

        @RequestMapping("/provider/get/employee/remote")
        public Employee getEmployeeRemote() {
            return new Employee(555, "tom555", 555.55);
        }
    }
    ```
4. 配置文件 application.yml
    ```yml
    server:
    port: 1000
    ```

### 10.1.4 新建子工程 consumer
1. 依赖
    ```xml
    <parent>
        <artifactId>my-spring-clound-parent</artifactId>
        <groupId>com.atguigu</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-consumer</artifactId>

    <properties>
        <maven.compiler.source>15</maven.compiler.source>
        <maven.compiler.target>15</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.atguigu</groupId>
            <artifactId>spring-cloud-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```




















2. 主启动类：同上
3. handler
    ```java
    @RestController
    public class HumanResourceHandler {
        @Autowired
        private RestTemplate restTemplate;

        @RequestMapping("/consumer/get/employee")
        public Employee getEmployeeRemote() {

            // 1.声明远程微服务主机地址和端口号
            String host = "http://localhost:1000";
            // 2.声明具体要调用的功能的 URL 地址
            String url = "/provider/get/employee/remote";

            // 3. 通过RestTemplate调用远程微服务
            return restTemplate.getForObject(host + url, Employee.class);
        }
    }
    ```
4. 配置类
    ```java
    package com.atguigu.spring.cloud.config;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.client.RestTemplate;

    @Configuration
    public class SpringCloudConfig {

        @Bean
        public RestTemplate getRestTemplate() {
            return new RestTemplate();
        }
    }
    ```

5. 配置文件 application.yml
    ```yml
    server:
    port: 4000
    ```

## 10.2 创建 Eureka 注册中心
### 10.2.1 新建子工程eureka
1. 依赖
    ```xml
    <parent>
        <artifactId>my-spring-clound-parent</artifactId>
        <groupId>com.atguigu</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>pro04-spring-cloud-Eureka</artifactId>

    <properties>
        <maven.compiler.source>15</maven.compiler.source>
        <maven.compiler.target>15</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

</project>
    ```
2. 主启动类
    ```java
    package com.atguigu.spring.cloud;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

    @EnableEurekaServer //启用Eureka服务端
    @SpringBootApplication
    public class AtguiguMainType {
        public static void main(String[] args) {
            SpringApplication.run(AtguiguMainType.class, args);
        }
    }
    ```
3. 配置
    ```yml
    server:
    port: 5000
    eureka:
    instance:
        hostname: localhost
    client:
        register-with-eureka: false   # 自己就是注册中心， 所以自己不注册自己
        fetch-registry: false         # 自己就是注册中心， 所以不需要“从注册中心取回信息”
        service-url:                  # 客户端访问 Eureka 时使用的地址
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    ```

### 10.2.2 将 provider 注册到 Eureka
1. provider中加入依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ```
2. 配置
    ```yml
    server:
    port: 1000
    eureka:
    client:
        service-url:
        defaultZone: http://localhost:5000/eureka/
    ```
3. 设置微服务名称（还是provider的配置文件）
    ```yml
    server:
    port: 1000
    eureka:
    client:
        service-url:
        defaultZone: http://localhost:5000/eureka/
    spring:
    application:
        name: atguigu-provider
    ```

### 10.2.3 consumer 访问 provider 时使用微服务名
1. consumer模块加入依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        <version>2.2.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ```
2. 修改 handler
    ```java
    // 1.声明远程微服务主机地址和端口号
    // String host = "http://localhost:1000";

    // 将远程微服务调用地址从ip地址+端口号改为微服务名
    String host = "http://ATGUIGU-PROVIDER";
    ```
3. 配置文件
    ```yml
    server:
    port: 4000
    spring:
    application:
        name: atguigu-consumer
    eureka:
    client:
        service-url:
        defaultZone: http://localhost:5000/eureka/
    ```
4. 修改配置类
    ```java
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
    ```
5. 报错
   * 启动后报错了：`No instances available for ATGUIGU-PROVIDER] with root cause`
   * 解决：consumer模块的pom中删除ribbon的依赖即可

### 10.2.4 使用 Feign 实现远程方法声明式调用
1. Feign 使用思路
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/10-2-4.png)
2. common 工程引入依赖
    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>
    ```
3. 新建接口 EmployeeRemoteService
    ```java
    // com/atguigu/spring/cloud/api/EmployeeRemoteService.java
    @FeignClient(value = "atguigu-provider")
    public interface EmployeeRemoteService {

        // 远程调用的接口方法
        // 要求@RequestMapping注解映射的地址一致
        // 方法声明一致
        @RequestMapping("/provider/get/employee/remote")
        public Employee getEmployeeRemote();
    }
    ```
4. 新建 module：pro06-spring-cloud-feign-consumer
    ```xml
    <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.atguigu</groupId>
        <artifactId>spring-cloud-common</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
    <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    </dependencies>
    ```
5. 主启动类
    ```java
    @EnableFeignClients
    @SpringBootApplication
    public class FeignMain {

        public static void main(String[] args) {
            SpringApplication.run(FeignMain.class,args);
        }

    }
    ```
6. handler
    ```java
    @RestController
    public class FeignHumanResourceHandler {

        @Autowired
        private EmployeeRemoteService employeeRemoteService;

        @RequestMapping("/feign/consumer/get/emp")
        public Employee getEmployeeRemote() {
            return employeeRemoteService.getEmployeeRemote();
        }
    }
    ```
7. 配置文件
    ```yml
    server:
    port: 7000

    spring:
    application:
        name: atguigu-feign-consumer

    eureka:
    client:
        service-url:
        defaultZone: http://localhost:5000/eureka/
    ```

## 10.3 Hystrix
### 10.3.1 服务熔断机制
1. provider模块添加依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        <version>2.2.6.RELEASE</version>
    </dependency>
    ```

2. 主启动类添加注解
    ```java
    @EnableCircuitBreaker
    @EnableEurekaClient
    @SpringBootApplication
    public class AtguiguMainType {

        public static void main(String[] args) {
            SpringApplication.run(AtguiguMainType.class,args);
        }
    }
    ```
3. common中新建类
    ```java
    package com.atguigu.spring.cloud.util;

    public class ResultEntity<T> {

        public static final String SUCCESS = "SUCCESS";
        public static final String FAILED = "FAILED";
        public static final String NO_MESSAGE = "NO_MESSAGE";
        public static final String NO_DATA = "NO_DATA";
        /**
        * 操作成功， 不需要返回数据
        * @return
        */
        public static ResultEntity<String> successWithoutData() {
            return new ResultEntity<String>(SUCCESS, NO_MESSAGE, NO_DATA);
        }
        /**
        * 操作成功， 需要返回数据
        * @param data
        * @return
        */
        public static <E> ResultEntity<E> successWithData(E data) {
            return new ResultEntity<>(SUCCESS, NO_MESSAGE, data);
        }
        /**
        * 操作失败， 返回错误消息
        * @param message
        * @return
        */
        public static <E> ResultEntity<E> failed(String message) {
            return new ResultEntity<>(FAILED, message, null);
        }
        private String result;
        private String message;
        private T data;
        public ResultEntity() {
        }
        public ResultEntity(String result, String message, T data) {

            super();
            this.result = result;
            this.message = message;
            this.data = data;
        } @
                Override
        public String toString() {
            return "ResultEntity [result=" + result + ", message=" + message + ", data=" + data + "]";
        }
        public String getResult() {
            return result;
        }
        public void setResult(String result) {
            this.result = result;
        }
        public String getMessage() {
            return message;
        }
        public void setMessage(String message) {
            this.message = message;
        }
        public T getData() {
            return data;
        }
        public void setData(T data) {
            this.data = data;
        }
    }
    ```
4. provider中handler
    ```java
    // 指定当前方法出问题时调用的备份方法
    @HystrixCommand(fallbackMethod = "getEmpWithCircuitBreakerBackup")
    @RequestMapping("/provider/get/emp/with/circuit/breaker")
    public ResultEntity<Employee> getEmpWithCircuitBreaker(@RequestParam("signal") String signal) throws InterruptedException {

        if ("quick-bang".equals(signal)) {
            throw new RuntimeException();
        }
        if ("slow-bang".equals(signal)) {
            TimeUnit.MILLISECONDS.sleep(5000);
        }
        return ResultEntity.successWithData(new Employee(666,"empName666", 666.66));
    }

    public ResultEntity<Employee> getEmpWithCircuitBreakerBackup(@RequestParam("signal") String signal) {
        return ResultEntity.failed("circuit break workded,with signal="+signal);
    }
    ```

### 10.3.2 服务降级机制
1. common中添加依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    ```
2. common中新建 MyFallbackFactory 类
    ```java
    package com.atguigu.spring.cloud.factory;

    /**
    * 实现Consumer端服务降级功能
    * 实现 FallbackFactory 接口时要传入 @FeignClient 注解标记的接口类型
    * 在 create 方法中返回 @FeignClient 注解标记的接口类型对象，当Provider调用失败后。会执行该对象对应方法
    */
    @Component
    public class MyFallbackFactory implements FallbackFactory<EmployeeRemoteService> {

        @Override
        public EmployeeRemoteService create(Throwable cause) {
            return new EmployeeRemoteService() {
                @Override
                public Employee getEmployeeRemote() {
                    return null;
                }

                @Override
                public List<Employee> getEmpListRemote(String keyword) {
                    return null;
                }

                @Override
                public ResultEntity<Employee> getEmpWithCircuitBreaker(String signal) {
                    return ResultEntity.failed(cause.getMessage());
                }
            };
        }
    }
    ```
3. 修改EmployeeRemoteService接口
    ```java
    /**
    * FeignClient 注解表示当前接口和一个Provider对应
    *      value指定要调用的Provider微服务名称
    *      fallbackFactory指定Provider不可用时提供备用方案的工厂类型
    */
    @FeignClient(value = "atguigu-provider", fallbackFactory = MyFallbackFactory.class)
    public interface EmployeeRemoteService {
        // ...
    ```
4. feign-consumer配置文件
   * springcloud 2020前的版本：
        ```yml
        feign:
        hystrix:
            enabled: true
        ```
   * 2020版本
        ```yml
        feign:
        circuitbreaker:
            enabled: true
        ```

5. feign-consumer模块handler
    ```java
    @RequestMapping("/feign/consumer/test/fallback")
    public ResultEntity<Employee> testFallBack(@RequestParam("signal") String signal) {
        return employeeRemoteService.getEmpWithCircuitBreaker(signal);
    }
    ```

6. 其它问题：
   * hystrix 已停更
   * 2020版本的SpringCloud中剔除了hystrix依赖，需要自己在父工程中指定依赖版本（最新为2.2.6.RELEASE）

### 10.3.3 监控
1. provider工程添加依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
2. provider工程配置文件
    ```yml
    management:
    endpoints:
        web:
        exposure:
            include: hystrix.stream
    ```
3. 新建module：spring-cloud-dashboard
    ```java
    @EnableHystrixDashboard
    @SpringBootApplication
    public class DashboardMain {

        public static void main(String[] args) {
            SpringApplication.run(DashboardMain.class, args);
        }
    }
    ```
4. dashboard工程依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        <version>2.2.6.RELEASE</version>
    </dependency>
    ````
5. dashboard工程配置文件
    ```yml
    server:
    port: 8000

    spring:
    application:
        name: atguigu-dashboard

    hystrix:
    dashboard:
        proxy-stream-allow-list: "*"
    ```

## 10.4 Zuul网关
### 10.4.1 








