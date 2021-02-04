---
title: Chapter15 — 前台 展示项目
excerpt: 首页显示真实数据、项目详情页
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
abbrlink: 17dad97d
date: 2021-01-27 23:41:29
updated: 2021-01-28 06:20:35
subtitle:
---
## 15.1 首页显示项目数据

### 15.1.1 目标思路

1. 目标
   * 在首页上加载真实保存到数据库的项目数据， 按分类显示
2. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/15-1-1.png)

### 15.1.2 实体类

1. PortalProjectVO

    ```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class PortalProjectVO {

        private Integer projectId;
        private String projectName;
        private String headerPicturePath;
        private Integer money;
        private String deployDate;
        private Integer percentage;
        private Integer supporter;
        
    }
    ```
2. PortalTypeVO

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class PortalTypeVO {

        private Integer id;
        private String name;
        private String remark;

        private List<PortalProjectVO> portalProjectVOList;
    }
    ```

### 15.1.3 sql

1. ProjectPOMapper.xml

    ```xml 
    <resultMap id="LoadPortalProjectListResultMap" type="com.atguigu.crowd.entity.vo.PortalTypeVO">
        <!-- 分类数据的常规属性 -->
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="remark" property="remark"/>

        <!-- 分类数据中包含的项目数据的 List -->
        <!-- property 属性： 对应 PortalTypeVO 中分类数据中的项目数据的 List 属性 -->
        <!-- column 属性： 接下来查询项目时需要用到分类 id， 就是通过 column 属性把值传入-->
        <!-- ofType 属性： 项目数据的实体类型 PortalProjectVO -->
        <collection property="portalProjectVOList" column="id"
                    ofType="com.atguigu.crowd.entity.vo.PortalProjectVO"
                    select="com.atguigu.crowd.mapper.MemberPOMapper.selectPortalProjectVOList"/>
    </resultMap>
    
    <select id="selectPortalTypeVOList" resultMap="LoadPortalProjectListResultMap">
        select id, name, remark from t_type
    </select>

    <select id="selectPortalProjectVOList" resultType="com.atguigu.crowd.entity.vo.PortalProjectVO">
        SELECT
        t_project.id projectId,
        project_name projectName,
        money,
        deploydate deployDate,
        supportmoney/money*100 percentage,
        supporter supporter,
        header_picture_path headerPicturePath
        FROM t_project LEFT JOIN t_project_type ON t_project.id=t_project_type.projectid
        WHERE t_project_type.typeid=#{id}
        ORDER BY t_project.id DESC
        LIMIT 0,4
    </select>
    ```

2. ProjectPOMapper

    ```java
    List<PortalTypeVO> selectPortalTypeVOList();
    ```

3. 数据库插入测试数据
    
    ```sql
    INSERT INTO `t_type` VALUES (1, '科技', '开启智慧生活');
    INSERT INTO `t_type` VALUES (2, '设计', '创建改变未来');
    INSERT INTO `t_type` VALUES (3, '农业', '网络天下肥美');
    INSERT INTO `t_type` VALUES (4, '公益', '汇集点点爱心');
    ```

4. test中进行测试

    ```java
    @Autowired
    private ProjectPOMapper projectPOMapper;

    @Test
    public void testLoadTypeData() {

        List<PortalTypeVO> typeVOList = projectPOMapper.selectPortalTypeVOList();
        for (PortalTypeVO portalTypeVO : typeVOList) {
            String name = portalTypeVO.getName();
            String remark = portalTypeVO.getRemark();
            log.info("name:"+name+" remark:"+remark);

            List<PortalProjectVO> portalProjectVOList = portalTypeVO.getPortalProjectVOList();
            for (PortalProjectVO portalProjectVO : portalProjectVOList) {
                if (portalProjectVO == null)
                    continue;
                log.info(portalProjectVO.toString());
            }
        }
    }
    ```

### 15.1.4 接口服务

1. ProjectService

    ```java
    List<PortalTypeVO> getPortalTypeVO();
    ```

    ```java
    @Override
    public List<PortalTypeVO> getPortalTypeVO() {

        return projectPOMapper.selectPortalTypeVOList();
    }
    ```

2. ProjectProviderHandler

    ```java
    @RequestMapping("/get/portal/type/project/data/remote")
    public ResultEntity<List<PortalTypeVO>> getPortalTypeProjectDataRemote() {

        try {
            List<PortalTypeVO> portalTypeVOList = projectService.getPortalTypeVO();

            return ResultEntity.successWithData(portalTypeVOList);
        } catch (Exception e) {
            e.printStackTrace();

            return ResultEntity.failed(e.getMessage());
        }
    }
    ```

3. api模块 MySQLRemoteService

    ```java
    @RequestMapping("/get/portal/type/project/data/remote")
    public ResultEntity<List<PortalTypeVO>> getPortalTypeProjectDataRemote();
    ```

### 15.1.5 页面显示

1. auth 模块 PortalHandler

    ```java
    @Autowired
    private MySQLRemoteService mySQLRemoteService;

    @RequestMapping("/")
    public String showPortalPage(Model model) {

        // 调用MySQLService 提供的方法查询首页数据
        ResultEntity<List<PortalTypeVO>> resultEntity = mySQLRemoteService.getPortalTypeProjectDataRemote();

        // 2.检查查询结果
        String result = resultEntity.getResult();
        if (ResultEntity.SUCCESS.equals(result)) {

            // 3.获取查询结果数据
            List<PortalTypeVO> list = resultEntity.getData();

            // 4.存入模型
            model.addAttribute(CrowdConstant.ATTR_NAME_PORTAL_DATA, list);
        }

        return "portal";
    }
    ```

2. 修改 portal.html

    ```html
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
                            <li><a href="login.html" th:href="@{/auth/member/to/login/page}">登录</a></li>
                            <li><a href="reg.html" th:href="@{/auth/member/to/reg/page}">注册</a></li>
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

        <div th:if="${#strings.isEmpty(portal_data)}">没有加载到任何分类数据</div>
        <div th:if="${not #strings.isEmpty(portal_data)}">
            <div th:each="type : ${portal_data}" class="container">
                <div class="row clearfix">
                    <div class="col-md-12 column">
                        <div class="box ui-draggable" id="mainBox">
                            <div class="mHd" style="">
                                <div class="path">
                                    <a href="projects.html">更多...</a>
                                </div>
                                <h3>
                                    <label th:text="${type.name}"> 科技 </label>  <small style="color:#FFF;" th:text="${type.remark}" >开启智慧未来</small>
                                </h3>
                            </div>
                            <div class="mBd" style="padding-top:10px;">
                                <div class="row">
                                    <div th:if="${#strings.isEmpty(type.portalProjectVOList)}">该分类下还没有项目</div>
                                    <div th:if="${not #strings.isEmpty(type.portalProjectVOList)}">
                                        <div th:each="project : ${type.portalProjectVOList}" class="col-md-3">
                                            <div class="thumbnail">
                                                <img alt="300x200" th:src="${project.headerPicturePath}" src="img/product-1.jpg" style="cursor: pointer;">
                                                <div class="caption">
                                                    <h3 class="break">
                                                        <a th:href="'http://www.crowd.com/project/get/project/detail/'+${project.projectId}" href="project.html" th:text="${project.projectName}">活性富氢净水直饮机</a>
                                                    </h3>
                                                    <p>
                                                    </p>
                                                    <div style="float:left;"><i class="glyphicon glyphicon-screenshot"
                                                                                title="目标金额"></i> $<span th:text="${project.money}">20,000</span>
                                                    </div>
                                                    <div style="float:right;"><i class="glyphicon glyphicon-calendar"
                                                                                title="截至日期"></i>
                                                        <span th:text="${project.deployDate}">2017-20-20</span>
                                                    </div>
                                                    <p></p>
                                                    <br>
                                                    <div class="progress" style="margin-bottom: 4px;">
                                                        <div aria-valuemax="100" aria-valuemin="0"
                                                            th:aria-valuenow="${project.percentage}" aria-valuenow="40"
                                                            class="progress-bar progress-bar-success" role="progressbar"
                                                            th:style="'width:'+${project.percentage}+'%'" style="width: 40%">
                                                            <span th:text="${project.percentage}+'% '">40% </span>
                                                        </div>
                                                    </div>
                                                    <div><span style="float:right;"><i
                                                            class="glyphicon glyphicon-star-empty"></i></span> <span><i
                                                            class="glyphicon glyphicon-user" title="支持人数"></i> <span th:text="${project.supporter}"> 12345</span></span></div>
                                                </div>
                                            </div>
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
    ```

## 15.2 显示项目详情

### 15.2.1 思路

1. 说明：首页点击项目，跳转到项目详情
2. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/15-2-1.png)

### 15.2.2 实体类

1. DetailReturnVO

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class DetailReturnVO {

        // 回报信息主键
        private Integer returnId;

        // 当前档位需支持的金额
        private Integer supportMoney;

        // 单笔限购， 取值为 0 时无限额， 取值为 1 时有限额
        private Integer signalPurchase;

        // 具体限额数量
        private Integer purchase;

        // 当前该档位支持者数量
        private Integer supproterCount;

        // 运费， 取值为 0 时表示包邮
        private Integer freight;
        
        // 众筹成功后多少天发货
        private Integer returnDate;

        // 回报内容
        private String content;
    }
    ```

2. DetailProjectVO

    ```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class DetailProjectVO {

        private Integer projectId;
        private String projectName;
        private String projectDesc;
        private Integer followerCount;
        private Integer status;
        private Integer day;
        private String statusText;
        private Integer money;
        private Integer supportMoney;
        private Integer percentage;
        private String deployDate;
        private Integer lastDay;
        private Integer supporterCount;
        private String headerPicturePath;
        private List<String> detailPicturePathList;
        private List<DetailReturnVO> detailReturnVOList;
    }
    ```

### 15.2.3 sql相关

1. ProjectPOMapper.xml

    ```xml
    <resultMap id="LoadProjectDetailResultMap" type="com.atguigu.crowd.entity.vo.DetailProjectVO">
        <id column="id" property="projectId"/>
        <result column="project_name" property="projectName"/>
        <result column="project_description" property="projectDesc"/>
        <result column="money" property="money"/>
        <result column="status" property="status"/>
        <result column="day" property="day"/>
        <result column="deploydate" property="deployDate"/>
        <result column="supportmoney" property="supportMoney"/>
        <result column="follower" property="followerCount"/>
        <result column="supporter" property="supporterCount"/>
        <result column="header_picture_path" property="headerPicturePath"/>
        <collection
                property="detailPicturePathList"
                select="com.atguigu.crowd.mapper.ProjectPOMapper.selectDetailPicturePath"
                column="id"/>
        <collection
                property="detailReturnVOList"
                select="com.atguigu.crowd.mapper.ProjectPOMapper.selectDetailReturnVO"
                column="id"/>
    </resultMap>

    <select id="selectDetailPicturePath" resultType="string">
        select item_pic_path from t_project_item_pic where projectid=#{id}
    </select>

    <select id="selectDetailReturnVO" resultType="com.atguigu.crowd.entity.vo.DetailReturnVO">
        select
        id returnId,
        supportmoney supportMoney,
        content,
        signalpurchase signalPurchase,
        purchase,
        freight,
        returndate returnDate
        from t_return
        where projectid=#{id}
    </select>

    <select id="selectDetailProjectVO" resultMap="LoadProjectDetailResultMap">
        select
        id,
        project_name,
        project_description ,
        money,
        status,
        day,
        deploydate,
        supportmoney,
        supporter,
        supportmoney/money*100 percentage,
        follower,
        header_picture_path
        from
        t_project
        where id=#{projectId}
    </select>
    ```

2. ProjectPOMapper

    ```java
    DetailProjectVO selectDetailProjectVO(Integer projectId);
    ```

3. 测试

    ```java
    @Test
    public void testLoadDetailProjectVO() {

        Integer projectId = 1;
        DetailProjectVO detailProjectVO = projectPOMapper.selectDetailProjectVO(projectId);
        if (detailProjectVO == null)
            log.warn("项目不存在，项目编号："+ projectId);
        else
            log.info(detailProjectVO.getProjectName());
    }
    ```

### 15.2.4 后端代码

1. service

    ```java
    @Override
    public DetailProjectVO getDetailProjectVO(Integer projectId) {

        // 1.查询得到 DetailProjectVO 对象
        DetailProjectVO detailProjectVO = projectPOMapper.selectDetailProjectVO(projectId);
        if (detailProjectVO == null) {

            log.error("项目不存在，项目编号："+ projectId);
            return null;
        }

        // 2.根据 status 确定 statusText
        Integer status = detailProjectVO.getStatus();

        switch (status){
            case 0:
                detailProjectVO.setStatusText("审核中");
                break;
            case 1:
                detailProjectVO.setStatusText("众筹中");
                break;
            case 2:
                detailProjectVO.setStatusText("众筹成功");
                break;
            case 3:
                detailProjectVO.setStatusText("已关闭");
                break;
            default:
                break;
        }

        // 3.根据 deployeDate 计算 lastDay
        String deployDate = detailProjectVO.getDeployDate();

        // 获取当前日期
        Date currentDay = new Date();

        // 把众筹日期解析成 Date 类型
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        try {
            Date deployDay = format.parse(deployDate);
            // 获取当前当前日期的时间戳
            long currentTimeStamp = currentDay.getTime();
            // 获取众筹日期的时间戳
            long deployTimeStamp = deployDay.getTime();
            // 两个时间戳相减计算当前已经过去的时间
            long pastDays = (currentTimeStamp - deployTimeStamp) / 1000 / 60 / 60 / 24;
            // 获取总的众筹天数
            Integer totalDays = detailProjectVO.getDay();
            // 使用总的众筹天数减去已经过去的天数得到剩余天数
            Integer lastDay = (int) (totalDays - pastDays);
            detailProjectVO.setLastDay(lastDay);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return detailProjectVO;
    }
    ```

2. handler

    ```java
    @RequestMapping("/get/project/detail/remote/{projectId}")
    public ResultEntity<DetailProjectVO> getDetailProjectVORemote(@PathVariable("projectId") Integer projectId) {

        try {
            DetailProjectVO detailProjectVO = projectService.getDetailProjectVO(projectId);
            return ResultEntity.successWithData(detailProjectVO);
        } catch (Exception e) {
            e.printStackTrace();
            return ResultEntity.failed(e.getMessage());
        }
    }
    ```

3. api 模块 MySQLRemoteService

    ```java
    @RequestMapping("/get/project/detail/remote/{projectId}")
    ResultEntity<DetailProjectVO> getDetailProjectVORemote(@PathVariable("projectId") Integer projectId);
    ```

### 15.2.5 显示详情页

1. ProjectConsumerHandler

    ```java
    @RequestMapping("/get/project/detail/{projectId}")
    public String getProjectDetail(@PathVariable("projectId") Integer projectId,
                                   Model model) {

        ResultEntity<DetailProjectVO> resultEntity = mySQLRemoteService.getDetailProjectVORemote(projectId);

        if (ResultEntity.SUCCESS.equals(resultEntity.getResult())) {
            DetailProjectVO detailProjectVO = resultEntity.getData();
            model.addAttribute("detailProjectVO", detailProjectVO);
        }

        return "project-show-detail";
    }
    ```

2. 新建 project-show-detail.html

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="author" content="">
        <base th:href="@{/}" />
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

            .nav-tabs>li.active>a, .nav-tabs>li.active>a:focus, .nav-tabs>li.active>a:hover
            {
                border-bottom-color: #ddd;
            }

            .nav-tabs>li>a {
                border-radius: 0;
            }
        </style>
    </head>
    <body>
    <div class="navbar-wrapper">
        <div class="container">
            <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
                <div class="container">
                    <div class="navbar-header">
                        <a class="navbar-brand" href="#" style="font-size: 32px;">尚筹网-创意产品众筹平台</a>
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
                                        class="glyphicon glyphicon-th-large"></i> 众筹项目</a></li>
                                <li><a rel="nofollow" href="start.html"><i
                                        class="glyphicon glyphicon-edit"></i> 发起众筹</a></li>
                                <li><a rel="nofollow" href="minecrowdfunding.html"><i
                                        class="glyphicon glyphicon-user"></i> 我的众筹</a></li>
                            </ul>
                        </div>
                    </nav>
                </div>
            </div>
        </div>

        <div th:if="${#strings.isEmpty(detailProjectVO)}">查询项目详情信息失败！</div>
        <div th:if="${not #strings.isEmpty(detailProjectVO)}">
            <div class="container">
                <div class="row clearfix">
                    <div class="col-md-12 column">
                        <div class="jumbotron nofollow" style="padding-top: 10px;">
                            <h3 th:text="${detailProjectVO.projectName}">酷驰触控龙头，智享厨房黑科技</h3>
                            <div style="float: left; width: 70%;" th:text="${detailProjectVO.projectDesc}">智能时代，酷驰触控厨房龙头，让煮妇解放双手，触发更多烹饪灵感，让美味信手拈来。</div>
                            <div style="float: right;">
                                <button type="button" class="btn btn-default">
                                    <i style="color: #f60" class="glyphicon glyphicon-heart"></i>
                                    关注[[${detailProjectVO.followerCount}]]
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="container">
                <div class="row clearfix">
                    <div class="col-md-12 column">
                        <div class="row clearfix">
                            <div class="col-md-8 column" th:if="${#strings.isEmpty(detailProjectVO.detailPicturePathList)}">加载详情信息失败</div>
                            <div class="col-md-8 column" th:if="${not #strings.isEmpty(detailProjectVO.detailPicturePathList)}">
                                <img alt="140x140" width="740" src="img/product_detail_head.jpg" th:src="${detailProjectVO.headerPicturePath}" />
                                <img alt="140x140" width="740" src="img/product_detail_body.jpg" th:each="detailPicturePath : ${detailProjectVO.detailPicturePathList}" th:src="${detailPicturePath}" />
                            </div>
                            <div class="col-md-4 column">
                                <div class="panel panel-default" style="border-radius: 0px;">
                                    <div class="panel-heading"
                                        style="background-color: #fff; border-color: #fff;">
                                            <span class="label label-success"><i
                                                    class="glyphicon glyphicon-tag"></i> [[${detailProjectVO.statusText}]]</span>
                                    </div>
                                    <div class="panel-body">
                                        <h3>已筹资金：￥[[${detailProjectVO.supportMoney}]]</h3>
                                        <p>
                                            <span>目标金额 ： [[${detailProjectVO.money}]]</span><span style="float: right;">达成
                                                    ： [[${detailProjectVO.percentage}]]%</span>
                                        </p>
                                        <div class="progress"
                                            style="height: 10px; margin-bottom: 5px;">
                                            <div class="progress-bar progress-bar-success"
                                                role="progressbar" aria-valuenow="[[${detailProjectVO.percentage}]]" aria-valuemin="0"
                                                aria-valuemax="100" style="width: 60%;" th:style="'width: '+${detailProjectVO.percentage}+'%;'"></div>
                                        </div>
                                        <p>剩余 [[${detailProjectVO.lastDay}]] 天</p>
                                        <div>
                                            <p>
                                                <span>已有[[${detailProjectVO.supporterCount}]]人支持该项目</span>
                                            </p>
                                            <button type="button"
                                                    class="btn  btn-warning btn-lg btn-block"
                                                    data-toggle="modal" data-target="#myModal">立即支持</button>
                                        </div>
                                    </div>
                                    <div class="panel-footer"
                                        style="background-color: #fff; border-top: 1px solid #ddd; border-bottom-right-radius: 0px; border-bottom-left-radius: 0px;">
                                        <div class="container-fluid">
                                            <div class="row clearfix">
                                                <div class="col-md-3 column" style="padding: 0;">
                                                    <img alt="140x140" src="img/services-box2.jpg"
                                                        data-holder-rendered="true"
                                                        style="width: 80px; height: 80px;">
                                                </div>
                                                <div class="col-md-9 column">
                                                    <div class="">
                                                        <h4>
                                                            <b>福建省南安厨卫</b> <span
                                                                style="float: right; font-size: 12px;"
                                                                class="label label-success">已认证</span>
                                                        </h4>
                                                        <p style="font-size: 12px">
                                                            酷驰是一家年轻的厨卫科技公司，我们有一支朝气蓬勃，有激情、有想法、敢实践的团队。</p>
                                                        <p style="font-size: 12px">客服电话:0595-86218855</p>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div th:if="${#strings.isEmpty(detailProjectVO.detailReturnVOList)}">没有加载到项目回报信息</div>
                                <div th:if="${not #strings.isEmpty(detailProjectVO.detailReturnVOList)}">
                                    <div th:each="return : ${detailProjectVO.detailReturnVOList}" class="panel panel-default" style="border-radius: 0px;">
                                        <div class="panel-heading">
                                            <h3>
                                                ￥[[${return.supportMoney}]]
                                                <span th:if="${return.signalPurchase == 0}" style="float: right; font-size: 12px;">无限额，447位支持者</span>
                                                <span th:if="${return.signalPurchase == 1}" style="float: right; font-size: 12px;">限额[[${return.purchase}]]位，剩余1966位</span>
                                            </h3>
                                        </div>
                                        <div class="panel-body">
                                            <p th:if="${return.freight==0}">配送费用：包邮</p>
                                            <p th:if="${return.freight>0}">配送费用：[[${return.freight}]]</p>
                                            <p>预计发放时间：项目筹款成功后的[[${return.returnDate}]]天内</p>
                                            <button type="button" class="btn  btn-warning btn-lg"
                                                    onclick="window.location.href='pay-step-1.html'">支持</button>
                                            <br>
                                            <br>
                                            <p th:text="${return.content}">感谢您的支持，在众筹开始后，您将以79元的优惠价格获得“遇见彩虹?”智能插座一件（参考价208元）。</p>
                                        </div>
                                    </div>
                                </div>
                                <div class=" panel panel-default" style="border-radius: 0px;">
                                    <div class="panel-heading">
                                        <h3>风险提示</h3>
                                    </div>
                                    <div class="panel-body">
                                        <p>
                                            1.众筹并非商品交易，存在一定风险。支持者根据自己的判断选择、支持众筹项目，与发起人共同实现梦想并获得发起人承诺的回报。<br>
                                            2.众筹平台仅提供平台网络空间及技术支持等中介服务，众筹仅存在于发起人和支持者之间，使用众筹平台产生的法律后果由发起人与支持者自行承担。<br>
                                            3.本项目必须在2017-06-09之前达到￥10000.00
                                            的目标才算成功，否则已经支持的订单将取消。订单取消或募集失败的，您支持的金额将原支付路径退回。<br>
                                            4.请在支持项目后15分钟内付款，否则您的支持请求会被自动关闭。<br>
                                            5.众筹成功后由发起人统一进行发货，售后服务由发起人统一提供；如果发生发起人无法发放回报、延迟发放回报、不提供回报后续服务等情况，您需要直接和发起人协商解决。<br>
                                            6.如您不同意上述风险提示内容，您有权选择不支持；一旦选择支持，视为您已确认并同意以上提示内容。
                                        </p>
                                    </div>
                                </div>

                                <div>
                                    <h2>为你推荐</h2>
                                    <hr>
                                </div>
                                <div class="prjtip panel panel-default"
                                    style="border-radius: 0px;">
                                    <div class="panel-body">
                                        <img src="img/product-3.png" width="100%" height="100%">
                                    </div>
                                </div>

                                <div class="prjtip panel panel-default"
                                    style="border-radius: 0px;">
                                    <div class="panel-body">
                                        <img src="img/product-4.jpg" width="100%" height="100%">
                                    </div>
                                </div>
                            </div>
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
                            <a rel="nofollow" href="http://www.atguigu.com">关于我们</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">服务条款</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">免责声明</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">网站地图</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">联系我们</a>
                        </div>
                        <div class="copyRight">Copyright ?2010-2014atguigu.com 版权所有
                        </div>
                    </div>

                </div>
            </div>
        </div>

    </div>
    <!-- /container -->


    <div class="modal fade" id="myModal" tabindex="-1" role="dialog"
        aria-labelledby="myModalLabel">
        <div class="modal-dialog " style="width: 960px; height: 400px;"
            role="document">
            <div class="modal-content" data-spy="scroll"
                data-target="#myScrollspy">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal"
                            aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                    <h4 class="modal-title" id="myModalLabel">选择支持项</h4>
                </div>
                <div class="modal-body">
                    <div class="container-fluid">
                        <div class="row clearfix">
                            <div class="col-sm-3 col-md-3 column" id="myScrollspy">
                                <ul class="nav nav-tabs nav-stacked">
                                    <li class="active"><a href="#section-1">￥1.00</a></li>
                                    <li class="active"><a href="#section-2">￥149.00</a></li>
                                    <li class="active"><a href="#section-3">￥249.00</a></li>
                                    <li class="active"><a href="#section-4">￥549.00</a></li>
                                    <li class="active"><a href="#section-5">￥1999.00</a></li>
                                </ul>
                            </div>
                            <div id="navList" class="col-sm-9 col-md-9 column"
                                style="height: 400px; overflow-y: auto;">
                                <h2 id="section-1" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥1.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-2" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥149.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-3" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥249.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-4" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥549.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-5" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥1999.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                            </div>
                        </div>
                    </div>
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
        $(".prjtip img").css("cursor", "pointer");
        $(".prjtip img").click(function() {
            window.location.href = 'project.html';
        });
    </script>
    </body>
    </html>
    ```

3. 新建 project-detail.html

    ```html
    <!DOCTYPE html>
    <html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="description" content="">
        <meta name="author" content="">
        <base th:href="@{/}" />
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

            .nav-tabs>li.active>a, .nav-tabs>li.active>a:focus, .nav-tabs>li.active>a:hover
            {
                border-bottom-color: #ddd;
            }

            .nav-tabs>li>a {
                border-radius: 0;
            }
        </style>
    </head>
    <body>
    <div class="navbar-wrapper">
        <div class="container">
            <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
                <div class="container">
                    <div class="navbar-header">
                        <a class="navbar-brand" href="#" style="font-size: 32px;">尚筹网-创意产品众筹平台</a>
                    </div>
                    <div id="navbar" class="navbar-collapse collapse"
                        style="float: right;">
                        <ul class="nav navbar-nav">
                            <li class="dropdown"><a href="#" class="dropdown-toggle"
                                                    data-toggle="dropdown"><i class="glyphicon glyphicon-user"></i>
                                张三<span class="caret"></span></a>
                                <ul class="dropdown-menu" role="menu">
                                    <li><a href="member.html"><i
                                            class="glyphicon glyphicon-scale"></i> 会员中心</a></li>
                                    <li><a href="#"><i class="glyphicon glyphicon-comment"></i>
                                        消息</a></li>
                                    <li class="divider"></li>
                                    <li><a href="index.html"><i
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
                                        class="glyphicon glyphicon-th-large"></i> 众筹项目</a></li>
                                <li><a rel="nofollow" href="start.html"><i
                                        class="glyphicon glyphicon-edit"></i> 发起众筹</a></li>
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
                    <div class="jumbotron nofollow" style="padding-top: 10px;">
                        <h3>酷驰触控龙头，智享厨房黑科技</h3>
                        <div style="float: left; width: 70%;">
                            智能时代，酷驰触控厨房龙头，让煮妇解放双手，触发更多烹饪灵感，让美味信手拈来。</div>
                        <div style="float: right;">
                            <button type="button" class="btn btn-default">
                                <i style="color: #f60" class="glyphicon glyphicon-heart"></i> 关注
                                111
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="container">
            <div class="row clearfix">
                <div class="col-md-12 column">
                    <div class="row clearfix">
                        <div class="col-md-8 column">
                            <img alt="140x140" width="740" src="img/product_detail_head.jpg" />
                            <img alt="140x140" width="740" src="img/product_detail_body.jpg" />

                        </div>
                        <div class="col-md-4 column">
                            <div class="panel panel-default" style="border-radius: 0px;">
                                <div class="panel-heading"
                                    style="background-color: #fff; border-color: #fff;">
                                        <span class="label label-success"><i
                                                class="glyphicon glyphicon-tag"></i> 众筹中</span>
                                </div>
                                <div class="panel-body">
                                    <h3>已筹资金：￥50000.00</h3>
                                    <p>
                                        <span>目标金额 ： 1000.00</span><span style="float: right;">达成
                                                ： 60%</span>
                                    </p>
                                    <div class="progress" style="height: 10px; margin-bottom: 5px;">
                                        <div class="progress-bar progress-bar-success"
                                            role="progressbar" aria-valuenow="60" aria-valuemin="0"
                                            aria-valuemax="100" style="width: 60%;"></div>
                                    </div>
                                    <p>剩余 15 天</p>
                                    <div>
                                        <p>
                                            <span>已有629人支持该项目</span>
                                        </p>
                                        <!-- <button type="button"
                                            class="btn  btn-warning btn-lg btn-block" data-toggle="modal"
                                            data-target="#myModal">立即支持</button> -->
                                        <a class="btn  btn-warning btn-lg btn-block" href="http://www.crowd.com/pay/confirm/order">立即支持</a>
                                    </div>
                                </div>
                                <div class="panel-footer"
                                    style="background-color: #fff; border-top: 1px solid #ddd; border-bottom-right-radius: 0px; border-bottom-left-radius: 0px;">
                                    <div class="container-fluid">
                                        <div class="row clearfix">
                                            <div class="col-md-3 column" style="padding: 0;">
                                                <img alt="140x140" src="img/services-box2.jpg"
                                                    data-holder-rendered="true"
                                                    style="width: 80px; height: 80px;">
                                            </div>
                                            <div class="col-md-9 column">
                                                <div class="">
                                                    <h4>
                                                        <b>福建省南安厨卫</b> <span
                                                            style="float: right; font-size: 12px;"
                                                            class="label label-success">已认证</span>
                                                    </h4>
                                                    <p style="font-size: 12px">
                                                        酷驰是一家年轻的厨卫科技公司，我们有一支朝气蓬勃，有激情、有想法、敢实践的团队。</p>
                                                    <p style="font-size: 12px">客服电话:0595-86218855</p>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="panel panel-default" style="border-radius: 0px;">
                                <div class="panel-heading">
                                    <h3>
                                        ￥1.00 <span style="float: right; font-size: 12px;">无限额，447位支持者</span>
                                    </h3>
                                </div>
                                <div class="panel-body">
                                    <p>配送费用：包邮</p>
                                    <p>预计发放时间：项目筹款成功后的50天内</p>
                                    <button type="button" class="btn  btn-warning btn-lg"
                                            onclick="window.location.href='pay-step-1.html'">支持</button>
                                    <br>
                                    <br>
                                    <p>感谢您的支持，在众筹开始后，您将以79元的优惠价格获得“遇见彩虹?”智能插座一件（参考价208元）。</p>
                                </div>
                            </div>

                            <div class="panel panel-default" style="border-radius: 0px;">
                                <div class="panel-heading">
                                    <h3>
                                        ￥149.00 <span style="float: right; font-size: 12px;">限额2000位，剩余1966位</span>
                                    </h3>
                                </div>
                                <div class="panel-body">
                                    <p>配送费用：包邮</p>
                                    <p>预计发放时间：项目筹款成功后的50天内</p>
                                    <button type="button" class="btn  btn-warning btn-lg"
                                            onclick="window.location.href='pay-step-1.html'">支持</button>
                                    <br>
                                    <br>
                                    <p>感谢您的支持，在众筹开始后，您将以79元的优惠价格获得“遇见彩虹?”智能插座一件（参考价208元）。</p>
                                </div>
                            </div>
                            <div class=" panel panel-default" style="border-radius: 0px;">
                                <div class="panel-heading">
                                    <h3>风险提示</h3>
                                </div>
                                <div class="panel-body">
                                    <p>
                                        1.众筹并非商品交易，存在一定风险。支持者根据自己的判断选择、支持众筹项目，与发起人共同实现梦想并获得发起人承诺的回报。<br>
                                        2.众筹平台仅提供平台网络空间及技术支持等中介服务，众筹仅存在于发起人和支持者之间，使用众筹平台产生的法律后果由发起人与支持者自行承担。<br>
                                        3.本项目必须在2017-06-09之前达到￥10000.00
                                        的目标才算成功，否则已经支持的订单将取消。订单取消或募集失败的，您支持的金额将原支付路径退回。<br>
                                        4.请在支持项目后15分钟内付款，否则您的支持请求会被自动关闭。<br>
                                        5.众筹成功后由发起人统一进行发货，售后服务由发起人统一提供；如果发生发起人无法发放回报、延迟发放回报、不提供回报后续服务等情况，您需要直接和发起人协商解决。<br>
                                        6.如您不同意上述风险提示内容，您有权选择不支持；一旦选择支持，视为您已确认并同意以上提示内容。
                                    </p>
                                </div>
                            </div>

                            <div>
                                <h2>为你推荐</h2>
                                <hr>
                            </div>
                            <div class="prjtip panel panel-default"
                                style="border-radius: 0px;">
                                <div class="panel-body">
                                    <img src="img/product-3.png" width="100%" height="100%">
                                </div>
                            </div>

                            <div class="prjtip panel panel-default"
                                style="border-radius: 0px;">
                                <div class="panel-body">
                                    <img src="img/product-4.jpg" width="100%" height="100%">
                                </div>
                            </div>
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
                            <a rel="nofollow" href="http://www.atguigu.com">关于我们</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">服务条款</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">免责声明</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">网站地图</a> | <a
                                rel="nofollow" href="http://www.atguigu.com">联系我们</a>
                        </div>
                        <div class="copyRight">Copyright ?2010-2014atguigu.com 版权所有
                        </div>
                    </div>

                </div>
            </div>
        </div>

    </div>
    <!-- /container -->


    <div class="modal fade" id="myModal" tabindex="-1" role="dialog"
        aria-labelledby="myModalLabel">
        <div class="modal-dialog " style="width: 960px; height: 400px;"
            role="document">
            <div class="modal-content" data-spy="scroll"
                data-target="#myScrollspy">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal"
                            aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                    <h4 class="modal-title" id="myModalLabel">选择支持项</h4>
                </div>
                <div class="modal-body">
                    <div class="container-fluid">
                        <div class="row clearfix">
                            <div class="col-sm-3 col-md-3 column" id="myScrollspy">
                                <ul class="nav nav-tabs nav-stacked">
                                    <li class="active"><a href="#section-1">￥1.00</a></li>
                                    <li class="active"><a href="#section-2">￥149.00</a></li>
                                    <li class="active"><a href="#section-3">￥249.00</a></li>
                                    <li class="active"><a href="#section-4">￥549.00</a></li>
                                    <li class="active"><a href="#section-5">￥1999.00</a></li>
                                </ul>
                            </div>
                            <div id="navList" class="col-sm-9 col-md-9 column"
                                style="height: 400px; overflow-y: auto;">
                                <h2 id="section-1" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥1.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-2" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥149.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-3" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥249.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-4" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥549.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                                <hr>
                                <h2 id="section-5" style="border-bottom: 1px dashed #ddd;">
                                    <span style="font-size: 20px; font-weight: bold;">￥1999.00</span><span
                                        style="font-size: 12px; margin-left: 60px;">无限额，223位支持者</span>
                                </h2>
                                <p>配送费用：全国包邮</p>
                                <p>预计发放时间：项目筹款成功后的30天内</p>
                                <button type="button" class="btn  btn-warning btn-lg "
                                        onclick="window.location.href='pay-step-1.html'">支持</button>
                                <br>
                                <br>
                                <p>每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</p>
                            </div>
                        </div>
                    </div>
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
        $(".prjtip img").css("cursor", "pointer");
        $(".prjtip img").click(function() {
            window.location.href = 'project.html';
        });
    </script>
    </body>
    </html>
    ```

