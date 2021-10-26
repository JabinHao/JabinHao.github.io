---
title: Chapter2 简单模式与工作模式
excerpt: RabbitMQ的六种核心模式之简单模式与工作模式
tags:
  - RabbitMQ
categories:
  - RabbitMQ
banner_img: /img/banner/doggie.png
index_img: /img/index/RabbitMQ.png
category: RabbitMQ
abbrlink: d8801619
date: 2021-06-18 20:28:39
updated: 2021-06-19 23:10:49
subtitle:
---
## 2.1 简单模式

### 2.1.1 简介

1. 简单队列
   * 简单队列也称为点对点，即一个生产者对应一个消费者，生产者发送消息到队列，消费者在队列中取出消息消费。
   * 组成：一个生产者 P 发送消息到队列 Q，一个消费者 C 接收

2. 生产者步骤
    * 获取连接
    * 创建通道
    * 创建队列声明
    * 发送消息
    * 关闭队列

3. 消费者步骤：
   * 获取连接
   * 获取通道
   * 监听队列
   * 消费消息

4. 缺点  
    如果有一些比较耗时的任务，则需要大量的时间才能处理完毕

### 2.1.2 环境准备

1. 新建maven工程
2. 依赖

    ```xml
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.12.0</version>
    </dependency>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
    ```

### 2.1.3 生产者与消费者

1. 新建类 Producer

    ```java
    import com.rabbitmq.client.*;
    public class Producer {

        public static final String QUEUE_NAME = "hello";

        public static void main(String[] args) throws IOException, TimeoutException {
            // 连接工厂
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("IP地址");

            // 用户名密码
            factory.setUsername("admin");
            factory.setPassword("password");

            // 创建连接
            Connection connection = factory.newConnection();
            // 获取信道
            Channel channel = connection.createChannel();

            /*
            * 生成一个队列
            * 1.队列名称
            * 2.队列里面的消息是否持久化 默认消息存储在内存中
            * 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
            * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
            * 5.其他参数
            */
            channel.queueDeclare(QUEUE_NAME, false,false, false, null);

            String message = "hello world";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println("消息发送完毕");
        }
    }
    ```

2. 新建类 Consumer

    ```java
    import com.rabbitmq.client.*;
    public class Consumer {

        // 队列名称
        public static final String QUEUE_NAME = "hello";

        public static void main(String[] args) throws IOException, TimeoutException {
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("139.224.113.9");
            factory.setUsername("admin");
            factory.setPassword("hjp356908");
            Connection connection = factory.newConnection();

            Channel channel = connection.createChannel();
            /*
            * 消费者消费消息
            * 1.消费哪个队列
            * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
            * 3.消费者未成功消费的回调
            */
            DeliverCallback deliverCallback = (consumerTag, message) -> {
                System.out.println(new String(message.getBody()));
            };
            channel.basicConsume(QUEUE_NAME, true,
                    deliverCallback,
                    consumerTag -> { System.out.println("消费消息被中断"); }
            );
        }

    }
    ```

3. 依次启动生产者与消费者


## 2.2 工作队列

### 2.2.1 简介

1. 工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。
2. 我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。 当有多个工作线程时，这些工作线程将一起处理这些任务

### 2.2.2 轮训分发消息

1. 首先将代码提取

    ```java
    public static Channel getChannel() throws IOException, TimeoutException {
        // 连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("IP地址");

        // 用户名密码
        factory.setUsername("admin");
        factory.setPassword("password");

        // 创建连接
        Connection connection = factory.newConnection();
        // 获取信道
        Channel channel = connection.createChannel();
        return channel;
    }
    ```

2. 消费者

    ```java
    public class Work01 {

        // 队列名称
        public static final String QUEUE_NAME = "hello";

        //接收消息
        public static void main(String[] args) throws Exception {

            Channel channel = RabbitMQUtils.getChannel();

            DeliverCallback deliverCallback = (consumerTag, message) -> {
                System.out.println("接收到消息" + new String(message.getBody()));
            };
            // 消息接收被取消时：
            CancelCallback cancelCallback = consumerTag -> {
                System.out.println(consumerTag + "消费者取消接口回调逻辑");
            };
            // 接收消息
            System.out.println("C1" + "等待接收消息");
            channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
        }
    }
    ```

    修改配置，启动两个线程：
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/2-0.png)
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/2-1.png)

3. 生产者
   
    ```java
    public class Task01 {
        public static final String QUEUE_NAME = "hello";

        public static void main(String[] args) throws Exception {
            final Channel channel = RabbitMQUtils.getChannel();

            channel.queueDeclare(QUEUE_NAME, false, false, false, null);

            // 从控制台接收信息
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()){
                String message = scanner.next();
                channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
                System.out.println("消息发送完成：" + message);
            }
        }
    }
    ```

    输入多条消息，则两个线程会依次交替接收

### 2.2.3  消息应答

1. 简介  
    消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉，则： 
    * 若 RabbitMQ 向消费者传递一条消息后立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费这的消息，因为它无法接收到。
    * rabbitmq 引入消息应答机制:消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了， rabbitmq 可以把该消息删除了

2. 自动应答
   * 消息发送后立即被认为已经传送成功
   * 这种模式仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用

3. 手动应答方法（channel中定义）：
    * `void basicAck(long deliveryTag, boolean multiple)`
        * multiple = true: 批量应答 channel 上未应答的消息
        * multiple = false: 只应答当前处理的消息
    * `void basicNack(long deliveryTag, boolean multiple, boolean requeue)`：拒绝
        * multiple：是否应用于多消息（该消费者先前接收未ack的所有消息）
        * requeue：是否重新入列
    * `void basicReject(long deliveryTag, boolean requeue)`

4. 手动应答的好处是可以批量应答并且减少网络拥堵


5. 消息自动重新入队 
   * 如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)， 导致消息未发送 ACK 确认， RabbitMQ 将了解到消息未完全处理，并将对其重新排队。
   * 如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。

### 2.2.4 代码示例

1. 生产者

    ```java
    public class Task2 {
        public static final String TASK_QUEUE_NAME = "ack_queue";

        public static void main(String[] args) throws IOException, TimeoutException {
            Channel channel = RabbitMQUtils.getChannel();
            // 生声明队列
            channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
            // 从控制台中输入信息
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()){
                String message = scanner.next();
                channel.basicPublish("", TASK_QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
                System.out.println("生产者发出消息：" + message);
            }
        }
    }
    ```

2. 消费者1

    ```java
    public class Work03 {
        public static final String TASK_QUEUE_NAME = "ack_queue";

        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();
            System.out.println("C1 等待接收消息处理时间较短");

            DeliverCallback deliverCallback = (consumerTag, message) -> {
                // 沉睡 1s
                RabbitMQUtils.sleep(1);
                System.out.println("接收到消息：" + new String(message.getBody(), StandardCharsets.UTF_8));

                // 手动应答
                channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
            };

            // 手动应答
            boolean autoAck = false;
            channel.basicConsume(TASK_QUEUE_NAME,
                    autoAck,
                    deliverCallback,
                    consumerTag -> {
                        System.out.println("消费者取消接口回调逻辑");
                    }
            );
        }
    }
    ```

3. 消费者2

    ```java
    public class Work04 {
        public static final String TASK_QUEUE_NAME = "ack_queue";

        public static void main(String[] args) throws Exception{
            final Channel channel = RabbitMQUtils.getChannel();
            System.out.println("C2 等待接收消息处理时间较长");

            DeliverCallback deliverCallback = (consumerTag, message) -> {
                // 沉睡 1s
                RabbitMQUtils.sleep(30);
                System.out.println("接收到消息：" + new String(message.getBody(), StandardCharsets.UTF_8));

                // 手动应答
                channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
            };

            // 手动应答
            boolean autoAck = false;
            channel.basicConsume(TASK_QUEUE_NAME,
                    autoAck,
                    deliverCallback,
                    consumerTag -> {
                        System.out.println("消费者取消接口回调逻辑");
                    }
            );
        }
    }
    ```

4. 操作
    * 依次启动生产者、两个消费者
    * 生产者发送四条消息
    * 消费者1会受到1、3两条
    * 消费者2过段时间会消费完第二条消息，然后处理第四条
    * 将消费者2关闭
    * 消费者1会收到第四条消息并处理

### 2.2.5  RabbitMQ 持久化

1. 概念
   * 默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息
   * 确保消息不会丢失需要做两件事：将队列和消息都标记为持久化。

2. 队列持久化
    * 非持久化的队列，RabbitMQ重启会丢失
    * 开启持久化：声明队列时 durable 设为 true

3. 消息持久化
    * 消息实现持久化需要在消息生产者修改代码，设置 MessageProperties.PERSISTENT_TEXT_PLAIN 属性
        ```java
        channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8));
        ```
    * 将消息标记为持久化并不能完全保证不会丢失消息.

4. 不公平分发
   * RabbitMQ默认采用轮训分发
   * 可以在消费者中设置为不公平分发
        ```java
        channel.basicQos(1);
        ```

### 2.2.6 预取值

1. 概念
   * 消息的发送是异步的，来自消费者的手动确认本质上也是异步的。因此这里就存在一个未确认的消息缓冲区，因此希望开发人员能限制此缓冲区的大小，以避免缓冲区里面无限制的未确认消息问题
   * 可以通过使用 basicQos 方法设置“预取计数”值，该值定义通道上允许的未确认消息的最大数量。
   * basicQos为1时为不公平分发，大于1则为预取值


## 2.3 发布确认

### 2.3.1 概念

1. 原理
    * 生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式， 所有在该信道上面发布的消息都将会被指派一个唯一的 ID(从 1 开始)
    * 一旦消息被投递到所有匹配的队列之后， broker就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列
    * 如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出， broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。

2. confirm 模式最大的好处在于是异步的

3. 发布确认默认是没有开启的，如果要开启需要调用方法 confirmSelect

    ```java
    channel.confirmSelect(); // 生产者
    ```

### 2.3.2 发布确认的策略

1. 单个确认发布

    ```java
    public static void publishMessageIndividually() throws Exception{
        Channel channel = RabbitMQUtils.getChannel();

        // 声明队列
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);

        // 开启发布确认
        channel.confirmSelect();

        //开始时间
        long begin = System.currentTimeMillis();

        // 批量发消息
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message = i + "";
            channel.basicPublish("", queueName, null, message.getBytes(StandardCharsets.UTF_8));
            boolean flag = channel.waitForConfirms();
            if (flag){
                System.out.println("消息发送成功");
            }
        }

        // 结束时间
        long end = System.currentTimeMillis();
        System.out.println("发布" + MESSAGE_COUNT + "个单独确认消息耗时：" + (end - begin));
    }
    ```

2. 批量确认发布

    ```java
    public static void publishMessageBatch() throws Exception{
        final Channel channel = RabbitMQUtils.getChannel();

        // 声明队列
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);

        // 开启发布确认
        channel.confirmSelect();

        //开始时间
        long begin = System.currentTimeMillis();

        // 批量确认消息大小
        int batchSize = 100;

        // 批量发布消息，批量确认
        for (int i = 1; i <= MESSAGE_COUNT; i++) {
            String message = i + "";
            channel.basicPublish("", queueName, null, message.getBytes(StandardCharsets.UTF_8));

            if (i % batchSize == 0){
                channel.waitForConfirms();
            }
        }

        // 结束时间
        long end = System.currentTimeMillis();
        System.out.println("发布" + MESSAGE_COUNT + "个批量确认消息耗时：" + (end - begin) + "毫秒");
    }
    ```

3. 异步确认发布

    ```java
    public static void publishMessageAsync() throws Exception{
        final Channel channel = RabbitMQUtils.getChannel();
        channel.confirmSelect();

        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);

        // 准备消息监听器
        channel.addConfirmListener(
                (deliveryTag, multiple) ->{System.out.println("确认消息" + deliveryTag);}, // 成功
                (deliveryTag, multiple) -> { System.out.println("未确认消息" + deliveryTag); } // 失败
        );

        final long start = System.currentTimeMillis();

        // 批量发送消息
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message = "消息" + i;
            channel.basicPublish("", queueName, null, message.getBytes(StandardCharsets.UTF_8));
        }

        final long end = System.currentTimeMillis();
        System.out.println("发表" + MESSAGE_COUNT + "个异步消息耗时：" + (end - start) + "毫秒");
    }
    ```
    处理异步未确认消息：
    * 把未确认的消息放到一个基于内存的能被发布线程访问的队列
    * 可以用 ConcurrentLinkedQueue 在 confirm callbacks 与发布线程之间进行消息的传递。

    ```java
    public static void publishMessageAsync() throws Exception{
        final Channel channel = RabbitMQUtils.getChannel();
        channel.confirmSelect();

        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);

        ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();

        // 准备消息监听器
        channel.addConfirmListener(
                (deliveryTag, multiple) ->{
                    if (multiple){
                    ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(deliveryTag);
                    confirmed.clear();
                    }else {
                        outstandingConfirms.remove(deliveryTag);
                    }
                    System.out.println("确认消息" + deliveryTag);}, // 成功
                (deliveryTag, multiple) -> {
                    final String message = outstandingConfirms.get(deliveryTag);
                    System.out.println("未确认消息" + message + deliveryTag); } // 失败
        );

        final long start = System.currentTimeMillis();

        // 批量发送消息
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message = "消息" + i;
            channel.basicPublish("", queueName, null, message.getBytes(StandardCharsets.UTF_8));
            outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
        }

        final long end = System.currentTimeMillis();
        System.out.println("发表" + MESSAGE_COUNT + "个异步消息耗时：" + (end - start) + "毫秒");
    }
    ```

4. 对比
   * 单独发布消息：同步等待确认， 简单，但吞吐量非常有限。
   * 批量发布消息：批量同步等待确认， 简单，合理的吞吐量， 一旦出现问题但很难推断出是那条消息出现了问题。
   * 异步处理：最佳性能和资源使用，在出现错误的情况下可以很好地控制，但是实现起来稍微难些


