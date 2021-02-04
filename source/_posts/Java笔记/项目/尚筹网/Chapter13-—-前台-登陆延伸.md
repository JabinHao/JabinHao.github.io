---
title: Chapter13 — 前台 登陆延伸
excerpt: Spring Session解决Session共享问题、阿里云OSS、跳转到众筹页面
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
abbrlink: 6d9ec2ce
date: 2021-01-25 03:53:10
updated: 2021-01-26 03:34:02
subtitle:
---
## 13.1 Session共享问题

### 13.1.1 背景

1. 问题引入  
  
   ![Session共享问题](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/13-1-1.png)

2. 分布式环境
   * 在分布式和集群环境下， 每个具体模块运行在单独的 Tomcat 上，
   * Session 被不同Tomcat 所“区隔” ，不能互通
   * 程序运行时， 用户会话数据发生错误：有的服务器上有， 有的服务器上没有
3. 目标：  
    使用 Session 共享技术解决 Session 不互通问题。

### 13.1.2 会话控制

1. Cookie 的工作机制
   * 服务器端返回 Cookie 信息给浏览器
     * Java 代码： response.addCookie(cookie 对象);
       * HTTP 响应消息头： Set-Cookie: Cookie 的名字=Cookie 的值
     * 浏览器接收到服务器端返回的 Cookie， 以后的每一次请求都会把 Cookie 带上
     * HTTP 请求消息头： Cookie： Cookie 的名字=Cookie 的值
2. Session 的工作机制
   * 获取 Session 对象： request.getSession()
   * 检查当前请求是否携带了 JSESSIONID 这个 Cookie
     * 带了： 根据这个 JSESSIONID 在服务器端查找对应的 Session 对象
       * 能找到： 就把找到的 Session 对象返回
       * 没找到： 新建 Session 对象返回， 同时返回 JSESSIONID 的 Cookie
   * 没带： 新建 Session 对象返回， 同时返回 JSESSIONID 的 Cookie

### 13.1.3 解决方案

1. Session 同步  

   ![同步](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/13-3-3_1.png)  
   * 问题 1： 造成 Session 在各个服务器上“同量” 保存。数据量太大的会导致 Tomcat 性能下降。
   * 问题 2： 数据同步对性能有一定影响
2. 将 Session 数据存储在 Cookie 中
   * 做法： 所有会话数据在浏览器端使用 Cookie 保存， 服务器端不存储任何会话数据。
   * 优点： 服务器端大大减轻了数据存储的压力。 不会有 Session 不一致问题
   * 缺点：
     * Cookie 能够存储的数据非常有限。 一般是 4KB。 不能存储丰富的数据。
     * Cookie 数据在浏览器端存储， 很大程度上不受服务器端控制， 如果浏览器端清理 Cookie， 相关数据会丢失
3. 反向代理 hash 一致性

   ![反向代理](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/13-3-3_2.png)
   * 问题 1： 具体一个浏览器， 专门访问某一个具体服务器， 如果服务器宕机，会丢失数据。 存在单点故障风险。
   * 问题 2： 仅仅适用于集群范围内， 超出集群范围， 负载均衡服务器无效
4. 后端统一存储 Session 数据

    ![后端统一存储 Session 数据](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/13-3-3_3.png)
   * 后端存储 Session 数据时， 一般需要使用 Redis 这样的内存数据库， 而一般不采用 MySQL 这样的关系型数据库。 原因如下：
     * Session 数据存取比较频繁。 内存访问速度快。
     * Session 有过期时间， Redis 这样的内存数据库能够比较方便实现过期释放
   * 优点
     * 访问速度比较快。 虽然需要经过网络访问， 但是现在硬件条件已经能够达到网络访问比硬盘访问还要快。
       * 硬盘访问速度： 200M/s
       * 网络访问速度： 1G/s
     * Redis 可以配置主从复制集群， 不担心单点故障

### 13.1.4 SpringSession

1. 原理

   ![原理](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/13-3-4.png)

   * Filter 原理，装饰模式
   * 存入 Session 域的实体类对象需要支持序列化

2. 配置
   * 依赖

      ```xml
      <!-- 引入 springboot&redis 整合场景 -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
      <!-- 引入 springboot&springsession 整合场景 -->
      <dependency>
          <groupId>org.springframework.session</groupId>
          <artifactId>spring-session-data-redis</artifactId>
      </dependency>
      ```

   * 配置文件

      ```yml
      spring:
        redis:
          host: 54.238.21.82
          jedis:
            pool:
              max-idle: 100
        session:
          store-type: redis
      ```

## 13.2 登录检查

### 13.2.1 目标思路

1. 目标  
   把项目中必须登录才能访问的功能保护起来， 如果没有登录就访问则跳转到登录页面
2. 思路

   ![思路](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/13-2-1.png)

### 13.2.2 设置Session共享

1. zuul工程加入依赖

    ```xml
    <!-- 引入 springboot&redis 整合场景 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!-- 引入 springboot&springsession 整合场景 -->
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    ```

2. 配置文件

    ```yml
    spring:
      application:
        name: atguigu-crowd-zuul
      redis:
        host: 54.238.**.**
      session:
        store-type: redis
    ```

3. auth工程
   * 依赖：同上
   * 配置：同上

### 13.2.3 不需要登录检查的资源

1. util模块新建常量类

    ```java
    public class AccessPassResources {
  
        public static final Set<String> PASS_RES_SET = new HashSet<>();
  
        static {
            PASS_RES_SET.add("/");
            PASS_RES_SET.add("/auth/member/to/reg/page");
            PASS_RES_SET.add("/auth/member/to/login/page");
            PASS_RES_SET.add("/auth/member/logout");
            PASS_RES_SET.add("/auth/member/do/login");
            PASS_RES_SET.add("/auth/do/member/register");
            PASS_RES_SET.add("/auth/member/send/short/message.json");
        }
  
        public static final Set<String> STATIC_RES_SET = new HashSet<>();
  
        static {
            STATIC_RES_SET.add("bootstrap");
            STATIC_RES_SET.add("css");
            STATIC_RES_SET.add("fonts");
            STATIC_RES_SET.add("img");
            STATIC_RES_SET.add("jquery");
            STATIC_RES_SET.add("layer");
            STATIC_RES_SET.add("script");
            STATIC_RES_SET.add("ztree");
        }

        /**
        * 用于判断某个 ServletPath 值是否对应一个静态资源
        * @param servletPath
        * @return
        *      true：是静态资源
        *      false：不是静态资源
        */
        public static boolean judgeCurrentServletPathWhetherStaticResource(String servletPath) {

            if (servletPath == null || servletPath.length() == 0) {
                throw new RuntimeException(CrowdConstant.MESSAGE_STRING_INVALIDATE);
            }

            String[] split = servletPath.split("/");

            String firstLevelPath = split[1];

            return STATIC_RES_SET.contains(firstLevelPath);
        }
    }
    ```

2. zuul模块创建过滤类

    ```java
    package com.atguigu.crowd.filter;
    /**
    * filterType：返回过滤器的类型。有pre、route、post、error等几种取值，分别对应几种过滤器。
    * filterOrder：返回一个int值来指定过滤器的执行顺序，不同的过滤器允许返回相同的数字。
    * shouldFilter：返回一个boolean值来判断该过滤器是否要执行，true表示执行，false表示不执行。
    * run：过滤器的具体逻辑。本例中，我们让它打印了请求的HTTP方法以及请求的地址
    */
    @Component
    public class CrowdAccessFilter extends ZuulFilter {
        @Override
        public String filterType() {

            return "pre";
        }

        @Override
        public int filterOrder() {
            return 0;
        }

        @Override
        public boolean shouldFilter() {

            // 1.获取RequestContext对象
            RequestContext requestContext = RequestContext.getCurrentContext();

            // 2.通过RequestContext对象获取当前请求对象（ 框架底层是借助 ThreadLocal 从当前线程上获取事先绑定的 Request 对象）
            HttpServletRequest request = requestContext.getRequest();

            // 3.获取 servletPath 值
            String servletPath = request.getServletPath();

            // 4.根据 servletPath 判断当前请求是否对应可以直接放行的特定功能
            boolean containsResult = AccessPassResources.PASS_RES_SET.contains(servletPath);
            // 如果当前请求是可以直接放行的特定功能请求则返回 false 放行
            if (containsResult){
                return false;
            }

            // 5.判断当前请求是否为静态资源
            return !AccessPassResources.judgeCurrentServletPathWhetherStaticResource(servletPath);
        }

        @Override
        public Object run() throws ZuulException {

            // 1.获取当前请求对象
            RequestContext requestContext = RequestContext.getCurrentContext();
            HttpServletRequest request = requestContext.getRequest();

            // 2.获取当前Session对象
            HttpSession session = request.getSession();

            // 3.尝试从 Session 对象中获取已登录用户
            Object loginMember = session.getAttribute(CrowdConstant.ATTR_NAME_LOGIN_MEMBER);

            // 4.判断 loginMember 是否为空
            if (loginMember == null) {

                // 5.从requestContext对象中获取Response对象
                HttpServletResponse response = requestContext.getResponse();

                // 6.将提示消息存入session域
                session.setAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_ACCESS_FORBIDDEN);

                // 7.重定向到 auth-consumer 工程中的登录页面
                try {
                    response.sendRedirect("/auth/member/to/login/page");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            return null;
        }
    }
    ```

## 13.3 错误修正

### 13.3.1 之前的错误

1. member-login.html

    ```html
    <p th:text="${session.message}">在这里显示登录失败的提示消息</p>
    ```

2. zuul模块加依赖

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
    ```

### 13.3.2 项目运行时出错

1. 超时错误：修改配置文件

    ```yml
    # 要加在eureka之前，几个工程都要加
    ribbon:
      ReadTimeout: 60000
      ConnectTimeout: 60000
    ```

2. 找不到username：重定向问题
    * 问题描述：<http://localhost:4000> 与 <http://loclahost:80> 是两个不同网站， 浏览器工作时不会使用相同的 Cookie
    * 解决：修改MemberHandler类doLogin方法

        ```java
        return "redirect:http://www.crowd.com/auth/member/to/center/page";
        ```

3. 序列化出错：
    * Spring Session要求对象支持序列化
    * 修改 MemberLoginVO

        ```java
        public class MemberLoginVO implements Serializable {

            private static final long serialVersionUID = 1L;
        ```

## 13.4 阿里云的 OSS 对象存储

### 13.4.1 问题引入

1. 文件上传保存位置：保存在应用中特定目录下

    ![文件保存位置](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/13-4-1.png)

2. 问题
    * Web 应用重新部署导致文件丢失
    * 集群环境下文件难以同步：分布式环境有多个Tomcat服务器
    * 文件数量太大时，影响 Tomcat 的运行效率
    * 服务器扩容：手动对服务器进行扩容， 有可能导致项目中其他地方需要进行连带修改

### 13.4.2 解决方案

1. 自己搭建文件服务器
    * 举例： FastDFS
    * 好处： 服务器可以自己维护、 自己定制。
    * 缺点： 需要投入的人力、 物力更多。
    * 适用： 规模比较大的项目， 要存储海量的文件
2. 使用第三方云服务
    * 举例： 阿里云提供的 OSS 对象存储服务。
    * 好处： 不必自己维护服务器的软硬件资源。 直接调用相关 API即可操作， 更加
    轻量级。
    * 缺点： 数据不在自己手里。 服务器不由自己维护。
    * 适用： 较小规模的应用， 文件数据不是绝对私密

### 13.4.3 开通 OSS 服务

1. project模块依赖

    ```xml
    <dependency>
        <groupId>com.atguigu.crowd</groupId>
        <artifactId>atcrowdfunding17-member-api</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
    </dependency>
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
    <!-- 引入 springboot&redis 整合场景 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!-- 引入 springboot&springsession 整合场景 -->
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    ```

2. 新建类

    ```java
    package com.atguigu.crowd.config;

    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.stereotype.Component;

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Component
    @ConfigurationProperties(prefix = "aliyun.oss")
    public class OSSProperties {
        private String endPoint;
        private String bucketName;
        private String accessKeyId;
        private String accessKeySecret;
        private String bucketDomain;
    }
    ```

3. 配置文件

    ```yml
    server:
    port: 5000

    spring:
    application:
        name: atguigu-crowd-project
    thymeleaf:
        prefix: classpath:/templates/
        suffix: .html
    redis:
        host: 54.238.77.83
    session:
        store-type: redis

    eureka:
    client:
        service-url:
        defaultZone: http://localhost:1000/eureka

    aliyun:
    oss:
        access-key-id: LTAI4GKG7FA4YiMvyP1PdtQm
        access-key-secret: PAf8u1Tmsltm7edDkvwUTIq4RcQaux
        bucket-domain: http://atguigu20210126.oss-cn-hangzhou.aliyuncs.com
        bucket-name: atguigu20210126
        end-point: http://oss-cn-hangzhou.aliyuncs.com
    ```

### 13.4.4 util模块

1. 依赖

    ```xml
    <dependency>
        <groupId>com.aliyun.oss</groupId>
        <artifactId>aliyun-sdk-oss</artifactId>
        <version>3.10.2</version>
    </dependency>
    ```

2. CrowdUtil类新建方法

    ```java
    /**
     * 专门负责上传文件到OSS服务器的工具方法
     * @param endpoint			OSS参数
     * @param accessKeyId		OSS参数
     * @param accessKeySecret	OSS参数
     * @param inputStream		要上传的文件的输入流
     * @param bucketName		OSS参数
     * @param bucketDomain		OSS参数
     * @param originalName		要上传的文件的原始文件名
     * @return	包含上传结果以及上传的文件在OSS上的访问路径
     */
    public static ResultEntity<String> uploadFileToOss(
            String endpoint,
            String accessKeyId,
            String accessKeySecret,
            InputStream inputStream,
            String bucketName,
            String bucketDomain,
            String originalName) {

        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);

        // 生成上传文件的目录
        String folderName = new SimpleDateFormat("yyyyMMdd").format(new Date());

        // 生成上传文件在OSS服务器上保存时的文件名
        // 原始文件名：beautfulgirl.jpg
        // 生成文件名：wer234234efwer235346457dfswet346235.jpg
        // 使用UUID生成文件主体名称
        String fileMainName = UUID.randomUUID().toString().replace("-", "");

        // 从原始文件名中获取文件扩展名
        String extensionName = originalName.substring(originalName.lastIndexOf("."));

        // 使用目录、文件主体名称、文件扩展名称拼接得到对象名称
        String objectName = folderName + "/" + fileMainName + extensionName;

        try {
            // 调用OSS客户端对象的方法上传文件并获取响应结果数据
            PutObjectResult putObjectResult = ossClient.putObject(bucketName, objectName, inputStream);

            // 从响应结果中获取具体响应消息
            ResponseMessage responseMessage = putObjectResult.getResponse();

            // 根据响应状态码判断请求是否成功
            if(responseMessage == null) {

                // 拼接访问刚刚上传的文件的路径
                String ossFileAccessPath = bucketDomain + "/" + objectName;

                // 当前方法返回成功
                return ResultEntity.successWithData(ossFileAccessPath);
            } else {
                // 获取响应状态码
                int statusCode = responseMessage.getStatusCode();

                // 如果请求没有成功，获取错误消息
                String errorMessage = responseMessage.getErrorResponseAsString();

                // 当前方法返回失败
                return ResultEntity.failed("当前响应状态码="+statusCode+" 错误消息="+errorMessage);
            }
        } catch (Exception e) {
            e.printStackTrace();

            // 当前方法返回失败
            return ResultEntity.failed(e.getMessage());
        } finally {

            if(ossClient != null) {

                // 关闭OSSClient。
                ossClient.shutdown();
            }
        }
    }
    ```

## 13.5 页面跳转 - 发起众筹

### 13.5.1 前端页面

1. 修改 member-center.html

    ```html
    <a th:href="@{/member/my/crowd}">我的众筹</a>
    ```

2. 新建 member-crowd.html

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="author" content="">
        <base th:href="@{/}"/>
        <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="css/font-awesome.min.css">
        <link rel="stylesheet" href="css/theme.css">
        <style>
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

        </style>
    </head>
    <body>
    <div class="navbar-wrapper">
        <div class="container">
            <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
                <div class="container">
                    <div class="navbar-header">
                        <a class="navbar-brand" href="#" style="font-size:32px;">尚筹网-创意产品众筹平台</a>
                    </div>
                    <div id="navbar" class="navbar-collapse collapse" style="float:right;">
                        <ul class="nav navbar-nav">
                            <li class="dropdown">
                                <a href="#" class="dropdown-toggle" data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i> [[${session.loginMember.username}]] <span class="caret"></span></a>
                                <ul class="dropdown-menu" role="menu">
                                    <li><a href="member.html"><i class="glyphicon glyphicon-scale"></i> 会员中心</a></li>
                                    <li><a href="#"><i class="glyphicon glyphicon-comment"></i> 消息</a></li>
                                    <li class="divider"></li>
                                    <li><a href="index.html" th:href="@{/auth/member/logout}"><i class="glyphicon glyphicon-off"></i> 退出系统</a></li>
                                </ul>
                            </li>
                        </ul>
                    </div>
                </div>
            </nav>

        </div>
    </div>
    <div class="container">
        <div class="row clearfix">
            <div class="col-sm-3 col-md-3 column">
                <div class="row">
                    <div class="col-md-12">
                        <div class="thumbnail" style="    border-radius: 0px;">
                            <img src="img/services-box1.jpg" class="img-thumbnail" alt="A generic square placeholder image with a white border around it, making it resemble a photograph taken with an old instant camera">
                            <div class="caption" style="text-align:center;">
                                <h3>
                                    [[${session.loginMember.username}]]
                                </h3>
                                <span class="label label-danger" style="cursor:pointer;" onclick="window.location.href='cert.html'">未实名认证</span>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="list-group">
                    <div class="list-group-item" style="cursor:pointer;" onclick="window.location.href='member.html'">
                        资产总览<span class="badge"><i class="glyphicon glyphicon-chevron-right"></i></span>
                    </div>
                    <div class="list-group-item active">
                        我的众筹<span class="badge"><i class="glyphicon glyphicon-chevron-right"></i></span>
                    </div>
                </div>
            </div>
            <div class="col-sm-9 col-md-9 column">
                <ul id="myTab" style="" class="nav nav-pills" role="tablist">
                    <li role="presentation" class="active"><a href="#home" role="tab" data-toggle="tab" aria-controls="home" aria-expanded="true">我的众筹</a></li>
                    <li role="presentation"><a href="#profile">众筹资产</a></li>
                </ul>
                <div id="myTabContent" class="tab-content" style="margin-top:10px;">
                    <div role="tabpanel" class="tab-pane fade active in" id="home" aria-labelledby="home-tab">

                        <ul id="myTab1" class="nav nav-tabs">
                            <li role="presentation" class="active"><a href="#support">我支持的</a></li>
                            <li role="presentation"><a href="#attension">我关注的</a></li>
                            <li role="presentation"><a href="#add">我发起的</a></li>
                            <li class=" pull-right">
                                <button type="button" class="btn btn-warning" onclick="window.location.href='start.html'">发起众筹</button>
                            </li>
                        </ul>
                        <div id="myTab1" class="tab-content" style="margin-top:10px;">
                            <div role="tabpanel" class="tab-pane fade active in" id="support" aria-labelledby="home-tab">
                                <div class="container-fluid">
                                    <div class="row clearfix">
                                        <div class="col-md-12 column">
                                            <span class="label label-warning">全部</span> <span class="label" style="color:#000;">已支付</span> <span class="label " style="color:#000;">未支付</span>
                                        </div>
                                        <div class="col-md-12 column" style="margin-top:10px;padding:0;">
                                            <table class="table table-bordered" style="text-align:center;">
                                                <thead>
                                                <tr style="background-color:#ddd;">
                                                    <td>项目信息</td>
                                                    <td width="90">支持日期</td>
                                                    <td width="120">支持金额（元）</td>
                                                    <td width="80">回报数量</td>
                                                    <td width="80">交易状态</td>
                                                    <td width="120">操作</td>
                                                </tr>
                                                </thead>
                                                <tbody>
                                                <tr>
                                                    <td style="vertical-align:middle;">
                                                        <div class="thumbnail">
                                                            <div class="caption">
                                                                <h3>
                                                                    活性富氢净水直饮机
                                                                </h3>
                                                                <p>
                                                                    订单编号:2x002231111
                                                                </p>
                                                                <p>
                                                                <div style="float:left;"><i class="glyphicon glyphicon-screenshot" title="目标金额" ></i> 已完成 100% </div>
                                                                <div style="float:right;"><i title="截至日期" class="glyphicon glyphicon-calendar"></i> 剩余8天 </div>
                                                                </p>
                                                                <br>
                                                                <div class="progress" style="margin-bottom: 4px;">
                                                                    <div class="progress-bar progress-bar-danger" role="progressbar" aria-valuenow="40" aria-valuemin="0" aria-valuemax="100" style="width: 40%">
                                                                        <span >众筹中</span>
                                                                    </div>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </td>
                                                    <td style="vertical-align:middle;">2017-05-23 11:31:22</td>
                                                    <td style="vertical-align:middle;">1.00<br>(运费：0.00 )</td>
                                                    <td style="vertical-align:middle;">1</td>
                                                    <td style="vertical-align:middle;">交易关闭</td>
                                                    <td style="vertical-align:middle;">
                                                        <div class="btn-group-vertical" role="group" aria-label="Vertical button group">
                                                            <button type="button" class="btn btn-default">删除订单</button>
                                                            <button type="button" class="btn btn-default">交易详情</button>
                                                        </div>
                                                    </td>
                                                </tr>
                                                <tr>
                                                    <td style="vertical-align:middle;">
                                                        <div class="thumbnail">
                                                            <div class="caption">
                                                                <h3>
                                                                    BAVOSN便携折叠移动电源台灯
                                                                </h3>
                                                                <p>
                                                                    订单编号:2x002231111
                                                                </p>
                                                                <p>
                                                                <div style="float:left;"><i class="glyphicon glyphicon-screenshot" title="目标金额" ></i> 已完成 100% </div>
                                                                <div style="float:right;"><i title="截至日期" class="glyphicon glyphicon-calendar"></i> 剩余8天 </div>
                                                                </p>
                                                                <br>
                                                                <div class="progress" style="margin-bottom: 4px;">
                                                                    <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="40" aria-valuemin="0" aria-valuemax="100" style="width: 40%">
                                                                        <span >众筹成功</span>
                                                                    </div>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </td>
                                                    <td style="vertical-align:middle;">2017-05-23 11:31:22</td>
                                                    <td style="vertical-align:middle;">1.00<br>(运费：0.00 )</td>
                                                    <td style="vertical-align:middle;">1</td>
                                                    <td style="vertical-align:middle;">交易关闭</td>
                                                    <td style="vertical-align:middle;">
                                                        <div class="btn-group-vertical" role="group" aria-label="Vertical button group">
                                                            <button type="button" class="btn btn-default">删除订单</button>
                                                            <button type="button" class="btn btn-default">交易详情</button>
                                                        </div>
                                                    </td>
                                                </tr>
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div role="tabpanel" class="tab-pane fade" id="attension" aria-labelledby="attension-tab">
                                <div class="container-fluid">
                                    <div class="row clearfix">
                                        <div class="col-md-12 column" style="padding:0;">
                                            <table class="table table-bordered" style="text-align:center;">
                                                <thead>
                                                <tr style="background-color:#ddd;">
                                                    <td>项目信息</td>
                                                    <td width="120">支持人数</td>
                                                    <td width="120">关注人数</td>
                                                    <td width="120">操作</td>
                                                </tr>
                                                </thead>
                                                <tbody>
                                                <tr>
                                                    <td style="vertical-align:middle;">
                                                        <div class="thumbnail">
                                                            <div class="caption">
                                                                <p>
                                                                    BAVOSN便携折叠移动电源台灯
                                                                </p>
                                                                <p>
                                                                    <i class="glyphicon glyphicon-jpy"></i> 已筹集 1000.0元
                                                                </p>
                                                                <p>
                                                                <div style="float:left;"><i class="glyphicon glyphicon-screenshot" title="目标金额" ></i> 已完成 100% </div>
                                                                <div style="float:right;"><i class="glyphicon glyphicon-calendar"></i> 剩余2天 </div>
                                                                </p>
                                                                <br>
                                                                <div class="progress" style="margin-bottom: 4px;">
                                                                    <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="40" aria-valuemin="0" aria-valuemax="100" style="width: 40%">
                                                                        <span >众筹中</span>
                                                                    </div>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </td>
                                                    <td style="vertical-align:middle;">1</td>
                                                    <td style="vertical-align:middle;">1</td>
                                                    <td style="vertical-align:middle;">
                                                        <div class="btn-group-vertical" role="group" aria-label="Vertical button group">
                                                            <button type="button" class="btn btn-default">取消关注</button>
                                                        </div>
                                                    </td>
                                                </tr>
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div role="tabpanel" class="tab-pane fade  " id="add" aria-labelledby="add-tab">
                                <div class="container-fluid">
                                    <div class="row clearfix">
                                        <div class="col-md-12 column">
                                            <span class="label label-warning">全部</span> <span class="label" style="color:#000;">众筹中</span> <span class="label " style="color:#000;">众筹成功</span>  <span class="label " style="color:#000;">众筹失败</span>
                                        </div>
                                        <div class="col-md-12 column" style="padding:0;margin-top:10px;">
                                            <table class="table table-bordered" style="text-align:center;">
                                                <thead>
                                                <tr style="background-color:#ddd;">
                                                    <td>项目信息</td>
                                                    <td width="120">募集金额（元）</td>
                                                    <td width="80">当前状态</td>
                                                    <td width="120">操作</td>
                                                </tr>
                                                </thead>
                                                <tbody>
                                                <tr>
                                                    <td style="vertical-align:middle;">
                                                        <div class="thumbnail">
                                                            <div class="caption">
                                                                <p>
                                                                    BAVOSN便携折叠移动电源台灯
                                                                </p>
                                                                <p>
                                                                <div style="float:left;"><i class="glyphicon glyphicon-screenshot" title="目标金额" ></i> 已完成 100% </div>
                                                                <div style="float:right;"><i title="截至日期" class="glyphicon glyphicon-calendar"></i> 剩余8天 </div>
                                                                </p>
                                                                <br>
                                                                <div class="progress" style="margin-bottom: 4px;">
                                                                    <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="40" aria-valuemin="0" aria-valuemax="100" style="width: 40%">
                                                                        <span >众筹中</span>
                                                                    </div>
                                                                </div>
                                                            </div>
                                                        </div>
                                                    </td>
                                                    <td style="vertical-align:middle;">1.00<br>(运费：0.00 )</td>
                                                    <td style="vertical-align:middle;">草稿</td>
                                                    <td style="vertical-align:middle;">
                                                        <div class="btn-group-vertical" role="group" aria-label="Vertical button group">
                                                            <button type="button" class="btn btn-default">项目预览</button>
                                                            <button type="button" class="btn btn-default">修改项目</button>
                                                            <button type="button" class="btn btn-default">删除项目</button>
                                                            <button type="button" class="btn btn-default">问题管理</button>
                                                        </div>
                                                    </td>
                                                </tr>
                                                </tbody>
                                            </table>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                    </div>
                    <div role="tabpanel" class="tab-pane fade" id="profile" aria-labelledby="profile-tab">
                        众筹资产
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="container" style="margin-top:20px;">
        <div class="row clearfix">
            <div class="col-md-12 column">
                <div id="footer">
                    <div class="footerNav">
                        <a rel="nofollow" href="http://www.atguigu.com">关于我们</a> | <a rel="nofollow" href="http://www.atguigu.com">服务条款</a> | <a rel="nofollow" href="http://www.atguigu.com">免责声明</a> | <a rel="nofollow" href="http://www.atguigu.com">网站地图</a> | <a rel="nofollow" href="http://www.atguigu.com">联系我们</a>
                    </div>
                    <div class="copyRight">
                        Copyright ?2017-2017atguigu.com 版权所有
                    </div>
                </div>

            </div>
        </div>
    </div>
    <script src="jquery/jquery-2.1.1.min.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
    <script src="script/docs.min.js"></script>
    <script src="script/back-to-top.js"></script>
    <script>
        $('#myTab a').click(function (e) {
            e.preventDefault()
            $(this).tab('show')
        })
        $('#myTab1 a').click(function (e) {
            e.preventDefault()
            $(this).tab('show')
        })
    </script>
    </body>
    </html>
    ```

### 13.5.2 后端代码

1. CrowdWebMvcConfig

    ```java
    registry.addViewController("/member/my/crowd").setViewName("member-crowd");
    ```
