---
title: Chapter7 其它知识点
excerpt: 幂等性问题与解决方案，优先队列，惰性队列
tags:
  - RabbitMQ
categories:
  - RabbitMQ
banner_img: /img/banner/doggie.png
index_img: /img/index/RabbitMQ.png
category: RabbitMQ
abbrlink: 9aefa79f
date: 2021-06-22 20:06:34
updated: 2021-06-22 23:42:59
subtitle:
---
## 7.1 幂等性

### 7.1.1 概念

1. 用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。
2. 消息重复消费  
   消费者在消费 MQ 中的消息时， MQ 已把消息发送给消费者，消费者在给 MQ 返回 ack 时网络中断，故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息

### 7.1.2 解决

1. 思路  
   使用全局 ID 或者写个唯一标识比如时间戳 或者 UUID 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过
2. 消费端的幂等性保障
   * 唯一 ID+指纹码机制,利用数据库主键去重
   * 利用 redis 的原子性去实现：利用 redis 执行 setnx 命令，天然具有幂等性

## 7.2 优先队列

### 7.2.1 说明

1. 概念
   * 优先队列可以为消息设置优先级
   * 优先级高的消息具备优先被消费的特权
2. 设置优先队列
   * 方法一：控制台
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/7-1.png)
   * 方法二：代码
        ```java
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-max-priority", 10); // 最大优先级为10（可以设置0-255）

        channel.queueDeclare(QUEUE_NAME, false,false, false, arguments);
        ```
3. 为消息添加优先级

    ```java
    AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
    channel.basicPublish("", QUEUE_NAME, properties, message.getBytes(StandardCharsets.UTF_8));
    ```

### 7.2.2 代码

1. 生产者

    ```java
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        final Channel channel = RabbitMQUtils.getChannel();
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-max-priority", 10); // 最大优先级为10（可以设置0-255）

        channel.queueDeclare(QUEUE_NAME, false,false, false, arguments);

        for (int i = 1; i <= 10; i++) {
            String message = String.valueOf(i);
            if (i == 5){
                final AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
                channel.basicPublish("", QUEUE_NAME, properties, message.getBytes(StandardCharsets.UTF_8));
            }else {
                channel.basicPublish("", QUEUE_NAME,null, message.getBytes(StandardCharsets.UTF_8));
            }
        }
        
        System.out.println("消息发送完毕");
    }
    ```

2. 消费者

    ```java
    public static void main(String[] args) throws IOException, TimeoutException {
        final Channel channel = RabbitMQUtils.getChannel();
 
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println(new String(message.getBody()));
        };
        channel.basicConsume(QUEUE_NAME, true,
                deliverCallback,
                consumerTag -> { System.out.println("消费消息被中断"); }
        );
    }
    ```

## 7.3 惰性队列

### 7.3.1 说明
1. RabbitMQ 从 3.6.0 版本开始引入了惰性队列的概念。
2. 惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中
3. 它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储
4. 应用场景：消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积

### 7.3.2 两种模式

1. 队列的模式
   * 默认的为 default 模式
   * lazy模式即为惰性队列的模式

2. 设置懒惰模式
   * 调用 channel.queueDeclare 方法的时候在参数中设置
   * 通过浏览器参数（Lazy mode）的方式设置（具有更高的优先级）

3. 代码

    ```java
    Map<String, Object> arguments = new HashMap<>();
    arguments.put("x-queue-mode", "lazy"); 
    channel.queueDeclare(QUEUE_NAME, false,false, false, arguments);
    ```

