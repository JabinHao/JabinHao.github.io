---
title: Chapter14 — 前台 发起项目
excerpt: 前台建模、发起众筹项目并保存信息
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
abbrlink: cfd916b7
date: 2021-01-26 20:12:58
updated: 2021-01-27 05:01:11
subtitle:
---
## 14.1 建模

### 14.1.1 创建数据库表

1. 分类表

    ```sql
    CREATE TABLE t_type
    (
        id      INT(11) NOT NULL AUTO_INCREMENT,
        name    VARCHAR(255) COMMENT '分类名称',
        remark  VARCHAR(255) COMMENT '分类介绍',
        PRIMARY KEY (id)
    );
    ```

2. 项目-分类中间表

    ```sql
    CREATE TABLE t_project_type
    (
        id INT NOT NULL AUTO_INCREMENT,
        projectid INT(11),
        typeid INT(11),
        PRIMARY KEY (id)
    );
    ````

3. 标签表

    ```sql
    CREATE TABLE t_tag
    (
        id INT(11) NOT NULL AUTO_INCREMENT,
        pid INT(11),
        name VARCHAR(255),
        PRIMARY KEY (id)
    );
    ````

4. 项目-标签中间表

    ```sql
    CREATE TABLE t_project_tag
    (
        id INT(11) NOT NULL AUTO_INCREMENT,
        projectid INT(11),
        tagid INT(11),
        PRIMARY KEY (id)
    );
    ````

5. 项目表

    ```sql
    CREATE TABLE t_project
    (
        id INT(11) NOT NULL AUTO_INCREMENT,
        project_name VARCHAR(255) COMMENT '项目名称',
        project_description VARCHAR(255) COMMENT '项目描述',
        money BIGINT (11) COMMENT '筹集金额',
        day INT(11) COMMENT '筹集天数',
        status INT(4) COMMENT '0-即将开始， 1-众筹中， 2-众筹成功， 3-众筹失败',
        deploydate VARCHAR(10) COMMENT '项目发起时间',
        supportmoney BIGINT(11) COMMENT '已筹集到的金额',
        supporter INT(11) COMMENT '支持人数',
        completion INT(3) COMMENT '百分比完成度',
        memberid INT(11) COMMENT '发起人的会员 id',
        createdate VARCHAR(19) COMMENT '项目创建时间',
        follower INT(11) COMMENT '关注人数',
        header_picture_path VARCHAR(255) COMMENT '头图路径',
        PRIMARY KEY (id)
    );
    ````

6. 项目-项目详情图片表

    ```sql
    CREATE TABLE t_project_item_pic
    (
        id INT(11) NOT NULL AUTO_INCREMENT,
        projectid INT(11),
        item_pic_path VARCHAR(255),
        PRIMARY KEY (id)
    );
    ```

7. 项目发起人信息表

    ```sql
    CREATE TABLE t_member_launch_info
    (
        id INT(11) NOT NULL AUTO_INCREMENT,
        memberid INT(11) COMMENT '会员 id',
        description_simple VARCHAR(255) COMMENT '简单介绍',
        description_detail VARCHAR(255) COMMENT '详细介绍',
        phone_num VARCHAR(255) COMMENT '联系电话',
        service_num VARCHAR(255) COMMENT '客服电话',
        PRIMARY KEY (id)
    );
    ```

8. 回报信息表

    ```sql
    CREATE TABLE t_return
    (
        id INT(11) NOT NULL AUTO_INCREMENT,
        projectid INT(11),
        type INT(4) COMMENT '0 - 实物回报， 1 虚拟物品回报',
        supportmoney INT(11) COMMENT '支持金额',
        content VARCHAR(255) COMMENT '回报内容',
        count INT(11) COMMENT '回报产品限额， “0” 为不限回报数量',
        signalpurchase INT(11) COMMENT '是否设置单笔限购',
        purchase INT(11) COMMENT '具体限购数量',
        freight INT(11) COMMENT '运费， “0” 为包邮',
        invoice INT(4) COMMENT '0 - 不开发票， 1 - 开发票',
        returndate INT(11) COMMENT '项目结束后多少天向支持者发送回报',
        describ_pic_path VARCHAR(255) COMMENT '说明图片路径',
        PRIMARY KEY (id)
    );
    ```

9. 发起人确认信息表

    ```sql
    CREATE TABLE t_member_confirm_info
    (
        id INT(11) NOT NULL AUTO_INCREMENT,
        memberid INT(11) COMMENT '会员 id',
        paynum VARCHAR(200) COMMENT '易付宝企业账号',
        cardnum VARCHAR(200) COMMENT '法人身份证号',
        PRIMARY KEY (id)
    );
    ```

### 14.1.2 逆向工程

1. 修改 generatorConfig.xml

    ```xml
    <!--  数据库表名字和我们的 entity 类对应的映射指定 -->
    <table tableName="t_type" domainObjectName="TypePO" />
    <table tableName="t_tag" domainObjectName="TagPO" />
    <table tableName="t_project" domainObjectName="ProjectPO" />
    <table tableName="t_project_item_pic" domainObjectName="ProjectItemPicPO" />
    <table tableName="t_member_launch_info" domainObjectName="MemberLaunchInfoPO" />
    <table tableName="t_return" domainObjectName="ReturnPO" />
    <table tableName="t_member_confirm_info" domainObjectName="MemberConfirmInfoPO" />
    ```

2. 资源归位
    * entity下的PO及POExample类移动到member-entity模块po包下
    * mapper类移动到mysql模块mapper目录下
    * mapper.xml文件移动到mysql模块resources/mybatis/mapper目录下

3. mysql模块handler

    ```java
    @RestController
    public class ProjectProviderHandler {

        @Autowired
        private ProjectService projectService;
    }
    ```

4. mysql模块service

    ```java
    public interface ProjectService {

    }
    ```

    ```java
    @Transactional(readOnly = true)
    @Service
    public class ProjectServiceImpl implements ProjectService {

    }
    ````

### 14.1.3 创建 VO 对象
1. ProjectVO

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class ProjectVO implements Serializable {
        private static final long serialVersionUID = 1L;
        // 分类 id 集合
        private List<Integer> typeIdList;
        // 标签 id 集合
        private List<Integer> tagIdList;
        // 项目名称
        private String projectName;
        // 项目描述
        private String projectDescription;
        // 计划筹集的金额
        private Integer money;
        // 筹集资金的天数
        private Integer day;
        // 创建项目的日期
        private String createdate;
        // 头图的路径
        private String headerPicturePath;
        // 详情图片的路径
        private List<String> detailPicturePathList;
        // 发起人信息
        private MemberLauchInfoVO memberLauchInfoVO;
        // 回报信息集合
        private List<ReturnVO> returnVOList;
        // 发起人确认信息
        private MemberConfirmInfoVO memberConfirmInfoVO;
    }
    ```

2. MemberLauchInfoVO

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class MemberLauchInfoVO implements Serializable {

        private static final long serialVersionUID = 1L;
        // 简单介绍
        private String descriptionSimple;
        // 详细介绍
        private String descriptionDetail;
        // 联系电话
        private String phoneNum;
        // 客服电话
        private String serviceNum;
    }
    ```

3. ReturnVO

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class ReturnVO implements Serializable {
        private static final long serialVersionUID = 1L;
        // 回报类型： 0 - 实物回报， 1 虚拟物品回报
        private Integer type;
        // 支持金额
        private Integer supportmoney;
        // 回报内容介绍
        private String content;
        // 总回报数量， 0 为不限制
        private Integer count;
        // 是否限制单笔购买数量， 0 表示不限购， 1 表示限购
        private Integer signalpurchase;
        // 如果单笔限购， 那么具体的限购数量
        private Integer purchase;
        // 运费， “0” 为包邮
        private Integer freight;
        // 是否开发票， 0 - 不开发票， 1 - 开发票
        private Integer invoice;
        // 众筹结束后返还回报物品天数
        private Integer returndate;
        // 说明图片路径
        private String describPicPath;
    }
    ```

4. MemberConfirmInfoVO

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class MemberConfirmInfoVO implements Serializable {
        private static final long serialVersionUID = 1L;
        // 易付宝账号
        private String paynum;
        // 法人身份证号
        private String cardnum;
    }
    ```

## 14.2 发起项目

### 14.2.1 目标思路

1. 目标：将各个表单页面提交的数据汇总到一起保存到数据库
2. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/14-2-1.png)

### 14.2.2 后端代码-project模块

1. 主启动类

    ```java
    // 启用Feign客户端功能
    @EnableFeignClients
    @EnableDiscoveryClient
    @SpringBootApplication
    public class ProjectConsumerMain {

        public static void main(String[] args) {
            SpringApplication.run(ProjectConsumerMain.class, args);
        }

    }
    ```

2. 新建 CrowdWebMvcConfig 类

    ```java
    @Configuration
    public class CrowdWebMvcConfig implements WebMvcConfigurer {

        @Override
        public void addViewControllers(ViewControllerRegistry registry) {

            // view-controller 是在 project-consumer 内部定义的， 所以这里是一个不经过 Zuul访问的地址， 所以这个路径前面不加路由规则中定义的前缀：“/project”
            registry.addViewController("/agree/protocol/page").setViewName("project-agree");
            registry.addViewController("/launch/project/page").setViewName("project-launch");
            registry.addViewController("/return/info/page").setViewName("project-return");
        }
    }
    ```

3. 修改 zuul 模块配置

    ```yml
    zuul:
    ignored-services: "*"
    sensitive-headers: "*"
    routes:
        crowd-portal:
        service-id: atguigu-crowd-auth
        path: /**
        crowd-project:
        service-id: atguigu-crowd-project
        path: /project/**
    ```

4. handler

    ```java
    @Controller
    public class ProjectConsumerHandler {

        @Autowired
        private OSSProperties ossProperties;

        @RequestMapping("/create/project/information")
        public String saveProjectBasicInfo(ProjectVO projectVO,
                                        MultipartFile headerPicture,
                                        List<MultipartFile> detailPictureList,
                                        HttpSession session,
                                        ModelMap modelMap) throws IOException {

            // 一、完成头图上传
            // 1.判断头图是否为空
            boolean headerPictureIsEmpty = headerPicture.isEmpty();

            if (headerPictureIsEmpty) {

                // 2.如果没有上传头图则返回到表单页面并显示错误消息
                modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_HEADER_PIC_EMPTY);
                return "project-launch";

            }

            // 3.如果用户确实上传了有内容的文件， 则执行上传
            ResultEntity<String> uploadHeaderPicResultEntity = CrowdUtil.uploadFileToOss(ossProperties.getEndPoint(),
                    ossProperties.getAccessKeyId(),
                    ossProperties.getAccessKeySecret(),
                    headerPicture.getInputStream(),
                    ossProperties.getBucketName(),
                    ossProperties.getBucketDomain(),
                    Objects.requireNonNull(headerPicture.getOriginalFilename()));

            String result = uploadHeaderPicResultEntity.getResult();

            // 4.判断头图是否上传成功
            if (ResultEntity.SUCCESS.equals(result)) {

                // 5.成功则获取图片访问路径
                String headerPicturePath = uploadHeaderPicResultEntity.getData();

                // 6.存入ProjectVO对象中
                projectVO.setHeaderPicturePath(headerPicturePath);

            }else {

                // 7.上传失败则返回上传页面，返回错误消息
                modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_HEADER_PIC_UPLOAD_FAILED);

                return "project-launch";
            }



            // 二、详情图片上传

            // 1.创建一个用来存放详情图片路径的集合
            List<String> detailPicturePathList = new ArrayList<String>();

            // 2.检查 detailPictureList 是否有效
            if (detailPictureList == null || detailPictureList.size() == 0) {

                modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_DETAIL_PIC_EMPTY);

                return "project-launch";
            }

            // 3.遍历 detailPictureList 集合
            for (MultipartFile detailPicture : detailPictureList) {

                // 4.判断是否为空
                if (detailPicture.isEmpty()) {

                    // 5.检测到详情图片中单个文件为空也是回去显示错误消息
                    modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_DETAIL_PIC_EMPTY);

                    return "project-launch";
                }

                // 6.执行上传
                ResultEntity<String> detailUploadResultEntity = CrowdUtil.uploadFileToOss(ossProperties.getEndPoint(),
                        ossProperties.getAccessKeyId(),
                        ossProperties.getAccessKeySecret(),
                        detailPicture.getInputStream(),
                        ossProperties.getBucketName(),
                        ossProperties.getBucketDomain(),
                        Objects.requireNonNull(detailPicture.getOriginalFilename()));

                // 7.检查上传结果
                String detailUploadResult = detailUploadResultEntity.getResult();
                if (ResultEntity.SUCCESS.equals(detailUploadResult)) {

                    String detailPicturePath = detailUploadResultEntity.getData();

                    // 8.收集刚刚上传的图片的访问路径
                    detailPicturePathList.add(detailPicturePath);
                }else {

                    // 9.如果上传失败则返回到表单页面并显示错误消息
                    modelMap.addAttribute(CrowdConstant.ATTR_NAME_MESSAGE, CrowdConstant.MESSAGE_DETAIL_PIC_UPLOAD_FAILED);

                    return "project-launch";
                }

            }
            // 10.将存放了详情图片访问路径的集合存入 ProjectVO 中
            projectVO.setDetailPicturePathList(detailPicturePathList);

            // 三、 后续操作
            // 1.将 ProjectVO 对象存入 Session 域
            session.setAttribute(CrowdConstant.ATTR_NAME_TEMPLE_PROJECT, projectVO);
            // 2.以完整的访问路径前往下一个收集回报信息的页面
            return "redirect:http://www.crowd.com/project/return/info/page";
        }
    }
    ```

5. 修改util模块 CrowdConstant

    ```java
    public static final String ATTR_NAME_TEMPLE_PROJECT = "tempProject";

    public static final String MESSAGE_HEADER_PIC_UPLOAD_FAILED = "头图上传失败";
    public static final String MESSAGE_HEADER_PIC_EMPTY = "头图不可为空！";
    public static final String MESSAGE_DETAIL_PIC_EMPTY = "详情图片不可为空！";
    public static final String MESSAGE_DETAIL_PIC_UPLOAD_FAILED = "详情图片上传失败";
    ```


### 14.2.3 前端代码

1. 修改 auth 模块 member-crowd.html

    ```html
    <button id="launchCrowdBtn" type="button" class="btn btn-warning"onclick="window.location.href='http://www.crowd.com.project/agree/protocol/page'">发起众筹</button>
    ```

2. project模块新建 project-agree.html

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

            .label-type, .label-status, .label-order {
                background-color: #fff;
                color: #f60;
                padding : 5px;
                border: 1px #f60 solid;
            }
            #typeList  :not(:first-child) {
                margin-top:20px;
            }
            .label-text {
                margin:0 10px;
            }
            .panel {
                border-radius:0;
            }
        </style>
        <script type="text/javascript" src="jquery/jquery-2.1.1.min.js"></script>
        <script type="text/javascript" src="bootstrap/js/bootstrap.min.js"></script>
        <script type="text/javascript" src="script/docs.min.js"></script>
        <script type="text/javascript" src="script/back-to-top.js"></script>
        <script type="text/javascript">
            $('#myTab a').click(function (e) {
                e.preventDefault()
                $(this).tab('show')
            })
        </script>
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
                                    <li><a href="member.html">会员中心</a></li>
                                    <li><a href="message.html">消息 <span class="badge badge-success">42</span></a></li>
                                    <li class="divider"></li>
                                    <li><a href="http://www.crowd.com/auth/member/logout">退出系统</a></li>
                                </ul>
                            </li>
                        </ul>
                    </div>
                </div>
            </nav>
        </div>
    </div>

    <div class="container theme-showcase" role="main">

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <nav class="navbar navbar-default" role="navigation">
                        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                            <ul class="nav navbar-nav">
                                <li>
                                    <a rel="nofollow" href="index.html"><i class="glyphicon glyphicon-home"></i> 众筹首页</a>
                                </li>
                                <li >
                                    <a rel="nofollow" href="projects.html"><i class="glyphicon glyphicon-th-large"></i> 项目总览</a>
                                </li>
                                <li class="active">
                                    <a rel="nofollow" href="javascript:;"><i class="glyphicon glyphicon-edit"></i> 发起项目</a>
                                </li>
                                <li>
                                    <a rel="nofollow" href="minecrowdfunding.html"><i class="glyphicon glyphicon-user"></i> 我的众筹</a>
                                </li>
                            </ul>
                        </div>
                    </nav>
                </div>
            </div>
        </div>


        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="panel panel-default" >
                        <div class="panel-heading" style="text-align:center;">
                            <h3 >
                                众筹平台项目发起人协议
                            </h3>
                        </div>
                        <div class="panel-body" style="height:400px;overflow-y:auto;padding:10px;">
                            <div class="test-box">
                                <p><b>在发起人使用尚筹网众筹平台提供的服务之前，请务必认真阅读本协议的全部内容。阅读本协议的过程中，如您不同意本协议或其中任何条款约定，您应立即停止注册（如：点击确认等后续操作），撤销注册流程。本协议一经发起人点击确认并同意接受，即产生法律效力。
                                </b></p>
                                <h5>第1条  签约主体</h5>
                                <p>本协议由尚筹网网站（包括但不限于www.atcrowd.com及尚筹网众筹平台）的所有者及其关联公司（以下简称为“尚筹网”），在尚筹网众筹平台（zc.atcrowd.com）与登录、使用尚筹网众筹平台的项目发起人（以下简称 “发起人”）共同签署（签约地：江苏省南京市）。</p>
                                <h5>第2条  定 义</h5>
                                <p>2.1 尚筹网众筹平台：尚筹网众筹平台是一个可以让发起人的项目计划变为现实的梦想平台，是“致力于中国创造，实现创新创业者梦想”的平台，发起人可以在尚筹网众筹平台上发起创新项目的筹款需求，并承诺提供不同形式的回报给项目的支持者，支持者可通过预购发起人的项目成果或相关产品来表达对发起人的支持。</p>
                                <p>2.2发起人：指在尚筹网众筹平台上发起项目的法人。</p>
                                <p>2.3支持者：指对发起人的项目表示支持的尚筹网用户，支持者通过支付一定金额的预购款的方式，预购项目的衍生成果及/或相关产品。</p>
                                <p>2.4筹款成功：指在发起人设定的时间内，发起人与支持者订立的以项目成果及/或相关产品为回报标的的合同金额达到或超过了发起人设定的筹款金额。</p>
                                <p>2.5项目成功：指项目筹款成功后，发起人执行项目计划并最终形成项目成果，并按照支持者预购订单的约定向支持者交付项目成果及/或相关产品。</p><p>
                            </p><h5>第3条  立约背景</h5>
                                <p>3.1为保护支持者的合法权益，规范发起人的行为，维护尚筹网众筹平台的秩序，特拟定本协议。</p>
                                <p>3.2尚筹网为发起人与支持者之间的交易行为提供平台网络空间、技术服务和支持，尚筹网并不是发起人或支持者中的任何一方，所有交易仅存在于发起人和支持者之间，使用尚筹网众筹平台产生的法律后果由发起人与支持者自行承担。</p>
                                <p>3.3尚筹网会对所发起的项目的合理性、项目内容与回报的匹配度等进行形式审核，审核通过后才可发布到尚筹网众筹平台，但是项目通过尚筹网的审核并不代表尚筹网保证项目合法或不侵犯任何第三方的权利，发起人仍需对所发起的项目独立承担法律责任。</p>
                                <h5>第4条  协议生效和适用范围</h5>
                                <p>4.1本协议内容包括协议正文以及尚筹网网站已经发布的或将来可能发布的各类规则、操作流程。所有规则为本协议不可分割的一部分，与协议正文具有同等的法律效力。</p>
                                <p>4.2当发起人在尚筹网众筹平台发起项目时，通过网络页面点击确认或以其他方式选择接受本协议，即表示与尚筹网达成协议并接受本协议的全部约定内容，本协议自发起人通过网络页面点击确认或以其他方式选择接受本协议之时起生效。本协议生效后，发起人不应以未阅读或不接受本协议的内容为由，主张本协议无效或要求撤销本协议。</p>
                                <p>4.3尚筹网有权根据需要不定时修改本协议或各类规则、操作流程，如本协议有任何变更，尚筹网将在网站上以公示形式通知，且无需征得发起人的事先同意。修改后的协议及规则一经公示即生效，成为本协议的一部分。如发起人继续登录或使用尚筹网众筹平台的，即视为已阅读并接受修改后的协议。</p>
                                <p>4.4发起人应该按照本协议约定行使权利并履行义务。如不能接受本协议的约定，包括但不限于不能接受修订后的协议及各类规则，则应立即停止使用尚筹网众筹平台提供的服务。如发起人继续使用服务，则表示同意并接受本协议及各类规则的约束。</p>
                                <h5>第5条  项目发起人资格</h5>
                                <p>5.1项目发起人应为尚筹网众筹平台的注册用户，完全理解并接受本协议。</p>
                                <p>5.2发起人可以为法人或其他组织且应是在中国境内合法成立、注册可独立承担法律责任的法律实体。</p>
                                <p>5.3发起人在发起项目前，应通过尚筹网易付宝完成必要的身份认证和资质认证，包括但不限于营业执照、法人身份证等的实名认证。发起人在发起项目时同意并授权尚筹网众筹平台公示认证信息。</p>
                                <p>5.4发起人应按照尚筹网的要求开立账户，以接收支持者的支持款。</p>
                                <p>5.5如发起人申请发起的项目为带有公益性质的项目，则发起人应是合法成立的公益组织或与合法成立的公益组织共同发起项目。</p>
                                <p>5.6尚筹网众筹平台将根据申请发起的项目类型不同，对发起人需要满足的其他资格进行限定和要求。</p>
                                <h5>第6条  项目内容规范</h5>
                                <p>6.1在尚筹网众筹平台上发起的项目应为具有创新性质且具有可执行性的项目，且项目目标须是明确、具体、可衡量的，如制作一个实物产品、拍一部微电影或完成一件艺术创作等。不得在无实质项目内容的情况下纯粹为公益组织发起募捐或以发起类似“资助奖学金”、“资助我去旅游”等为满足发起人个人需求之目的筹款。</p>
                                <p>6.2项目内容须符合法律法规及尚筹网网站的相关规定；尚筹网众筹平台有权对项目提出特殊要求。不得抄袭、盗用他人的成果发起众筹项目，创意类产品必须为原创。</p>
                                <p>6.3项目的内容必须包含“我想要做什么事情”、“项目风险”、“项目回报”、“为什么需要支持”等信息。同时，应向项目支持者充分说明项目存在的风险及挑战，以便于项目支持者对项目有全面充分的了解，从而独立慎重做出是否投资的决定。</p>
                                <p>6.4项目内容及发起人上传的相关项目信息（包含但不限于文字、图片、视频等）须为发起人原创，如非发起人原创，则发起人应已获得权利人的相应授权，且权利人允许发起人转授权给尚筹网及尚筹网的关联公司在尚筹网网站及尚筹网关联公司的其他官方网站及线下媒体出于宣传尚筹网众筹平台的目的而进行永久的免费宣传、推广、使用。</p>
                                <p>6.5项目不得含有攻击性、侮辱性言论，不得含有违反国家法律法规、政策的内容及其他尚筹网认为不适宜的内容。包括但不限于以下内容：</p>
                                <p>6.5.1违反国家法律规定的违禁品，如毒品、枪支弹药及管制刀具相关；</p>
                                <p>6.5.2非法、色情、淫秽、赌博、暴力、恐怖、反动、政治与宗教相关；</p>
                                <p>6.5.3开办公司、网站、店铺等相关；</p>
                                <p>6.5.4其他国家法律规定和尚筹网网站规定的禁限售等违禁品信息。</p>
                                <p>6.6项目回报发放完成前不允许对已经完成生产的商品进行销售，公益相关项目除外。</p>
                                <p>6.7 项目的排他性，项目在尚筹网众筹平台募集期内不得在其他任意平台同时发起类似项目或进行销售。一经发现，项目强行下架，并追究违约责任。</p>
                                <p>6.8尚筹网众筹平台对项目的审核仅针对项目的合理性、项目内容与回报的匹配度等进行审核，发起人应保证发起的项目内容合法，且不侵犯他人合法权益。</p>
                                <p>6.9在尚筹网众筹平台上发起项目时，应明确众筹开始时间和结束时间，以及项目众筹成功后的回报时间。截止项目结束时间，如项目众筹金额等于或大于预定众筹金额，则项目众筹成功；如项目众筹金额低于预定众筹金额，则项目众筹失败。</p>
                                <h5>第7条  项目回报规范</h5>
                                <p>7.1项目回报必须与项目具有关联性。回报应为项目进行过程中发起人产出的衍生成果或相关商品，公益相关项目除外。</p>
                                <p>7.2 发起人承诺，如项目众筹成功，及时兑现对项目支持者承诺的回报；如项目众筹失败，同意尚筹网将众筹款项及时退还项目支持者，并由发起人就项目众筹失败的原因等对项目支持者做出解释。</p>
                                <p>7.3项目回报产品在尚筹网众筹平台设置的最高单价应低于项目成功结束后<b>两个月内</b>的产品市场售价的<b>90%</b> ，否则应将售价差额发还支持者。</p>
                                <p>7.4项目回报为实物的，产品量产后，应最先发放给尚筹网众筹平台支持该项目的支持者，否则将视为发起人违约，需承担违约责任及罚金。</p>
                                <p>7.5如发起人与项目支持者在兑现回报过程中发生纠纷，一切责任由发起人承担，如因此给尚筹网造成经济或名誉损失，发起人应赔偿尚筹网相关损失。</p>
                                <p>7.6如发起人在兑现对项目支持者的回报的过程中，与第三方（包括但不限于物流公司）发生纠纷，一切责任由发起人承担。</p>
                                <h5>第8条  发起人行为规范</h5>
                                <p>8.1项目审核通过后，发起人除可修改项目启动时间外，不得编辑、修改其他项目内容。</p>
                                <p>8.2项目筹款成功前，发起人如因故需取消项目的，需向尚筹网众筹平台提交取消申请，尚筹网众筹平台审核通过后项目取消，所有筹集到的款项由尚筹网退回给支持者。</p>
                                <p>8.3项目筹款成功后，发起人应严格按照项目计划执行，并在项目计划执行过程中积极与支持者互动。</p>
                                <p>8.4项目筹款成功后，如项目因故延期或无法按原定项目计划执行的，或者因任何主观或客观因素导致项目最终无法完成的，发起人应第一时间通知尚筹网众筹平台，并及时通过尚筹网众筹平台及其他途径告知支持者，向支持者支付违约金，全额退还未履行订单的项目支持款。</p>
                                <p>8.5发起人在项目募集期间，不能以任何形式在其他平台上发起该项目。如违反该规定，尚筹网有权对发起人追究责任。</p>
                                <p>8.6发起人应按照承诺发放项目回报（项目衍生成果及/或相关商品），发放的项目回报应无质量或权利瑕疵，如因项目回报发生纠纷，参照国家相关法规处理(例如:《消费者权益保护法》)。</p>
                                <p>8.7发起人在项目回报日前未完成回报，且无合理理由的，视为项目延期。<b>项目延期30天（含）以上的，视为项目失败</b>。尚筹网众筹平台有权通过系统直接向未确认收货的支持者退还支持款，发起人应当将支持款全额返还尚筹网众筹平台。</p>
                                <p>8.8公益相关项目发起人应在完成项目后的合理时间内上传筹款项用于公益用途的相关凭证。</p>
                                <p>8.9发起人应提交真实、准确的项目信息（项目信息如有任何更新，应及时向尚筹网众筹平台提交更新后的信息），并自主上传、提交项目。</p>
                                <p>8.10发起人应自行承担准备或提交、完成项目而发生的费用，自行缴纳因从事本协议项下行为而产生的相应税款。</p>
                                <p>8.11发起人应妥善保管尚筹网众筹平台的账号和密码，任何情况下发起人须对在该账号下发生的所有活动（包括但不限于信息披露、发布信息、上传图片或视频、网上点击同意或提交各类规则协议等）承担法律责任。</p>
                                <p>8.12发起人了解并同意，不得自行或允许任何第三方使用发起人的账号通过尚筹网众筹平台从事任何违反法律法规或侵犯第三方权利的行为，包含但不限于：</p>
                                <p>8.12.1侵犯任何第三方的专利、商标、著作权、商业秘密或其他合法权利，或违反任何法律或合同的；</p>
                                <p>8.12.2发起人的行为或项目信息是虚假的、误导性的、不准确的；</p>
                                <p>8.12.3发起人的行为或项目信息涉嫌非法、威胁、辱骂、骚扰、诽谤、中伤、欺骗、欺诈、侵权、淫秽、冒犯、亵渎或侵犯他人隐私的；</p>
                                <p>8.12.4未经接收方允许而向接收方发布任何邮件、宣传材料或广告信息；</p>
                                <p>8.12.5进行任何危害信息网络安全的行为，故意传播恶意程序或病毒以及其他破坏、干扰正常网络信息服务的行为。</p>
                                <p>8.13对于发起人通过尚筹网众筹平台发布的涉嫌违法或涉嫌侵犯他人合法权利或违反本协议的信息，尚筹网有权依据尚筹网的判断不经通知发起人即予以修改、编辑、删除等。</p>
                                <p>8.14实物回报类项目上线之前，发起人有义务主动提供样品进行评估审核，且项目一经上线无论众筹成功与否，样机不予退还。</p>
                                <p>8.15发起人承诺不得未经尚筹网众筹平台同意擅自使用尚筹网众筹及其关联公司的任何知识产权。特别地，除非经尚筹网众筹平台书面批准，发起人在任何情形下都不在任何国家或地区将尚筹网众筹及其关联公司的商标及其类似商标、标识的全部或部分单独地或与其他任何商标、商号、文字或符号、域名合并使用或进行申请、注册等权力化行为。合作过程中，发起人应当在其知晓任何第三方可能侵犯尚筹网众筹平台知识产权时及时通知尚筹网众筹平台。</p>
                                <p>8.16发起人违反上述行为规范对任何第三方造成损害的，发起人均应当以自己的名义独立承担所有的法律责任，并赔偿尚筹网因此产生的损失或费用，包括但不限于商誉损失等。</p>
                                <h5>第9条  支持款交付及平台服务费</h5>
                                <p>9.1尚筹网众筹平台对支持款的交付及平台服务费的收取均通过第三方支付平台<b>尚筹网易付宝</b>进行收付。</p>
                                <p>9.2发起人使用尚筹网众筹平台服务，尚筹网将向发起人收取募集总金额3%的平台服务费，发起人与尚筹网另有约定的除外。</p>
                                <p>9.3筹款成功后尚筹网众筹平台将在<b>3</b>个工作日内将募集总金额扣除平台服务费以及质量保证金（如有）后剩余款项的<b>60%</b>交付给发起人，并预留余下的<b>40%</b>作为确保项目成功并保证支持者获得回报的保证金(以下简称“保证金”)。在项目成功、无纠纷且所有支持者得到承诺回报的情况下，尚筹网将把这部分款项交付给发起人。发起人与尚筹网另有约定的除外。</p>
                                <p>9.4根据市场与技术的发展，尚筹网保留变更保证金及平台服务费比例的权利。在变更前，尚筹网将以适当的方式通知发起人变更情况，包括但不限于在尚筹网网站上公示。发起人有权选择接受或拒绝接受，如果发起人选择拒绝的，应立即停止使用尚筹网众筹平台的服务；发起人继续使用尚筹网众筹平台的，即视为发起人同意尚筹网的费用变更。变更后的收费比例对已经发布的项目无溯及力。</p>
                                <h5>第10条  知识产权</h5>
                                <p>10.1发起人承诺，对于通过尚筹网众筹平台发布、上传的所有内容均拥有合法权利，不侵犯任何第三方的肖像权、隐私权、专利权、商标权、著作权等合法权利及其他合同权利。</p>
                                <p>10.2发起人通过尚筹网众筹平台发布、上传的任何内容，发起人授予尚筹网及其关联公司非独家的、可转授权的、不可撤销的、全球通用的、永久的、免费的许可使用权利，并可对上述内容进行修改、改写、改编、发行、翻译、创造衍生性内容及/或可以将前述部分或全部内容加以传播、表演、展示，及/或可以将前述部分或全部内容放入任何现在已知和未来开发出的以任何形式、媒体或科技承载的作品当中。</p>
                                <p>10.3尚筹网众筹平台向发起人提供的服务含有受到相关知识产权及其他法律保护的专有保密资料或信息，亦可能受到著作权、商标、专利等相关法律的保护。未经尚筹网或相关权利人书面授权，发起人不得修改、出售、传播部分或全部该等信息，或加以制作衍生服务或软件，或通过进行还原工程、反向组译及其他方式破译原代码。</p>
                                <h5>第11条  纠纷处理</h5>
                                <p>11.1虚拟类回报不支持退换货，也不支持退款，如有发货错误、漏发或支持者收到的虚拟类回报存在质量问题的，发起人应在七日内免费予以重发或补发。</p>
                                <p>11.2实物类回报如有错发、漏发的，发起人应在七日内免费予以重发或补发，发起人无法重发或补发的，应退相应款项给支持者。</p>
                                <p>11.3项目回报有质量问题的，参照国家相关法规处理(例如:《消费者权益保护法》)。</p>
                                <p>11.4发起人怠于处理纠纷或处理纠纷不符合尚筹网相关规定的，尚筹网众筹平台有权利用预留的保证金/质保金处理支持者纠纷。如保证金不足以处理纠纷，发起人有义务向尚筹网支付处理纠纷所产生的费用。</p>
                                <h5>第12条  违规处理</h5>
                                <p>12.1对于违反本协议或尚筹网网站规则的发起人，尚筹网有权进行违规处理，涉及的罚款可以从保证金/质保金中扣除。</p>
                                <p>12.2如发起人在筹款时间截止前非因不可抗力主动取消项目的，则发起人在一个月内不得再次发起项目。</p>
                                <p>12.3如发起人在项目筹款成功后、发放回报前，非因不可抗力宣布项目失败或因项目延期而被认定为项目失败的，则发起人在三个月内不得再次发起项目，但如发起人已将支持者所支付的款项全额退回的，再次发起项目的时间可以不受限制。</p>
                                <p>12.4尚筹网有权对发起人是否涉嫌违规做出单方认定，并根据单方认定结果中止、终止对发起人的平台使用许可或采取其他限制措施。</p>
                                <p>12.5如发起人严重违反本协议、尚筹网网站规则或违反国家法律法规规定的，将被清退出尚筹网众筹平台，涉嫌犯罪的尚筹网将移送司法机关处理。</p>
                                <h5>第13条  违约责任</h5>
                                <p>13.1如发起人涉嫌违反有关法律或者本协议之约定，使尚筹网及/或尚筹网的关联公司遭受任何损失，或受到任何第三方的索赔，或受到任何行政管理部门的处罚，发起人应当赔偿尚筹网因此遭受的损失及/或发生的费用，包括合理的律师费用、调查取证费用等，相关费用尚筹网有权从保证金/质保金中扣除，不足部分，由发起人另行支付补足。</p>
                                <p>13.2任何一方违反其于本协议项下的陈述、保证或承诺，而使另一方遭受任何诉讼、纠纷、索赔、处罚等的，违约方应负责解决，使另一方发生任何费用、额外责任或遭受经济损失的，应当负责赔偿。如一方发生违约行为，守约方可以书面通知方式要求违约方在指定的时限内停止违约行为，要求其消除影响。如违约方未能按时停止违约行为，则守约方有权立即终止本协议。如因尚筹网自身原因造成发起人的损失，尚筹网向发起人承担的最大总体责任和赔偿限额不应超过在本协议项下尚筹网已向发起人收取的服务费总额。</p>
                                <p>13.3对于发起人应当承担的违约金，尚筹网有权从保证金/质保金中扣除，不予返还。</p>
                                <h5>第14条  协议终止及争议解决</h5>
                                <p>14.1在下列情况下，尚筹网可以随时无须承担任何义务和责任地全部或部分暂停、中止或终止履行本协议的义务或提供本协议项下的服务，直至解除本协议，且无须征得发起人的同意：</p>
                                <p>14.1.1发起的项目违反尚筹网众筹平台使用规则或本协议约定条款；</p>
                                <p>14.1.2发起人不同意接受本协议约定（含尚筹网发布的各类规则），且已停止使用尚筹网众筹平台针对项目发起人提供的服务；</p>
                                <p>14.1.3发起人不符合本协议约定的项目发起人应具备的资格的；</p>
                                <p>14.1.4发起的项目违反法律法规、监管政策或其他相关规定；</p>
                                <p>14.1.5发起的项目将引发或可能引发尚筹网运营的重大风险；</p>
                                <p>14.1.6发起的项目存在或可能存在明显危害支持者利益的风险；</p>
                                <p>14.1.7因不可归责于尚筹网的原因造成项目无法继续进行或项目回报无法实现的。</p>
                                <p>14.2因不可归责于尚筹网的原因造成协议终止，在协议终止前的行为所导致的任何赔偿和责任，发起人必须完全且独立地承担责任。</p>
                                <p>14.3无论本协议因何种原因终止，并不影响本协议终止前已经筹款成功项目的效力，发起人均应将筹款成功的项目履行完毕，或依照约定或本协议的规定对支持者承担责任。</p>
                                <p>14.4本协议及本协议项下的所有行为均适用中华人民共和国法律。</p>
                                <p>14.5协议各方因本协议的签订、履行或解释发生争议的，各方应友好协商解决。如协商不成，任何一方均应向合同签订地法院（即：南京市玄武区人民法院）提起诉讼。</p>
                                <p>14.6本协议的解释权归尚筹网众筹平台所有。</p>
                                <p>
                                    <b>附件：</b>
                                </p>
                                <h5>《尚筹网众筹回报服务协议》</h5>
                                <h5>（注：本协议只适用于尚筹网众筹平台发布的科技、设计类众筹）</h5>
                                <p>鉴于：</p>
                                <p>发起人在隶属于尚筹网（www.atcrowd.com）网站旗下尚筹网众筹平台（以下简称“尚筹网”）上发起项目筹款需求，并承诺给项目支持者提供回报。为保护支持者的合法权益，规范发起人的行为，维护众筹平台秩序，特拟定本协议。发起人承诺严格履行本协议约定，按照项目承诺为客户提供回报及相关服务。</p>
                                <h5>1、 回报产品的退换服务</h5>
                                <p>1.1发起人承诺自支持者收到回报产品的15天内提供无理由退换货服务，退换货过程中产生的快递费用，由发起人承担。如支持者选择退货的，发起人应于收到退回货物之日起3个工作日内为支持者全额退款。如支持者选择换货的，发起人应于收到更换货物之日起5个工作日内完成换货处理。</p>
                                <p>1.2发起人承诺自支持者收到回报产品的第16天至1年内，提供免费维修或换新服务。如回报产品出现质量问题或因非人为原因无法正常使用，发起人应在收到支持者所更换货物之日起5个工作日内完成换货处理。货物更换过程中产生的快递费用，由发起人承担。</p>
                                <p>1.3以下产品不适用于无理由退换货:</p>
                                <p class="indent">1.3.1 个人定做类产品；</p>
                                <p class="indent">1.3.2 文化类产品；</p>
                                <p class="indent">1.3.3 在线下载或者已经拆封的音像制品，计算机软件等数字化商品；</p>
                                <p class="indent">1.3.4 交付的报纸期刊类商品；</p>
                                <p class="indent">1.3.5 未经授权的维修、误用、碰撞、疏忽、滥用、进液、事故、改动、不正确的安装所造成的产品质量问题，或撕毁、涂改标贴、机器序号、防伪标记；</p>
                                <p class="indent">1.3.6其他根据产品性质不适宜退换货，或在众筹项目发起时已经明确表示不适用退换货的产品。</p>

                                <p>1.4为保证产品在<strong><u>1年保修期</u></strong>内的售后服务质量，发起人须向尚筹网缴纳质量保证金（简称“质保金”），作为尚筹网处理客户投诉或产品质量问题用：
                                </p><p> 若发起人为尚筹网（或尚筹网关联公司）已有合作方的，与已有合作业务关联，以尚筹网（或尚筹网关联公司）对发起人的应付账款作为质押，包括但不限于货款、保证金等。
                            </p><p>若发起人非尚筹网（或尚筹网关联公司）已有合作方的，在项目筹款成功后，尚筹网从项目所筹得款项中自动扣取下述约定金额作为质量保证金，自所有支持者收到回报产品<strong><u> 1年</u></strong>以后，且发起人已经妥善处理所有客户投诉后，尚筹网将此质保金返还发起人，具体金额要求如下：</p>
                                <table class="table-initiate">
                                    <tbody><tr>
                                        <th align="center"><strong>众筹项目分类 </strong></th>
                                        <th align="center"><strong>质保金 </strong></th>
                                        <th align="center"><strong>质保金上下限 </strong></th>
                                    </tr>
                                    <tr>
                                        <td>手机、电脑、数码、办公设备、网络设备、智能设备等 </td>
                                        <td rowspan="2">所筹金额×2.5%</td>
                                        <td rowspan="9">下限：1000元<br>
                                            上限：10万元<br>
                                            （即：最低收取1000元，最高收取10万元） </td>
                                    </tr>
                                    <tr>
                                        <td>孕婴用品、儿童用品（童装、童鞋、玩具）等 </td>
                                    </tr>
                                    <tr>
                                        <td>空调、电视、冰箱、洗衣机、影音产品等 </td>
                                        <td rowspan="4">所筹金额×2%</td>
                                    </tr>
                                    <tr>
                                        <td>厨卫大家电、厨房小家电、西式厨房电器、生活小家电、个人护理等 </td>
                                    </tr>
                                    <tr>
                                        <td>户外装备、体育器材、骑行装备、健身器材、垂钓用品、车载电器、汽车配件等 </td>
                                    </tr>
                                    <tr>
                                        <td>家纺、家具、灯具、建材、厨房卫浴用品、清洁用品、餐厨用具、生活日用、成人用品、宠物用品等 </td>
                                    </tr>
                                    <tr>
                                        <td>日化用品（美妆、护肤、洗护用品）等 </td>
                                        <td rowspan="3">所筹金额×1%</td>
                                    </tr>
                                    <tr>
                                        <td>图书、文化用品类 </td>
                                    </tr>
                                    <tr>
                                        <td>其它 </td>
                                    </tr>
                                    </tbody></table>
                                <h5>2、 延迟回报（发货）补偿</h5>
                                <p>众筹项目成功后，发起人应确保按照项目承诺及时为支持者提供回报。如出现回报延迟或未及时发货的情况，发起人应按照以下标准对支持者进行补偿。除项目失败造成的全额退款的情形外，其余情况的补偿金支付时间不得晚于支持者收到货物的时间。</p>
                                <p>
                                </p><table class="table-initiate">
                                <tbody><tr>
                                    <th>延迟发货天数（单位：日）</th>
                                    <th>延迟交付违约金/其他处理措施</th>
                                </tr>
                                <tr>
                                    <td>X≤30</td>
                                    <td>违约金为：未履行订单总金额的5%</td>
                                </tr>
                                <tr>
                                    <td>X&gt;30</td>
                                    <td>项目失败，按照已履行订单总金额的5%支付违约金，并全额退还未履行订单的众筹款给支持者，未履行订单取消</td>
                                </tr>
                                </tbody></table>
                                <p></p>
                                <p>X为延迟发货天数，自承诺回报期限的最后一天的次日起算。如延迟天数达到30日，即视为发起人已经无法提供回报，项目失败，发起人应向所有支持者发表致歉声明或阐述说明，并在3日内完成未履行订单的退款。若发起人在延迟天数未满30天时，与支持者达成一致，并另行约定发货时间的除外。</p>
                                <h5>3、 服务响应时效</h5>
                                <p>在项目回报过程中，发起人应积极响应支持者、尚筹网或尚筹网众筹平台提出的各项问题和需求。对于提问邮件或各项需求邮件，应于2个工作日内回复处理意见或对申请进行处理。如发起人未按以上规定时间回复意见或进行相应处理，尚筹网或尚筹网众筹平台均有权从客户为先理念角度代发起人回复或处理，发起人应按照尚筹网或尚筹网众筹平台的处理意见为客户提供相关服务，因此造成的损失或产生的费用由发起人承担。</p>
                                <h5>4、 违约责任</h5>
                                <p>4.1如发起人涉嫌违反有关法律或者本协议之约定，使尚筹网或尚筹网众筹平台遭受任何损失包括但不限于，受到任何第三方的诉讼、纠纷、索赔，受到任何行政管理部门的处罚，发起人应当赔偿尚筹网或尚筹网众筹平台因此遭受的损失及/或发生的费用，包括合理的律师费用、调查取证费用等。</p>
                                <p>4.2对于发起人应当承担的违约金，尚筹网或尚筹网众筹平台有权从发起人的保证金及质保金或其他由尚筹网或尚筹网众筹平台控制的款项中直接抵扣，不予返还。<b>保证金及质保金不足以支付的，尚筹网或尚筹网众筹平台均有权继续向发起人追偿。</b></p>
                                <h5>5、 其他</h5>
                                <p>5.1发起人同意遵守尚筹网网站（www.atcrowd.com）及其旗下任何网站已经发布的或将来可能发布的各类规则、操作流程。对违反约定或规定给客户造成损失的，发起人愿意接受尚筹网或尚筹网众筹平台的各项处罚。</p>
                                <p>5.2凡因本协议引起的或与本协议有关的任何争议，由双方友好协商解决。协商不成时，任何一方均应向合同签订地法院（即：南京市玄武区人民法院）提起诉讼。</p>
                            </div>
                        </div>
                        <div class="panel-footer" style="text-align:center;">
                            <a class="btn btn-warning btn-lg" href="project/launch/project/page">阅读并同意协议</a>
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
                            <a rel="nofollow" href="http://www.layoutit.cn">关于我们</a> | <a rel="nofollow" href="http://www.layoutit.cn">服务条款</a> | <a rel="nofollow" href="http://www.layoutit.cn">免责声明</a> | <a rel="nofollow" href="http://www.layoutit.cn">网站地图</a> | <a rel="nofollow" href="http://www.layoutit.cn">联系我们</a>
                        </div>
                        <div class="copyRight">
                            Copyright ?2017-2017layoutit.cn 版权所有
                        </div>
                    </div>

                </div>
            </div>
        </div>

    </div>
    </body>
    </html>
    ```


3. project模块新建 project-launch.html

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="author" content="">
        <link rel="stylesheet" th:href="@{/bootstrap/css/bootstrap.min.css}"
            href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" th:href="@{/css/font-awesome.min.css}"
            href="css/font-awesome.min.css">
        <link rel="stylesheet" th:href="@{/css/theme.css}" href="css/theme.css">
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

            .label-type, .label-status, .label-order {
                background-color: #fff;
                color: #f60;
                padding: 5px;
                border: 1px #f60 solid;
            }

            #typeList  :not (:first-child ) {
                margin-top: 20px;
            }

            .label-txt {
                margin: 10px 10px;
                border: 1px solid #ddd;
                padding: 4px;
                font-size: 14px;
            }

            .panel {
                border-radius: 0;
            }

            .progress-bar-default {
                background-color: #ddd;
            }

            .selected {
                color: yellow;
                background-color: green;
            }
        </style>
    </head>
    <body>
    <div class="navbar-wrapper">
        <div class="container">
            <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
                <div class="container">
                    <div class="navbar-header">
                        <a class="navbar-brand" href="index.html" style="font-size: 32px;">尚筹网-创意产品众筹平台</a>
                    </div>
                    <div id="navbar" class="navbar-collapse collapse"
                        style="float: right;">
                        <ul class="nav navbar-nav">
                            <li class="dropdown"><a href="#" class="dropdown-toggle"
                                                    data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i>
                                [[${session.loginMember.username}]]<span class="caret"></span></a>
                                <ul class="dropdown-menu" role="menu">
                                    <li><a href="member.html"><i
                                            class="glyphicon glyphicon-scale"></i> 会员中心</a></li>
                                    <li><a href="#"><i class="glyphicon glyphicon-comment"></i>
                                        消息</a></li>
                                    <li class="divider"></li>
                                    <li><a href="http://www.crowd.com/auth/member/logout"><i
                                            class="glyphicon glyphicon-off"></i> 退出系统</a></li>
                                </ul></li>
                        </ul>
                    </div>
                </div>
            </nav>
        </div>
    </div>

    <div class="container theme-showcase" role="main">

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <nav class="navbar navbar-default" role="navigation">
                        <div class="collapse navbar-collapse"
                            id="bs-example-navbar-collapse-1">
                            <ul class="nav navbar-nav">
                                <li><a rel="nofollow" href="index.html"><i
                                        class="glyphicon glyphicon-home"></i> 众筹首页</a></li>
                                <li><a rel="nofollow" href="projects.html"><i
                                        class="glyphicon glyphicon-th-large"></i> 项目总览</a></li>
                                <li class="active"><a rel="nofollow" href="javascript:;"><i
                                        class="glyphicon glyphicon-edit"></i> 发起项目</a></li>
                                <li><a rel="nofollow" href="minecrowdfunding.html"><i
                                        class="glyphicon glyphicon-user"></i> 我的众筹</a></li>
                            </ul>
                        </div>
                    </nav>
                </div>
            </div>
        </div>


        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <form id="projectForm" th:action="@{/project/create/project/information}" method="post" enctype="multipart/form-data" class="form-horizontal">
                        <p th:text="${message}"></p>
                        <div class="panel panel-default">
                            <div class="panel-heading" style="text-align: center;">
                                <div class="container-fluid">
                                    <div class="row clearfix">
                                        <div class="col-md-3 column">
                                            <div class="progress"
                                                style="height: 50px; border-radius: 0; position: absolute; width: 75%; right: 49px;">
                                                <div class="progress-bar progress-bar-success"
                                                    role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                    aria-valuemax="100" style="width: 100%; height: 50px;">
                                                    <div style="font-size: 16px; margin-top: 15px;">1.
                                                        项目及发起人信息</div>
                                                </div>
                                            </div>
                                            <div
                                                    style="right: 0; border: 10px solid #ddd; border-left-color: #88b7d5; border-width: 25px; position: absolute; border-left-color: rgba(92, 184, 92, 1); border-top-color: rgba(92, 184, 92, 0); border-bottom-color: rgba(92, 184, 92, 0); border-right-color: rgba(92, 184, 92, 0);">
                                            </div>
                                        </div>
                                        <div class="col-md-3 column">
                                            <div class="progress"
                                                style="height: 50px; border-radius: 0; position: absolute; width: 75%; right: 49px;">
                                                <div class="progress-bar progress-bar-default"
                                                    role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                    aria-valuemax="100" style="width: 100%; height: 50px;">
                                                    <div style="font-size: 16px; margin-top: 15px;">2.
                                                        回报设置</div>
                                                </div>
                                            </div>
                                            <div
                                                    style="right: 0; border: 10px solid #ddd; border-left-color: #88b7d5; border-width: 25px; position: absolute; border-left-color: rgba(221, 221, 221, 1); border-top-color: rgba(221, 221, 221, 0); border-bottom-color: rgba(221, 221, 221, 0); border-right-color: rgba(221, 221, 221, 0);">
                                            </div>
                                        </div>
                                        <div class="col-md-3 column">
                                            <div class="progress"
                                                style="height: 50px; border-radius: 0; position: absolute; width: 75%; right: 49px;">
                                                <div class="progress-bar progress-bar-default"
                                                    role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                    aria-valuemax="100" style="width: 100%; height: 50px;">
                                                    <div style="font-size: 16px; margin-top: 15px;">3.
                                                        确认信息</div>
                                                </div>
                                            </div>
                                            <div
                                                    style="right: 0; border: 10px solid #ddd; border-left-color: #88b7d5; border-width: 25px; position: absolute; border-left-color: rgba(221, 221, 221, 1); border-top-color: rgba(221, 221, 221, 0); border-bottom-color: rgba(221, 221, 221, 0); border-right-color: rgba(221, 221, 221, 0);">
                                            </div>
                                        </div>
                                        <div class="col-md-3 column">
                                            <div class="progress" style="height: 50px; border-radius: 0;">
                                                <div class="progress-bar progress-bar-default"
                                                    role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                    aria-valuemax="100" style="width: 100%; height: 50px;">
                                                    <div style="font-size: 16px; margin-top: 15px;">4. 完成</div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="panel-body">
                                <div class="container-fluid">
                                    <div class="row clearfix">
                                        <div class="col-md-12 column">
                                            <blockquote
                                                    style="border-left: 5px solid #f60; color: #f60; padding: 0 0 0 20px;">
                                                <b> 项目及发起人信息 </b>
                                            </blockquote>
                                        </div>
                                        <div class="col-md-12 column">
                                            <div class="page-header" style="border-bottom-style: dashed;">
                                                <h3>项目信息</h3>
                                            </div>
                                            <div class="form-group">
                                                <label for="inputEmail3" class="col-sm-2 control-label">分类信息</label>
                                                <div class="col-sm-10">
                                                    <label class="radio-inline">
                                                        <input type="checkbox" name="typeIdList"
                                                            id="inlineRadio1"
                                                            value="1"> 科技
                                                    </label>
                                                    <label class="radio-inline">
                                                        <input
                                                                type="checkbox" name="typeIdList"
                                                                id="inlineRadio2" value="2" checked="checked"> 设计
                                                    </label>
                                                    <label class="radio-inline">
                                                        <input
                                                                type="checkbox" name="typeIdList"
                                                                id="inlineRadio3" value="3"> 公益
                                                    </label>
                                                    <label class="radio-inline">
                                                        <input
                                                                type="checkbox" name="typeIdList"
                                                                id="inlineRadio3" value="4" checked="checked"> 农业
                                                    </label>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label for="inputEmail3" class="col-sm-2 control-label">标签</label>
                                                <div class="col-sm-10">
                                                    <ul style="list-style: none; padding-left: 0;">
                                                        <li style="margin: 10px 0">[手机] <span
                                                                class="tagLable label-txt" id="4">大屏</span> <span
                                                                class="tagLable label-txt" id="5">美颜</span> <span
                                                                class="tagLable label-txt" id="6">续航</span>
                                                        </li>
                                                        <li style="margin: 10px 0">[数码] <span
                                                                class="tagLable label-txt" id="7">高解析度</span> <span
                                                                class="tagLable label-txt" id="8">高清</span>
                                                        </li>
                                                        <li style="margin: 10px 0">[电脑] <span
                                                                class="tagLable label-txt" id="9">大内存</span> <span
                                                                class="tagLable label-txt" id="10">固态</span>
                                                        </li>
                                                    </ul>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">项目名称</label>
                                                <div class="col-sm-10">
                                                    <input type="text" name="projectName" value="brotherMao" class="form-control">
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">一句话简介</label>
                                                <div class="col-sm-10">
                                                    <textarea name="projectDescription" class="form-control" rows="5">就是帅！</textarea>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">筹资金额（元）</label>
                                                <div class="col-sm-10">
                                                    <input type="text" name="money" value="100000" class="form-control"
                                                        style="width: 100px;"> <label
                                                        class="control-label">筹资金额不能低于100元,不能高于1000000000元</label>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">筹资天数（天）</label>
                                                <div class="col-sm-10">
                                                    <input type="text" name="day" value="30" class="form-control"
                                                        style="width: 100px;"> <label
                                                        class="control-label">一般10-90天，建议30天</label>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">项目头图</label>
                                                <div class="col-sm-10">
                                                    <input type="file" name="headerPicture" style="display: none;" />
                                                    <button id="uploadHeadBtn" type="button" class="btn btn-primary btn-lg active">上传图片</button>
                                                    <label class="control-label">图片上无文字,支持jpg、jpeg、png、gif格式，大小不超过2M，建议尺寸：740*457px</label>
                                                    <br /><img style="display: none;" />
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">项目详情</label>
                                                <div class="col-sm-10">
                                                    <input type="file" multiple="multiple" name="detailPictureList" style="display: none;" />
                                                    <button id="uploadDetailBtn" type="button" class="btn btn-primary btn-lg active">上传图片</button>
                                                    <label class="control-label">支持jpg、jpeg、png、gif格式，大小不超过2M，建议尺寸：宽740px</label>
                                                    <div id="showDetailPicture"></div>
                                                </div>
                                            </div>
                                            <!-- </form> -->
                                        </div>
                                        <div class="col-md-12 column">
                                            <div class="page-header" style="border-bottom-style: dashed;">
                                                <h3>发起人信息</h3>
                                            </div>
                                            <!-- <form class="form-horizontal"> -->
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">自我介绍</label>
                                                <div class="col-sm-10">
                                                    <input type="text" name="memberLauchInfoVO.descriptionSimple" value="i am mao" class="form-control"
                                                        placeholder="一句话自我介绍，不超过40字">
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">详细自我介绍</label>
                                                <div class="col-sm-10">
                                                        <textarea name="memberLauchInfoVO.descriptionDetail" class="form-control" rows="5"
                                                                placeholder="向支持者详细介绍你自己或你的团队及项目背景，让支持者在最短时间内了解你，不超过160字">我是猫哥</textarea>
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">联系电话</label>
                                                <div class="col-sm-10">
                                                    <input type="text" name="memberLauchInfoVO.phoneNum" value="123456" class="form-control"
                                                        placeholder="此信息不会显示在项目页面">
                                                </div>
                                            </div>
                                            <div class="form-group">
                                                <label class="col-sm-2 control-label">客服电话</label>
                                                <div class="col-sm-10">
                                                    <input type="text" name="memberLauchInfoVO.serviceNum" value="654321" class="form-control"
                                                        placeholder="此信息显示在项目页面">
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="panel-footer" style="text-align: center;">
                                <button id="submitBtn" type="button" class="btn  btn-warning btn-lg">下一步</button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>


        <div class="container" style="margin-top: 20px;">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div id="footer">
                        <div class="footerNav">
                            <a rel="nofollow" href="http://www.layoutit.cn">关于我们</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">服务条款</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">免责声明</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">网站地图</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">联系我们</a>
                        </div>
                        <div class="copyRight">Copyright ?2017-2017layoutit.cn 版权所有
                        </div>
                    </div>

                </div>
            </div>
        </div>

    </div>
    <!-- /container -->
    <script type="text/javascript" th:src="@{/jquery/jquery-2.1.1.min.js}"
            src="jquery/jquery-2.1.1.min.js"></script>
    <script type="text/javascript"
            th:src="@{/bootstrap/js/bootstrap.min.js}"
            src="bootstrap/js/bootstrap.min.js"></script>
    <script type="text/javascript" th:src="@{/script/docs.min.js}"
            src="script/docs.min.js"></script>
    <script type="text/javascript" th:src="@{/script/back-to-top.js}"
            src="script/back-to-top.js"></script>
    <script type="text/javascript">
        $('#myTab a').click(function(e) {
            e.preventDefault()
            $(this).tab('show')
        });

        // 声明全局变量用于存储选中的标签的id
        var tagIdList = new Array();

        $(".tagLable").click(function() {

            // 标签文本显示框如果有selected就去掉，如果没有就加上
            $(this).toggleClass("selected");

            // 获取当前文本显示框的id（也就是标签的id）
            var tagId = this.id;

            // 判定tagId是否在数组中，如果在则删除，如果不在则添加
            checkExitsts(tagIdList, tagId);

            console.log(tagIdList);

        });

        function checkExitsts(arr, id) {
            for (var i = 0; i < arr.length; i++) {
                var value = arr[i];
                if (value == id) {

                    // 从数组中删除元素，i是元素索引，1是删除数量
                    arr.splice(i, 1);

                    return;
                }
            }

            // 将当前标签的id加入数组
            arr.push(id);
        }

        $("#uploadHeadBtn").click(function() {

            // 调用click()函数，相当于被点击了一下
            $("[name=headerPicture]").click();
        });

        $("[name=headerPicture]").change(function(event) {

            // 获取用户选中的文件
            var files = event.target.files;

            // 使用下标0，选择唯一的一个文件
            var file = files[0];

            // 获取URL对象
            var url = window.url || window.webkitURL;

            // 调用url对象的createObjectURL()方法获取当前选中的文件在系统中的路径
            var path = url.createObjectURL(file);

            // 使用path修改img标签的src属性
            $(this).next().next().next().next().attr("src", path).show();

        });

        $("#uploadDetailBtn").click(function() {

            $("[name=detailPictureList]").click();

        });

        $("[name=detailPictureList]").change(function(event) {

            $("#showDetailPicture").empty();

            var files = event.target.files;

            var url = window.url || window.webkitURL;

            for (var i = 0; i < files.length; i++) {
                var file = files[i];

                var path = url.createObjectURL(file);

                var imgHtml = "<img src='"+path+"' /><br/>";

                $("#showDetailPicture").append(imgHtml);
            }

        });

        // 点击下一步按钮提交表单
        $("#submitBtn").click(function(){

            // 将表单中标签id的值组成的数组转换成表单内的隐藏域
            for(var i = 0; i < tagIdList.length; i++) {
                var tagId = tagIdList[i];

                var hiddenInputHTML = "<input type='hidden' name='tagIdList' value='"+tagId+"' />";

                $("#projectForm").append(hiddenInputHTML);
            }

            // 提交表单
            $("#projectForm").submit();
        });

    </script>
    </body>
    </html>
    ```


4. project模块新建 project-return.html

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="author" content="">
        <link rel="stylesheet" th:href="@{/bootstrap/css/bootstrap.min.css}"
            href="bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" th:href="@{/css/font-awesome.min.css}"
            href="css/font-awesome.min.css">
        <link rel="stylesheet" th:href="@{/css/theme.css}" href="css/theme.css">
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

            .label-type, .label-status, .label-order {
                background-color: #fff;
                color: #ff6600;
                padding: 5px;
                border: 1px #f60 solid;
            }

            #typeList  :not (:first-child ) {
                margin-top: 20px;
            }

            .label-txt {
                margin: 10px 10px;
                border: 1px solid #ddd;
                padding: 4px;
                font-size: 14px;
            }

            .panel {
                border-radius: 0;
            }

            .progress-bar-default {
                background-color: #ddd;
            }
        </style>
    </head>
    <body>
    <div class="navbar-wrapper">
        <div class="container">
            <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
                <div class="container">
                    <div class="navbar-header">
                        <a class="navbar-brand" href="index.html" style="font-size: 32px;">尚筹网-创意产品众筹平台</a>
                    </div>
                    <div id="navbar" class="navbar-collapse collapse"
                        style="float: right;">
                        <ul class="nav navbar-nav">
                            <li class="dropdown"><a href="#" class="dropdown-toggle"
                                                    data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i>
                                [[${session.loginMember.username}]]<span class="caret"></span></a>
                                <ul class="dropdown-menu" role="menu">
                                    <li><a href="member.html"><i
                                            class="glyphicon glyphicon-scale"></i> 会员中心</a></li>
                                    <li><a href="#"><i class="glyphicon glyphicon-comment"></i>
                                        消息</a></li>
                                    <li class="divider"></li>
                                    <li><a href="http://www.crowd.com/auth/member/logout"><i
                                            class="glyphicon glyphicon-off"></i> 退出系统</a></li>
                                </ul></li>
                        </ul>
                    </div>
                </div>
            </nav>
        </div>
    </div>

    <div class="container theme-showcase" role="main">

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <nav class="navbar navbar-default" role="navigation">
                        <div class="collapse navbar-collapse"
                            id="bs-example-navbar-collapse-1">
                            <ul class="nav navbar-nav">
                                <li><a rel="nofollow" href="index.html"><i
                                        class="glyphicon glyphicon-home"></i> 众筹首页</a></li>
                                <li><a rel="nofollow" href="projects.html"><i
                                        class="glyphicon glyphicon-th-large"></i> 项目总览</a></li>
                                <li class="active"><a rel="nofollow" href="javascript:;"><i
                                        class="glyphicon glyphicon-edit"></i> 发起项目</a></li>
                                <li><a rel="nofollow" href="minecrowdfunding.html"><i
                                        class="glyphicon glyphicon-user"></i> 我的众筹</a></li>
                            </ul>
                        </div>
                    </nav>
                </div>
            </div>
        </div>


        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="panel panel-default">
                        <div class="panel-heading" style="text-align: center;">
                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-3 column">
                                        <div class="progress"
                                            style="height: 50px; border-radius: 0; position: absolute; width: 75%; right: 49px;">
                                            <div class="progress-bar progress-bar-default"
                                                role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                aria-valuemax="100" style="width: 100%; height: 50px;">
                                                <div style="font-size: 16px; margin-top: 15px;">1.
                                                    项目及发起人信息</div>
                                            </div>
                                        </div>
                                        <div
                                                style="right: 0; border: 10px solid #ddd; border-left-color: #88b7d5; border-width: 25px; position: absolute; border-left-color: rgba(221, 221, 221, 1); border-top-color: rgba(221, 221, 221, 0); border-bottom-color: rgba(221, 221, 221, 0); border-right-color: rgba(221, 221, 221, 0);">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress"
                                            style="height: 50px; border-radius: 0; position: absolute; width: 75%; right: 49px;">
                                            <div class="progress-bar progress-bar-success"
                                                role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                aria-valuemax="100" style="width: 100%; height: 50px;">
                                                <div style="font-size: 16px; margin-top: 15px;">2.
                                                    回报设置</div>
                                            </div>
                                        </div>
                                        <div
                                                style="right: 0; border: 10px solid #ddd; border-left-color: #88b7d5; border-width: 25px; position: absolute; border-left-color: rgba(92, 184, 92, 1); border-top-color: rgba(92, 184, 92, 0); border-bottom-color: rgba(92, 184, 92, 0); border-right-color: rgba(92, 184, 92, 0);">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress"
                                            style="height: 50px; border-radius: 0; position: absolute; width: 75%; right: 49px;">
                                            <div class="progress-bar progress-bar-default"
                                                role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                aria-valuemax="100" style="width: 100%; height: 50px;">
                                                <div style="font-size: 16px; margin-top: 15px;">3.
                                                    确认信息</div>
                                            </div>
                                        </div>
                                        <div
                                                style="right: 0; border: 10px solid #ddd; border-left-color: #88b7d5; border-width: 25px; position: absolute; border-left-color: rgba(221, 221, 221, 1); border-top-color: rgba(221, 221, 221, 0); border-bottom-color: rgba(221, 221, 221, 0); border-right-color: rgba(221, 221, 221, 0);">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height: 50px; border-radius: 0;">
                                            <div class="progress-bar progress-bar-default"
                                                role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                                aria-valuemax="100" style="width: 100%; height: 50px;">
                                                <div style="font-size: 16px; margin-top: 15px;">4. 完成</div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="panel-body">

                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-12 column">
                                        <blockquote
                                                style="border-left: 5px solid #f60; color: #f60; padding: 0 0 0 20px;">
                                            <b> 回报设置 </b>
                                        </blockquote>
                                    </div>
                                    <div class="col-md-12 column">
                                        <table class="table table-bordered"
                                            style="text-align: center;">
                                            <thead>
                                            <tr style="background-color: #ddd;">
                                                <td>序号</td>
                                                <td>支付金额</td>
                                                <td>名额</td>
                                                <td>单笔限购</td>
                                                <td>回报内容</td>
                                                <td>回报时间</td>
                                                <td>运费</td>
                                                <td>操作</td>
                                            </tr>
                                            </thead>
                                            <tbody id="returnTableBody">
                                            <!-- <tr>
                                                <td scope="row">1</td>
                                                <td>￥1.00</td>
                                                <td>无限制</td>
                                                <td>1</td>
                                                <td>1</td>
                                                <td>项目结束后的30天</td>
                                                <td>包邮</td>
                                                <td>
                                                    <button type="button" class="btn btn-primary btn-xs">
                                                        <i class=" glyphicon glyphicon-pencil"></i>
                                                    </button>
                                                    <button type="button" class="btn btn-danger btn-xs">
                                                        <i class=" glyphicon glyphicon-remove"></i>
                                                    </button>
                                                </td>
                                            </tr> -->
                                            </tbody>
                                        </table>
                                        <button id="addReturnBtn" type="button" class="btn btn-primary btn-lg">
                                            <i class="glyphicon glyphicon-plus"></i> 添加回报
                                        </button>
                                        <div class="returnFormDiv"
                                            style="border: 10px solid #f60; border-bottom-color: #eceeef; border-width: 10px; width: 20px; margin-left: 20px; border-left-color: rgba(221, 221, 221, 0); border-top-color: rgba(221, 221, 221, 0); border-right-color: rgba(221, 221, 221, 0);"></div>
                                        <div class="panel panel-default returnFormDiv"
                                            style="border-style: dashed; background-color: #eceeef">
                                            <div class="panel-body">

                                                <div class="col-md-12 column">
                                                    <form class="form-horizontal">
                                                        <div class="form-group">
                                                            <label for="inputEmail3" class="col-sm-2 control-label">回报类型</label>
                                                            <div class="col-sm-10">
                                                                <label class="radio-inline"> <input type="radio"
                                                                                                    name="type" id="inlineRadio1"
                                                                                                    value="0"> 实物回报
                                                                </label> <label class="radio-inline"> <input
                                                                    type="radio" name="type"
                                                                    id="inlineRadio2" value="1"> 虚拟物品回报
                                                            </label>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">支持金额（元）</label>
                                                            <div class="col-sm-10">
                                                                <input type="text" name="supportmoney" value="10" class="form-control"
                                                                    style="width: 100px;">
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">回报内容</label>
                                                            <div class="col-sm-10">
                                                                    <textarea class="form-control" name="content" rows="5"
                                                                            placeholder="简单描述回报内容，不超过200字">以身相许</textarea>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">说明图片</label>
                                                            <div class="col-sm-10">
                                                                <input type="file" name="returnPicture" style="display: none;" />
                                                                <button type="button" id="uploadBtn"
                                                                        class="btn btn-primary btn-lg active">上传图片</button>
                                                                <label class="control-label">支持jpg、jpeg、png、gif格式，大小不超过2M，建议尺寸：760*510px选择文件</label>
                                                                <br/>
                                                                <img style="display: none" />
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">回报数量（名）</label>
                                                            <div class="col-sm-10">
                                                                <input type="text" name="count" value="5" class="form-control"
                                                                    style="width: 100px; display: inline"> <label
                                                                    class="control-label">“0”为不限回报数量</label>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label for="inputEmail3" class="col-sm-2 control-label">单笔限购</label>
                                                            <div class="col-sm-10">
                                                                <label class="radio-inline"> <input type="radio"
                                                                                                    name="signalpurchase" id="inlineRadio1"
                                                                                                    value="0"> 不限购
                                                                </label> <label class="radio-inline"> <input
                                                                    type="radio" name="signalpurchase"
                                                                    id="inlineRadio2" value="1"> 限购
                                                            </label> <input type="text" name="purchase" value="8" class="form-control"
                                                                            style="width: 100px; display: inline"> <label
                                                                    class="radio-inline">默认数量为1，且不超过上方已设置的回报数量</label>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">运费(元)</label>
                                                            <div class="col-sm-10">
                                                                <input type="text" name="freight" value="0" class="form-control"
                                                                    style="width: 100px; display: inline">
                                                                <label class="control-label">“0”为包邮</label>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label for="inputEmail3" class="col-sm-2 control-label">发票</label>
                                                            <div class="col-sm-10">
                                                                <label class="radio-inline"> <input type="radio"
                                                                                                    name="invoice" id="inlineRadio1"
                                                                                                    value="0"> 不可开发票
                                                                </label> <label class="radio-inline"> <input
                                                                    type="radio" name="invoice"
                                                                    id="inlineRadio2" value="1">
                                                                可开发票（包括个人发票和自定义抬头发票）
                                                            </label>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label for="inputEmail3" class="col-sm-2 control-label">回报时间</label>
                                                            <div class="col-sm-10">
                                                                <label class="radio-inline"> 项目结束后 </label> <input
                                                                    type="text" name="returndate" value="15" class="form-control"
                                                                    style="width: 100px; display: inline"> <label
                                                                    class="radio-inline">天，向支持者发送回报</label>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label for="inputEmail3" class="col-sm-2 control-label"></label>
                                                            <div class="col-sm-10">
                                                                <button type="button" class="btn btn-primary" id="okBtn">确定</button>
                                                                <button type="button" class="btn btn-default">取消</button>
                                                            </div>
                                                        </div>

                                                    </form>
                                                </div>


                                            </div>
                                        </div>
                                    </div>

                                    <div class="container">
                                        <div class="row clearfix">
                                            <div class="col-md-12 column">
                                                <blockquote>
                                                    <p>
                                                        <i class="glyphicon glyphicon-info-sign"
                                                        style="color: #2a6496;"></i> 提示
                                                    </p>
                                                    <small>3个以上的回报：多些选择能提高项目的支持率。几十、几百、上千元的支持：3个不同档次的回报，能让你的项目更快成功。回报最好是项目的衍生品：<br>与项目内容有关的回报更能吸引到大家的支持。
                                                    </small>
                                                </blockquote>
                                            </div>
                                        </div>
                                    </div>


                                </div>
                            </div>
                        </div>
                        <div class="panel-footer" style="text-align: center;">
                            <button type="button" class="btn  btn-default btn-lg"
                                    onclick="window.location.href='start-step-1.html'">上一步</button>
                            <!-- <button type="button" class="btn  btn-warning btn-lg"
                                onclick="window.location.href='start-step-3.html'">下一步</button> -->
                            <a th:href="@{/project/create/confirm/page.html}" class="btn btn-warning btn-lg">下一步</a>
                            <a class="btn"> 预览 </a>
                        </div>
                    </div>
                </div>
            </div>
        </div>


        <div class="container" style="margin-top: 20px;">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div id="footer">
                        <div class="footerNav">
                            <a rel="nofollow" href="http://www.layoutit.cn">关于我们</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">服务条款</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">免责声明</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">网站地图</a> | <a
                                rel="nofollow" href="http://www.layoutit.cn">联系我们</a>
                        </div>
                        <div class="copyRight">Copyright ?2017-2017layoutit.cn 版权所有
                        </div>
                    </div>

                </div>
            </div>
        </div>

    </div>
    <!-- /container -->
    <script type="text/javascript" th:src="@{/jquery/jquery-2.1.1.min.js}"
            src="jquery/jquery-2.1.1.min.js"></script>
    <script type="text/javascript"
            th:src="@{/bootstrap/js/bootstrap.min.js}"
            src="bootstrap/js/bootstrap.min.js"></script>
    <script type="text/javascript" th:src="@{/script/docs.min.js}"
            src="script/docs.min.js"></script>
    <script type="text/javascript" th:src="@{/script/back-to-top.js}"
            src="script/back-to-top.js"></script>
    <script type="text/javascript">
        $('#myTab a').click(function(e) {
            e.preventDefault()
            $(this).tab('show')
        });

        $(function() {

            // 回报信息的表单部分默认隐藏
            $(".returnFormDiv").hide();

            // 点击添加回报按钮切换表单部分的显示状态
            $("#addReturnBtn").click(function(){

                // toggle()把显示的元素隐藏，把隐藏的元素显示
                $(".returnFormDiv").toggle();
            });
        });

        // 声明一个全局的returnObj对象用于存储整个表单的数据（包括上传到OSS的图片访问路径）
        var returnObj = {};

        // 点击上传图片按钮打开文件选择框
        $("#uploadBtn").click(function(){
            $("[name=returnPicture]").click();
        });

        // 在文件上传框的值改变事件响应函数中预览并上传图片
        $("[name=returnPicture]").change(function(event){

            var file = event.target.files[0];

            var url = window.url || window.webkitURL;

            var path = url.createObjectURL(file);

            $(this).next().next().next().next().attr("src",path).show();

            // 将上传的文件封装到FormData对象中
            var formData = new FormData();

            formData.append("returnPicture", file);

            // 发送Ajax请求上传文件
            $.ajax({
                "url":"[[@{/project/create/upload/return/picture.json}]]",
                "type":"post",
                "data":formData,
                "contentType":false,
                "processData":false,
                "dataType":"json",
                "success":function(response){

                    var result = response.result;

                    if(result == "SUCCESS") {
                        alert("上传成功！");

                        // 如果上传成功，则从响应体数据中获取图片的访问路径
                        returnObj.describPicPath = response.data;
                    }

                    if(result == "FAILED") {
                        alert(response.message);
                    }

                },
                "error":function(response){
                    alert(response.status + " " + response.statusText);
                }
            });

        });

        // 声明序号保存表格中数据的序号
        var order = 0;

        // 点击确定按钮，绑定单击响应函数
        $("#okBtn").click(function(){

            // 1.收集表单数据
            returnObj.type = $("[name=type]:checked").val();
            returnObj.supportmoney = $("[name=supportmoney]").val();
            returnObj.content = $("[name=content]").val();
            returnObj.count = $("[name=count]").val();
            returnObj.signalpurchase = $("[name=signalpurchase]:checked").val();
            returnObj.purchase = $("[name=purchase]").val();
            returnObj.freight = $("[name=freight]").val();
            returnObj.invoice = $("[name=invoice]:checked").val();
            returnObj.returndate = $("[name=returndate]").val();

            // 2.发送Ajax请求
            $.ajax({
                "url" : "[[@{/project/create/save/return.json}]]",
                "type": "post",
                "dataType": "json",
                "data": returnObj,
                "success": function(response) {
                    var result = response.result;
                    if(result == "SUCCESS") {
                        alert("这一条保存成功！");

                        // 使用returnObj填充表格
                        var orderTd = "<td>"+(++order)+"</td>";
                        var supportmoneyTd = "<td>"+returnObj.supportmoney+"</td>";
                        var countTd = "<td>"+returnObj.count+"</td>";
                        var signalpurchaseTd = "<td>"+(returnObj.signalpurchase == 0?"不限购":("限购"+returnObj.purchase))+"</td>";
                        var contentTd = "<td>"+returnObj.content+"</td>";
                        var returndateTd = "<td>"+returnObj.returndate+"天以后返还</td>";
                        var freightTd = "<td>"+(returnObj.freight==0?"包邮":returnObj.freight)+"</td>";
                        var operationTd = "<td><button type='button' class='btn btn-primary btn-xs'><i class=' glyphicon glyphicon-pencil'></i></button>&nbsp;<button type='button' class='btn btn-danger btn-xs'><i class=' glyphicon glyphicon-remove'></i></button></td>";
                        var trHTML = "<tr>"+orderTd+supportmoneyTd+countTd+signalpurchaseTd+contentTd+returndateTd+freightTd+operationTd+"</tr>";

                        $("#returnTableBody").append(trHTML);

                        $("#returnPictureImage").hide();
                    }

                    if(result == "FAILED") {
                        alert("这一条保存失败！");
                    }

                    // 后续操作
                    // 仅仅调用click()函数而不传入回调函数表示点击一下这个按钮
                    $("#resetBtn").click();

                    // 将表单部分div隐藏
                    $(".returnFormDiv").hide();
                }
            });
        });
    </script>
    </body>
    </html>
    ```

## 14.3 收集回报信息

思路：![思路](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/14-3-1.png)

### 14.3.1 接收每条回报信息

1. 接收页面异步上传的图片

    ```java
    @ResponseBody
    @RequestMapping("/create/upload/return/picture.json")
    public ResultEntity<String> uploadReturnPicture(@RequestParam("returnPicture") MultipartFile returnPicture) throws IOException {

        // 1.执行文件上传
        ResultEntity<String> uploadReturnPicResultEntity = CrowdUtil.uploadFileToOss(
                ossProperties.getEndPoint(),
                ossProperties.getAccessKeyId(),
                ossProperties.getAccessKeySecret(),
                returnPicture.getInputStream(),
                ossProperties.getBucketName(),
                ossProperties.getBucketDomain(),
                Objects.requireNonNull(returnPicture.getOriginalFilename()));

        // 2.返回上传的结果
        return uploadReturnPicResultEntity;
    }
    ```

2. 接收整个回报信息数据

    ```java
    @ResponseBody
    @RequestMapping("/create/save/return.json")
    public ResultEntity<String> saveReturn(ReturnVO returnVO, HttpSession session) {

        try {
            // 1.从 session 域中读取之前缓存的 ProjectVO 对象
            ProjectVO projectVO = (ProjectVO) session.getAttribute(CrowdConstant.ATTR_NAME_TEMPLE_PROJECT);

            // 2.判断 projectVO 是否为 null
            if (projectVO == null) {
                return ResultEntity.failed(CrowdConstant.MESSAGE_TEMPLE_PROJECT_MISSING);
            }

            // 3.从 projectVO 对象中获取存储回报信息的集合
            List<ReturnVO> returnVOList = projectVO.getReturnVOList();

            // 4.判断 returnVOList 集合是否有效
            if (returnVOList == null || returnVOList.size() == 0) {
                // 5.创建集合对象对 returnVOList 进行初始化
                returnVOList = new ArrayList<>();
                // 6.为了让以后能够正常使用这个集合， 设置到 projectVO 对象中
                projectVO.setReturnVOList(returnVOList);
            }

            // 7.将收集了表单数据的 returnVO 对象存入集合
            returnVOList.add(returnVO);

            // 8.把数据有变化的 ProjectVO 对象重新存入 Session 域， 以确保新的数据最终能够存入 Redis
            session.setAttribute(CrowdConstant.ATTR_NAME_TEMPLE_PROJECT, projectVO);

            // 9.所有操作成功完成返回成功
            return ResultEntity.successWithoutData();
        } catch (Exception e) {
            e.printStackTrace();

            return ResultEntity.failed(e.getMessage());
        }
    }
    ```

### 14.3.2 跳转页面

1. 修改 project-return.html

    ```html
    <a th:href="@{/project/create/confirm/page}" class="btn btn-warning btn-lg">下一步</a>
    ```

2. 添加 view-controller

    ```java
    registry.addViewController("/create/confirm/page").setViewName("project-confirm");
    ```

3. 新建 project-confirm.html 页面

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="www.thymeleaf.org">
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

            .label-type, .label-status, .label-order {
                background-color: #fff;
                color: #f60;
                padding : 5px;
                border: 1px #f60 solid;
            }
            #typeList  :not(:first-child) {
                margin-top:20px;
            }
            .label-txt {
                margin:10px 10px;
                border:1px solid #ddd;
                padding : 4px;
                font-size:14px;
            }
            .panel {
                border-radius:0;
            }

            .progress-bar-default {
                background-color: #ddd;
            }
        </style>
        <script src="jquery/jquery-2.1.1.min.js"></script>
        <script src="bootstrap/js/bootstrap.min.js"></script>
        <script src="script/docs.min.js"></script>
        <script src="script/back-to-top.js"></script>
        <script>
            $('#myTab a').click(function (e) {
                e.preventDefault()
                $(this).tab('show')
            })
        </script>
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
                                <a href="#" class="dropdown-toggle" data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i> 张三<span class="caret"></span></a>
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

    <div class="container theme-showcase" role="main">

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <nav class="navbar navbar-default" role="navigation">
                        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                            <ul class="nav navbar-nav">
                                <li>
                                    <a rel="nofollow" href="index.html"><i class="glyphicon glyphicon-home"></i> 众筹首页</a>
                                </li>
                                <li >
                                    <a rel="nofollow" href="projects.html"><i class="glyphicon glyphicon-th-large"></i> 项目总览</a>
                                </li>
                                <li class="active">
                                    <a rel="nofollow" href="javascript:;"><i class="glyphicon glyphicon-edit"></i> 发起项目</a>
                                </li>
                                <li>
                                    <a rel="nofollow" href="minecrowdfunding.html"><i class="glyphicon glyphicon-user"></i> 我的众筹</a>
                                </li>
                            </ul>
                        </div>
                    </nav>
                </div>
            </div>
        </div>


        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="panel panel-default" >
                        <div class="panel-heading" style="text-align:center;">
                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;    position: absolute;width: 75%;right:49px;">
                                            <div class="progress-bar progress-bar-default" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">1. 项目及发起人信息</div>
                                            </div>
                                        </div>
                                        <div style="right: 0;border:10px solid #ddd;border-left-color: #88b7d5;border-width: 25px;position: absolute;
                                                border-left-color: rgba(221, 221, 221, 1);
                                                border-top-color: rgba(221, 221, 221, 0);
                                                border-bottom-color: rgba(221, 221, 221, 0);
                                                border-right-color: rgba(221, 221, 221, 0);
                                            ">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;position: absolute;width: 75%;right:49px;">
                                            <div class="progress-bar progress-bar-default" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">2. 回报设置</div>
                                            </div>
                                        </div>
                                        <div style="right: 0;border:10px solid #ddd;border-left-color: #88b7d5;border-width: 25px;position: absolute;
                                                border-left-color: rgba(221, 221, 221, 1);
                                                border-top-color: rgba(221, 221, 221, 0);
                                                border-bottom-color: rgba(221, 221, 221, 0);
                                                border-right-color: rgba(221, 221, 221, 0);
                                            ">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;position: absolute;width: 75%;right:49px;">
                                            <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">3. 确认信息</div>
                                            </div>
                                        </div>
                                        <div style="right: 0;border:10px solid #ddd;border-left-color: #88b7d5;border-width: 25px;position: absolute;
                                                border-left-color: rgba(92, 184, 92, 1);
                                                border-top-color: rgba(92, 184, 92, 0);
                                                border-bottom-color: rgba(92, 184, 92, 0);
                                                border-right-color: rgba(92, 184, 92, 0);
                                            ">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;">
                                            <div class="progress-bar progress-bar-default" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">4. 完成</div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="panel-body">

                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-12 column">
                                        <blockquote style="border-left: 5px solid #f60;color:#f60;padding: 0 0 0 20px;">
                                            <b>
                                                确认信息（请填写易付宝企业账号用于收款及身份核实）
                                            </b>
                                        </blockquote>
                                    </div>
                                    <div class="col-md-12 column">


                                        <div class="row clearfix">
                                            <div class="col-md-6 column">
                                                <form id="confirmFomr" th:action="@{/project/create/confirm}" method="post" role="form">
                                                    <div class="form-group">
                                                        <label for="exampleInputEmail1">易付宝企业账号：</label><input type="email" name="paynum" class="form-control" id="exampleInputEmail1" />
                                                    </div>
                                                    <div class="form-group">
                                                        <label for="exampleInputPassword1">法人身份证号：</label><input type="password" name="cardnum" class="form-control" id="exampleInputPassword1" />
                                                    </div>
                                                </form>
                                            </div>
                                            <div class="col-md-6 column">
                                                <div class="panel panel-default">
                                                    <div class="panel-body" style="padding:40px;">
                                                        <i class="glyphicon glyphicon-user"></i> 易购账户名：18801282948<br><br><span style="margin-left:60px;">您正在使用该账号发起众筹项目</span>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="panel-footer" style="text-align:center;">
                            <button type="button" class="btn  btn-default btn-lg">上一步</button>

                            <script type="text/javascript">
                                $(function(){
                                    $("#submitBtn").click(function(){
                                        $("#confirmFomr").submit();
                                    });
                                });
                            </script>
                            <button type="button" id="submitBtn" class="btn  btn-warning btn-lg">提交</button>
                            <a class="btn" > 保存草稿 </a>
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
                            <a rel="nofollow" href="http://www.layoutit.cn">关于我们</a> | <a rel="nofollow" href="http://www.layoutit.cn">服务条款</a> | <a rel="nofollow" href="http://www.layoutit.cn">免责声明</a> | <a rel="nofollow" href="http://www.layoutit.cn">网站地图</a> | <a rel="nofollow" href="http://www.layoutit.cn">联系我们</a>
                        </div>
                        <div class="copyRight">
                            Copyright ?2017-2017layoutit.cn 版权所有
                        </div>
                    </div>

                </div>
            </div>
        </div>

    </div> <!-- /container -->
    </body>
    </html>
    ```

## 14.4 收集确认信息

### 14.4.1 前端代码

1. 修改提交按钮
2. 跳转表代码
3. 给提交按钮绑定单击响应函数
4. 以上三条上面的页面中已完成

### 14.4.2 后端代码

1. handler

    ```java

    ```
2. view-controller

    ```java
    registry.addViewController("/create/success").setViewName("project-success");
    ```

3. api模块 MySQLRemoteService

    ```java
    @RequestMapping("/save/project/vo/remote")
    ResultEntity<String> saveProjectVORemote(@RequestBody ProjectVO projectVO, @RequestParam("memberId") Integer memberId);
    ```

### 14.4.3 前端代码

1. 新建 project-success.html 页面

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

            .label-type, .label-status, .label-order {
                background-color: #fff;
                color: #f60;
                padding : 5px;
                border: 1px #f60 solid;
            }
            #typeList  :not(:first-child) {
                margin-top:20px;
            }
            .label-txt {
                margin:10px 10px;
                border:1px solid #ddd;
                padding : 4px;
                font-size:14px;
            }
            .panel {
                border-radius:0;
            }

            .progress-bar-default {
                background-color: #ddd;
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
                                <a href="#" class="dropdown-toggle" data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i> 张三<span class="caret"></span></a>
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

    <div class="container theme-showcase" role="main">

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <nav class="navbar navbar-default" role="navigation">
                        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                            <ul class="nav navbar-nav">
                                <li>
                                    <a rel="nofollow" href="index.html"><i class="glyphicon glyphicon-home"></i> 众筹首页</a>
                                </li>
                                <li >
                                    <a rel="nofollow" href="projects.html"><i class="glyphicon glyphicon-th-large"></i> 项目总览</a>
                                </li>
                                <li class="active">
                                    <a rel="nofollow" href="javascript:;"><i class="glyphicon glyphicon-edit"></i> 发起项目</a>
                                </li>
                                <li>
                                    <a rel="nofollow" href="minecrowdfunding.html"><i class="glyphicon glyphicon-user"></i> 我的众筹</a>
                                </li>
                            </ul>
                        </div>
                    </nav>
                </div>
            </div>
        </div>


        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="panel panel-default" >
                        <div class="panel-heading" style="text-align:center;">
                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;    position: absolute;width: 75%;right:49px;">
                                            <div class="progress-bar progress-bar-default" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">1. 项目及发起人信息</div>
                                            </div>
                                        </div>
                                        <div style="right: 0;border:10px solid #ddd;border-left-color: #88b7d5;border-width: 25px;position: absolute;
                                                border-left-color: rgba(221, 221, 221, 1);
                                                border-top-color: rgba(221, 221, 221, 0);
                                                border-bottom-color: rgba(221, 221, 221, 0);
                                                border-right-color: rgba(221, 221, 221, 0);
                                            ">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;position: absolute;width: 75%;right:49px;">
                                            <div class="progress-bar progress-bar-default" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">2. 回报设置</div>
                                            </div>
                                        </div>
                                        <div style="right: 0;border:10px solid #ddd;border-left-color: #88b7d5;border-width: 25px;position: absolute;
                                                border-left-color: rgba(221, 221, 221, 1);
                                                border-top-color: rgba(221, 221, 221, 0);
                                                border-bottom-color: rgba(221, 221, 221, 0);
                                                border-right-color: rgba(221, 221, 221, 0);
                                            ">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;position: absolute;width: 75%;right:49px;">
                                            <div class="progress-bar progress-bar-default" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">3. 确认信息</div>
                                            </div>
                                        </div>
                                        <div style="right: 0;border:10px solid #ddd;border-left-color: #88b7d5;border-width: 25px;position: absolute;
                                                border-left-color: rgba(221, 221, 221, 1);
                                                border-top-color: rgba(221, 221, 221, 0);
                                                border-bottom-color: rgba(221, 221, 221, 0);
                                                border-right-color: rgba(221, 221, 221, 0);
                                            ">
                                        </div>
                                    </div>
                                    <div class="col-md-3 column">
                                        <div class="progress" style="height:50px;border-radius:0;">
                                            <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="width: 100%;height:50px;">
                                                <div style="font-size:16px;margin-top:15px;">4. 完成</div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="panel-body">

                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-12 column">
                                        <blockquote style="border-left: 5px solid #f60;color:#f60;padding: 0 0 0 20px;">
                                            <b>
                                                完成
                                            </b>
                                        </blockquote>
                                    </div>
                                    <div class="col-md-12 column">


                                        <div class="row clearfix">
                                            <div class="col-md-12 column">
                                                <div class="panel panel-default">
                                                    <div class="panel-body" style="padding:60px;">
                                                        <i class="glyphicon glyphicon-ok" style="color:green;font-size:40px;"></i> <span style="margin-left:20px;">你发起的众筹项目信息已经提交完毕，我们会在5~7个工作日内对项目进行审核，请耐心等候</span>
                                                        <p style="margin-left:60px;">你能在会员中心-我的众筹- 发起的项目中查看</p>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="panel-footer" style="text-align:center;">
                            <button type="button" class="btn  btn-warning btn-lg" onclick="window.location.href='minecrowdfunding.html'">我的众筹</button>
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

    </div> <!-- /container -->
    <script src="jquery/jquery-2.1.1.min.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
    <script src="script/docs.min.js"></script>
    <script src="script/back-to-top.js"></script>
    <script>
        $('#myTab a').click(function (e) {
            e.preventDefault()
            $(this).tab('show')
        })
    </script>
    </body>
    </html>
    ```

### 14.4.4 执行数据库保存 - mysql模块

1. 思路

    ![思路](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/14-4-4.png)

2. ProjectProviderHandler

    ```java
    @RequestMapping("/save/project/vo/remote")
    ResultEntity<String> saveProjectVORemote(@RequestBody ProjectVO projectVO,
                                             @RequestParam("memberId") Integer memberId){

        try {
            projectService.saveProject(projectVO, memberId);

            return ResultEntity.successWithoutData();
        } catch (Exception e) {
            e.printStackTrace();

            return ResultEntity.failed(e.getMessage());
        }
    }
    ```

3. service

    ```java
    void saveProject(ProjectVO projectVO, Integer memberId);
    ```

    ```java
    @Autowired
    private ProjectPOMapper projectPOMapper;

    @Autowired
    private ProjectItemPicPOMapper projectItemPicPOMapper;

    @Autowired
    private MemberLaunchInfoPOMapper mapperLaunchInfoPOMapper;

    @Autowired
    private MemberConfirmInfoPOMapper memberConfirmInfoPOMapper;

    @Autowired
    private ReturnPOMapper returnPOMapper;

    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
    @Override
    public void saveProject(ProjectVO projectVO, Integer memberId) {

        // 一、保存ProjectPO对象
        // 1.创建空的ProjectPO对象
        ProjectPO projectPO = new ProjectPO();

        // 2.把projectVO中的属性复制到projectPO中
        BeanUtils.copyProperties(projectVO, projectPO);

        // 3.保存projectPO
        projectPOMapper.insertSelective(projectPO);

        // 4.从projectPO对象中获取自增主键
        Integer projectId = projectPO.getId();

        // 二、保存项目、分类的关联信息
        // 1. 从projectVO中获取typeList
        List<Integer> typeIdList = projectVO.getTypeIdList();
        projectPOMapper.insertTypeRelationShip(typeIdList, projectId);

        // 三、保存项目、标签的关联信息
        List<Integer> tagIdList = projectVO.getTagIdList();
        projectPOMapper.insertTagRelationShip(tagIdList, projectId);

        // 四、保存项目中详情图片路径信息
        List<String> detailPicturePathList = projectVO.getDetailPicturePathList();
        projectItemPicPOMapper.insertPathList(detailPicturePathList, projectId);

        // 五、保存项目发起人信息
        MemberLauchInfoVO memberLauchInfoVO = projectVO.getMemberLauchInfoVO();
        MemberLaunchInfoPO memberLaunchInfoPO = new MemberLaunchInfoPO();
        BeanUtils.copyProperties(memberLauchInfoVO, memberLaunchInfoPO);
        memberLaunchInfoPO.setMemberid(memberId);

        mapperLaunchInfoPOMapper.insert(memberLaunchInfoPO);

        // 六、保存项目回报信息
        List<ReturnVO> returnVOList = projectVO.getReturnVOList();
        
        List<ReturnPO> returnPOList = new ArrayList<>();

        for (ReturnVO returnVO : returnVOList) {

            ReturnPO returnPO = new ReturnPO();

            BeanUtils.copyProperties(returnVO, returnPO);

            returnPOList.add(returnPO);
        }

        returnPOMapper.insertReturnPOBatch(returnPOList, projectId);

        // 七、保存项目确认信息
        MemberConfirmInfoVO memberConfirmInfoVO = projectVO.getMemberConfirmInfoVO();
        MemberConfirmInfoPO memberConfirmInfoPO = new MemberConfirmInfoPO();
        BeanUtils.copyProperties(memberConfirmInfoVO, memberConfirmInfoPO);
        memberConfirmInfoPO.setMemberid(memberId);
        memberConfirmInfoPOMapper.insert(memberConfirmInfoPO);
    }
    ```

4. ProjectPOMapper.java

    ```java
    void insertTypeRelationShip(@Param("typeIdList") List<Integer> typeIdList, @Param("projectId") Integer projectId);

    void insertTagRelationShip(@Param("tagIdList") List<Integer> tagIdList, @Param("projectId") Integer projectId);
    ```

5. ProjectPOMapper.xml
    
    修改：
    ```xml
    <insert id="insertSelective" useGeneratedKeys="true" keyProperty="id" parameterType="com.atguigu.crowd.entity.po.ProjectPO">
    ```

    新增：
    ```xml
    <insert id="insertTypeRelationShip">
        insert into t_project_type(`projectid`, `typeid`) values 
        <foreach collection="typeIdList" item="typeId" separator=",">(#{projectId},#{typeId})</foreach>
    </insert>
    <insert id="insertTagRelationShip">
        insert into t_project_tag(`projectid`, `tagid`) values
        <foreach collection="tagIdList" item="tagId" separator=",">(#{projectId},#{tagId})</foreach>
    </insert>
    ```

6. ProjectItemPicPOMapper

    ```java
    void insertPathList(@Param("detailPicturePathList") List<String> detailPicturePathList, @Param("projectId") Integer projectId);
    ```

7. ProjectItemPicPOMapper.xml

    ```xml
    <insert id="insertPathList">
        insert into t_project_item_pic(projectid, item_pic_path) values
        <foreach collection="detailPicturePathList" item="detailPath" separator=",">
        (#{projectId}, #{detailPath})
        </foreach>
    </insert>
    ```

8. ReturnPOMapper

    ```java
    void insertReturnPOBatch(@Param("returnPOList") List<ReturnPO> returnPOList, @Param("projectId") Integer projectId);
    ```

9. ReturnPOMapper.xml

    ```xml
    <insert id="insertReturnPOBatch">
        insert into t_return (projectid, type,
                            supportmoney, content, count,
                            signalpurchase, purchase, freight,
                            invoice, returndate, describ_pic_path
        )
        values
        <foreach collection="returnPOList" item="returnPO" separator=",">
        (#{projectId},#{returnPO.type},#{returnPO.supportmoney},#{returnPO.content},#{returnPO.count},
        #{returnPO.signalpurchase},#{returnPO.purchase},#{returnPO.freight},
        #{returnPO.invoice},#{returnPO.returndate},#{returnPO.describPicPath})
        </foreach>
    </insert>
    ```

### 14.4.5 bug 修复

1. ProjectPO中 money 类型改为 Integer （getter、setter）
2. ProjectServiceImpl.java

    ```java
    // 2.把projectVO中的属性复制到projectPO中
    BeanUtils.copyProperties(projectVO, projectPO);

    // bug 修复
    projectPO.setMemberid(memberId);
    String createDate = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
    projectPO.setCreatedate(createDate);
    projectPO.setStatus(0);
    ```





