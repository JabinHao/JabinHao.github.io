---
title: Chapter5 延迟队列
excerpt: RabbitMQ延迟队列的实现，基于插件的延迟队列，整合SpringBoot
tags:
  - RabbitMQ
categories:
  - RabbitMQ
banner_img: /img/banner/doggie.png
index_img: /img/index/RabbitMQ.png
category: RabbitMQ
abbrlink: d09d57cc
date: 2021-06-21 15:40:35
updated: 2021-06-21 23:32:35
subtitle:
---
## 5.1 简介

### 5.1.1 概念

1. 延时队列,队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望在指定时间到了以后或之前取出和处理
2. 简单来说，延时队列就是用来存放需要在指定时间被处理的元素的队列。
3. rabbitmq本身是不直接支持延时队列的，RabbitMQ的延迟队列基于消息的存活时间TTL（Time To Live）和死信交换机DLE（Dead Letter Exchanges）实现


### 5.1.2 应用场景

1. 订单在十分钟之内未支付则自动取消
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
3. 用户注册成功后，如果三天内没有登陆则进行短信提醒。
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

## 5.2 实现

### 5.2.1 构造
1. rabbitmq通过TTL + 死信交换机实现延时队列
2. TTL：RabbitMQ可以对队列和消息各自设置存活时间，规则是两者中较小的值，即队列无消费者连接的消息过期时间，或者消息在队列中一直未被消费的过期时间
3. DLE：过期的消息通过绑定的死信交换机，路由到指定的死信队列，消费者实际上消费的是死信队列上的消息

### 5.2.2 TTL

1. 什么是TTL
   * TTL 是 RabbitMQ 中一个消息或者队列的属性，表明一条消息或者该队列中的所有消息的最大存活时间，单位是毫秒
   * 如果同时配置了队列的 TTL 和消息的 TTL，那么较小的那个值将会被使用
2. 消息设置 TTL

    ```java
    // 设置过期时间为 6s
    AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
    builder.expiration("6000");
    AMQP.BasicProperties properties = builder.build();
    channel.basicPublish(exchangeName, routingKey, mandatory, properties, "msg body".getBytes());
    ```

    * 如果不设置 TTL，表示消息永远不会过期
    * 如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃

3. 队列设置 TTL

    ```java
    Map<String, Object> args = new HashMap<String, Object>();
    args.put("x-message-ttl", 6000);
    channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);
    ```

4. 区别
    * 如果设置了队列的 TTL 属性，那么一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)
    * 对于设置消息 TTL，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间；

### 5.3 实战

### 5.3.1 SpringBoot整合

1. 创建项目
2. 依赖

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>3.0.0</version>
        </dependency>
    </dependencies>
    ```

3. 配置文件 application.properties

    ```properties
    spring.rabbitmq.host=139.224.113.9
    spring.rabbitmq.port=5672
    spring.rabbitmq.username=admin
    spring.rabbitmq.password=hjp356908
    ```

4. 配置类 config/SwaggerConfig.java

    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import springfox.documentation.builders.ApiInfoBuilder;
    import springfox.documentation.service.ApiInfo;
    import springfox.documentation.service.Contact;
    import springfox.documentation.spi.DocumentationType;
    import springfox.documentation.spring.web.plugins.Docket;
    import springfox.documentation.swagger2.annotations.EnableSwagger2;

    @Configuration
    @EnableSwagger2
    public class SwaggerConfig {

        @Bean
        public Docket webApiConfig(){
            return new Docket(DocumentationType.SWAGGER_2)
                    .groupName("webApi")
                    .apiInfo(webApiInfo())
                    .select()
                    .build();
        }
        private ApiInfo webApiInfo(){
            return new ApiInfoBuilder()
                    .title("rabbitmq 接口文档")
                    .description("本文档描述了 rabbitmq 微服务接口定义")
                    .version("1.0")
                    .contact(new Contact("enjoy6288", "http://atguigu.com",
                            "1551388580@qq.com"))
                    .build();
        }
    }
    ```

### 5.3.2 队列 TTL

1. 代码架构

   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/5-1.png)

   * 两个队列 QA 和 QB，两者队列 TTL 分别设置为 10S 和 40S
   * 一个交换机 X 和死信交换机 Y，它们的类型都是 direct，一个死信队列 QD
   * 通过不同的 routing key选择不同的队列，实现不同的延迟时间

2. 配置文件类（声明五个组件）

    ```java
    @Configuration
    public class TTLQueueConfig {

        public static final String X_EXCHANGE = "X";
        public static final String QUEUE_A = "QA";
        public static final String QUEUE_B = "QB";

        public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
        public static final String DEAD_LETTER_QUEUE = "QD";

        // 声明 X Exchange
        @Bean("xExchange")
        public DirectExchange xExchange(){
            return new DirectExchange(X_EXCHANGE);
        }

        // 声明 Y Exchange
        @Bean("yExchange")
        public DirectExchange yExchange(){
            return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
        }

        // 声明队列A（10s）
        @Bean("queueA")
        public Queue queueA(){
            Map<String, Object> args = new HashMap<>();
            args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
            args.put("x-dead-letter-routing-key", "YD");
            args.put("x-message-ttl", 10000);
            return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
        }

        // 声明队列B（40s）
        @Bean("queueB")
        public Queue queueB(){
            Map<String, Object> args = new HashMap<>();
            args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
            args.put("x-dead-letter-routing-key", "YD");
            args.put("x-message-ttl", 40000);
            return QueueBuilder.durable(QUEUE_B).withArguments(args).build();
        }

        // 声明死信队列 QD
        @Bean("queueD")
        public Queue queueD(){
            return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
        }

        // 绑定交换机
        @Bean
        public Binding queueABindingX(@Qualifier("queueA") Queue queueA,
                                    @Qualifier("xExchange") DirectExchange xExchange){
            return BindingBuilder.bind(queueA).to(xExchange).with("XA");
        }
        @Bean
        public Binding queueBBindingX(@Qualifier("queueB") Queue queueB,
                                    @Qualifier("xExchange") DirectExchange xExchange){
            return BindingBuilder.bind(queueB).to(xExchange).with("XB");
        }
        @Bean
        public Binding queueDBindingY(@Qualifier("queueD") Queue queueD,
                                    @Qualifier("yExchange") DirectExchange yExchange){
            return BindingBuilder.bind(queueD).to(yExchange).with("YD");
        }
    }
    ```

3. 生产者

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.controller;

    import lombok.extern.slf4j.Slf4j;
    import org.springframework.amqp.rabbit.core.RabbitTemplate;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    import java.util.Date;

    /**
    * 发送延迟消息
    */
    @Slf4j
    @RestController
    @RequestMapping("ttl")
    public class SendMsgController {

        @Autowired
        private RabbitTemplate rabbitTemplate;

        @GetMapping("sendMsg/{message}")
        public void sendMsg(@PathVariable String message){
            log.info("当前时间：{}，发送一条信息给两个TTL队列：{}", new Date().toString(), message);

            rabbitTemplate.convertAndSend("X", "XA", "消息来自ttl为10s的队列" + message);
            rabbitTemplate.convertAndSend("X", "XB", "消息来自ttl为40s的队列" + message);
        }
    }
    ```

4. 消费者

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.consumer;

    import com.rabbitmq.client.Channel;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.amqp.core.Message;
    import org.springframework.amqp.rabbit.annotation.RabbitListener;
    import org.springframework.stereotype.Component;

    import java.nio.charset.StandardCharsets;
    import java.util.Date;

    /**
    * 消费者
    */
    @Slf4j
    @Component
    public class DeadLetterQueueConsumer {

        // 接收消息
        @RabbitListener(queues = "QD")
        public void receiveD(Message message, Channel channel) throws Exception{
            String msg = new String(message.getBody(), StandardCharsets.UTF_8);
            log.info("当前时间：{}，收到死信队列的消息：{}", new Date().toString(), msg);
        }
    }
    ```

### 5.3.3 消息 TTL

1. 说明：
   * 对于队列TTL，每增加一个新的时间需求，就要新增一个队列
   * 改用消息TTL则只需一个队列

2. 代码架构
   
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/5-2.png)

3. 配置文件类

    ```java
    public static final String QUEUE_C = "QC";

    // 声明队列C
    @Bean("queueC")
    public Queue queueC(){
        Map<String, Object> args = new HashMap<>();
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        args.put("x-dead-letter-routing-key", "YD");
        return QueueBuilder.durable(QUEUE_C).withArguments(args).build();
    }

    @Bean
    public Binding queueCBindX(@Qualifier("queueC") Queue queueC,
                               @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueC).to(xExchange).with("XC");
    }
    ```

4. 生产者

    ```java
    @GetMapping("sendExpirationMsg/{message}/{ttlTime}")
    public void sendExpirationMsg(@PathVariable String message,
                        @PathVariable String ttlTime){

        rabbitTemplate.convertAndSend("X", "XC", message, correlationData -> {
            correlationData.getMessageProperties().setExpiration(ttlTime);
            return correlationData;
        });
        log.info("当前时间：{}，发送一条时长{}毫秒的TTL信息给队列C：{}", new Date().toString(), ttlTime, message);
    }
    ```

5. 访问：
   * 20000ms: [http://localhost:8080/ttl/sendExpirationMsg/你好1/20000](http://localhost:8080/ttl/sendExpirationMsg/你好1/20000)
   * 2000ms: [http://localhost:8080/ttl/sendExpirationMsg/你好2/2000](http://localhost:8080/ttl/sendExpirationMsg/你好1/20000)

6. 问题
   * 消息可能不会按时过期
   * RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列，如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行

### 5.3.4 Rabbitmq 插件实现延迟队列

1. 安装
    * 官网下载插件 rabbitmq_delayed_message_exchange
        ```sh
        wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.8.0/rabbitmq_delayed_message_exchange-3.8.0.ez
        ```

    * 移动到指定目录下

        ```sh
        mv rabbitmq_delayed_message_exchange-3.8.0.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.17/plugins/
        ```

    * 开启

        ```sh
        rabbitmq-plugins enable rabbitmq_delayed_message_exchange
        ```

    * 重启 RabbitMQ

        ```sh
        /sbin/service rabbitmq-server restart
        ```
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/5-4.png)

2. 代码架构

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/5-5.png)

3. 配置文件类

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.config;

    import org.springframework.amqp.core.*;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    import java.util.HashMap;
    import java.util.Map;

    @Configuration
    public class DelayQueueConfig {
        public static final String DELAYED_QUEUE_NAME = "delayed.queue";
        public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
        public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

        @Bean
        public Queue delayedQueue(){
            return new Queue(DELAYED_QUEUE_NAME);
        }

        // 自定义交换机
        @Bean
        public CustomExchange delayedExchange(){
            Map<String, Object> args = new HashMap<>();
            args.put("x-delayed-type", "direct");
            return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, args);
        }

        @Bean
        public Binding bindingDelayedQueue(@Qualifier("delayedQueue") Queue delayedQueue,
                                        @Qualifier("delayedExchange") CustomExchange customExchange){
            return BindingBuilder.bind(delayedQueue).to(customExchange).with(DELAYED_ROUTING_KEY).noargs();
        }

    }
    ```

4. 生产者

    ```java
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";
    @GetMapping("sendDelayMsg/{message}/{delayTime}")
    public void sendMsg(@PathVariable String message,
                        @PathVariable Integer delayTime){
        rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME, DELAYED_ROUTING_KEY, message, correlationData -> {
            correlationData.getMessageProperties().setDelay(delayTime);
            return correlationData;
        });

        log.info("当前时间：{}，发送一条延迟{}毫秒的信息给队列 delayed.queue：{}", new Date().toString(), delayTime, message);
    }
    ```

5. 消费者

    ```java
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    @RabbitListener(queues = DELAYED_QUEUE_NAME)
    public void receiveDelayedQueue(Message message){
        String msg = new String(message.getBody(), StandardCharsets.UTF_8);
        log.info("当前时间：{}，收到延迟队列的消息：{}", new Date().toString(), msg);
    }
    ```
