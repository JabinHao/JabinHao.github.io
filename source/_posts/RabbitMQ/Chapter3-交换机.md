---
title: Chapter3 交换机
excerpt: 交换机与基于交换机的三种模式（发布/订阅、路由、主题）
tags:
  - RabbitMQ
categories:
  - RabbitMQ
banner_img: /img/banner/doggie.png
index_img: /img/index/RabbitMQ.png
category: RabbitMQ
abbrlink: 43055b81
date: 2021-06-19 23:11:22
updated: 2021-06-19 23:46:02
subtitle:
---
## 3.1 背景

### 3.1.1 发布/订阅模式

1. 在工作队列中，消息会通过轮询的方式送达到每个队列，但是每个消息只会被唯一送达至某一个队列
2. 把一个消息转发给全部消费者，这种模式称之为发布-订阅模式，即生产者发布的消息将会以广播的形式转发到所有的消费者
3. 对于每个队列，消息只能被消费一次，因此我们需要交换机来实现该模式
4. 交换机还能实现路由、主题两种模式
   
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/3-1.png)

### 3.1.2 应用场景

1. 电商下单后，订单需要同步给库存系统以及邮件系统等等其他的业务模块


## 3.2 Exchanges

### 3.2.1 概念

1. RabbitMQ 消息传递模型的核心思想是: 生产者生产的消息从不会直接发送到队列
2. 生产者只能将消息发送到交换机(exchange)，由交换机的类型决定如何处理消息
3. 交换机类型
   * 直接（direct）
   * 主题（topic）
   * 标题（headers）
   * 扇出（fanout）

### 3.2.2 无名 exchange

1. 发布消息时需要指定交换机
   ```java
    void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body)
   ```
   第一个参数 exchange 即为交换机

2. 交换机可以为空字符串，表示默认或无名称交换机： 消息能路由发送到队列中其实是由 routingKey(bindingkey)绑定 key 指定的，如果它存在的话


## 3.3 其它概念

### 3.3.1 临时队列

1. 可以创建临时队列用于消费者读取消息，一旦断开了消费者的连接，队列将被自动删除。
2. 创建方式
   ```java
   String queueName = channel.queueDeclare().getQueue();
   ```

### 3.3.2 绑定（bindings）

1. 概念：binding 是 exchange 和 queue 之间的桥梁，指明 exchange 和那个队列进行了绑定关系
2. 方法：
   ```java
   Queue.BindOk queueBind(String queue, String exchange, String routingKey)
   ```
   也可以在浏览器管理界面中绑定

3. 当 routingKey 与 发布消息时的 routingKey 匹配时，消息才会传送到指定队列

## 3.4 Fanout (发布/订阅)

### 3.4.1 简介

1. Fanout即发布/订阅，广播接收到的消息给所有队列
2. Fanout不会处理 routingKey，而是转发给所有已绑定队列

### 3.4.2 使用

1. 生产者发送消息

   ```java
   public class EmitLog {

      private static final String EXCHANGE_NAME = "logs";
      public static void main(String[] args) {
         try {
               final Channel channel = RabbitMQUtils.getChannel();

               channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
               Scanner scanner = new Scanner(System.in);
               System.out.println("输入信息：");
               while (scanner.hasNext()){
                  String message = scanner.next();
                  channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes(StandardCharsets.UTF_8));
                  System.out.println("生产者发出消息：" + message);
               }
         } catch (IOException | TimeoutException e) {
               e.printStackTrace();
         }
      }
   }
   ```

2. 消费者1接收消息打印到控制台

    ```java
   public class ReceiveLogs01 {

      private static final String EXCHANGE_NAME = "logs";
      public static void main(String[] args) throws Exception{
         final Channel channel =RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

         final String queueName = channel.queueDeclare().getQueue();
         channel.queueBind(queueName,EXCHANGE_NAME, "");
         System.out.println("ReceiveLogs01等待接收消息");

         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
               String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
               System.out.println("ReceiveLogs01接收到消息：" + message);
         };
         CancelCallback cancelCallback = consumerTag -> {};

         channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
      }
   }
    ```
3. 消费者2接收消息存储到磁盘

    ```java
   public class ReceiveLogs02 {

      private static final String EXCHANGE_NAME = "logs";
      public static void main(String[] args) throws Exception{
         final Channel channel = RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

         final String queueName = channel.queueDeclare().getQueue();
         channel.queueBind(queueName,EXCHANGE_NAME, "");
         System.out.println("ReceiveLogs02等待接收消息");

         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
               String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
               FileUtils.writeStringToFile(new File("f:\\code\\java\\test\\log.txt"), message, StandardCharsets.UTF_8);
               System.out.println("ReceiveLogs02将消息" + message + "存储到磁盘");
         };
         CancelCallback cancelCallback = consumerTag -> {};

         channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
      }
   }    
    ```


## 3.5 Direct(路由)

### 3.5.1 简介

1. 路由模式跟发布订阅模式类似，然后在订阅模式的基础上加上了类型，订阅模式是分发到所有绑定到交换机的队列，路由模式只分发到绑定在交换机上面指定路由键(routingKey )的队列
2. 路由模式与发布/订阅模式设置上不同之处在于两个参数，exchange类型和 routingKey
3. 路由模式示意：（[图片来源](https://www.cnblogs.com/niceyoo/p/11448093.html)）
   
   ![路由模式](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/5-3.png)

4. 多重绑定  
   如果 exchange 的绑定类型是 direct， 但是它绑定的多个队列的 key 如果都相同，在这种情况下虽然绑定类型是 direct 但是它表现与 fanout 类似

### 3.5.2 代码示意

1. 接收错误日志

   ```java
   public class ReceiveLogsDirect01 {

      public static final String EXCHANGE_NAME = "direct_logs";

      public static void main(String[] args) throws IOException, TimeoutException {
         final Channel channel = RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
         final String QUEUE_NAME = "disk";
         channel.queueDeclare(QUEUE_NAME,false, false, false, null);
         channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "error");

         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
               String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
               FileUtils.writeStringToFile(new File("f:\\code\\java\\test\\error-log.txt"), "接收绑定键：" + delivery.getEnvelope().getRoutingKey() + ", 消息：" + message + "\n", StandardCharsets.UTF_8, true);
               System.out.println("已写入错误日志");
         };
         CancelCallback cancelCallback = consumerTag -> {};

         channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
      }
   }
   ```

2. 接收普通信息
   
   ```java
   public class ReceiveLogsDirect02 {
      public static final String EXCHANGE_NAME = "direct_logs";

      public static void main(String[] args) throws IOException, TimeoutException {
         final Channel channel = RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
         final String QUEUE_NAME = "console";
         channel.queueDeclare(QUEUE_NAME,false, false, false, null);
         channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "info");
         channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "warning");

         System.out.println("等待接收消息.....");
         DeliverCallback deliverCallback = (consumerTag, delivery) -> {
               String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
               System.out.println("接收绑定键：" + delivery.getEnvelope().getRoutingKey() + ", 消息：" + message);
         };
         CancelCallback cancelCallback = consumerTag -> {
         };

         channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
      }
   }
   ```

3. 发送方

   ```java
   public class EmitLogDirect {
      public static final String EXCHANGE_NAME = "direct_logs";

      public static void main(String[] args) throws Exception{
         final Channel channel = RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
         Map<String, String> map = new HashMap<>();
         map.put("info", "信息");
         map.put("warning", "警告");
         map.put("error", "错误");
         map.put("debug", "调试");
         for (Map.Entry<String, String> entry : map.entrySet()) {
               channel.basicPublish(EXCHANGE_NAME, entry.getKey(), null, entry.getValue().getBytes(StandardCharsets.UTF_8));
               System.out.println("生产者发出消息：" + entry.getValue());
         }
      }
   }
   ```

## 3.6 Topics(主题交换机)

### 3.6.1 简介

1. direct 不支持匹配 routingKey，而 topic 主题模式支持规则匹配，只要符合 routingKey 就能发送到绑定的队列上
2. topic 交换机的消息的 routing_key 必须是一个单词列表，以点号分隔开
3. 匹配规则
   * "*" 表示任何一个词
   * "#" 表示0或1个词

### 3.6.2 代码示例

1. 生产者

   ```java
   public class EmitLogTopic {
      private static final String EXCHANGE_NAME = "topic_logs";

      public static void main(String[] args) throws Exception{
         final Channel channel = RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, "topic");
         Map<String, String> bindingKeyMap = new HashMap<>();
         bindingKeyMap.put("quick.orange.rabbit","被队列 Q1Q2 接收到");
         bindingKeyMap.put("lazy.orange.elephant","被队列 Q1Q2 接收到");
         bindingKeyMap.put("quick.orange.fox","被队列 Q1 接收到");
         bindingKeyMap.put("lazy.brown.fox","被队列 Q2 接收到");
         bindingKeyMap.put("lazy.pink.rabbit","虽然满足两个绑定但只被队列 Q2 接收一次");
         bindingKeyMap.put("quick.brown.fox","不匹配任何绑定不会被任何队列接收到会被丢弃");
         bindingKeyMap.put("quick.orange.male.rabbit","是四个单词不匹配任何绑定会被丢弃");
         bindingKeyMap.put("lazy.orange.male.rabbit","是四个单词但匹配 Q2");

         for (Map.Entry<String, String> entry : bindingKeyMap.entrySet()) {
               channel.basicPublish(EXCHANGE_NAME, entry.getKey(), null, entry.getValue().getBytes(StandardCharsets.UTF_8));
               System.out.println("生产者发出消息：" + entry.getValue());
         }
      }
   }
   ```

2. 消费者1

   ```java
   public class ReceiveLogsTopic01 {
      private static final String EXCHANGE_NAME = "topic_logs";

      public static void main(String[] args) throws Exception{
         final Channel channel = RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, "topic");
         final String QUEUE_NAME = channel.queueDeclare().getQueue();
         channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "*.orange.*");

         System.out.println("等待接收消息....");
         DeliverCallback deliverCallback = ((consumerTag, delivery) -> {
               String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
               System.out.println("接收队列：" + QUEUE_NAME + ", 绑定键：" + delivery.getEnvelope().getRoutingKey() + ",消息：" + message);
         });
         CancelCallback cancelCallback = consumerTag -> {};

         channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
      }
   }
   ```

3. 消费者2

   ```java
   public class ReceiveLogsTopic02 {
      private static final String EXCHANGE_NAME = "topic_logs";

      public static void main(String[] args) throws Exception{
         final Channel channel = RabbitMQUtils.getChannel();
         channel.exchangeDeclare(EXCHANGE_NAME, "topic");
         final String QUEUE_NAME = channel.queueDeclare().getQueue();
         channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "*.*.rabbit");
         channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "lazy.#");

         System.out.println("等待接收消息....");
         DeliverCallback deliverCallback = ((consumerTag, delivery) -> {
               String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
               System.out.println("接收队列：" + QUEUE_NAME + ", 绑定键：" + delivery.getEnvelope().getRoutingKey() + ",消息：" + message);
         });
         CancelCallback cancelCallback = consumerTag -> {};

         channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
      }
   }
   ```



