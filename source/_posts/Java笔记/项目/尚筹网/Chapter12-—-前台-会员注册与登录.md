---
title: Chapter12 — 前台 会员注册与登录
excerpt: 短信注册、会员登录
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
abbrlink: 838ceb88
date: 2021-01-24 01:49:26
updated: 2021-01-25 02:50:33
subtitle:
---
## 12.1 短信功能
### 12.1.1 申请阿里云短信服务

### 12.1.2 工具类：util模块 
1. CrowdUtil
    ```java
    /**
     * 阿里官方新API
     * @param SignName 签名
     * @param TemplateCode 模板编号
     * @param phoneNumbers 手机号码
     * @return 成功则返回验证码，失败返回错误信息
     */
    public static ResultEntity<String> sendShortMessage(String SignName, String TemplateCode, String phoneNumbers) {
        DefaultProfile profile = DefaultProfile.getProfile( "cn-hangzhou", ACCESSKEYID, ACCESSKEYSECRET );
        IAcsClient client = new DefaultAcsClient( profile );
        CommonRequest request = new CommonRequest();
        String authCode = RandomUtil.randomNumbers( 4 );
        request.setSysMethod( MethodType.POST );
        request.setSysDomain( "dysmsapi.aliyuncs.com" );
        request.setSysVersion( "2017-05-25" );
        request.setSysAction( "SendSms" );
        request.putQueryParameter( "RegionId", "cn-hangzhou" );
        request.putQueryParameter( "PhoneNumbers", phoneNumbers );
        request.putQueryParameter( "SignName", SignName );//短信签名名称。请在控制台签名管理页面签名名称一列查看。
        request.putQueryParameter( "TemplateCode", TemplateCode );
        request.putQueryParameter( "TemplateParam", "{\"code\":\"" + authCode + "\"}" ); //拼接成官网指定格式
        try {
            CommonResponse response = client.getCommonResponse( request );
            //System.out.println( response.getData() ); //回显消息
            if(response.getHttpResponse().isSuccess()) {
                //请求成功
                return ResultEntity.successWithData(authCode);
            }else{
                return ResultEntity.failed(response.getData());
            }
        } catch (ServerException e) {
            e.printStackTrace();
            return ResultEntity.failed(e.getMessage());
        } catch (ClientException e) {
            e.printStackTrace();
            return ResultEntity.failed(e.getMessage());
        }
    }

    /**
     * 阿里旧版的官方API
     * @param signName 签名
     * @param templateCode 模板编号
     * @param phoneNumbers 手机号码
     * @return 成功则返回验证码，失败返回错误信息
     */
    public static ResultEntity<String> sendSms(String signName, String templateCode, String phoneNumbers) {
        //设置超时时间-可自行调整
        System.setProperty("sun.net.client.defaultConnectTimeout", "10000");
        System.setProperty("sun.net.client.defaultReadTimeout", "10000");
        //初始化ascClient需要的几个参数
        final String product = "Dysmsapi";//短信API产品名称（短信产品名固定，无需修改）
        final String domain = "dysmsapi.aliyuncs.com";//短信API产品域名（接口地址固定，无需修改）
        //替换成你的AK
        final String accessKeyId = CrowdUtil.ACCESSKEYID;//你的accessKeyId,参考本文档步骤2
        final String accessKeySecret = CrowdUtil.ACCESSKEYSECRET;//你的accessKeySecret，参考本文档步骤2
        //初始化ascClient,暂时不支持多region（请勿修改）
        IClientProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessKeyId,
                accessKeySecret);
        try {
            DefaultProfile.addEndpoint("cn-hangzhou", "cn-hangzhou", product, domain);
        } catch (ClientException e) {
            e.printStackTrace();
        }
        IAcsClient acsClient = new DefaultAcsClient(profile);

        // 随机验证码
        String authCode = RandomUtil.randomNumbers( 4 );

        //组装请求对象
        SendSmsRequest request = new SendSmsRequest();
        //使用post提交
        request.setMethod(MethodType.POST);
        //必填:待发送手机号。支持以逗号分隔的形式进行批量调用，批量上限为1000个手机号码,批量调用相对于单条调用及时性稍有延迟,验证码类型的短信推荐使用单条调用的方式；发送国际/港澳台消息时，接收号码格式为国际区号+号码，如“85200000000”
        request.setPhoneNumbers(phoneNumbers);
        //必填:短信签名-可在短信控制台中找到
        request.setSignName(signName);
        //必填:短信模板-可在短信控制台中找到，发送国际/港澳台消息时，请使用国际/港澳台短信模版
        request.setTemplateCode(templateCode);
        //可选:模板中的变量替换JSON串,如模板内容为"亲爱的${name},您的验证码为${code}"时,此处的值为
        //友情提示:如果JSON中需要带换行符,请参照标准的JSON协议对换行符的要求,比如短信内容中包含\r\n的情况在JSON中需要表示成\\r\\n,否则会导致JSON在服务端解析失败
        //参考：request.setTemplateParam("{\"变量1\":\"值1\",\"变量2\":\"值2\",\"变量3\":\"值3\"}")
        request.setTemplateParam("{\"code\":\"" + authCode + "\"}");
        //可选-上行短信扩展码(扩展码字段控制在7位或以下，无特殊需求用户请忽略此字段)
        //request.setSmsUpExtendCode("90997");

        //可选:outId为提供给业务方扩展字段,最终在短信回执消息中将此值带回给调用者
        //request.setOutId("yourOutId");

        try {
            //请求失败这里会抛ClientException异常
            SendSmsResponse sendSmsResponse = acsClient.getAcsResponse(request);
            if(sendSmsResponse.getCode() != null && sendSmsResponse.getCode().equals("OK")) {
                //请求成功
                return ResultEntity.successWithData(authCode);
            }else{
                return ResultEntity.failed(sendSmsResponse.getMessage());
            }
        } catch (ClientException e) {
            e.printStackTrace();
            return ResultEntity.failed(e.getMessage());
        }
    }

    /**
     * 短信服务不可用时的模拟方法
     * @param signName 签名，随便填
     * @param templateCode 模板编号，随便填
     * @param phoneNumbers 手机号码
     * @return 返回生成的随机验证码
     */
    public static ResultEntity<String> sendSmsSimulation(String signName, String templateCode, String phoneNumbers){

        String authCode = RandomUtil.randomNumbers( 4 );
        return ResultEntity.successWithData(authCode);
    }
    ```
2. 


## 12.2 发送短信验证码
### 12.2.1 思路


### 12.2.2 前端代码
1. auth模块新建类
    ```java
    package com.atguigu.crowd.config;

    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

    @Configuration
    public class CrowdWebMvcConfig implements WebMvcConfigurer {

        @Override
        public void addViewControllers(ViewControllerRegistry registry) {

            // 浏览器访问的地址
            String urlPath = "/auth/member/to/reg/page.html";

            // 目标视图的名称，将来拼接前后缀
            String viewName = "member-reg";

            // 添加view-controller
            registry.addViewController(urlPath).setViewName(viewName);
        }
    }
    ```
2. 修改前端页面
    ```html
    <li><a href="reg.html" th:href="@{/auth/member/to/reg/page.html}">注册</a></li>
    ```
3. 新建注册页面 member-reg.html
    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="keys" content="">
        <meta name="author" content="">
        <base th:href="@{/}"/>
        <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="css/font-awesome.min.css">
        <link rel="stylesheet" href="css/login.css">
        <script src="jquery/jquery-2.1.1.min.js"></script>
        <script src="bootstrap/js/bootstrap.min.js"></script>
        <script type="text/javascript" src="layer/layer.js"></script>
        <script type="text/javascript">
            layer.msg("test");
        </script>
    </head>
    <body>
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
        <div class="container">
            <div class="navbar-header">
                <div><a class="navbar-brand" href="index.html" style="font-size:32px;">尚筹网-创意产品众筹平台</a></div>
            </div>
        </div>
    </nav>

    <div class="container">

        <form class="form-signin" role="form">
            <h2 class="form-signin-heading"><i class="glyphicon glyphicon-log-in"></i> 用户注册</h2>
            <div class="form-group has-success has-feedback">
                <input type="text" class="form-control" id="inputSuccess4" placeholder="请输入登录账号" autofocus>
                <span class="glyphicon glyphicon-user form-control-feedback"></span>
            </div>
            <div class="form-group has-success has-feedback">
                <input type="text" class="form-control" id="inputSuccess4" placeholder="请输入登录密码" style="margin-top:10px;">
                <span class="glyphicon glyphicon-lock form-control-feedback"></span>
            </div>
            <div class="form-group has-success has-feedback">
                <input type="text" class="form-control" id="inputSuccess4" placeholder="请输入邮箱地址" style="margin-top:10px;">
                <span class="glyphicon glyphicon glyphicon-envelope form-control-feedback"></span>
            </div>
            <div class="form-group has-success has-feedback">
                <input type="text" class="form-control" id="inputSuccess4" placeholder="请输入手机号" style="margin-top:10px;">
                <span class="glyphicon glyphicon glyphicon-earphone form-control-feedback"></span>
            </div>
            <div class="form-group has-success has-feedback">
                <input type="text" class="form-control" id="inputSuccess4" placeholder="请输入验证码" style="margin-top:10px;">
                <span class="glyphicon glyphicon glyphicon-comment form-control-feedback"></span>
            </div>
            <button class="btn btn-lg btn-success btn-block"> 获取验证码</button>
            <a class="btn btn-lg btn-success btn-block" href="login.html" > 注册</a>
        </form>
    </div>
    </body>
    </html>
    ```
4. 修改注册页面
    ```html
    <script type="text/javascript">

    $(function (){
        $("#sendBtn").click(function (){
            // 1.获取接收短信的手机号
            let phoneNum = $.trim($("[name=phoneNum]").val());

            // 2.发送请求
            $.ajax({
                url: "auth/member/send/short/message.json",
                type: "post",
                data: {
                    phoneNum: phoneNum
                },
                dataType: "json",
                success: function (resp){

                    var result = resp.result;
                    if (result == "SUCCESS") {
                        layer.msg("发送成功");
                    }
                    if (result == "FAILED") {
                        layer.msg("发送失败，请重新发送！")
                    }
                },
                error: function (resp){
                    layer.msg(resp.status + " " + resp.statusText);
                }
            });
        });
    });
    </script>
    ```
    ```html
    <input type="text" class="form-control" name="phoneNum" id="inputSuccess4" placeholder="请输入手机号" style="margin-top:10px;">
    ```
    ```html
    <button id="sendBtn" type="button" class="btn btn-lg btn-success btn-block"> 获取验证码</button>
    ```

### 12.2.3 后端代码
1. 主启动类添加 feign 功能
    ```java
    @EnableFeignClients
    @SpringBootApplication
    @EnableEurekaClient
    public class AuthenticationConsumerMain {
    ```
2. 新建 MemberHandler
    ```java
    @Controller
    public class MemberHandler {

        @Autowired
        private ShortMessageProperties shortMessageProperties;

        @Autowired
        private RedisRemoteService redisRemoteService;

        @ResponseBody
        @RequestMapping("/auth/member/send/short/message.json")
        public ResultEntity<String> sendMessage(@RequestParam("phoneNum") String phoneNum) {

            // 1.发送验证码到 phoneNum
            ResultEntity<String> resp = CrowdUtil.sendSms(
                    shortMessageProperties.getSignName(),
                    shortMessageProperties.getTemplateCode(),
                    phoneNum
            );

            // 2.判断是否发送成功
            if (ResultEntity.SUCCESS.equals(resp.getResult())) {

                // 3.如果成功，则将验证码存入Redis
                String code = resp.getData();
                String key = CrowdConstant.REDIS_CODE_PREFIX + phoneNum;
                ResultEntity<String> saveCodeResultEntity = redisRemoteService.setRedisKeyValueRemoteWithTimeout(key, code, 100, TimeUnit.SECONDS);

                // 4. 判断是否成功存入 Redis
                if (ResultEntity.SUCCESS.equals(saveCodeResultEntity.getResult())) {
                    return ResultEntity.successWithoutData();
                }else {
                    return saveCodeResultEntity;
                }
            }else {
                return resp;
            }
        }
    }
    ```
3. 配置文件
    ```yml
    short:
    message:
        signName: 1
        templateCode: SMS_21*****2
    ```
4. 新建配置类
    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Component
    @ConfigurationProperties(prefix = "short.message")
    public class ShortMessageProperties {

        private String signName;
        private String templateCode;
    }
    ```

## 12.3 注册
### 12.3.1 思路
1. 目标：用户点击注册，实现功能
2. 思路  
   
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/12-3-1-1.png)

### 12.3.2 后端代码
1. mysql模块handler
    ```java
    @RequestMapping("/save/member/remote")
    public ResultEntity<String> saveMember(@RequestBody MemberPO memberPO) {

        try {
            memberService.saveMember(memberPO);

            return ResultEntity.successWithoutData();
        } catch (Exception e) {

            if (e instanceof DuplicateKeyException)
                return ResultEntity.failed(CrowdConstant.MESSAGE_LOGIN_ACCT_ALREADY_IN_USE);

            return ResultEntity.failed(e.getMessage());
        }
    }
    ```
2. mysql模块service
    ```java
    @Transactional(propagation = Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
    @Override
    public void saveMember(MemberPO memberPO) {

        memberPOMapper.insertSelective(memberPO);
    }
    ```
3. api模块
    ```java
    @RequestMapping("/save/member/remote")
    public ResultEntity<String> saveMember(@RequestBody MemberPO memberPO);
    ```
4. entity模块创建MemberVO
    ```java
    package com.atguigu.crowd.entity.vo;

    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class MemberVO {

        private String loginacct;

        private String userpswd;

        private String username;

        private String email;

        private String phoneNum;

        private String code;
    }
    ```
5. MemberHandler
    ```java
    @Autowired
    private MySQLRemoteService mySQLRemoteService;

    @RequestMapping("/auth/do/member/register")
    public String register(MemberVO memberVO, ModelMap modelMap) {


        // 1.获取用户输入的手机号
        String phoneNum = memberVO.getPhoneNum();

        // 2.拼接Redis中存储的Key
        String key = CrowdConstant.REDIS_CODE_PREFIX + phoneNum;

        // 3.从Redis读取Key对应的value
        ResultEntity<String> resultEntity = redisRemoteService.getRedisStringValueByKey(key);

        // 4.检查查询操作是否有效
        String result = resultEntity.getResult();
        if (ResultEntity.FAILED.equals(result)) {

            modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, resultEntity.getMessage());
            return "member-reg";
        }

        String redisCode = resultEntity.getData();
        if (redisCode == null) {
            modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_CODE_NOT_EXISTS);
            return "member-reg";
        }

        // 5.能从Redis查询到，则比较两者是否一致
        String formCode = memberVO.getCode();
        if (!formCode.equals(redisCode)) {

            modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_CODE_INVALID);
        }
        // 6.一致，将value从Redis中删除
        ResultEntity<String> removeResultEntity = redisRemoteService.removeRedisKeyRemote(key);
        /*if (ResultEntity.FAILED.equals(removeResultEntity.getResult())) {
            return "";
        }*/

        // 7.执行密码加密
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        memberVO.setUserpswd(passwordEncoder.encode(memberVO.getUserpswd()));

        // 8. 执行保存
        MemberPO memberPO = new MemberPO();
        BeanUtils.copyProperties(memberVO, memberPO);
        ResultEntity<String> saveMemberResultEntity = mySQLRemoteService.saveMember(memberPO);

        if (ResultEntity.FAILED.equals(saveMemberResultEntity)) {

            modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, saveMemberResultEntity.getMessage());

            return "member-reg";
        }

        return "redirect:/auth/member/to/login/page";
    }
    ```
6. auth模块config类
    ```java
    @Configuration
    public class CrowdWebMvcConfig implements WebMvcConfigurer {

        @Override
        public void addViewControllers(ViewControllerRegistry registry) {

            // 浏览器访问的地址
            String urlPath = "/auth/member/to/reg/page";

            // 目标视图的名称，将来拼接前后缀
            String viewName = "member-reg";

            // 添加view-controller
            registry.addViewController(urlPath).setViewName(viewName);
            registry.addViewController("/auth/member/to/login/page").setViewName("member-login");;
        }
    }
    ```
7. 另一个config类
    ```java

    ```

### 12.3.3 前端
1. 修改 member-reg.html
   * 添加表单提交地址、method
   * 输入文本框添加 name
   * 修改注册按钮
    ```html
    <form action="/auth/do/member/register" method="post" class="form-signin" role="form">
        <h2 class="form-signin-heading"><i class="glyphicon glyphicon-log-in"></i> 用户注册</h2>
        <p th:text="${message}">在这里显示从请求域取出的提示消息</p>
        <div class="form-group has-success has-feedback">
            <input type="text" name="loginacct" class="form-control" id="inputSuccess1" placeholder="请输入登录账号" autofocus>
            <span class="glyphicon glyphicon-user form-control-feedback"></span>
        </div>
        <div class="form-group has-success has-feedback">
            <input type="text" name="userpswd" class="form-control" id="inputSuccess2" placeholder="请输入登录密码" style="margin-top:10px;">
            <span class="glyphicon glyphicon-lock form-control-feedback"></span>
        </div>
        <div class="form-group has-success has-feedback">
            <input type="text" name="username" class="form-control" id="inputSuccess12" placeholder="请输入用户昵称" style="margin-top:10px;">
            <span class="glyphicon glyphicon-lock form-control-feedback"></span>
        </div>
        <div class="form-group has-success has-feedback">
            <input type="text" name="email" class="form-control" id="inputSuccess3" placeholder="请输入邮箱地址" style="margin-top:10px;">
            <span class="glyphicon glyphicon glyphicon-envelope form-control-feedback"></span>
        </div>
        <div class="form-group has-success has-feedback">
            <input type="text" name="phoneNum" class="form-control" id="inputSuccess4" placeholder="请输入手机号" style="margin-top:10px;">
            <span class="glyphicon glyphicon glyphicon-earphone form-control-feedback"></span>
        </div>
        <div class="form-group has-success has-feedback">
            <input type="text" name="code" class="form-control" id="inputSuccess5" placeholder="请输入验证码" style="margin-top:10px;">
            <span class="glyphicon glyphicon glyphicon-comment form-control-feedback"></span>
        </div>
        <button id="sendBtn" type="button" class="btn btn-lg btn-success btn-block"> 获取验证码</button>
        <button type="button" class="btn btn-lg btn-success btn-block">注册</button>
    </form>
    ```
2. 新建 member-login.html
    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="keys" content="">
        <meta name="author" content="">
        <base th:href="@{/}"/>
        <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="css/font-awesome.min.css">
        <link rel="stylesheet" href="css/login.css">
        <script src="jquery/jquery-2.1.1.min.js"></script>
        <script src="bootstrap/js/bootstrap.min.js"></script>
        <style>
        </style>
    </head>
    <body>
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
        <div class="container">
            <div class="navbar-header">
                <div><a class="navbar-brand" href="index.html" style="font-size:32px;">尚筹网-创意产品众筹平台</a></div>
            </div>
        </div>
    </nav>

    <div class="container">

        <form action="/auth/member/do/login" method="post" class="form-signin" role="form">
            <h2 class="form-signin-heading">
                <i class="glyphicon glyphicon-log-in"></i> 用户登录
            </h2>
            <p th:text="${message}">在这里显示登录失败的提示消息</p>
            <div class="form-group has-success has-feedback">
                <input type="text" name="loginacct" class="form-control" id="loginacct" placeholder="请输入登录账号" autofocus>
                <span class="glyphicon glyphicon-user form-control-feedback"></span>
            </div>
            <div class="form-group has-success has-feedback">
                <input type="text" name="userpswd" class="form-control" id="userpswd" placeholder="请输入登录密码" style="margin-top:10px;">
                <span class="glyphicon glyphicon-lock form-control-feedback"></span>
            </div>
            <div class="checkbox" style="text-align:right;">
                <a href="reg.html" th:href="@{/auth/member/to/login/page/}">我要注册</a>
            </div>
            <button type="submit" class="btn btn-lg btn-success btn-block"> 登录</button>
        </form>
    </div>
    </body>
    </html>
    ```
3. 修改portal.html
    ```html
    <li><a href="login.html" th:href="@{/auth/member/to/login/page}">登录</a></li>
    <li><a href="reg.html" th:href="@{/auth/member/to/reg/page}">注册</a></li>
    ```

## 12.4 登录
### 12.4.1 目标思路
1. 目标
   * 检查账号密码是否正确
   * 将正确的账号信息存入Session，跳转页面
2. 思路
    
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/12-4-1.png)


### 12.4.2 后端代码
1. entity模块新建vo
    ```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class MemberLoginVO {

        private Integer id;

        private String username;

        private String email;
    }
    ```
2. Memberhandler
    ```java
    @RequestMapping("/auth/member/do/login")
    public String doLogin(@RequestParam("loginacct") String logincacct,
                          @RequestParam("userpswd") String userpswd,
                          ModelMap modelMap,
                          HttpSession session) {

        // 1.调用远程接口查询登录账号
        ResultEntity<MemberPO> resultEntity = mySQLRemoteService.getMemberPOByLoginAcctRemote(logincacct);

        // 2.失败则返回登录页面
        if (ResultEntity.FAILED.equals(resultEntity.getResult())) {
            modelMap.addAttribute(CrowdConstant.ATTR_NAME_EXCEPTION, resultEntity.getMessage());
            return "member-login";
        }

        MemberPO memberPO = resultEntity.getData();

        if (memberPO == null) {
            modelMap.addAttribute(CrowdConstant.ATTR_NAME_EXCEPTION, CrowdConstant.MESSAGE_LOGIN_FAILED);
            return "member-login";
        }

        // 3.比较密码
        String userpswdDB = memberPO.getUserpswd();
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        boolean matches = passwordEncoder.matches(userpswd, userpswdDB);
        if (!matches) {
            modelMap.addAttribute(CrowdConstant.ATTR_NAME_EXCEPTION, CrowdConstant.MESSAGE_LOGIN_FAILED);
            return "member-login";
        }

        // 4.创建MemberLoginVO对象存入Session域
        MemberLoginVO memberLoginVO = new MemberLoginVO(memberPO.getId(), memberPO.getUsername(), memberPO.getEmail());
        session.setAttribute(CrowdConstant.ATTR_NAME_LOGIN_MEMBER, memberLoginVO);

        return "redirect:/auth/member/to/center/page";
    }
    ```
3. CrowdWebMvcConfig
    ```java
        registry.addViewController("/auth/member/to/center/page").setViewName("member-center");
        ```

### 12.4.3 前端代码
1. 新建 member-center.html
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
                        <a class="navbar-brand" href="index.html" style="font-size:32px;">尚筹网-创意产品众筹平台</a>
                    </div>
                    <div id="navbar" class="navbar-collapse collapse" style="float:right;">
                        <ul class="nav navbar-nav">
                            <li class="dropdown">
                                <a href="#" class="dropdown-toggle" data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i> [[${session.loginMember.username}]]<span class="caret"></span></a>
                                <ul class="dropdown-menu" role="menu">
                                    <li><a href="member.html"><i class="glyphicon glyphicon-scale"></i> 会员中心</a></li>
                                    <li><a href="#"><i class="glyphicon glyphicon-comment"></i> 消息</a></li>
                                    <li class="divider"></li>
                                    <li><a href="index.html"><i class="glyphicon glyphicon-off"></i> 退出系统</a></li>
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
                            <img src="img/services-box1.jpg" class="img-thumbnail" alt="">
                            <div class="caption" style="text-align:center;">
                                <h3>
                                    [[${session.loginMember.username}]]
                                </h3>
                                <span class="label label-danger" style="cursor:pointer;" onclick="window.location.href='accttype.html'">未实名认证</span>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="list-group">
                    <div class="list-group-item active">
                        资产总览<span class="badge"><i class="glyphicon glyphicon-chevron-right"></i></span>
                    </div>
                    <div class="list-group-item " style="cursor:pointer;" onclick="window.location.href='minecrowdfunding.html'">
                        我的众筹<span class="badge"><i class="glyphicon glyphicon-chevron-right"></i></span>
                    </div>
                </div>
            </div>
            <div class="col-sm-9 col-md-9 column">
                <blockquote style="border-left: 5px solid #f60;color:#f60;padding: 0 0 0 20px;">
                    <b>
                        我的钱包
                    </b>
                </blockquote>
                <div id="main" style="width: 600px;height:400px;"></div>
                <blockquote style="border-left: 5px solid #f60;color:#f60;padding: 0 0 0 20px;">
                    <b>
                        理财
                    </b>
                </blockquote>
                <div id="main1" style="width: 600px;height:400px;"></div>
                <blockquote style="border-left: 5px solid #f60;color:#f60;padding: 0 0 0 20px;">
                    <b>
                        比例
                    </b>
                </blockquote>
                <div id="main2" style="width: 600px;height:400px;"></div>
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
    <script src="script/echarts.js"></script>
    <script>
        $('#myTab a').click(function (e) {
            e.preventDefault()
            $(this).tab('show')
        })
        $('#myTab1 a').click(function (e) {
            e.preventDefault()
            $(this).tab('show')
        })

        var myChart = echarts.init(document.getElementById('main'));

        // 指定图表的配置项和数据
        option = {
            title: {
                text: '七日年化收益率(%)'
            },
            tooltip: {
                trigger: 'axis'
            },
            legend: {
                data:['基金','股票']
            },
            toolbox: {
                show: false,
                feature: {
                    dataZoom: {
                        yAxisIndex: 'none'
                    },
                    dataView: {readOnly: false},
                    magicType: {type: ['line', 'bar']},
                    restore: {},
                    saveAsImage: {}
                }
            },
            xAxis:  {
                type: 'category',
                boundaryGap: false,
                data: ['2017-05-16','2017-05-17','2017-05-18','2017-05-19','2017-05-20','2017-05-21','2017-05-22']
            },
            yAxis: {
                type: 'value',
                axisLabel: {
                    formatter: '{value} '
                }
            },
            series: [
                {
                    name:'基金',
                    type:'line',
                    data:[1, 1, 5, 3, 2, 3, 2],
                    markPoint: {
                        data: [
                            {type: 'max', name: '最大值'},
                            {type: 'min', name: '最小值'}
                        ]
                    },
                    markLine: {
                        data: [
                            {type: 'average', name: '平均值'}
                        ]
                    }
                },
                {
                    name:'股票',
                    type:'line',
                    data:[1, -2, 2, 5, 3, 2, 4],
                    markPoint: {
                        data: [
                            {name: '周最低', value: -2, xAxis: 1, yAxis: -1.5}
                        ]
                    },
                    markLine: {
                        data: [
                            {type: 'average', name: '平均值'},
                            [{
                                symbol: 'none',
                                x: '90%',
                                yAxis: 'max'
                            }, {
                                symbol: 'circle',
                                label: {
                                    normal: {
                                        position: 'start',
                                        formatter: '最大值'
                                    }
                                },
                                type: 'max',
                                name: '最高点'
                            }]
                        ]
                    }
                }
            ]
        };
        myChart.setOption(option);
        var myChart1 = echarts.init(document.getElementById('main1'));

        // 指定图表的配置项和数据
        option1 = {
            color: ['#3398DB'],
            tooltip : {
                trigger: 'axis',
                axisPointer : {            // 坐标轴指示器，坐标轴触发有效
                    type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
                }
            },
            grid: {
                left: '3%',
                right: '4%',
                bottom: '3%',
                containLabel: true
            },
            xAxis : [
                {
                    type : 'category',
                    data : ['基金', '票据', '定期理财', '变现贷'],
                    axisTick: {
                        alignWithLabel: true
                    }
                }
            ],
            yAxis : [
                {
                    type : 'value'
                }
            ],
            series : [
                {
                    name:'直接访问',
                    type:'bar',
                    barWidth: '60%',
                    data:[10, 52, 200, 334, 390, 330, 220]
                }
            ]
        };

        // 使用刚指定的配置项和数据显示图表。
        myChart1.setOption(option1);

        var myChart2 = echarts.init(document.getElementById('main2'));

        // 指定图表的配置项和数据
        option2 = {
            title : {
                text: '某站点用户访问来源',
                subtext: '纯属虚构',
                x:'center'
            },
            tooltip : {
                trigger: 'item',
                formatter: "{a} <br/>{b} : {c} ({d}%)"
            },
            legend: {
                orient: 'vertical',
                left: 'left',
                data: ['直接访问','邮件营销','联盟广告','视频广告','搜索引擎']
            },
            series : [
                {
                    name: '访问来源',
                    type: 'pie',
                    radius : '55%',
                    center: ['50%', '60%'],
                    data:[
                        {value:335, name:'直接访问'},
                        {value:310, name:'邮件营销'},
                        {value:234, name:'联盟广告'},
                        {value:135, name:'视频广告'},
                        {value:1548, name:'搜索引擎'}
                    ],
                    itemStyle: {
                        emphasis: {
                            shadowBlur: 10,
                            shadowOffsetX: 0,
                            shadowColor: 'rgba(0, 0, 0, 0.5)'
                        }
                    }
                }
            ]
        };

        // 使用刚指定的配置项和数据显示图表。
        myChart2.setOption(option2);
    </script>
    </body>
    </html>
    ```


### 12.4.4 退出登录
1. 修改 member-center.html
    ```html
    <li><a href="index.html" th:href="@{/auth/member/logout}"><i class="glyphicon glyphicon-off"></i> 退出系统</a></li>
    ```
2. handler
    ```java
    @RequestMapping("/auth/member/logout")
    public String logout(HttpSession session) {

        session.invalidate();

        return "redirect:/";
    }
    ```



