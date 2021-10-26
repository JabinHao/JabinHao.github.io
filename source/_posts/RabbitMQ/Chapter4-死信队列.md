---
title: Chapter4 死信队列
excerpt: 什么是死信队列，消息进入死信队列的三种情况
tags:
  - RabbitMQ
categories:
  - RabbitMQ
banner_img: /img/banner/doggie.png
index_img: /img/index/RabbitMQ.png
category: RabbitMQ
abbrlink: 97d9cc69
date: 2021-06-20 19:32:32
updated: 2021-06-21 15:38:00
subtitle:
---
## 4.1 简介

### 4.1.1 概念

1. 死信（Dead Letter）即无法被消费的消息：
    * 由于特定的原因导致 queue 中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信
    * “死信”消息会被RabbitMQ进行特殊处理，如果配置了死信队列信息，那么该消息将会被丢进死信队列中，如果没有配置，则该消息将会被丢弃

2. 来源
   * 消息 TTL 过期
   * 队列达到最大长度
   * 消息被拒绝(basic.reject 或 basic.nack)并且 requeue=false.

3. 死信示意

    ![死信架构](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/4-1.png)

## 4.2 代码示例

### 4.2.1 消息 TTL 过期

1. 生产者

    ```java
    public class Producer {
        private static final String NORMAL_EXCHANGE = "normal_exchange";

        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);

            // 设置过期时间
            AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();
            for (int i = 1; i <= 10; i++) {
                String message = String.valueOf(i);
                channel.basicPublish(NORMAL_EXCHANGE, "mei", properties, message.getBytes(StandardCharsets.UTF_8));
                System.out.println("生产者发送消息：" + message);
            }
        }
    }
    ```

2. 消费者 C1：启动之后关闭该消费者 模拟其接收不到消息

    ```java
    public class Consumer01 {
        // 普通交换机
        private static final String NORMAL_EXCHANGE = "normal_exchange";
        // 私信交换机
        private static final String DEAD_EXCHANGE = "dead_exchange";
        
        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
            channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

            // 死信队列
            String deadQueue = "dead-queue";
            channel.queueDeclare(deadQueue, false, false, false, null);
            channel.queueBind(deadQueue, DEAD_EXCHANGE, "rui");

            // 绑定
            Map<String, Object> map = new HashMap<>();
            map.put("x-dead-letter-exchange", DEAD_EXCHANGE); // 为正常队列绑定死信交换机
            map.put("x-dead-letter-routing-key", "rui");      // 为正常队列绑定死信 routing-key

            // 正常队列
            String normalQueue = "normal-queue";
            channel.queueDeclare(normalQueue, false, false, false, map); // 绑定死信队列
            channel.queueBind(normalQueue, NORMAL_EXCHANGE, "mei");

            System.out.println("等待接收消息....");
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println("Consumer01收到消息：" + message);
            };

            channel.basicConsume(normalQueue, true, deliverCallback, consumer -> {});
        }
    }
    ```

3. 消费者 C2

    ```java
    public class Consumer02 {
        private static final String DEAD_EXCHANGE = "dead-exchange";

        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();

            String queueName = "dead-queue";
            channel.queueDeclare(queueName, false, false, false, null);
            channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
            channel.queueBind(queueName, DEAD_EXCHANGE, "rui");

            System.out.println("等待接收死信队列消息.....");
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println("Consumer02 接收死信队列的消息" + message);
            };
            channel.basicConsume(queueName, true, deliverCallback, consumer -> {});
        }
    }
    ```

4. 操作顺序
   * 启动Consumer01，关闭
   * 启动生产者，此时可以在浏览器中看到 normal-queue中有 10 条消息
   * 10s后，消息进入 dead-queue
   * 启动 Consumer02，消费到死信队列中的 10 条消息

### 4.2.2 队列达到最大长度

1. 生产者（取消TTL设置）

    ```java
    public class Producer {
        private static final String NORMAL_EXCHANGE = "normal_exchange";

        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);

            for (int i = 1; i <= 10; i++) {
                String message = String.valueOf(i);
                channel.basicPublish(NORMAL_EXCHANGE, "mei", null, message.getBytes(StandardCharsets.UTF_8));
                System.out.println("生产者发送消息：" + message);
            }
        }
    }
    ```

2. 消费者 C1（设置队列长度）

    ```java
    public class Consumer01 {
        // 普通交换机
        private static final String NORMAL_EXCHANGE = "normal_exchange";
        // 私信交换机
        private static final String DEAD_EXCHANGE = "dead_exchange";

        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
            channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

            // 死信队列
            String deadQueue = "dead-queue";
            channel.queueDeclare(deadQueue, false, false, false, null);
            channel.queueBind(deadQueue, DEAD_EXCHANGE, "rui");

            // 绑定
            Map<String, Object> map = new HashMap<>();
            map.put("x-dead-letter-exchange", DEAD_EXCHANGE); // 为正常队列绑定死信交换机
            map.put("x-dead-letter-routing-key", "rui");      // 为正常队列绑定死信 routing-key
            map.put("x-max-length", 6);                       // 设置队列长度

            // 正常队列
            String normalQueue = "normal-queue";
            channel.queueDeclare(normalQueue, false, false, false, map); // 绑定死信队列
            channel.queueBind(normalQueue, NORMAL_EXCHANGE, "mei");

            System.out.println("等待接收消息....");
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println("Consumer01收到消息：" + message);
            };

            channel.basicConsume(normalQueue, true, deliverCallback, consumer -> {});
        }
    }
    ```

3. 消费者C2不变
4. 操作顺序
   * 删除之前的 normal 队列
   * 启动C1，关闭C1
   * 启动C2
   * 启动生产者，发送10条消息

### 4.2.3 消息被拒

1. 生产者：不变
2. 消费者 C1 (取消自动应答)

    ```java
    public class Consumer01 {
        // 普通交换机
        private static final String NORMAL_EXCHANGE = "normal_exchange";
        // 私信交换机
        private static final String DEAD_EXCHANGE = "dead_exchange";

        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
            channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

            // 死信队列
            String deadQueue = "dead-queue";
            channel.queueDeclare(deadQueue, false, false, false, null);
            channel.queueBind(deadQueue, DEAD_EXCHANGE, "rui");

            // 绑定
            Map<String, Object> map = new HashMap<>();
            map.put("x-dead-letter-exchange", DEAD_EXCHANGE); // 为正常队列绑定死信交换机
            map.put("x-dead-letter-routing-key", "rui");      // 为正常队列绑定死信 routing-key

            // 正常队列
            String normalQueue = "normal-queue";
            channel.queueDeclare(normalQueue, false, false, false, map); // 绑定死信队列
            channel.queueBind(normalQueue, NORMAL_EXCHANGE, "mei");

            System.out.println("等待接收消息....");
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                if (message.equals("5")){
                    System.out.println("拒绝此消息：" + message);
                    channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
                }else
                    System.out.println("Consumer01收到消息：" + message);
            };

            // 开启手动应答
            channel.basicConsume(normalQueue, false, deliverCallback, consumer -> {});
        }
    }
    ```

3. 消费者 C2 不变
4. 操作顺序
   * 删除 normal 队列
   * 启动两个消费者
   * 启动生产者

