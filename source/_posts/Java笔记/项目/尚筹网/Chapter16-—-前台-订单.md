---
title: Chapter16 — 前台 订单
excerpt: 项目支持页面，创建支持订单，保存收货地址
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
abbrlink: 1f72ca5
date: 2021-01-28 09:18:32
updated: 2021-01-28 23:57:44
subtitle:
---
## 16.1 搭建 order-consumer 开发环境

### 16.1.1 order 模块

1. 依赖：与project模块相同
2. 主启动类

    ```java
    @EnableFeignClients
    @EnableEurekaClient
    @SpringBootApplication
    public class OrderConsumerMain {

        public static void main(String[] args) {

            SpringApplication.run(OrderConsumerMain.class, args);
        }
    }
    ```

3. 配置文件

    ```yml
    server:
    port: 6000

    spring:
    application:
        name: atguigu-crowd-order
    thymeleaf:
        prefix: classpath:/templates/
        suffix: .html
    redis:
        host: 54.238.77.83
    session:
        store-type: redis

    ribbon:
    ReadTimeout: 60000
    ConnectTimeout: 60000

    eureka:
    client:
        service-url:
        defaultZone: http://localhost:1000/eureka
    ```

### 16.1.2 其它模块

1. zuul模块添加配置

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
        crowd-order:
        service-id: atguigu-crowd-order
        path: /order/**
    ```

## 16.2 建模

### 16.2.1 结构分析

1. 前端页面

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/16-2-1_1.png)

2. 结构分析

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/16-2-1_2.png)

### 16.2.2 建表

1. 订单表

    ```sql
    CREATE TABLE `project_crowd`.`t_order`
    (
        `id`                INT NOT NULL AUTO_INCREMENT COMMENT '主键',
        `order_num`         CHAR(100) COMMENT '订单号',
        `pay_order_num`     CHAR(100) COMMENT '支付宝流水号',
        `order_amount`      DOUBLE(10,5) COMMENT '订单金额',
        `invoice`           INT COMMENT '是否开发票（0 不开， 1 开） ',
        `invoice_title`     CHAR(100) COMMENT '发票抬头',
        `order_remark`      CHAR(100) COMMENT '订单备注',
        `address_id`        CHAR(100) COMMENT '收货地址 id',
        PRIMARY KEY (`id`)
    );
    ```

2. 收货地址表

    ```sql
    CREATE TABLE `project_crowd`.`t_address`
    (
        `id`                INT NOT NULL AUTO_INCREMENT COMMENT '主键',
        `receive_name`      CHAR(100) COMMENT '收件人',
        `phone_num`         CHAR(100) COMMENT '手机号',
        `address`           CHAR(200) COMMENT '收货地址',
        `member_id`         INT COMMENT '用户 id',
        PRIMARY KEY (`id`)
    );
    ```

3. 项目信息表

    ```sql
    CREATE TABLE `project_crowd`.`t_order_project`
    (
        `id`                INT NOT NULL AUTO_INCREMENT COMMENT '主键',
        `project_name`      CHAR(200) COMMENT '项目名称',
        `launch_name`       CHAR(100) COMMENT '发起人',
        `return_content`    CHAR(200) COMMENT '回报内容',
        `return_count`      INT COMMENT '回报数量',
        `support_price`     INT COMMENT '支持单价',
        `freight`           INT COMMENT '配送费用',
        `order_id`          INT COMMENT '订单表的主键',
        PRIMARY KEY (`id`)
    );
    ```

### 16.2.3 逆向工程

1. 修改 generatorConfig.xml

    ```xml
    <!--  数据库表名字和我们的 entity 类对应的映射指定 -->
    <table tableName="t_order" domainObjectName="OrderPO" />
    <table tableName="t_address" domainObjectName="AddressPO" />
    <table tableName="t_order_project" domainObjectName="OrderProjectPO" />
    ```

2. 生成
3. 资源归位

## 16.3 确认回报内容

### 16.3.1 分析

1. 页面

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/16-3-1.png)

2. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/16-3-1_2.png)

### 16.3.2 前端代码

1. 修改 project-show-detail.html

    ```html
    <!--<button type="button" class="btn  btn-warning btn-lg" onclick="window.location.href='pay-step-1.html'">支持</button>-->
    <a th:href="'http://www.crowd.com/order/confirm/return/info/'+${detailProjectVO.projectId}+'/'+${return.returnId}"
        class="btn  btn-warning btn-lg">支持</a>
    ```

2. 新建 confirm-return.html

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
                                                    确认回报内容</div>
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
                                                    确认订单</div>
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
                                                <div style="font-size: 16px; margin-top: 15px;">3. 付款</div>
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
                                            <b> 请确认您所选择的回报项信息和购买数量 </b>
                                        </blockquote>
                                    </div>
                                    <div class="col-md-12 column">
                                        <table class="table table-bordered"
                                            style="text-align: center;">
                                            <thead>
                                            <tr style="background-color: #ddd;">
                                                <td>项目名称</td>
                                                <td>发起人</td>
                                                <td width="300">回报内容</td>
                                                <td width="80">回报数量</td>
                                                <td>支持单价</td>
                                                <td>配送费用</td>
                                            </tr>
                                            </thead>
                                            <tbody>
                                            <tr>
                                                <td th:text="${session.orderProjectVO.projectName}">活性富氢净水直饮机</td>
                                                <td th:text="${session.orderProjectVO.launchName}">深圳市博实永道电子商务有限公司</td>
                                                <td th:text="${session.orderProjectVO.returnContent}">每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</td>
                                                <td><input id="returnCountInput" type="text" class="form-control"
                                                        style="width: 60px;" th:value="${session.orderProjectVO.returnCount}"></td>
                                                <td style="color: #F60" th:text="${session.orderProjectVO.supportPrice}">￥ 1.00</td>
                                                <td th:if="${session.orderProjectVO.freight == 0}">免运费</td>
                                                <td th:if="${session.orderProjectVO.freight != 0}" th:text="${session.orderProjectVO.freight}">免运费</td>
                                            </tr>
                                            </tbody>
                                        </table>
                                        <div style="float: right;">
                                            <p>
                                                总价(含运费)：<span id="totalAmount" style="font-size: 16px; color: #F60;">￥[[${session.orderProjectVO.returnCount*session.orderProjectVO.supportPrice}]]</span>
                                            </p>
                                            <button id="submitBtn" type="button" class="btn btn-warning btn-lg"
                                                    style="float: right;">
                                                <i class="glyphicon glyphicon-credit-card"></i> 去结算
                                            </button>
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
                                                    <br> <span style="font-size: 12px;">1.众筹并非商品交易，存在一定风险。支持者根据自己的判断选择、支持众筹项目，与发起人共同实现梦想并获得发起人承诺的回报。<br>
                                                            2.众筹平台仅提供平台网络空间及技术支持等中介服务，众筹仅存在于发起人和支持者之间，使用众筹平台产生的法律后果由发起人与支持者自行承担。<br>
                                                            3.本项目必须在2017-06-04之前达到 ￥1000000.00
                                                            的目标才算成功，否则已经支持的订单将取消。订单取消或募集失败的，您支持的金额将原支付路径退回。<br>
                                                            4.请在支持项目后15分钟内付款，否则您的支持请求会被自动关闭。<br>
                                                            5.众筹成功后由发起人统一进行发货，售后服务由发起人统一提供；如果发生发起人无法发放回报、延迟发放回报、不提供回报后续服务等情况，您需要直接和发起人协商解决。<br>
                                                            6.如您不同意上述风险提示内容，您有权选择不支持；一旦选择支持，视为您已确认并同意以上提示内容。
                                                        </span>
                                                </blockquote>
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
    <script src="jquery/jquery-2.1.1.min.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
    <script src="script/docs.min.js"></script>
    <script src="script/back-to-top.js"></script>
    <script>

        var signalPurchase = [[${session.orderProjectVO.signalPurchase}]];
        var purchase = [[${session.orderProjectVO.purchase}]];

        $('#myTab a').click(function(e) {
            e.preventDefault()
            $(this).tab('show')
        });

        $("#returnCountInput").change(function(){
            var returnCount = $.trim($(this).val());

            if(returnCount == null || returnCount == "") {
                alert("请输入有效数据！");

                $(this).val(this.defaultValue);

                return ;
            }

            if(signalPurchase == 1 && returnCount > purchase) {
                alert("不能超过限购数量！");
                return ;
            }

            var supportPrice = [[${session.orderProjectVO.supportPrice}]];

            $("#totalAmount").text("￥"+(supportPrice * returnCount));
        });

        $("#submitBtn").click(function(){
            var returnCount = $("#returnCountInput").val();
            window.location.href = "order/confirm/order/"+returnCount;
        });
    </script>
    </body>
    </html>
    ```

### 16.3.3 后端代码

1. entity 模块创建 OrderProjectVO

    ```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class OrderProjectVO implements Serializable {

        private static final long serialVersionUID = 1L;

        private Integer id;

        private String projectName;

        private String launchName;

        private String returnContent;

        private Integer returnCount;

        private Integer supportPrice;

        private Integer freight;

        private Integer orderId;

        private Integer signalPurchase;

        private Integer purchase;

    }
    ```

2. order 模块新建 OrderHandler 类

    ```java
    @Controller
    public class OrderHandler {

        @Autowired
        private MySQLRemoteService mySQLRemoteService;

        @RequestMapping("/confirm/return/info/{projectId}/{returnId}")
        public String showReturnConfirmInfo(@PathVariable("projectId") Integer projectId,
                                            @PathVariable("returnId") Integer returnId,
                                            HttpSession session) {

            ResultEntity<OrderProjectVO> resultEntity = mySQLRemoteService.getOrderProjectVORemote(projectId, returnId);

            if (ResultEntity.SUCCESS.equals(resultEntity.getResult())) {
                OrderProjectVO orderProjectVO = resultEntity.getData();

                session.setAttribute("orderProjectVO", orderProjectVO);
            }

            return "confirm-return";
        }

    }
    ```

3. api 模块 MySQLRemoteService

    ```java
    @RequestMapping("/get/order/project/vo/remote")
    ResultEntity<OrderProjectVO> getOrderProjectVORemote(@RequestParam("projectId") Integer projectId, @RequestParam("returnId") Integer returnId);
    ```

### 16.3.4 sql相关 - mysql模块

1. 新建 OrderProviderHandler

    ```java
    @RestController
    public class OrderProviderHandler {

        @Autowired
        private OrderService orderService;

        @RequestMapping("/get/order/project/vo/remote")
        ResultEntity<OrderProjectVO> getOrderProjectVORemote(@RequestParam("projectId") Integer projectId, @RequestParam("returnId") Integer returnId) {

            try {
                OrderProjectVO orderProjectVO = orderService.getOrderProjectVO(projectId, returnId);

                return ResultEntity.successWithData(orderProjectVO);
            } catch (Exception e) {
                e.printStackTrace();

                return ResultEntity.failed(e.getMessage());
            }

        }
    }
    ```

2. 新建 OrderService

    ```java
    @Service
    public class OrderServiceImpl implements OrderService {

        @Autowired
        private OrderPOMapper orderPOMapper;

        @Autowired
        private OrderProjectPOMapper orderProjectPOMapper;

        @Override
        public OrderProjectVO getOrderProjectVO(Integer projectId, Integer returnId) {

            return orderProjectPOMapper.selectOrderProjectVO(returnId);
        }
    }
    ```

3. OrderProjectPOMapper

    ```java
    OrderProjectVO selectOrderProjectVO(Integer returnId);
    ```

4. OrderProjectPOMapper.xml

    ```xml
    <select id="selectOrderProjectVO" resultType="com.atguigu.crowd.entity.vo.OrderProjectVO">
        select distinct
            project_name projectName,
            content returnContent,
            description_simple launchName,
            t_return.supportmoney supportPrice,
            freight,
            count returnCount,
            signalpurchase signalPurchase,
            purchase
        from t_project
            left join t_member_launch_info tmli on t_project.memberid = tmli.memberid
            left join t_return on t_project.id=t_return.projectid
        where t_return.id=#{returnId}
    </select>
    ```

## 16.4 确认订单

### 16.4.1 思路

1. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/16-4-1.png)

2. 确认页面

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/16-2-1_1.png)

3. 功能
   * 显示收货地址
   * 保存新的收货地址
   * 保存信息，跳转到付款页面

### 16.4.2 页面显示-后端代码

1. 创建 AddressVO
   
    ```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class AddressVO implements Serializable {

        private static final long serialVersionUID = 1L;

        private Integer id;
        
        private String receiveName;
        
        private String phoneNum;
        
        private String address;
        
        private Integer memberId;
        
    }
    ```

2. OrderHandler

    ```java
    /**
     * 跳转到确认订单页面
     * @param returnCount 回报数量
     * @param session  将orderProjectVO存进session域
     * @return 返回确认订单页面视图
     */
    @RequestMapping("/confirm/order/{returnCount}")
    public String showConfirmOrderInfo(@PathVariable Integer returnCount,
                                       HttpSession session) {

        // 1.把接收到的回报数量合并到 Session 域
        OrderProjectVO orderProjectVO = (OrderProjectVO) session.getAttribute("orderProjectVO");
        orderProjectVO.setReturnCount(returnCount);
        session.setAttribute("orderProjectVO", orderProjectVO);

        // 2.获取当前已登录用户的 id
        MemberLoginVO memberLoginVO = (MemberLoginVO) session.getAttribute(CrowdConstant.ATTR_NAME_LOGIN_MEMBER);
        Integer memberId = memberLoginVO.getId();

        // 3.查询目前的收货地址数据
        ResultEntity<List<AddressVO>>  resultEntity = mySQLRemoteService.getAddressVORemote(memberId);

        if (ResultEntity.SUCCESS.equals(resultEntity.getResult())) {
            List<AddressVO> list = resultEntity.getData();

            session.setAttribute("addressVOList", list);
        }

        return "confirm-order";
    }
    ```

3. api接口

    ```java
    @RequestMapping("/get/address/vo/remote")
    ResultEntity<List<AddressVO>> getAddressVORemote(@RequestParam("memberId") Integer memberId);
    ```

### 16.4.3 sql相关-地址查询

1. OrderProviderHandler

    ```java
    @RequestMapping("/get/address/vo/remote")
    ResultEntity<List<AddressVO>> getAddressVORemote(@RequestParam("memberId") Integer memberId) {

        try {
            List<AddressVO> addressVOList =  orderService.getAddressVOList(memberId);

            return ResultEntity.successWithData(addressVOList);
        } catch (Exception e) {
            e.printStackTrace();

            return ResultEntity.failed(e.getMessage());
        }
    }
    ```

2. OrderService

    ```java
    @Override
    public List<AddressVO> getAddressVOList(Integer memberId) {

        AddressPOExample example = new AddressPOExample();
        example.createCriteria().andMemberIdEqualTo(memberId);
        List<AddressPO> addressPOList = addressPOMapper.selectByExample(example);

        List<AddressVO> addressVOList = new ArrayList<>();
        for (AddressPO addressPO : addressPOList) {

            AddressVO addressVO = new AddressVO();
            BeanUtils.copyProperties(addressPO, addressVO);
            addressVOList.add(addressVO);
        }
        return addressVOList;
    }
    ```

### 16.4.4 前端页面

1. confirm-order.html

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
                                                    确认回报内容</div>
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
                                                    确认订单</div>
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
                                                <div style="font-size: 16px; margin-top: 15px;">3. 付款</div>
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
                                        <div class="alert alert-warning alert-dismissable"
                                            style="color: red;">
                                            <button type="button" class="close" data-dismiss="alert"
                                                    aria-hidden="true">×</button>
                                            <i class="glyphicon glyphicon-info-sign"></i> <strong>请在下单后15分钟内付款，否则您的订单会被自动关闭。</strong>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div id="address" class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-12 column">
                                        <blockquote
                                                style="border-left: 5px solid #f60; color: #f60; padding: 0 0 0 20px;">
                                            <b> 收货地址 </b>
                                        </blockquote>
                                    </div>
                                    <div class="col-md-12 column" style="padding: 0 120px;">
                                        <div th:if="${#strings.isEmpty(session.addressVOList)}">尚未创建收货地址</div>
                                        <div th:if="${not #strings.isEmpty(session.addressVOList)}">
                                            <div th:each="address : ${session.addressVOList}" class="radio">
                                                <label> <input type="radio" name="addressId" th:value="${address.id}"
                                                            id="optionsRadios1"/> [[${address.receiveName}]]
                                                    [[${address.phoneNum}]] [[${address.address}]]
                                                </label>
                                            </div>
                                        </div>
                                        <div class="radio">
                                            <label> <input type="radio" name="optionsRadios"
                                                        id="optionsRadios2" value="option2"> 新地址
                                            </label>
                                        </div>
                                        <div
                                                style="border: 10px solid #f60; border-bottom-color: #eceeef; border-width: 10px; width: 20px; margin-left: 20px; margin-top: -20px; border-left-color: rgba(221, 221, 221, 0); border-top-color: rgba(221, 221, 221, 0); border-right-color: rgba(221, 221, 221, 0);"></div>
                                        <div class="panel panel-default"
                                            style="border-style: dashed; background-color: #eceeef">
                                            <div class="panel-body">
                                                <div class="col-md-12 column">
                                                    <form action="order/save/address" method="post" class="form-horizontal">
                                                        <input type="hidden" name="memberId" th:value="${session.loginMember.id}" />
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">收货人（*）</label>
                                                            <div class="col-sm-10">
                                                                <input type="text" name="receiveName" class="form-control"
                                                                    style="width: 200px;" placeholder="姓名：xxxx">
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">手机（*）</label>
                                                            <div class="col-sm-10">
                                                                <input class="form-control" name="phoneNum" type="text"
                                                                    style="width: 200px;" placeholder="请填写11位手机号码"></input>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">地址（*）</label>
                                                            <div class="col-sm-10">
                                                                <input class="form-control" name="address" type="text"
                                                                    style="width: 400px;" placeholder="请填写11位手机号码"></input>
                                                            </div>
                                                        </div>
                                                        <div class="form-group">
                                                            <label for="inputEmail3" class="col-sm-2 control-label"></label>
                                                            <div class="col-sm-10">
                                                                <button type="submit" class="btn btn-primary">确认配送信息</button>
                                                            </div>
                                                        </div>
                                                    </form>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-12 column">
                                        <blockquote
                                                style="border-left: 5px solid #f60; color: #f60; padding: 0 0 0 20px;">
                                            <b> 发票信息 </b>
                                        </blockquote>
                                    </div>
                                    <div class="col-md-12 column" style="padding: 0 120px;">
                                        <div class="radio">
                                            <label> <input type="radio" name="optionsRadio"
                                                        id="optionsRadio1" value="option1" checked> 无需发票
                                            </label>
                                        </div>
                                        <div class="radio">
                                            <label> <input type="radio" name="optionsRadio"
                                                        id="optionsRadio2" value="option2"> 需要发票
                                            </label>
                                        </div>
                                        <div
                                                style="border: 10px solid #f60; border-bottom-color: #eceeef; border-width: 10px; width: 20px; margin-left: 20px; margin-top: -20px; border-left-color: rgba(221, 221, 221, 0); border-top-color: rgba(221, 221, 221, 0); border-right-color: rgba(221, 221, 221, 0);"></div>
                                        <div class="panel panel-default"
                                            style="border-style: dashed; background-color: #eceeef">
                                            <div class="panel-body">
                                                <div class="col-md-12 column">
                                                    <form class="form-horizontal">
                                                        <div class="form-group">
                                                            <label class="col-sm-2 control-label">发票抬头（*）</label>
                                                            <div class="col-sm-10">
                                                                <input type="text" class="form-control"
                                                                    style="width: 200px;" placeholder="个人">
                                                            </div>
                                                        </div>
                                                    </form>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="container-fluid">
                                <div class="row clearfix">
                                    <div class="col-md-12 column">
                                        <blockquote
                                                style="border-left: 5px solid #f60; color: #f60; padding: 0 0 0 20px;">
                                            <b> 项目信息 <a style="font-size: 12px;"
                                                        href="pay-step-1.html">修改数量</a>
                                            </b>
                                        </blockquote>
                                    </div>
                                    <div class="col-md-12 column">
                                        <table class="table table-bordered"
                                            style="text-align: center;">
                                            <thead>
                                            <tr style="background-color: #ddd;">
                                                <td>项目名称</td>
                                                <td>发起人</td>
                                                <td width="300">回报内容</td>
                                                <td width="80">回报数量</td>
                                                <td>支持单价</td>
                                                <td>配送费用</td>
                                            </tr>
                                            </thead>
                                            <tbody>
                                            <tr>
                                                <td th:text="${session.orderProjectVO.projectName}">活性富氢净水直饮机</td>
                                                <td th:text="${session.orderProjectVO.launchName}">深圳市博实永道电子商务有限公司</td>
                                                <td th:text="${session.orderProjectVO.returnContent}">每满1750人抽取一台活性富氢净水直饮机，至少抽取一台。抽取名额（小数点后一位四舍五入）=参与人数÷1750人，由苏宁官方抽取。</td>
                                                <td th:text="${session.orderProjectVO.returnCount}">55</td>
                                                <td style="color: #F60" th:text="${session.orderProjectVO.supportPrice}">￥ 1.00</td>
                                                <td th:if="${session.orderProjectVO.freight == 0}">免运费</td>
                                                <td th:if="${session.orderProjectVO.freight != 0}" th:text="${session.orderProjectVO.freight}">免运费</td>
                                            </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                    <div class="col-md-12 column">
                                        <div class="form-group">
                                            <label class="col-sm-2 control-label">备注</label>
                                            <div class="col-sm-10">
                                                    <textarea class="form-control" rows="1"
                                                            placeholder="填写关于回报或发起人希望您备注的信息"></textarea>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-md-12 column">
                                        <blockquote
                                                style="border-left: 5px solid #f60; color: #f60; padding: 0 0 0 20px;">
                                            <b> 账单 </b>
                                        </blockquote>
                                    </div>

                                    <div class="col-md-12 column">
                                        <div class="alert alert-warning alert-dismissable"
                                            style="text-align: right; border: 2px solid #ffc287;">
                                            <ul style="list-style: none;">
                                                <li style="margin-top: 10px;">支持金额：<span
                                                        style="color: red;">￥[[${session.orderProjectVO.returnCount*session.orderProjectVO.supportPrice}]]</span></li>
                                                <li style="margin-top: 10px;">配送费用：<span
                                                        style="color: red;">￥[[${session.orderProjectVO.freight}]]</span></li>
                                                <li style="margin-top: 10px; margin-bottom: 10px;"><h2>
                                                    支付总金额：<span style="color: red;">￥[[${session.orderProjectVO.returnCount*session.orderProjectVO.supportPrice+session.orderProjectVO.freight}]]</span>
                                                </h2></li>
                                                <li
                                                        style="margin-top: 10px; padding: 5px; border: 1px solid #F00; display: initial; background: #FFF;">
                                                    <i class="glyphicon glyphicon-info-sign"></i> <strong>您需要先
                                                    <a href="#address">设置配送信息</a> ,再提交订单
                                                </strong>
                                                </li>
                                                <li style="margin-top: 10px;">
                                                    请在下单后15分钟内付款，否则您的订单会被自动关闭。</li>
                                                <li style="margin-top: 10px;">
                                                    <button id="payButton" disabled="disabled" type="button"
                                                            class="btn btn-warning btn-lg"
                                                            onclick="window.location.href='pay-step-3.html'">
                                                        <i class="glyphicon glyphicon-credit-card"></i> 立即付款
                                                    </button>
                                                </li>
                                                <li style="margin-top: 10px;">
                                                    <div class="checkbox">
                                                        <label>
                                                            <input id="knowRoleCheckBox" type="checkbox">我已了解风险和规则
                                                        </label>
                                                    </div>
                                                </li>
                                            </ul>


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
                                                    <br> <span style="font-size: 12px;">1.众筹并非商品交易，存在一定风险。支持者根据自己的判断选择、支持众筹项目，与发起人共同实现梦想并获得发起人承诺的回报。<br>
                                                            2.众筹平台仅提供平台网络空间及技术支持等中介服务，众筹仅存在于发起人和支持者之间，使用众筹平台产生的法律后果由发起人与支持者自行承担。<br>
                                                            3.本项目必须在2017-06-04之前达到 ￥1000000.00
                                                            的目标才算成功，否则已经支持的订单将取消。订单取消或募集失败的，您支持的金额将原支付路径退回。<br>
                                                            4.请在支持项目后15分钟内付款，否则您的支持请求会被自动关闭。<br>
                                                            5.众筹成功后由发起人统一进行发货，售后服务由发起人统一提供；如果发生发起人无法发放回报、延迟发放回报、不提供回报后续服务等情况，您需要直接和发起人协商解决。<br>
                                                            6.如您不同意上述风险提示内容，您有权选择不支持；一旦选择支持，视为您已确认并同意以上提示内容。
                                                        </span>
                                                </blockquote>
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
    <script src="jquery/jquery-2.1.1.min.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
    <script src="script/docs.min.js"></script>
    <script src="script/back-to-top.js"></script>
    <script>
        $("#knowRoleCheckBox").click(function(){
            var currentStatus = this.checked;
            if(currentStatus) {
                $("#payButton").prop("disabled", "");
            }else{
                $("#payButton").prop("disabled","disabled");
            }
        });
        $('#myTab a').click(function(e) {
            e.preventDefault()
            $(this).tab('show')
        })
    </script>
    </body>
    </html>
    ```

### 16.4.5 新增收获地址

1. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/16-4-4.png)

2. OrderHandler

    ```java
    /**
     * 保存地址信息
     * @param addressVO  新增的地址信息
     * @param session    通过session获取地址信息
     * @return    通过session中的信息跳转回原页面
     */
    @RequestMapping("/save/address")
    public String saveAddress(AddressVO addressVO, HttpSession session) {

        // 1.保存地址信息
        ResultEntity<String> resultEntity = mySQLRemoteService.saveAddressRemote(addressVO);
        log.info(resultEntity.getResult());

        // 2.从Session域中获取orderProjectVO
        OrderProjectVO orderProjectVO = (OrderProjectVO) session.getAttribute("orderProjectVO");

        // 3.获取returnCount
        Integer returnCount = orderProjectVO.getReturnCount();

        // 4.重定向到指定地址，重新进入确定订单页面（数据库中地址信息已更新，页面中显示的是新的地址信息）
        return "redirect:http://crowd.com/order/confirm/order/"+returnCount;
    }
    ```

3. api

    ```java
    @RequestMapping("/save/address/vo/remote")
    ResultEntity<String> saveAddressRemote(@RequestBody AddressVO addressVO);
    ```

4. OrderProviderHandler

    ```java
    @RequestMapping("/save/address/vo/remote")
    ResultEntity<String> saveAddressRemote(@RequestBody AddressVO addressVO) {

        try {
            orderService.saveAddressVO(addressVO);
            
            return ResultEntity.successWithoutData();
        } catch (Exception e) {
            e.printStackTrace();
            
            return ResultEntity.failed(e.getMessage());
        }
    }
    ```

5. OrderService

    ```java
    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
    @Override
    public void saveAddressVO(AddressVO addressVO) {

        AddressPO addressPO = new AddressPO();

        BeanUtils.copyProperties(addressVO, addressPO);

        addressPOMapper.insert(addressPO);

    }
    ```




