---
title: Chapter17 — 前台 支付
excerpt: 实现订单支付功能，将订单信息保存到数据库
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
abbrlink: 4b38d148
date: 2021-01-29 01:02:06
updated: 2021-01-30 01:14:38
subtitle:
---
## 17.1 准备工作

### 17.1.1 建模

1. 订单模型回顾

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/17-1-1.png)

2. 创建OrderVO类

    ```java
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class OrderVO implements Serializable {

        private static final long serialVersionUID = 1L;
        // 主键
        private Integer id;

        // 订单号
        private String orderNum;

        // 支付宝流水单号
        private String payOrderNum;

        // 订单金额
        private Double orderAmount;

        // 是否开发票
        private Integer invoice;

        // 发票抬头
        private String invoiceTitle;

        // 备注
        private String orderRemark;

        private Integer addressId;

        private OrderProjectVO orderProjectVO;
    }
    ```

### 17.1.2 环境搭建

1. 依赖

    ```xml
    <dependencies>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
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
    </dependencies>
    ```

2. 配置类

    ```java
    package com.atguigu.crowd.config;

    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.stereotype.Component;

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    @Component
    @ConfigurationProperties(prefix = "ali.pay")
    public class PayProperties {

        private String appId;
        private String merchantPrivateKey;
        private String alipayPublicKey;
        private String notifyUrl;
        private String returnUrl;
        private String signType;
        private String charset;
        private String gatewayUrl;
        
    }
    ```
3. 配置文件

    ```yml
    server:
    port: 7000
    spring:
    application:
        name: atguigu-crowd-pay
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

    ali:
    pay:
        alipay-public-key: 
        app-id: 2021000117606212
        charset: utf-8
        gateway-url: https://openapi.alipaydev.com/gateway.do
        merchant-private-key: 
        notify-url: http://grekcv.natappfree.cc/pay/notify
        return-url: http://www.crowd.com/pay/return
        sign-type: RSA2
    ```

4. zuul配置

    ```yml
    crowd-pay:
      service-id: atguigu-crowd-pay
      path: /pay/**
    ```

## 17.2 支付功能

### 17.2.1 思路

1. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/17-2-1.png)

### 17.2.2 前端代码

1. 修改 confirm-order.html

    ```html
    <!-- /container -->

    <!-- 为了收集当前页面中的所有数据，构造空表单 -->
    <form id="summaryForm" action="pay/generate/order" method="post"></form>

    <script src="jquery/jquery-2.1.1.min.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
    <script src="script/docs.min.js"></script>
    <script src="script/back-to-top.js"></script>
    <script>

        $("#payButton").click(function(){

            // 1.收集所有要提交的表单项的数据
            var addressId = $("[name=addressId]:checked").val();
            var invoice = $("[name=invoiceRadio]:checked").val();
            var invoiceTitle = $.trim($("[name=invoiceTitle]").val());
            var remark = $.trim($("[name=remark]").val());

            // 2.将上面收集到的表单数据填充到空表单中并提交
            $("#summaryForm")
                .append("<input type='hidden' name='addressId' value='"+addressId+"'/>")
                .append("<input type='hidden' name='invoice' value='"+invoice+"'/>")
                .append("<input type='hidden' name='invoiceTitle' value='"+invoiceTitle+"'/>")
                .append("<input type='hidden' name='orderRemark' value='"+remark+"'/>")
                .submit();

        });

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
    ```

### 17.2.3 提交订单表单 - PayHandler

1. 处理支付链接

    ```java
    @Autowired
    private PayProperties payProperties;

    @ResponseBody
    @RequestMapping("/generate/order")
    public String generateOrder(HttpSession session, OrderVO orderVO) throws AlipayApiException {

        // 1.从 Session 域获取 orderProjectVO 对象
        OrderProjectVO orderProjectVO = (OrderProjectVO) session.getAttribute("orderProjectVO");

        // 2.将 orderProjectVO 对象和 orderVO 对象组装到一起
        orderVO.setOrderProjectVO(orderProjectVO);

        // 3.生成订单号并设置到 orderVO 对象中
        String time = new SimpleDateFormat("yyyyMMddHHmmss").format(new Date()); // 根据当前日期时间生成字符串
        String user = UUID.randomUUID().toString().replace("-", "").toUpperCase(); // 使用 UUID 生成用户 ID 部分
        String orderNum = time + user;
        orderVO.setOrderNum(orderNum);

        // 4.计算订单总金额并设置到 orderVO 对象中
        Double orderAmount = (double) (orderProjectVO.getSupportPrice() * orderProjectVO.getReturnCount() + orderProjectVO.getFreight());
        orderVO.setOrderAmount(orderAmount);

        // 将OrderVO对象存入Session域
        session.setAttribute("orderVO", orderVO);

        // 5.调用专门封装好的方法给支付宝接口发送请求
        return sendRequestToAliPay(orderNum, orderAmount, orderProjectVO.getProjectName(), orderProjectVO.getReturnContent());
    }
    ```

2. 定义调用支付宝接口的方法

    ```java
    /**
     * 为了调用支付宝接口专门封装的方法
     * @param outTradeNo	外部订单号，也就是商户订单号，也就是我们生成的订单号
     * @param totalAmount	订单的总金额
     * @param subject		订单的标题，这里可以使用项目名称
     * @param body			商品的描述，这里可以使用回报描述
     * @return				返回到页面上显示的支付宝登录界面
     * @throws AlipayApiException
     */
    private String sendRequestToAliPay(String outTradeNo, Double totalAmount, String subject, String body) throws AlipayApiException {
        //获得初始化的AlipayClient
        AlipayClient alipayClient = new DefaultAlipayClient(
                payProperties.getGatewayUrl(),
                payProperties.getAppId(),
                payProperties.getMerchantPrivateKey(),
                "json",
                payProperties.getCharset(),
                payProperties.getAlipayPublicKey(),
                payProperties.getSignType());

        //设置请求参数
        AlipayTradePagePayRequest alipayRequest = new AlipayTradePagePayRequest();
        alipayRequest.setReturnUrl(payProperties.getReturnUrl());
        alipayRequest.setNotifyUrl(payProperties.getNotifyUrl());

        alipayRequest.setBizContent("{\"out_trade_no\":\""+ outTradeNo +"\","
                + "\"total_amount\":\""+ totalAmount +"\","
                + "\"subject\":\""+ subject +"\","
                + "\"body\":\""+ body +"\","
                + "\"product_code\":\"FAST_INSTANT_TRADE_PAY\"}");

        //若想给BizContent增加其他可选请求参数，以增加自定义超时时间参数timeout_express来举例说明
        //alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\","
        //		+ "\"total_amount\":\""+ total_amount +"\","
        //		+ "\"subject\":\""+ subject +"\","
        //		+ "\"body\":\""+ body +"\","
        //		+ "\"timeout_express\":\"10m\","
        //		+ "\"product_code\":\"FAST_INSTANT_TRADE_PAY\"}");
        //请求参数可查阅【电脑网站支付的API文档-alipay.trade.page.pay-请求参数】章节

        //请求
        return alipayClient.pageExecute(alipayRequest).getBody();

    }
    ```

3. returnUrl 方法

    ```java
    @ResponseBody
    @RequestMapping("/return")
    public String returnUrlMethod(HttpServletRequest request, HttpSession session) throws AlipayApiException,
            UnsupportedEncodingException {

        // 获取支付宝 GET 过来反馈信息
        Map<String,String> params = new HashMap<String,String>();
        Map<String,String[]> requestParams = request.getParameterMap();
        for (Iterator<String> iter = requestParams.keySet().iterator(); iter.hasNext();) {
            String name = (String) iter.next();
            String[] values = (String[]) requestParams.get(name);
            String valueStr = "";
            for (int i = 0; i < values.length; i++) {
                valueStr = (i == values.length - 1) ? valueStr + values[i]
                        : valueStr + values[i] + ",";
            }
            // 乱码解决， 这段代码在出现乱码时使用
            valueStr = new String(valueStr.getBytes("ISO-8859-1"), "utf-8");
            params.put(name, valueStr);
        }
        boolean signVerified = AlipaySignature.rsaCheckV1(
                params,
                payProperties.getAlipayPublicKey(),
                payProperties.getCharset(),
                payProperties.getSignType()); //调用 SDK 验证签名

        // ——请在这里编写您的程序（以下代码仅作参考）——
        if(signVerified) {
            // 商户订单号
            String orderNum = new String(request.getParameter("out_trade_no").getBytes("ISO-8859-1"),"UTF-8");

            // 支付宝交易号
            String payOrderNum = new String(request.getParameter("trade_no").getBytes("ISO-8859-1"),"UTF-8");

            // 付款金额
            String orderAmount = new String(request.getParameter("total_amount").getBytes("ISO-8859-1"),"UTF-8");

            // 保存到数据库

            return "trade_no:"+payOrderNum+"<br/>out_trade_no:"+orderNum+"<br/>total_amount:"+orderAmount;
        }else {

            // 页面显示信息：验签失败
            return "验签失败";
        }
    }
    ```

4. notify方法

    ```java
    @RequestMapping("/notify")
    public void notifyUrlMethod(HttpServletRequest request) throws UnsupportedEncodingException, AlipayApiException {
        //获取支付宝POST过来反馈信息
        Map<String,String> params = new HashMap<String,String>();
        Map<String,String[]> requestParams = request.getParameterMap();
        for (Iterator<String> iter = requestParams.keySet().iterator(); iter.hasNext();) {
            String name = (String) iter.next();
            String[] values = (String[]) requestParams.get(name);
            String valueStr = "";
            for (int i = 0; i < values.length; i++) {
                valueStr = (i == values.length - 1) ? valueStr + values[i]
                        : valueStr + values[i] + ",";
            }
            //乱码解决，这段代码在出现乱码时使用
            valueStr = new String(valueStr.getBytes("ISO-8859-1"), "utf-8");
            params.put(name, valueStr);
        }

        boolean signVerified = AlipaySignature.rsaCheckV1(
                params,
                payProperties.getAlipayPublicKey(),
                payProperties.getCharset(),
                payProperties.getSignType()); //调用SDK验证签名

        //——请在这里编写您的程序（以下代码仅作参考）——

	/* 实际验证过程建议商户务必添加以下校验：
	1、需要验证该通知数据中的out_trade_no是否为商户系统中创建的订单号，
	2、判断total_amount是否确实为该订单的实际金额（即商户订单创建时的金额），
	3、校验通知中的seller_id（或者seller_email） 是否为out_trade_no这笔单据的对应的操作方（有的时候，一个商户可能有多个seller_id/seller_email）
	4、验证app_id是否为该商户本身。
	*/
        if(signVerified) {//验证成功
            //商户订单号
            String out_trade_no = new String(request.getParameter("out_trade_no").getBytes("ISO-8859-1"),"UTF-8");

            //支付宝交易号
            String trade_no = new String(request.getParameter("trade_no").getBytes("ISO-8859-1"),"UTF-8");

            //交易状态
            String trade_status = new String(request.getParameter("trade_status").getBytes("ISO-8859-1"),"UTF-8");

            log.info("out_trade_no="+out_trade_no);
            log.info("trade_no="+trade_no);
            log.info("trade_status="+trade_status);

        }else {//验证失败
            log.info("验证失败");

            //调试用，写文本函数记录程序运行情况是否正常
            //String sWord = AlipaySignature.getSignCheckContentV1(params);
            //AlipayConfig.logResult(sWord);
        }

        //——请在这里编写您的程序（以上代码仅作参考）——
    }
    ```

## 17.3 订单信息保存到数据库

### 17.3.1 思路

1. 思路

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/17-3-1.png)

### 17.3.2 后端代码

1. 完善 return 方法

    ```java
    // 保存到数据库
    // 1.从Session域中获取OrderVO对象
    OrderVO orderVO = (OrderVO) session.getAttribute("orderVO");

    // 2.将支付宝交易号设置到OrderVO对象中
    orderVO.setPayOrderNum(payOrderNum);

    // 3.发送给MySQL的远程接口
    ResultEntity<String> resultEntity = mySQLRemoteService.saveOrderRemote(orderVO);
    log.info("Order save result="+resultEntity.getResult());
    ```

2. api 接口

    ```java
    @RequestMapping("/save/order/remote")
    ResultEntity<String> saveOrderRemote(@RequestBody OrderVO orderVO);
    ```

### 17.3.3 SQL部分

1. handler

    ```java
    @RequestMapping("/save/order/remote")
    ResultEntity<String> saveOrderRemote(@RequestBody OrderVO orderVO) {

        try {
            orderService.saveOrderVO(orderVO);

            return ResultEntity.successWithoutData();
        } catch (Exception e) {
            e.printStackTrace();

            return ResultEntity.failed(e.getMessage());
        }
    }
    ```

2. service

    ```java
    @Override
    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
    public void saveOrderVO(OrderVO orderVO) {

        OrderPO orderPO = new OrderPO();

        BeanUtils.copyProperties(orderVO, orderPO);

        OrderProjectPO orderProjectPO = new OrderProjectPO();

        BeanUtils.copyProperties(orderVO.getOrderProjectVO(), orderProjectPO);

        orderPOMapper.insert(orderPO);

        // 保存OrderProjectPO需要orderPO的id作为外键
        Integer id = orderPO.getId();
        // 这里需要先修改mapper，
        orderProjectPO.setOrderId(id);

        orderProjectPOMapper.insert(orderProjectPO);
    }
    ```

3. 修改 OrderPOMapper.xml

    ```xml
    <insert id="insert" parameterType="com.atguigu.crowd.entity.po.OrderPO" useGeneratedKeys="true" keyProperty="id">
        insert into t_order (id, order_num, pay_order_num, 
        order_amount, invoice, invoice_title, 
        order_remark, address_id)
        values (#{id,jdbcType=INTEGER}, #{orderNum,jdbcType=CHAR}, #{payOrderNum,jdbcType=CHAR}, 
        #{orderAmount,jdbcType=DOUBLE}, #{invoice,jdbcType=INTEGER}, #{invoiceTitle,jdbcType=CHAR}, 
        #{orderRemark,jdbcType=CHAR}, #{addressId,jdbcType=CHAR})
    </insert>
    ```




