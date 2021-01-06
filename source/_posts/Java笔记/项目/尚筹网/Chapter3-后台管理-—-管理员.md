---
title: Chapter3 后台管理一 — 管理员信息维护
excerpt: 管理员登录与退出、管理员信息的增删改查
tags:
  - java
  - ssm
categories:
  - Java笔记
  - 项目
  - 尚筹网
banner_img: /img/post/banner/mandao.png
index_img: /img/post/ssm.png
abbrlink: '5e387227'
date: 2021-01-03 01:04:16
updated: 2021-01-06 16:27:21
subtitle:
---
## 3.1 管理员登录
### 3.1.1 设计思路

![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/3-1-1.png)

### 3.1.2 具体实现
1. 创建工具方法执行MD5加密
   * CrowdUtil类：
        ```java
        public static String md5(String source) {
            // 1.判断source是否有效
            if (source == null || source.length() == 0) {
                // 2.如果不是有效数据则抛异常
                throw new RuntimeException(CrowdConstant.MESSAGE_STRING_INVALIDATE);
            }

            try {
                // 3.获取MessageDigest对象
                String algorithm = "md5";
                MessageDigest messageDigest = MessageDigest.getInstance(algorithm);
                // 4.获取明文字符串对应的字节数组
                byte[] input = source.getBytes();
                // 5.执行加密
                byte[] output = messageDigest.digest(input);
                // 6.创建BigInteger对象
                int signum = 1;
                BigInteger bigInteger = new BigInteger(signum, output);
                // 7.按照16进制将bigInteger的值转换为字符串
                int radix = 16;
                String encoded = bigInteger.toString(radix).toUpperCase();

                return encoded;
            } catch (NoSuchAlgorithmException e) {
                e.printStackTrace();
            }
            return null;
        }
        ```

   * CrowdConstant类中添加常量
        ```java
        public static final String MESSAGE_STRING_INVALIDATE = "数据不合法，禁止传入空数据！";
        ```

2. 创建登录失败异常类（util模块）
    ```java
    package com.atguigu.crowd.exception;

    /**
    * 登录失败后抛出异常
    * @author hjp
    */
    public class LoginFailedException extends RuntimeException{

        private static final long serialVersionUID = 1L;

        public LoginFailedException() {
        }

        public LoginFailedException(String message) {
            super(message);
        }

        public LoginFailedException(String message, Throwable cause) {
            super(message, cause);
        }

        public LoginFailedException(Throwable cause) {
            super(cause);
        }

        public LoginFailedException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
            super(message, cause, enableSuppression, writableStackTrace);
        }
    }
    ```
3. 异常处理器类增加登录失败异常处理方法（component模块下CrowdExceptionResolver类）
    ```java
    @ExceptionHandler(value = LoginFailedException.class)
    public ModelAndView resolveLoginFailedException(NullPointerException exception, HttpServletRequest request, HttpServletResponse response) throws IOException {

        String viewName = "admin-login";
        return commonResolve(viewName,exception,request,response);
    }
    ```
4. 登录页面弹出消息
    ```html
    <h2 class="form-signin-heading"><i class="glyphicon glyphicon-log-in"></i>管理员登录</h2>
    <p>${requestScope.exception.message}</p>
    ```
5. handler方法:component模块下，com/atguigu/crowd/mvc/handler/AdminHandler.java
    ```java
    @Controller
    public class AdminHandler {

        @Autowired
        private AdminService adminService;

        @RequestMapping(value = "/admin/do/login.do")
        public String doLogin(
                @RequestParam("loginAcct") String loginAcct,
                @RequestParam("userPswd") String userPswd,
                HttpSession session
            ) {
            // 1.调用service方法执行登录检查
            // 返回admin对象说明登录成功，否则会抛出异常
            Admin admin = adminService.getAdminByLoginAcct(loginAcct, userPswd);

            // 2.将登录成功返回的admin对象存入Session域
            session.setAttribute(CrowdConstant.ATTR_NAME_LOGIN_ADMIN, admin);
            return "admin-main";
        }
    }
    ```
6. service方法
    ```java
    Admin getAdminByLoginAcct(String loginAcct, String userPswd);
    ```
    service实现类
    ```java
    @Override
    public Admin getAdminByLoginAcct(String loginAcct, String userPswd) {
        // 1. 根据登录账号查询Admin对象
        // 创建AdminExample对象
        System.out.println("到1了");
        AdminExample adminExample = new AdminExample();
        // 创建Criteria对象
        Criteria criteria = adminExample.createCriteria();
        // 在Criteria对象中封装查询条件
        criteria.andLoginEqualTo(loginAcct); // 这里跟视频中不太一样，因为方法名不同
        // 调用AdminMapper的方法执行查询
        List<Admin> list = adminMapper.selectByExample(adminExample);

        // 2.判断Admin对象是否为null
        if (list == null || list.size() == 0) {
            throw new LoginFailedException(CrowdConstant.MESSAGE_LOGIN_FAILED);
        }

        if (list.size()>1) {
            throw new RuntimeException(CrowdConstant.MESSAGE_SYSTEM_ERROR_LOGIN_NOT_UNIQUE);
        }

        Admin admin = list.get(0);
        // 3.为null则抛出异常
        if (admin == null) {
            throw new LoginFailedException(CrowdConstant.MESSAGE_LOGIN_FAILED);
        }
        System.out.println("查到用户了");
        // 4.Admin对象不为null，取出密码
        String userPswdDB = admin.getUserPswd();
        // 5.将表单提交的明文密码进行加密
        String userPswmForm = CrowdUtil.md5(userPswd);
        // 6.比较密码
        System.out.println("登录密码"+userPswmForm);
        System.out.println("用户密码"+userPswdDB);
        if (!Objects.equals(userPswdDB, userPswmForm)) {
            // 7.结果不一致，抛异常
            throw new LoginFailedException(CrowdConstant.MESSAGE_LOGIN_FAILED);
        }
        // 8.如果一致则返回Admin对象
        return admin;
    }
    ```
7. 后台主页面(WEB-INF目录下，admin-main.jsp)
    ```html
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <!DOCTYPE html>
    <html lang="zh-CN">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="author" content="">
        <base href="http://${pageContext.request.serverName}:${pageContext.request.serverPort}${pageContext.request.contextPath}/">

        <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="css/font-awesome.min.css">
        <link rel="stylesheet" href="css/main.css">
        <style>
            .tree li {
                list-style-type: none;
                cursor:pointer;
            }
            .tree-closed {
                height : 40px;
            }
            .tree-expanded {
                height : auto;
            }
        </style>
        <script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-2.1.1.min.js"></script>
        <script src="bootstrap/js/bootstrap.min.js"></script>
        <script src="script/docs.min.js"></script>
        <script type="text/javascript">
            $(function () {
                $(".list-group-item").click(function(){
                    if ( $(this).find("ul") ) {
                        $(this).toggleClass("tree-closed");
                        if ( $(this).hasClass("tree-closed") ) {
                            $("ul", this).hide("fast");
                        } else {
                            $("ul", this).show("fast");
                        }
                    }
                });
            });
        </script>
    </head>

    <body>

    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
        <div class="container-fluid">
            <div class="navbar-header">
                <div><a class="navbar-brand" style="font-size:32px;" href="#">众筹平台 - 控制面板</a></div>
            </div>
            <div id="navbar" class="navbar-collapse collapse">
                <ul class="nav navbar-nav navbar-right">
                    <li style="padding-top:8px;">
                        <div class="btn-group">
                            <button type="button" class="btn btn-default btn-success dropdown-toggle" data-toggle="dropdown">
                                <i class="glyphicon glyphicon-user"></i> ${sessionScope.loginAdmin.userName} <span class="caret"></span>
                            </button>
                            <ul class="dropdown-menu" role="menu">
                                <li><a href="#"><i class="glyphicon glyphicon-cog"></i> 个人设置</a></li>
                                <li><a href="#"><i class="glyphicon glyphicon-comment"></i> 消息</a></li>
                                <li class="divider"></li>
                                <li><a href="admin/do/logout.do"><i class="glyphicon glyphicon-off"></i> 退出系统</a></li>
                            </ul>
                        </div>
                    </li>
                    <li style="margin-left:10px;padding-top:8px;">
                        <button type="button" class="btn btn-default btn-danger">
                            <span class="glyphicon glyphicon-question-sign"></span> 帮助
                        </button>
                    </li>
                </ul>
                <form class="navbar-form navbar-right">
                    <input type="text" class="form-control" placeholder="查询">
                </form>
            </div>
        </div>
    </nav>
    <div class="container-fluid">
        <div class="row">
            <div class="col-sm-3 col-md-2 sidebar">
                <div class="tree">
                    <ul style="padding-left:0px;" class="list-group">
                        <li class="list-group-item tree-closed" >
                            <a href="main.html"><i class="glyphicon glyphicon-dashboard"></i> 控制面板</a>
                        </li>
                        <li class="list-group-item tree-closed">
                            <span><i class="glyphicon glyphicon glyphicon-tasks"></i> 权限管理 <span class="badge" style="float:right">3</span></span>
                            <ul style="margin-top:10px;display:none;">
                                <li style="height:30px;">
                                    <a href="user.html"><i class="glyphicon glyphicon-user"></i> 用户维护</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="role.html"><i class="glyphicon glyphicon-king"></i> 角色维护</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="permission.html"><i class="glyphicon glyphicon-lock"></i> 菜单维护</a>
                                </li>
                            </ul>
                        </li>
                        <li class="list-group-item tree-closed">
                            <span><i class="glyphicon glyphicon-ok"></i> 业务审核 <span class="badge" style="float:right">3</span></span>
                            <ul style="margin-top:10px;display:none;">
                                <li style="height:30px;">
                                    <a href="auth_cert.html"><i class="glyphicon glyphicon-check"></i> 实名认证审核</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="auth_adv.html"><i class="glyphicon glyphicon-check"></i> 广告审核</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="auth_project.html"><i class="glyphicon glyphicon-check"></i> 项目审核</a>
                                </li>
                            </ul>
                        </li>
                        <li class="list-group-item tree-closed">
                            <span><i class="glyphicon glyphicon-th-large"></i> 业务管理 <span class="badge" style="float:right">7</span></span>
                            <ul style="margin-top:10px;display:none;">
                                <li style="height:30px;">
                                    <a href="cert.html"><i class="glyphicon glyphicon-picture"></i> 资质维护</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="type.html"><i class="glyphicon glyphicon-equalizer"></i> 分类管理</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="process.html"><i class="glyphicon glyphicon-random"></i> 流程管理</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="advertisement.html"><i class="glyphicon glyphicon-hdd"></i> 广告管理</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="message.html"><i class="glyphicon glyphicon-comment"></i> 消息模板</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="project_type.html"><i class="glyphicon glyphicon-list"></i> 项目分类</a>
                                </li>
                                <li style="height:30px;">
                                    <a href="tag.html"><i class="glyphicon glyphicon-tags"></i> 项目标签</a>
                                </li>
                            </ul>
                        </li>
                        <li class="list-group-item tree-closed" >
                            <a href="param.html"><i class="glyphicon glyphicon-list-alt"></i> 参数管理</a>
                        </li>
                    </ul>
                </div>
            </div>
            <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
                <h1 class="page-header">控制面板</h1>

                <div class="row placeholders">
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/sky" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/vine" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/sky" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/vine" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </body>
    </html>
    ```

8. 常量类添加相应常量
    ```java
    public static final String ATTR_NAME_LOGIN_ADMIN = "loginAdmin";
    public static final String MESSAGE_SYSTEM_ERROR_LOGIN_NOT_UNIQUE = "查询结果不唯一";
    ```
9. 测试
    * 数据库中删除所有数据，新建一条，将密码改为md5加密后的值
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/3-1-2_1.png)
    * 启动tomcat服务器，访问 http://localhost:8080/atcrowdfunding02_admin_webui/admin/to/login.do
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/3-1-2_2.png)

10. 完善
    * 问题：登陆成功后，地址为 http://ip:port/工程名/admin/do/login.do
    * 此时若若刷新页面，则再次调用handler，重复数据库查询操作
    * 可以修改为重定向到当前页面：修改handler：
        ```java
        return "redirect:/admin/to/main.do";
        ```

    * 在mvc配置文件中加入：
        ``` xml
        <mvc:view-controller path="/admin/to/main.do" view-name="admin-main"/>
        ```

    * 此时的地址为：http://ip:port/工程名/admin/to/main.do 刷新不会重新执行handler

11. 登出操作
    * handler：
        ```java
        @RequestMapping(value = "/admin/do/logout.do")
        public String doLogout(HttpSession session) {

            // 强制Session失效
            session.invalidate();
            return "redirect:/admin/to/login.do";
        }
        ```

### 3.1.3 前端代码抽取
1. 可以将前端页面中重复的代码抽取出来，创建公共的模板
2. 头部：include-head.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="author" content="">
        <base href="http://${pageContext.request.serverName}:${pageContext.request.serverPort}${pageContext.request.contextPath}/">

        <link rel="stylesheet" href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="css/font-awesome.min.css">
        <link rel="stylesheet" href="css/main.css">
        <style>
            .tree li {
                list-style-type: none;
                cursor:pointer;
            }
            .tree-closed {
                height : 40px;
            }
            .tree-expanded {
                height : auto;
            }
        </style>
        <script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-2.1.1.min.js"></script>
        <script src="bootstrap/js/bootstrap.min.js"></script>
        <script src="script/docs.min.js"></script>
        <script type="text/javascript">
            $(function () {
                $(".list-group-item").click(function(){
                    if ( $(this).find("ul") ) {
                        $(this).toggleClass("tree-closed");
                        if ( $(this).hasClass("tree-closed") ) {
                            $("ul", this).hide("fast");
                        } else {
                            $("ul", this).show("fast");
                        }
                    }
                });
            });
        </script>
    </head>
    ```

3. 导航栏：include-nav.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
        <div class="container-fluid">
            <div class="navbar-header">
                <div><a class="navbar-brand" style="font-size:32px;" href="#">众筹平台 - 控制面板</a></div>
            </div>
            <div id="navbar" class="navbar-collapse collapse">
                <ul class="nav navbar-nav navbar-right">
                    <li style="padding-top:8px;">
                        <div class="btn-group">
                            <button type="button" class="btn btn-default btn-success dropdown-toggle" data-toggle="dropdown">
                                <i class="glyphicon glyphicon-user"></i> ${sessionScope.loginAdmin.userName} <span class="caret"></span>
                            </button>
                            <ul class="dropdown-menu" role="menu">
                                <li><a href="#"><i class="glyphicon glyphicon-cog"></i> 个人设置</a></li>
                                <li><a href="#"><i class="glyphicon glyphicon-comment"></i> 消息</a></li>
                                <li class="divider"></li>
                                <li><a href="admin/do/logout.do"><i class="glyphicon glyphicon-off"></i> 退出系统</a></li>
                            </ul>
                        </div>
                    </li>
                    <li style="margin-left:10px;padding-top:8px;">
                        <button type="button" class="btn btn-default btn-danger">
                            <span class="glyphicon glyphicon-question-sign"></span> 帮助
                        </button>
                    </li>
                </ul>
                <form class="navbar-form navbar-right">
                    <input type="text" class="form-control" placeholder="查询">
                </form>
            </div>
        </div>
    </nav>
    ```

4. 侧边栏：include-sidebar.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div class="col-sm-3 col-md-2 sidebar">
        <div class="tree">
            <ul style="padding-left:0px;" class="list-group">
                <li class="list-group-item tree-closed" >
                    <a href="main.html"><i class="glyphicon glyphicon-dashboard"></i> 控制面板</a>
                </li>
                <li class="list-group-item tree-closed">
                    <span><i class="glyphicon glyphicon glyphicon-tasks"></i> 权限管理 <span class="badge" style="float:right">3</span></span>
                    <ul style="margin-top:10px;display:none;">
                        <li style="height:30px;">
                            <a href="user.html"><i class="glyphicon glyphicon-user"></i> 用户维护</a>
                        </li>
                        <li style="height:30px;">
                            <a href="role.html"><i class="glyphicon glyphicon-king"></i> 角色维护</a>
                        </li>
                        <li style="height:30px;">
                            <a href="permission.html"><i class="glyphicon glyphicon-lock"></i> 菜单维护</a>
                        </li>
                    </ul>
                </li>
                <li class="list-group-item tree-closed">
                    <span><i class="glyphicon glyphicon-ok"></i> 业务审核 <span class="badge" style="float:right">3</span></span>
                    <ul style="margin-top:10px;display:none;">
                        <li style="height:30px;">
                            <a href="auth_cert.html"><i class="glyphicon glyphicon-check"></i> 实名认证审核</a>
                        </li>
                        <li style="height:30px;">
                            <a href="auth_adv.html"><i class="glyphicon glyphicon-check"></i> 广告审核</a>
                        </li>
                        <li style="height:30px;">
                            <a href="auth_project.html"><i class="glyphicon glyphicon-check"></i> 项目审核</a>
                        </li>
                    </ul>
                </li>
                <li class="list-group-item tree-closed">
                    <span><i class="glyphicon glyphicon-th-large"></i> 业务管理 <span class="badge" style="float:right">7</span></span>
                    <ul style="margin-top:10px;display:none;">
                        <li style="height:30px;">
                            <a href="cert.html"><i class="glyphicon glyphicon-picture"></i> 资质维护</a>
                        </li>
                        <li style="height:30px;">
                            <a href="type.html"><i class="glyphicon glyphicon-equalizer"></i> 分类管理</a>
                        </li>
                        <li style="height:30px;">
                            <a href="process.html"><i class="glyphicon glyphicon-random"></i> 流程管理</a>
                        </li>
                        <li style="height:30px;">
                            <a href="advertisement.html"><i class="glyphicon glyphicon-hdd"></i> 广告管理</a>
                        </li>
                        <li style="height:30px;">
                            <a href="message.html"><i class="glyphicon glyphicon-comment"></i> 消息模板</a>
                        </li>
                        <li style="height:30px;">
                            <a href="project_type.html"><i class="glyphicon glyphicon-list"></i> 项目分类</a>
                        </li>
                        <li style="height:30px;">
                            <a href="tag.html"><i class="glyphicon glyphicon-tags"></i> 项目标签</a>
                        </li>
                    </ul>
                </li>
                <li class="list-group-item tree-closed" >
                    <a href="param.html"><i class="glyphicon glyphicon-list-alt"></i> 参数管理</a>
                </li>
            </ul>
        </div>
    </div>
    ```
5. admin-main.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <!DOCTYPE html>
    <html lang="zh-CN">
    <%@include file="include-head.jsp"%>

    <body>
    <%@include file="include-nav.jsp"%>
    <div class="container-fluid">
        <div class="row">
            <%@include file="include-sidebar.jsp"%>
            <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
                <h1 class="page-header">控制面板</h1>

                <div class="row placeholders">
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/sky" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/vine" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/sky" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                    <div class="col-xs-6 col-sm-3 placeholder">
                        <img data-src="holder.js/200x200/auto/vine" class="img-responsive" alt="Generic placeholder thumbnail">
                        <h4>Label</h4>
                        <span class="text-muted">Something else</span>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </body>
    </html>
    ```

### 3.1.4 登录状态检查
1. 创建异常类:util模块 com/atguigu/crowd/exception/AccessForbidden.java
    ```java
    /**
     * 用户未登录访问收保护资源时抛出的异常
     * @author hjp
     */
    public class AccessForbidden extends RuntimeException{

        private static final long serialVersionUID = 1L;

        public AccessForbidden() {
        }

        public AccessForbidden(String message) {
            super(message);
        }

        public AccessForbidden(String message, Throwable cause) {
            super(message, cause);
        }

        public AccessForbidden(Throwable cause) {
            super(cause);
        }

        public AccessForbidden(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
            super(message, cause, enableSuppression, writableStackTrace);
        }
    }
    ```
2. 创建拦截器类component模块com/atguigu/crowd/mvc/interceptor/LoginInterceptor.java
    ```java
    public class LoginInterceptor extends HandlerInterceptorAdapter {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            // 1.通过request对象获取Session对象
            HttpSession session = request.getSession();
            // 2.尝试从Session域获取Admin对象
            Admin admin = (Admin) session.getAttribute(CrowdConstant.ATTR_NAME_LOGIN_ADMIN);
            // 3.判断Admin对象是否为空
            if (admin == null) {
                // 4.抛出异常
                throw new AccessForbidden(CrowdConstant.MESSAGE_ACCESS_FORBIDDEN);
            }
            // 5. 若admin不为空，则返回true放行
            return true;
        }
    }
    ```
3. 注册拦截器类：mvc配置文件
    ```xml
    <!--注册拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
            <!--要拦截的路径-->
            <mvc:mapping path="/**"/>
            <!--不拦截的资源-->
            <mvc:exclude-mapping path="/admin/to/login.do"/>
            <mvc:exclude-mapping path="/admin/do/login.do"/>
            <mvc:exclude-mapping path="/admin/do/logout.do"/>
            <!--配置拦截器类-->
            <bean class="com.atguigu.crowd.mvc.interceptor.LoginInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
    ```
4. 异常映射
   * 配置异常映射，当出现未登录异常时跳转到登录页面
   * 基于xml：mvc配置文件
        ```xml
        <!--配置基于xml的异常映射-->
        <bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
            <!-- 配置异常类型和具体视图页面对应关系 -->
            <property name="exceptionMappings">
                <props>
                    <!-- key属性指定异常全类名，标签体中指定对应的视图(前后缀拼接) -->
                    <prop key="java.lang.Exception">system-error</prop>
                    <prop key="com.atguigu.crowd.exception.AccessForbiddenException">admin-login</prop>
                </props>
            </property>
        </bean>
        ```
   * 基于注解：component模块 `CrowdExceptionResolver.java`
        ```java
        // 未登录异常
        @ExceptionHandler(value = AccessForbiddenException.class)
        public ModelAndView resolveAccessForbiddenException(AccessForbiddenException exception,
                                                            HttpServletRequest request,
                                                            HttpServletResponse response) throws IOException {
            String viewName = "admin-login";
            return commonResolve(viewName,exception,request,response);
        }
        ```

## 3.2 管理员维护
### 3.2.1 分页显示管理员信息
1. 思路
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/3-2-1.png)
2. 技术点
   * 让 SQL 语句针对 keyword 时有时无的情况进行适配，使用 SQL 中做字符串连接的函数： CONCAT("%",#{keyword},"%")
     * keyword 有值： “like %tom%”
     * keyword 无值： “like %%”
   * PageHelper 使用：数据分页
3. mybatis配置文件中配置插件
    ```xml
    <!--配置SqlSessionFactoryBean整合MyBatis-->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--指定mybatis全局配置文件位置-->
        <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
        <!--指定mapper配置文件位置-->
        <property name="mapperLocations" value="classpath:mybatis/mapper/*Mapper.xml"/>
        <!--装配数据源：引用前面的dataSource数据源-->
        <property name="dataSource" ref="dataSource"/>

        <!--配置插件-->
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageInterceptor">
                    <property name="properties">
                        <value>
                            <!--配置数据库方言，告诉PageHelper-->
                            helperDialect=mysql
                            <!-- 配置页码的合理化修正，在1~ 总页数之间修正页码-->
                            reasonable=true
                        </value>
                    </property>
                </bean>
            </array>
        </property>
    </bean>
    ```
4. mapper
   * AdminMapper.java
      ```java
      List<Admin> selectAdminByKeyword(String keyword);
      ```
   * AdminMapper.xml
      ```xml
      <select id="selectAdminByKeyword" resultMap="BaseResultMap">
        select id, login_acct, user_pswd, user_name, email,   create_time
          from t_admin
          where login_acct like concat("%",#{keyword},"%") or
                user_name like concat("%",#{keyword},"%") or
                email like concat("%",#{keyword},"%")
      </select>
      ```
5. service
   * AdminService
        ```java
        PageInfo<Admin> getPageInfo(String keyword, Integer pageNum, Integer pageSize)
        ```

   * AdminServiceImpl
        ```java
        @Override
        public PageInfo<Admin> getPageInfo(String keyword, Integer pageNum, Integer pageSize) {
            // 1.调用PageHelper的静态方法开启分页功能
            PageHelper.startPage(pageNum,pageSize);
            // 2.执行查询
            List<Admin> list = adminMapper.selectAdminByKeyword(keyword);
            // 3.封装到PageInfo对象
            return new PageInfo<>(list);
        }
        ```

6. handler
    ```java
    @RequestMapping(value = "/admin/get/page.do")
    public String getPageInfo(
            // 使用defaultValue指定默认值
            @RequestParam(value = "keyword",defaultValue = "") String keyword,
            @RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum,
            @RequestParam(value = "pageSize", defaultValue = "5") Integer pageSize,
            ModelMap modelMap
        ) {
        // 调用service方法获取PageInfo对象
        PageInfo<Admin> pageInfo = adminService.getPageInfo(keyword, pageNum, pageSize);
        // 将PageInfo对象存入模型
        modelMap.addAttribute(CrowdConstant.ATTR_NAME_PAGE_INFO, pageInfo);
        return "admin-page";
    }
    ```
7. admin-page.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <!DOCTYPE html>
    <html lang="zh-CN">
    <%@include file="include-head.jsp" %>

    <body>
    <%@include file="include-nav.jsp" %>
    <div class="container-fluid">
        <div class="row">
            <%@include file="include-sidebar.jsp" %>
            <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
                <div class="panel panel-default">
                    <div class="panel-heading">
                        <h3 class="panel-title"><i class="glyphicon glyphicon-th"></i> 数据列表</h3>
                    </div>
                    <div class="panel-body">
                        <form class="form-inline" role="form" style="float:left;">
                            <div class="form-group has-feedback">
                                <div class="input-group">
                                    <div class="input-group-addon">查询条件</div>
                                    <input class="form-control has-success" type="text" placeholder="请输入查询条件">
                                </div>
                            </div>
                            <button type="button" class="btn btn-warning"><i class="glyphicon glyphicon-search"></i> 查询</button>
                        </form>
                        <button type="button" class="btn btn-danger" style="float:right;margin-left:10px;"><i class=" glyphicon glyphicon-remove"></i> 删除</button>
                        <button type="button" class="btn btn-primary" style="float:right;" onclick="window.location.href='add.html'"><i class="glyphicon glyphicon-plus"></i> 新增</button>
                        <br>
                        <hr style="clear:both;">
                        <div class="table-responsive">
                            <table class="table  table-bordered">
                                <thead>
                                <tr>
                                    <th width="30">#</th>
                                    <th width="30"><input type="checkbox"></th>
                                    <th>账号</th>
                                    <th>名称</th>
                                    <th>邮箱地址</th>
                                    <th width="100">操作</th>
                                </tr>
                                </thead>
                                <tbody>
                                <c:if test="${empty requestScope.pageInfo.list}">
                                    <tr>
                                        <td colspan="6" align="center">抱歉！没有查询到你要的数据</td>
                                    </tr>
                                </c:if>
                                <c:if test="${!empty requestScope.pageInfo.list}">
                                    <c:forEach items="${requestScope.pageInfo.list}" var="admin" varStatus="myStatus">
                                        <tr>
                                            <td>${myStatus.count}</td>
                                            <td><input type="checkbox"></td>
                                            <td>${admin.login}</td>
                                            <td>${admin.userName}</td>
                                            <td>${admin.email}</td>
                                            <td>
                                                <button type="button" class="btn btn-success btn-xs"><i class=" glyphicon glyphicon-check"></i></button>
                                                <button type="button" class="btn btn-primary btn-xs"><i class=" glyphicon glyphicon-pencil"></i></button>
                                                <button type="button" class="btn btn-danger btn-xs"><i class=" glyphicon glyphicon-remove"></i></button>
                                            </td>
                                        </tr>
                                    </c:forEach>
                                </c:if>

                                </tbody>
                                <tfoot>
                                <tr>
                                    <td colspan="6" align="center">
                                        <ul class="pagination">
                                            <li class="disabled"><a href="#">上一页</a></li>
                                            <li class="active"><a href="#">1 <span class="sr-only">(current)</span></a></li>
                                            <li><a href="#">2</a></li>
                                            <li><a href="#">3</a></li>
                                            <li><a href="#">4</a></li>
                                            <li><a href="#">5</a></li>
                                            <li><a href="#">下一页</a></li>
                                        </ul>
                                    </td>
                                </tr>

                                </tfoot>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </body>
    </html>
    ```
8. 在component中加入依赖，以使用JSTL
    ```xml
    <!-- https://mvnrepository.com/artifact/taglibs/standard -->
    <dependency>
        <groupId>taglibs</groupId>
        <artifactId>standard</artifactId>
        <version>1.1.2</version>
    </dependency>
    ```

9. 在页面上使用pagination实现分页
   * 将pagination.css和jquery.pagination.js文件拷贝到相应目录
   * 在页面中引入
        ```jsp
        <%@include file="include-head.jsp" %>
        <link rel="stylesheet" href="css/pagination.css">
        <script type="text/javascript" src="jquery/jquery.pagination.js"></script>
        ```

   * 修改页码显示部分
        ```html
        <tfoot>
        <tr>
            <td colspan="6" align="center">
                <div id="Pagination" class="pagination"><!-- 这里显示分页 --></div>
            </td>
        </tr>
        </tfoot>
        ```
   * 增加js代码
        ```jsp
        <script type="text/javascript">
            $(function (){
                // 调用后面声明的函数对页码导航条进行初始化操作
                initPagination();
            });
            function initPagination() {
                // 获取总记录数
                var totalRecord = ${requestScope.pageInfo.total};
                // 声明一个JSON对象存储Pagination要设置的属性
                var properties = {
                    num_edge_entries: 3, // 边缘页数
                    num_display_entries: 5, // 主体页数
                    callback: pageSelectCallback,
                    items_per_page:${requestScope.pageInfo.pageSize}, // 每页显示1项
                    current_page: ${requestScope.pageInfo.pageNum - 1}, // Pagination内部使用pageIndex来管理页码，从0开始，而pageNum从1开始
                    prev_text: "上一页",
                    next_text: "下一页"
                };
                // 生成页码导航条
                $("#Pagination").pagination(totalRecord, properties);
            }
            // 用户点击页码时调用该函数实现跳转
            function pageSelectCallback(pageIndex,jQuery) {
                // 根据pageIndex计算得到pageNum
                var pageNum = pageIndex + 1;
                // 跳转页码
                window.location.href = "admin/get/page.do?pageNum="+pageNum;
                // 由于每一个页码按钮都是超链接，所以在这个函数最后取消超链接的默认行为
                return false;
            }
        </script>
        ```
   * 修改pagination.js
        ```js
        // 回调函数
		// opts.callback(current_page, this);
        ``` 
10. 测试数据（CrowdTest）
    ```java
    @Test
    public void testCreateData(){
        Admin admin;
        for (int i = 0; i < 10; i++) {
            String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
            String password = String.valueOf((int)(Math.random()*900000 + 100000));;
            String userPswd = CrowdUtil.md5(password);
            String login = getStringRandom(5);
            String userName = login.substring(0,3);
            admin = new Admin(null,captureName(login),userPswd,userName,userName+"@qq.com",date);
            adminMapper.insert(admin);
        }
    }
    //生成随机用户名，数字和字母组成,
    public static String getStringRandom(int length) {
        String val = "";
        Random random = new Random();

        //参数length，表示生成几位随机数
        for(int i = 0; i < length; i++) {
            val += (char)(random.nextInt(26)+97);
        }
        return val;
    }
    public static String captureName(String name) {
        char[] cs=name.toCharArray();
        cs[0]-=32;
        return String.valueOf(cs);
    }
    ```

### 3.2.2 关键词查询
1. 设置查询表单
    ```html
    <form action="admin/get/page.do" class="form-inline" role="form" style="float:left;" method="post">
        <div class="form-group has-feedback">
            <div class="input-group">
                <div class="input-group-addon">查询条件</div>
                <input name="keyword" class="form-control has-success" type="text" placeholder="请输入查询条件">
            </div>
        </div>
        <button type="submit" class="btn btn-warning"><i class="glyphicon glyphicon-search"></i> 查询</button>
    </form>
    ```
2. 翻页时保持 keyword 值，修改js代码：
    ```js
    // 跳转页码
    window.location.href = "admin/get/page.do?pageNum="+pageNum+"&keyword=${param.keyword}";
    ```

### 3.2.2 删除成员
1. handler
    ```java
    // 删除成员
    @RequestMapping("/admin/remove/{adminId}/{pageNum}/{keyword}.do")
    public String remove(@PathVariable("adminId") Integer adminId,
                         @PathVariable("pageNum") Integer pageNum,
                         @PathVariable("keyword") String keyword) {
        // 执行删除
        adminService.remove(adminId);
        // 页面跳转：回到分页页面
        return "redirect:/admin/get/page.do?pageNum="+pageNum+"&keyword="+keyword;
    }
    ```

2. service
    ```java
    void remove(Integer adminId);
    ```
    ```java
    @Override
    public void remove(Integer adminId) {
        adminMapper.deleteByPrimaryKey(adminId);
    }
    ```

3. 前端页面
    ```html
    <a href="admin/remove/${admin.id}/${requestScope.pageInfo.pageNum}/${param.keyword}.do" class="btn btn-danger btn-xs"><i class="glyphicon glyphicon-remove"></i> </a>
    ```

### 3.2.3 新增成员
1. 修改数据库
    ```mysql
    # 为用户名添加唯一约束
    ALTER TABLE `t_admin` ADD UNIQUE INDEX (`login`);
    ```
2. handler
    ```java
    // 新增成员
    @RequestMapping("/admin/save.do")
    public String save(Admin admin) {
        adminService.saveAdmin(admin);
        return "redirect:/admin/get/page.do?pageNum="+Integer.MAX_VALUE;
    }
    ```

3. service
    ```java
    @Override
    public void saveAdmin(Admin admin) {
        // 密码加密
        admin.setUserPswd(CrowdUtil.md5(admin.getUserPswd()));
        // 创建时间
        admin.setCreateTime(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
        // 用户名已存在错误：
        try {
            adminMapper.insert(admin);
        } catch (Exception e) {
            e.printStackTrace();
            logger.info("异常全类名"+e.getClass().getName());
            if (e instanceof DuplicateKeyException) {
                throw new LoginAcctAlreadyInUseException(CrowdConstant.MESSAGE_LOGIN_ACCT_ALREADY_IN_USE);
            }
        }
    }
    ```
4. 前端页面  
    admin-add.jsp：
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <!DOCTYPE html>
    <html lang="zh-CN">
    <%@include file="include-head.jsp" %>

    <body>
    <%@include file="include-nav.jsp" %>
    <div class="container-fluid">
        <div class="row">
            <%@include file="include-sidebar.jsp" %>
            <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
                <ol class="breadcrumb">
                    <li><a href="admin/to/main.do">首页</a></li>
                    <li><a href="admin/get/page.do">数据列表</a></li>
                    <li class="active">新增</li>
                </ol>
                <div class="panel panel-default">
                    <div class="panel-heading">表单数据<div style="float:right;cursor:pointer;" data-toggle="modal" data-target="#myModal"><i class="glyphicon glyphicon-question-sign"></i></div></div>
                    <div class="panel-body">
                        <form action="admin/save.do" method="post" role="form">
                            <p>${requestScope.exception.message}</p>
                            <div class="form-group">
                                <label for="InputLogin">登录账号</label>
                                <input type="text" name="login" class="form-control" id="InputLogin" placeholder="请输入登陆账号">
                            </div>
                            <div class="form-group">
                                <label for="InputPassword">登录密码</label>
                                <input type="text" name="userPswd" class="form-control" id="InputPassword" placeholder="请输入登陆账号">
                            </div>
                            <div class="form-group">
                                <label for="InputUserNmae">用户昵称</label>
                                <input type="text" name="userName" class="form-control" id="InputUserNmae" placeholder="请输入用户名称">
                            </div>
                            <div class="form-group">
                                <label for="exampleInputEmail1">邮箱地址</label>
                                <input type="email" name="email" class="form-control" id="exampleInputEmail1" placeholder="请输入邮箱地址">
                                <p class="help-block label label-warning">请输入合法的邮箱地址, 格式为： xxxx@xxxx.com</p>
                            </div>
                            <button type="submit" class="btn btn-success"><i class="glyphicon glyphicon-plus"></i> 新增</button>
                            <button type="reset" class="btn btn-danger"><i class="glyphicon glyphicon-refresh"></i> 重置</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </body>
    </html>
    ```
    修改admin-page.jsp：
    ```jsp
    <%--<button type="button" class="btn btn-success btn-xs"><i class=" glyphicon glyphicon-check"></i></button>--%>
    <a href="admin/to/add.do" class="btn btn-success btn-xs"><i class=" glyphicon glyphicon-check"></i></a>
    ```
5. 异常处理
   * 新建异常类
        ```java
        package com.atguigu.crowd.exception;

        /**
        * 保存或更新Admin时账号重复
        * @author hjp
        */
        public class LoginAcctAlreadyInUseException extends RuntimeException{

            private static final long serialVersion = 1L;

            public LoginAcctAlreadyInUseException() {
            }

            public LoginAcctAlreadyInUseException(String message) {
                super(message);
            }

            public LoginAcctAlreadyInUseException(String message, Throwable cause) {
                super(message, cause);
            }

            public LoginAcctAlreadyInUseException(Throwable cause) {
                super(cause);
            }

            public LoginAcctAlreadyInUseException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
                super(message, cause, enableSuppression, writableStackTrace);
            }
        }
        ```

   * 异常处理
        ```java
        // 用户名重复异常
        @ExceptionHandler(value = LoginAcctAlreadyInUseException.class)
        public ModelAndView resolveLoginAcctAlreadyInUseException(LoginAcctAlreadyInUseException exception,
                                                        HttpServletRequest request,
                                                        HttpServletResponse response) throws IOException {
            String viewName = "admin-add";
            return commonResolve(viewName, exception, request, response);
        }
        ```

### 3.2.4 更新信息
1. 前端页面
   * admin-page.jsp
        ```jsp
        <%--<button id="testAdmin" type="button" class="btn btn-primary btn-xs"><i class=" glyphicon glyphicon-pencil"></i></button>--%>
        <a href="admin/to/edit.do?adminId=${admin.id }&pageNum=${requestScope.pageInfo.pageNum}&keyword=${param.keyword}" class="btn btn-primary btn-xs"><i class=" glyphicon glyphicon-pencil"></i></a>
        ```
   * admin-edit.jsp
        ```jsp
        <%@ page contentType="text/html;charset=UTF-8" language="java" %>
        <!DOCTYPE html>
        <html lang="zh-CN">
        <%@include file="include-head.jsp" %>

        <body>
        <%@include file="include-nav.jsp" %>
        <div class="container-fluid">
            <div class="row">
                <%@include file="include-sidebar.jsp" %>
                <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
                    <ol class="breadcrumb">
                        <li><a href="admin/to/main.do">首页</a></li>
                        <li><a href="admin/get/page.do">数据列表</a></li>
                        <li class="active">更新</li>
                    </ol>
                    <div class="panel panel-default">
                        <div class="panel-heading">表单数据<div style="float:right;cursor:pointer;" data-toggle="modal" data-target="#myModal"><i class="glyphicon glyphicon-question-sign"></i></div></div>
                        <div class="panel-body">
                            <form action="admin/update.do" method="post" role="form">
                                <input type="hidden" name="id" value="${requestScope.admin.id}">
                                <input type="hidden" name="pageNum" value="${param.pageNum}">
                                <input type="hidden" name="keyword" value="${param.keyword}">
                                <p>${requestScope.exception.message}</p>
                                <div class="form-group">
                                    <label for="InputLogin">登录账号</label>
                                    <input type="text" name="login" value="${requestScope.admin.login}" class="form-control" id="InputLogin" placeholder="请输入登陆账号">
                                </div>
                                <div class="form-group">
                                    <label for="InputUserNmae">用户昵称</label>
                                    <input type="text" name="userName" value="${requestScope.admin.userName}" class="form-control" id="InputUserNmae" placeholder="请输入用户名称">
                                </div>
                                <div class="form-group">
                                    <label for="exampleInputEmail1">邮箱地址</label>
                                    <input type="email" name="email" value="${requestScope.admin.email}" class="form-control" id="exampleInputEmail1" placeholder="请输入邮箱地址">
                                    <p class="help-block label label-warning">请输入合法的邮箱地址, 格式为： xxxx@xxxx.com</p>
                                </div>
                                <button type="submit" class="btn btn-success"><i class="glyphicon glyphicon-edit"></i> 更新</button>
                                <button type="reset" class="btn btn-danger"><i class="glyphicon glyphicon-refresh"></i> 重置</button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        </body>
        </html>
        ```
2. handler
    * 转到提交页面
        ```java
        // 修改页面
        @RequestMapping("/admin/to/edit.do")
        public String toEdit(@RequestParam("adminId") Integer adminId,
                            ModelMap modelMap) {
            Admin admin = adminService.getAdminById(adminId);
            modelMap.addAttribute("admin",admin);
            return "admin-edit";
        }
        ```

    * 提交信息
        ```java
        // 提交更新信息
        @RequestMapping(value = "/admin/update.do")
        public String update(Admin admin,
                            @RequestParam(value = "keyword",defaultValue = "") String keyword,
                            @RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum) {
            adminService.update(admin);
            return "redirect:/admin/get/page.do?pageNum="+pageNum+"&keyword="+keyword;
        }
        ```
3. servcice
   * AdminService
        ```java
        void update(Admin admin);
        ```
   * AdminServiceImpl
        ```java
        @Override
        public void update(Admin admin) {
            // 有选择的更新，对于null值字段不更新
            try {
                adminMapper.updateByPrimaryKeySelective(admin);
            } catch (Exception e) {
                e.printStackTrace();
                // 用户名重复：
                logger.info("异常全类名"+e.getClass().getName());
                if (e instanceof DuplicateKeyException) {
                    throw new LoginAcctAlreadyInUseForUpdateException(CrowdConstant.MESSAGE_LOGIN_ACCT_ALREADY_IN_USE);
                }
            }
        }
        ```

4. 异常
   * 新建异常类
        ```java
        package com.atguigu.crowd.exception;
        public class LoginAcctAlreadyInUseForUpdateException extends RuntimeException{

            private static final long serialVersion = 1L;

            public LoginAcctAlreadyInUseForUpdateException() {
            }

            public LoginAcctAlreadyInUseForUpdateException(String message) {
                super(message);
            }

            public LoginAcctAlreadyInUseForUpdateException(String message, Throwable cause) {
                super(message, cause);
            }

            public LoginAcctAlreadyInUseForUpdateException(Throwable cause) {
                super(cause);
            }

            public LoginAcctAlreadyInUseForUpdateException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
                super(message, cause, enableSuppression, writableStackTrace);
            }
        }
        ```

   * 异常处理
        ```java
        // 用户名重复异常(更新时)
        @ExceptionHandler(value = LoginAcctAlreadyInUseForUpdateException.class)
        public ModelAndView resolveLoginAcctAlreadyInUseForUpdateException(LoginAcctAlreadyInUseForUpdateException exception,
                                                        HttpServletRequest request,
                                                        HttpServletResponse response) throws IOException {
            String viewName = "system-error";
            System.out.println(viewName);
            return commonResolve(viewName, exception, request, response);
        }
        ```



