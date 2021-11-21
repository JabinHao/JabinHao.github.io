---
title: Kafka简单使用(1)
excerpt: 通过 docker-compose 搭建 Kafka 集群，在 Spring Boot 中使用 Kafka
tags:
  - kafka
categories:
  - Kafka
banner_img: /img/banner/kafka.png
index_img: /img/index/kafka.png
category: Kafka
abbrlink: fb3cac4b
date: 2021-10-31 20:20:35
updated: 2021-11-16 21:30:39
subtitle:
---


## 1. 搭建集群

### 1.1 通过 docker 搭建

1. 创建网络

    ```sh
    docker network create --subnet 172.30.0.0/23 --gateway 172.30.0.1 kafka
    ```

2. docker-compose-zookeeper

    ```sh
    version: '3.9'

    services: 
        zoo1:
            image: bitnami/zookeeper:3.7.0
            restart: always
            hostname: zoo1
            container_name: zoo1
            ports:
                - 2184:2181
            volumes: 
                - "/home/jphao/kafka/zkcluster/zoo1/data:/data"
                - "/home/jphao/kafka/zkcluster/zoo1/datalog:/datalog"
            environment: 
                ZOO_SERVER_ID: 1
                ZOO_SERVERS: 0.0.0.0:2888:3888, zoo2:2888:3888, zoo3:2888:3888
                ALLOW_ANONYMOUS_LOGIN: 'true'
            networks:
                kafka:
                    ipv4_address: 172.30.0.11

        zoo2:
            image: bitnami/zookeeper:3.7.0
            restart: always
            hostname: zoo2
            container_name: zoo2
            ports:
                - 2185:2181
            volumes: 
                - "/home/jphao/kafka/zkcluster/zoo2/data:/data"
                - "/home/jphao/kafka/zkcluster/zoo2/datalog:/datalog"
            environment: 
                ZOO_SERVER_ID: 2
                ZOO_SERVERS: zoo1:2888:3888, 0.0.0.0:2888:3888, zoo3:2888:3888
                ALLOW_ANONYMOUS_LOGIN: 'true'
            networks:
                kafka:
                    ipv4_address: 172.30.0.12

        zoo3:
            image: bitnami/zookeeper:3.7.0
            restart: always
            hostname: zoo3
            container_name: zoo3
            ports:
                - 2186:2181
            volumes: 
                - "/home/jphao/kafka/zkcluster/zoo3/data:/data"
                - "/home/jphao/kafka/zkcluster/zoo3/datalog:/datalog"
            environment: 
                ZOO_SERVER_ID: 3
                ZOO_SERVERS: zoo1:2888:3888, zoo2:2888:3888, 0.0.0.0:2888:3888
                ALLOW_ANONYMOUS_LOGIN: 'true'
            networks:
                kafka:
                    ipv4_address: 172.30.0.13

    networks: 
        kafka:
            external: 
                name: kafka 
    ```

3. docker-compose-kafka

    ```sh
    version: '3.9'

    services: 
        kafka1:
            image: bitnami/kafka:2.7.0
            restart: always
            hostname: kafka1
            container_name: kafka1
            privileged: true
            ports:
                - 9092:9092
            environment:
                KAFKA_ADVERTISED_HOST_NAME: kafka1
                KAFKA_LISTENERS: PLAINTEXT://kafka1:9092
                KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
                KAFKA_ADVERTISED_PORT: 9092
                KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
                ALLOW_PLAINTEXT_LISTENER: 'true'
            volumes:
                - /home/jphao/kafka/kafkaCluster/kafka1/logs:/kafka
            networks:
                kafka:
                    ipv4_address: 172.30.1.11
            extra_hosts: 
                zoo1: 172.30.0.11
                zoo2: 172.30.0.12
                zoo3: 172.30.0.13

        kafka2:
            image: bitnami/kafka:2.7.0
            restart: always
            hostname: kafka2
            container_name: kafka2
            privileged: true
            ports:
                - 9093:9093
            environment:
                KAFKA_ADVERTISED_HOST_NAME: kafka2
                KAFKA_LISTENERS: PLAINTEXT://kafka2:9093
                KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9093
                KAFKA_ADVERTISED_PORT: 9093
                KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
                ALLOW_PLAINTEXT_LISTENER: 'true'
            volumes:
                - /home/jphao/kafka/kafkaCluster/kafka2/logs:/kafka
            networks:
                kafka:
                    ipv4_address: 172.30.1.12
            extra_hosts: 
                zoo1: 172.30.0.11
                zoo2: 172.30.0.12
                zoo3: 172.30.0.13

        kafka3:
            image: bitnami/kafka:2.7.0
            restart: always
            hostname: kafka3
            container_name: kafka3
            privileged: true
            ports:
                - 9094:9094
            environment:
                KAFKA_ADVERTISED_HOST_NAME: kafka3
                KAFKA_LISTENERS: PLAINTEXT://kafka3:9094
                KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9094
                KAFKA_ADVERTISED_PORT: 9094
                KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
                ALLOW_PLAINTEXT_LISTENER: 'true'
            volumes:
                - /home/jphao/kafka/kafkaCluster/kafka3/logs:/kafka
            networks:
                kafka:
                    ipv4_address: 172.30.1.13
            extra_hosts: 
                zoo1: 172.30.0.11
                zoo2: 172.30.0.12
                zoo3: 172.30.0.13                            

    networks: 
        kafka:
            external: 
                name: kafka

    ```

4. 启动

    ```sh
    # 创建容器
    docker-compose -f docker-compose-zookeeper.yml up -d
    docker-compose -f docker-compose-kafka.yml up -d

    # 查看容器
    docker ps

    # 查看主题
    kafka-topics.sh --bootstrap-server kafka2:9093 --list
    kafka-topics.sh --bootstrap-server kafka1:9092 --describe --topic kraft-test
    ```


### 1.2 配置

1. 在 `/etc/hosts` 文件中添加：

    ```
    127.0.0.1 kafka1

    127.0.0.1 kafka2

    127.0.0.1 kafka3
    ```
    也可以将docker-compose-kafka.yml中
    ```yml
    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
    ```
    中的 kafka1、kafka2、kafka3 替换为宿主机 ip

2. 查看容器的网络信息

    ```sh
    docker network inspect kafka
    ```

## 2. 在 Spring Boot 中应用

###  2.1 环境搭建

1. 依赖

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    ```

2. 配置

    ```yml
    spring:
    kafka:
        bootstrap-servers: localhost:9092
        consumer:

        auto-offset-reset: earliest
        key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        producer:
        key-serializer: org.apache.kafka.common.serialization.StringSerializer
        value-serializer: org.apache.kafka.common.serialization.StringSerializer
    ```

3. 手动提交

    ```yml
    consumer:
      enable-auto-commit: false
    ```
    手动提交时 `ack-mode` 设置
    ```yml
    kafka:
        listener:
        ack-mode: manual
    ```

    ackMode | 说明
    :-:|:-
    MANUAL | 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后, 手动调用 Acknowledgment.acknowledge() 后提交
    MANUAL_IMMEDIATE | 手动调用 Acknowledgment.acknowledge() 后立即提交
    RECORD | 当每一条记录被消费者监听器（ListenerConsumer）处理之后提交
    BATCH(默认) | 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后提交
    TIME | 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，距离上次提交时间大于TIME时提交
    COUNT | 当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后，被处理record数量大于等于COUNT时提交
    COUNT_TIME | TIME或COUNT　有一个条件满足时提交


4. 项目源码

    [github](https://github.com/JabinHao/spring-boot-demo)

### 2.2 生产者

1. 说明
   * 理想结构：controller层接受请求，service层作为生产者，将消息存入 kafka
   * 这里为了简化省去service层

2. UserController.java

    ```java
    @PostMapping(value = "/save/user")
    public void saveUser(@RequestBody User user){

        log.info("save user: {}", user);
        String json = JSON_UTIL.toJson(user);
        kafkaTemplate.send("User", json);
    }
    ```

### 2.3 消费者

1. 消费者接收消息，存入数据库
2. Consumer.java

    ```java
    @Service
    public class Consumer {

        private final Logger logger = LoggerFactory.getLogger(Producer.class);

        private final UserMapper userMapper;

        public Consumer(UserMapper userMapper) {
            this.userMapper = userMapper;
        }

        @KafkaListener(topics = "User", groupId = "group_id")
        public void consumer(String message) throws IOException {
            logger.info(String.format("#### -> Consumed message -> %s", message));
            User user = CommonConstant.JSON_UTIL.fromJson(message, User.class);
            userMapper.insert(user);
        }
    }
    ```

4. 设置消费组、多topic、指定分区、指定偏移量消费及设置消费者个数
   
    ```java
    @KafkaListener(groupId = "group_id", topicPartitions = {
            @TopicPartition(topic = "topic1", partitions = {"0", "1"}),
            @TopicPartition(topic = "topic2", partitions = "0",
                    partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "100"))
    },concurrency = "3")//concurrency就是同组下的消费者个数，就是并发消费数, 建议⼩于等于分区总数
    public void listenGroup(ConsumerRecord<String, String> record, Acknowledgment ack) {
        String value = record.value();
        System.out.println(value);
        System.out.println(record);
        //⼿动提交offset
        ack.acknowledge();
    }
    ```

## Reference

1. [Apache Kafka](https://kafka.apache.org/)
2. [Spring for Apache Kafka](https://docs.spring.io/spring-kafka/docs/current/reference/html/)
3. [How to Work with Apache Kafka in Your Spring Boot Application](https://www.confluent.io/blog/apache-kafka-spring-boot-application/)
