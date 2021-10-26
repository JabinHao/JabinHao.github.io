---
title: Chapter6 发布确认高级
excerpt: RabbitMQ工作模式之发布确认，如何实现确认，回退消息，备份交换机
tags:
  - RabbitMQ
categories:
  - RabbitMQ
banner_img: /img/banner/doggie.png
index_img: /img/index/RabbitMQ.png
category: RabbitMQ
abbrlink: f914028b
date: 2021-06-21 23:33:09
updated: 2021-06-22 19:59:00
subtitle:
---
## 6.1 发布确认 SpringBoot 版本

### 6.1.1 说明

1. 问题  
   RabbitMQ宕机时，生产者发送的消息会丢失，如何保证消息的可靠投递？
2. 确认机制

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/6-1.png)

3. 代码架构

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/6-2.png)

### 6.1.2 环境搭建

1. 配置文件

    ```properties
    spring.rabbitmq.publisher-confirm-type=correlated
    ```
    * none: 禁用发布确认模式，默认值
    * simple
    * correlated: 发布消息成功到交换器后会触发回调方法
2. 配置类

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.config;

    import org.springframework.amqp.core.*;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class ConfirmConfig {

        public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
        public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

        @Bean
        public DirectExchange confirmExchange(){
            return new DirectExchange(CONFIRM_EXCHANGE_NAME);
        }

        @Bean
        public Queue confirmQueue(){
            return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
        }

        @Bean
        public Binding queueBinding(@Qualifier("confirmQueue") Queue confirmQueue,
                                    @Qualifier("confirmExchange") DirectExchange confirmExchange){

            return BindingBuilder.bind(confirmQueue).to(confirmExchange).with("key1");
        }
    }
    ```

3. 生产者

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.controller;

    import com.atguigu.rabbitmq.springbootrabbitmq.config.ConfirmConfig;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.amqp.rabbit.connection.CorrelationData;
    import org.springframework.amqp.rabbit.core.RabbitTemplate;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    @Slf4j
    @RestController
    @RequestMapping("confirm")
    public class ProducerController {

        private final RabbitTemplate rabbitTemplate;

        @Autowired
        public ProducerController(RabbitTemplate rabbitTemplate) {
            this.rabbitTemplate = rabbitTemplate;
        }

        // 发消息
        @GetMapping("sendMessage/{message}")
        public void sendMsg(@PathVariable("message") String message){

            CorrelationData correlationData = new CorrelationData("1");
            rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME, ConfirmConfig.CONFIRM_ROUTING_KEY,message, correlationData);
            log.info("发送消息内容：{}",message);

            // 路由错误，交换机能收到但队列收不到
            CorrelationData correlationData2 = new CorrelationData("2");
            rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME, "key2",message, correlationData2);
            log.info("发送消息内容：{}",message);
        }
    }
    ```
    
4. 回调接口

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.config;

    import lombok.extern.slf4j.Slf4j;
    import org.springframework.amqp.rabbit.connection.CorrelationData;
    import org.springframework.amqp.rabbit.core.RabbitTemplate;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;

    import javax.annotation.PostConstruct;

    @Slf4j
    @Component
    public class MyCallBack implements RabbitTemplate.ConfirmCallback {

        private final RabbitTemplate rabbitTemplate;

        @Autowired
        public MyCallBack(RabbitTemplate rabbitTemplate) {
            this.rabbitTemplate = rabbitTemplate;
        }

        // 注入
        @PostConstruct
        public void init(){
            rabbitTemplate.setConfirmCallback(this);
        }

        /**
        * 交换机确认回调方法
        * @param correlationData 保存回调消息的ID及相关信息
        * @param ack 交换机是否成功接收消息
        * @param cause 失败原因（成功为null）
        */
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {

            String id = correlationData != null ? correlationData.getId() : "";
            if (ack){
                log.info("交换机已经收到ID为：{}的消息",id);
            }else {
                log.info("交换机还未收到ID为：{}的消息，原因为：{}", id, cause);
            }
        }
    }
    ```
5. 消费者

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.consumer;

    import com.atguigu.rabbitmq.springbootrabbitmq.config.ConfirmConfig;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.amqp.core.Message;
    import org.springframework.amqp.rabbit.annotation.RabbitListener;
    import org.springframework.stereotype.Component;

    @Slf4j
    @Component
    public class Consumer {

        @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE_NAME)
        public void receiveConfirmMessage(Message message){
            String msg = new String(message.getBody());
            log.info("接收到队列confirm.queue的消息：{}", msg);
        }
    }
    ```

6. 收不到消息两种情况
   * 交换机错误（如交换机名称错误）
   * 队列错误（如路由错误）

## 6.2 回退消息

### 6.2.1 说明

1. 问题
   * 生产者确认机制下，交换机收不到消息可以通过回调方法通知到生产者
   * 交换机接收到消息，如果发现该消息不可路由，那么消息会被直接丢弃，生产者不会被通知到
2. Mandatory 参数  
   在当消息传递过程中不可达目的地时将消息返回给生产者

### 6.2.2 代码

1. 配置文件

    ```properties
    spring.rabbitmq.publisher-returns=true
    ```

2. 生产者

    ```java
    @Slf4j
    @RestController
    @RequestMapping("confirm")
    public class ProducerController {

        private final RabbitTemplate rabbitTemplate;
        private final MyCallBack myCallBack;

        @Autowired
        public ProducerController(RabbitTemplate rabbitTemplate, MyCallBack myCallBack) {
            this.rabbitTemplate = rabbitTemplate;
            this.myCallBack = myCallBack;
        }

        // 注入
        @PostConstruct
        public void init(){
            rabbitTemplate.setConfirmCallback(myCallBack);
            rabbitTemplate.setReturnsCallback(myCallBack);
        }

        // 发消息
        @GetMapping("sendMessage/{message}")
        public void sendMsg(@PathVariable("message") String message){

            CorrelationData correlationData = new CorrelationData("1");
            rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME, ConfirmConfig.CONFIRM_ROUTING_KEY,message, correlationData);
            log.info("发送消息内容：{}",message);

            CorrelationData correlationData2 = new CorrelationData("2");
            rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME, "key2",message, correlationData2);
            log.info("发送消息内容：{}",message);
        }
    }
    ```

3. 回调接口

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.config;

    import lombok.extern.slf4j.Slf4j;
    import org.springframework.amqp.core.ReturnedMessage;
    import org.springframework.amqp.rabbit.connection.CorrelationData;
    import org.springframework.amqp.rabbit.core.RabbitTemplate;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;

    import javax.annotation.PostConstruct;
    import java.nio.charset.StandardCharsets;

    @Slf4j
    @Component
    public class MyCallBack implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {

        /**
        * 交换机确认回调方法
        * @param correlationData 保存回调消息的ID及相关信息
        * @param ack 交换机是否成功接收消息
        * @param cause 失败原因（成功为null）
        */
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {

            String id = correlationData != null ? correlationData.getId() : "";
            if (ack){
                log.info("交换机已经收到ID为：{}的消息",id);
            }else {
                log.info("交换机还未收到ID为：{}的消息，原因为：{}", id, cause);
            }
        }

        /**
        * 无法路由时的回调方法
        * @param returnedMessage 回调信息
        */
        @Override
        public void returnedMessage(ReturnedMessage returnedMessage) {
            log.error("消息：{}被交换机：{}退回，路由为：{}", new String(returnedMessage.getMessage().getBody(), StandardCharsets.UTF_8), returnedMessage.getExchange(), returnedMessage.getRoutingKey());
        }
    }
    ```
    
4. 消费者  
    不变 


## 6.3 备份交换机

### 6.3.1 说明

1. 通过mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力
2. 但这些消息需要手动处理，麻烦而且容易出错，也增加了生产者的复杂性

3. 以上问题可以通过备份交换机来解决
   * 当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理
   * 备份交换机的类型为 fanout
   * 在备份交换机下绑定一个队列，用于处理无法被路由的消息
   * 建立一个报警队列，用独立的消费者来进行监测和报警

4. 代码架构 
   
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/6-3.png)

### 6.3.2 代码

1. 配置类

    ```java
    public static final String BACKUP_EXCHANGE_NAME = "backup.exchange";
    public static final String BACKUP_QUEUE_NAME = "backup.queue";
    public static final String WARNING_QUEUE_NAME = "warning.queue";

    @Bean
    public FanoutExchange backupExchange(){
        return new FanoutExchange(BACKUP_EXCHANGE_NAME);
    }

    @Bean
    public Queue backupQueue(){
        return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }

    @Bean
    public Queue warningQueue(){
        return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
    }

    @Bean
    public Binding backupBinding(@Qualifier("backupQueue") Queue backupQueue,
                                 @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(backupQueue).to(backupExchange);
    }

    @Bean
    public Binding warningBinding(@Qualifier("warningQueue") Queue warningQueue,
                                 @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(warningQueue).to(backupExchange);
    }

    // 为确认交换机绑定备份交换机
    @Bean
    public DirectExchange confirmExchange(){
        ExchangeBuilder exchangeBuilder = ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME).durable(true).withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME);
        return exchangeBuilder.build();
    }
    ```

2. 报警消费者

    ```java
    package com.atguigu.rabbitmq.springbootrabbitmq.consumer;

    import lombok.extern.slf4j.Slf4j;
    import org.springframework.amqp.core.Message;
    import org.springframework.amqp.rabbit.annotation.RabbitListener;
    import org.springframework.stereotype.Component;

    import java.nio.charset.StandardCharsets;

    @Slf4j
    @Component
    public class WarningConsumer {

        public static final String WARNING_QUEUE_NAME = "warning.queue";

        @RabbitListener(queues = WARNING_QUEUE_NAME)
        public void receiveWarningMsg(Message message){
            String msg = new String(message.getBody(), StandardCharsets.UTF_8);
            log.warn("报警发现不可路由消息{}", msg);
        }
    }
    ```

3. 操作
   * 删除原来的交换机
   * 启动

