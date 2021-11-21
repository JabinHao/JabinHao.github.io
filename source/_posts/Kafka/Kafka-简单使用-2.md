---
title: Kafka 简单使用(2)
excerpt: Kafka 命令行工具简单使用，Kafka Java客户端的简单实现
tags:
  - kafka
categories:
  - Kafka
banner_img: /img/banner/kafka.png
index_img: /img/index/kafka.png
category: Kafka
date: 2021-11-19 12:13:06
updated: 2021-11-21 15:20:06
subtitle:
---

## 1. Kafka 命令行工具

Kafka 提供了一些命令行工具，可以对 Kafka 进行管理，其实是 shell 脚本，放在 bin 目录下：  

![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Kafka/commandline.png)

### 1.1 Topic 相关

1. 创建 topic
    ```sh
    kafka-topics.sh --zookeeper zoo1:2181 --create --topic "ustc" --partitions 3 --replication-factor 2
    ```

2. 修改 topic 

    ```sh
    # 增加分区数
    kafka-topics.sh --bootstrap-server kafka2:9093 --alter --topic "ustc" --partitions 5
    ```

3. 查看 topic 列表

    ```sh
    $ kafka-topics.sh --bootstrap-server kafka2:9093 --list
    __consumer_offsets
    insert_order
    ustc
    ```
    * 可以看到一个名为 `__consumer_offsets` 的主题，Kafka 0.9.0.0 以后，topic 的 offset 信息不再存放在 zookeeper 中，而是放在该 topic中
    * --zookeeper zookeeper:port 也可以用 --bootstrap-server kafka:port 来替换

4. 查看 topic 具体信息

    ```sh
    kafka-topics.sh --bootstrap-server kafka1:9092 --describe --topic "ustc"
    ```

5. 删除 topic

    ```sh
    kafka-topics.sh --zookeeper zoo1:2181 --delete --topic "ustc"
    ```

### 1.2 生产者 Producer

1. 创建生产者

    ```sh
    kafka-console-producer.sh --bootstrap-server kafka1:9092 --topic "ustc"
    ```

### 1.3 消费组 Consumer

1. 创建消费者

    ```sh
    kafka-console-consumer.sh --bootstrap-server kafka1:9092 --topic insert_order 
    ```

2. 其它参数
    * --from-beginning：从头开始消费
    * --offset \<offset>: 从指定的 offset 开始消费
    * --group \<group id>: 指定消费者组

### 1.4 消费组组

1. 查看消费组组列表

    ```sh
    kafka-consumer-groups.sh --bootstrap-server kafka1:9092 --list
    ```

2. 查看消费组的具体信息

    ```sh
    kafka-consumer-groups.sh --bootstrap-server kafka1:9092 --describe --group myId
    ```

3. 删除组

    ```sh
    kafka-consumer-groups.sh --bootstrap-server kafka1:9092 --group myId --delete
    ```

## 2. 生产者 Java 客户端

### 2.1 环境搭建

1. 依赖

    ```xml
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>2.7.1</version>
    </dependency>
    ```

2. 生产者

    ```java
    Properties props = new Properties();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "222.195.87.224:9092");
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG , StringSerializer.class.getName());
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

    KafkaProducer<String, String> producer =  new KafkaProducer<>(props);
    ```

3. 发送消息

    ```java
    ProducerRecord<String, String> record = new ProducerRecord<>("user", UUID.randomUUID().toString(), "hello world");

    RecordMetadata metadata = null;
    try {
        metadata = producer.send(record).get();
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }

    log.info("topic: {}, partition: {}, offset: {}", metadata.topic(), metadata.partition(), metadata.offset());
    ```

### 2.2 同步发送消息

1. 同步的两种方式
    * 上面代码用的是同步发送消息的方式，在收到kafka的ack通知之前⼀直处于阻塞状态
    * 另一种方式是发送后不等待通知

2. 同步的第二种方式

    ```java
    userKafkaProducer.send(record);
    userKafkaProducer.close();
    ```

### 2.3 异步发送消息

1. 说明
    * Producer 提供了异步发送消息的方法 `Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback);`
    * `callback` 是一个接口，用于回调
    * 异步可能会有消息丢失的问题，使用较少

2. 异步发送

    ```java
    producer.send(record, (RecordMetadata metadata, Exception exception)->{
            System.out.println("进入异步发送回调函数");
    if(exception != null){
        System.out.println("出现异常："+exception.getMessage());
    }
    System.out.println(String.format("发送结果：topic:%s,存储的partition:%s，offset:%s",
            metadata.topic(),
            metadata.partition(),
            metadata.offset()));
    });
    ```

### 2.4 生产者参数

1. 消息确认 ack 设置
    ```java
    props.put(ProducerConfig.ACKS_CONFIG, "0");
    ```

2. 发送失败重试设置
    ```java
    // 重试次数
    props.put(ProducerConfig.RETRIES_CONFIG, 3);
    // 重试时间间隔, 默认 100 ms
    props.put(ProducerConfig.RECONNECT_BACKOFF_MS_CONFIG, 300);
    ```

3. 缓冲区相关设置

    ```java
    // 消息缓冲区，先将要发送的消息存入缓冲区，提高性能，默认32M（单位byte）
    props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
    // 本地线程拉取该参数指定的大小的数据发送到 broker
    props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16);
    // batch 最大等待时间，数据达不到指定大小时超时也会发出(ms)
    props.put(ProducerConfig.LINGER_MS_CONFIG, 10);
    ```

## 3. 消费者 Java 客户端

### 3.1 环境搭建

1. 依赖同上
2. 消费者

    ```java
    @Slf4j
    public class AnotherConsumer {

        public static final String TOPIC_NAME = "user";
        public static final String CONSUMER_GROUP_NAME = "testGroup";

        public static void main(String[] args) {
            Properties props = new Properties();
            props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "222.195.87.224:9092");
            props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG , StringDeserializer.class.getName());
            props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
            props.put(ConsumerConfig.GROUP_ID_CONFIG, CONSUMER_GROUP_NAME);
            KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
            consumer.subscribe(Arrays.asList(TOPIC_NAME));

            while (true){
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
                for (ConsumerRecord<String, String> record : records) {
                    log.info("partition:{}， offset:{}, key:{}, value:{}", record.partition(), record.offset(), record.key(), record.value());
                }
            }
        }
    }
    ```

### 3.2 消费者参数

1. 自动提交 offset

    ```java
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
    // 时间间隔
    props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
    ```

2. 手动提交 offset

    ```java
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
    ```
    手动提交可以同步也可以异步：
    ```java
    /**
     * 同步提交
     */
    if (records.count() > 0){
        consumer.commitSync();;
    }
    ```
    ```java
    /**
     * 异步提交
     */
    if (records.count() > 0){
        consumer.commitAsync(new OffsetCommitCallback() {
            @Override
            public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
                // ...
            }
        });
    }
    ```

3. poll() 消息

    ```java
    // 一次 poll 的最大消息条数
    props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500);
    ```
    长轮询时间内未 poll 够指定数量消息则继续poll;  
    超过长轮询时间仍未 poll 到指定数量的消息，则直接向下执行：
    ```java
    // 长轮询时间
    consumer.poll(Duration.ofMillis(1000));
    ```

4. reblance相关

    ```java
    // 两次poll的时间间隔超出该时间(30s)则消费者会被踢出消费者组
    props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 30*1000);
    ```

    ```java
    // 消费者每1s向集群发送心跳，超过10s未收到则踢出
    props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10*1000);
    ```

5. 指定分区等

    ```java
    // 指定分区
    consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 1)));
    // 回溯消费（从头开始）
    consumer.seekToBeginning(Arrays.asList(new TopicPartition(TOPIC_NAME, 1)));
    // 指定 offset
    consumer.seek(new TopicPartition(TOPIC_NAME, 1), 1);
    ```

6. 指定时间点

    ```java
    List<PartitionInfo> topicPartitions = consumer.partitionsFor(TOPIC_NAME);
    //从1⼩时前开始消费
    long fetchDataTime = new Date().getTime() - 1000 * 60 * 60;
    Map<TopicPartition, Long> map = new HashMap<>();
    for (PartitionInfo par : topicPartitions) {
        map.put(new TopicPartition(TOPIC_NAME, par.partition()),
                fetchDataTime);
    }
    Map<TopicPartition, OffsetAndTimestamp> parMap =
            consumer.offsetsForTimes(map);
    for (Map.Entry<TopicPartition, OffsetAndTimestamp> entry : parMap.entrySet()) {
        TopicPartition key = entry.getKey();
        OffsetAndTimestamp value = entry.getValue();
        if (key == null || value == null) continue;
        Long offset = value.offset();
        System.out.println("partition-" + key.partition() + "|offset-" + offset);
        //根据消费⾥的timestamp确定offset
        if (value != null) {
            consumer.assign(Arrays.asList(key));
            consumer.seek(key, offset);

            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(String.format("partition:%s， offset:%s, key:%s, value:%s", record.partition(), record.offset(), record.key(), record.value()) );

            }
        }
    }
    ```

7. 新消费者 offset 设置
    ```java
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    ```
    * earliest 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
    * latest 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据


