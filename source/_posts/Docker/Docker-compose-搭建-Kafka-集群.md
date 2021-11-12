---
title: Docker-compose 搭建 Kafka 集群
excerpt: >-
  使用 Docker-compose搭建Kafka集群，zookeeper+Kafka、zookeeper集群+Kafka集群，zookeeper-less
  Kafka集群
tags:
  - Other
categories:
  - Others
banner_img: /img/banner/catiger.png
index_img: /img/index/code.jpg
category: Docker
abbrlink: c9a5c246
date: 2021-11-12 20:52:17
updated: 2021-11-12 20:52:17
subtitle:
---

## 1. 说明

### 1.1 版本选择
1. docker image 选择
    * 比较流行的两个 kafka image：Bitmami/kafka 和 wurstmeister/kafka
    * Bitmami/kafka 由 IBM 维护，且文档较全，推荐使用
    * 还有一个 confluentinc/cp-kafka， 是由 Kafka 团队出的，比较大

2. 环境及版本
    * Manjaro  21.1.6
    * Docker 20.10.9
    * docker-compose 1.29.2
    * Kafka：bitnami/kafka
    * Zookeeper：bitnami/zookeeper


### 1.2 集群架构

1. 单台 Zookeeper + Kafka 集群
    * 只用一台 zookeeper，配置比较简单
    * zookeeper 与 kafka 在同一个 compose 里配置

2. zookeeper 集群 + kafka 集群
    * zookeeper 通过集群保证高可用，Kafka的节点Broker通过domain与ip注册到Zookeeper集群中，通过Zookeeper的协调能力，选出唯一的Leader节点，集群服务启动并对外提供服务
    * zookeeper内部的选举机制，在集群中有其他节点挂掉的话，至少保证n+1个节点用来选举leader，所以集群至少三个节点
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Kafka/zookeeper-kafka.png)

## 2. 集群搭建

### 2.1 Zookeeper + Kafka 集群

1. 创建网络

    ```docker
    docker network create --subnet 172.30.0.0/23 --gateway 172.30.0.1 kafka
    ```
2. docker-compose

    ```yml
    version: '3.9'

    services: 
        zookeeper:
            image: bitnami/zookeeper:3.5.9
            container_name: kafak-zookeeper
            ports:
                - "2181:2181"
            restart: always
            environment: 
                - ALLOW_ANONYMOUS_LOGIN=yes
            volumes: 
                - "/home/kafka/zookeeper/data:/data"
                - "/home/kafka/zookeeper/datalog:/datalog"
            networks:
                kafka:
                    ipv4_address: 172.30.0.11

        kafka1:
            image: bitnami/kafka:2.7.0
            restart: always
            hostname: kafka1
            container_name: kafka1
            privileged: true
            ports:
                - "9092:9092"
            environment:
                KAFKA_CFG_ADVERTISED_HOST_NAME: kafka1
                KAFKA_CFG_LISTENERS: PLAINTEXT://kafka1:9092
                KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
                KAFKA_CFG_ADVERTISED_PORT: 9092
                KAFKA_CFG_ZOOKEEPER_CONNECT: zookeeper:2181
                ALLOW_PLAINTEXT_LISTENER: 'true'
            volumes:
                - /home/kafka/kafkaCluster/kafka1/logs:/kafka
            networks:
                kafka:
                    ipv4_address: 172.30.1.11
            depends_on:
                - zookeeper

        kafka2:
            image: bitnami/kafka:2.7.0
            restart: always
            hostname: kafka2
            container_name: kafka2
            privileged: true
            ports:
                - "9093:9093"
            environment:
                KAFKA_CFG_ADVERTISED_HOST_NAME: kafka2
                KAFKA_CFG_LISTENERS: PLAINTEXT://kafka2:9093
                KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9093
                KAFKA_CFG_ADVERTISED_PORT: 9093
                KAFKA_CFG_ZOOKEEPER_CONNECT: zookeeper:2181
                ALLOW_PLAINTEXT_LISTENER: 'true'
            volumes:
                - /home/kafka/kafkaCluster/kafka2/logs:/kafka
            networks:
                kafka:
                    ipv4_address: 172.30.1.12
            depends_on:
                - zookeeper  

        kafka3:
            image: bitnami/kafka:2.7.0
            restart: always
            hostname: kafka3
            container_name: kafka3
            privileged: true
            ports:
                - "9094:9094"
            environment:

                KAFKA_CFG_ADVERTISED_HOST_NAME: kafka3
                KAFKA_CFG_LISTENERS: PLAINTEXT://kafka3:9094
                KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9094
                KAFKA_CFG_ADVERTISED_PORT: 9094
                KAFKA_CFG_ZOOKEEPER_CONNECT: zookeeper:2181
                ALLOW_PLAINTEXT_LISTENER: 'true'
            volumes:
                - /home/kafka/kafkaCluster/kafka3/logs:/kafka
            networks:
                kafka:
                    ipv4_address: 172.30.1.13
            depends_on:
                - zookeeper                       

    networks: 
        kafka:
            # 使用已存在的网络
            external: 
                name: kafka
    ```

    管理工具 kafka-manager
    ```yml
    version: "2"

    services:
    kafka-manager:
        image: sheepkiller/kafka-manager:latest
        restart: always
        container_name: kafka-manager
        hostname: kafka-manager
        ports:
        - "9000:9000"
    #    links:      # 连接本compose文件创建的container
    #     - kafka1
    #     - kafka2
    #     - kafka3
    #    external_links:  # 连接本compose文件以外的container
    #     - zoo1
    #     - zoo2
    #     - zoo3
        environment:
        ZK_HOSTS: zookeeper:2181
        KAFKA_BROKERS: kafka1:9092,kafka2:9092,kafka3:9092
        APPLICATION_SECRET: letmein
        KM_ARGS: -Djava.net.preferIPv4Stack=true
        networks:
        kafka:
        ipv4_address: 172.30.1.17

    networks:
    kafka:
        external:
        name: kafka
    ```

3. 创建主题

    ```sh
    # 连接任意 kafka 容器
    docker exec -it kafka1 bash
    kafka-topics.sh --create --topic mytopic --partitions 5 --zookeeper 172.30.0.11:2181 --replication-factor 3
    ```

4. 生产消费

    发送消息：
    ```sh
    kafka-console-producer.sh --broker-list kafka1:9092 --topic mytopic
    # 随便发送几条消息
    > hello
    > this is producer
    ```

    消费：
    ```sh
    # 连接其它任意kafka容器
    docker exec -it kafka3 bash
    kafka-console-consumer.sh --bootstrap-server kafka2:9093 --topic chat --from-beginning
    ```

5. 管理

    * 打开浏览器，访问 [宿主机ip:9000](http://192.168.255.128:9000/)
    * 添加集群即可
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Kafka/kafka-manager.png)

### 2.2 Zookeeper 集群 + Kafka 集群
1. zookeeper docker-compose.yml

    ```yml
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

2. kafka docker-compose.yml

    ```yml
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

3. 生产消费

    ```sh
    docker exec -it kafka1 bash

    # 创建主题， 192.168.255.128 为宿主机ip
    kafka-topics.sh --create --topic test1 --replication-factor 3 --partitions 2 --zookeeper 192.168.255.128:2184

    # 生产者
    kafka-console-producer.sh --broker-list 192.168.255.128:9092 --topic test1

    # 消费者
    kafka-console-consumer.sh --bootstrap-server 192.168.255.128:9092 --topic test1 --from-beginning
    ```

### 2.3 Kafka cluster without Zookeeper

1. 说明
    * Kafka 2.8 以后的版本可以不依赖于 Zookeeper 启动
    * 不依赖Zookeeper的情况下，Kafka 使用 Kafka Raft metadata mode (KRaft)，该模式下，Kafka 的信息作为一个 partition 存储在 Kafka 当中
    * 之前的模式中只有一个 Controller，由zookeeper来控制， KRaft 架构下，有多个Controller（Quorum原则）

2. 创建网络

    ```docker
    docker network create --subnet 172.30.16.0/20 --gateway 172.30.16.1 kafka-quorum 
    ```

3. docker compose

    ```yml
    version: "2"

    services:
    kafka-1:
        image: debezium/kafka:1.7
        restart: always
        container_name: kafka1
        ports:
        - 9092:9092
        environment:
        - CLUSTER_ID=5Yr1SIgYQz-b-dgRabWx4g
        - BROKER_ID=1
        - KAFKA_NODE_ID=1
        - KAFKA_ADVERTISED_PORT=9092
        - ADVERTISED_HOST_NAME=kafka1
        - KAFKA_PROCESS_ROLES=broker,controller
        - KAFKA_LISTENERS=PLAINTEXT://kafka1:9092, CONTROLLER://kafka1:19092
        - KAFKA_CONTROLLER_QUORUM_VOTERS=1@kafka-1:19092,2@kafka-2:19093,3@kafka-3:19094
        - KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL      
        - KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT
        - KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
        networks:
        kafka-quorum:
            ipv4_address: 172.30.16.11

    kafka-2:
        image: debezium/kafka:1.7
        restart: always
        container_name: kafka2
        ports:
        - 9093:9093
        environment:
        - CLUSTER_ID=5Yr1SIgYQz-b-dgRabWx4g
        - BROKER_ID=2
        - KAFKA_NODE_ID=2      
        - KAFKA_ADVERTISED_PORT=9093
        - ADVERTISED_HOST_NAME=kafka2
        - KAFKA_PROCESS_ROLES=broker,controller
        - KAFKA_LISTENERS=PLAINTEXT://kafka2:9093, CONTROLLER://kafka2:19093
        - KAFKA_CONTROLLER_QUORUM_VOTERS=1@kafka-1:19092,2@kafka-2:19093,3@kafka-3:19094
        - KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL      
        - KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT
        - KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
        networks:
        kafka-quorum:
            ipv4_address: 172.30.16.12

    kafka-3:
        image: debezium/kafka:1.7
        restart: always
        container_name: kafka3
        ports:
        - 9094:9094
        environment:
        - CLUSTER_ID=5Yr1SIgYQz-b-dgRabWx4g
        - BROKER_ID=3
        - KAFKA_NODE_ID=3      
        - KAFKA_ADVERTISED_PORT=9094
        - ADVERTISED_HOST_NAME=kafka3
        - KAFKA_PROCESS_ROLES=broker,controller
        - KAFKA_LISTENERS=PLAINTEXT://kafka3:9094, CONTROLLER://kafka3:19094 
        - KAFKA_CONTROLLER_QUORUM_VOTERS=1@kafka-1:19092,2@kafka-2:19093,3@kafka-3:19094
        - KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
        - KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT
        - KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
        networks:
        kafka-quorum:
            ipv4_address: 172.30.16.13

    networks: 
    kafka-quorum:
        external:
            name: kafka-quorum
    ```

4. 创建集群

    ```docker
    docker-compose -f docker-compose-quorum.yml up -d
    ```
    运行过程中发现 kafka1 经常停止，重新启动可以通过以下命令：

    ```docker
    docker-compose -f docker-compose-quorum.yml up -d --no-recreate
    ```

5. 创建主题

    ```sh
    kafka-topics.sh --create --topic kraft-test --partitions 3 --replication-factor 3 --bootstrap-server kafka1:9092

    # 查看主题
    kafka-topics.sh --bootstrap-server kafka2:9093 --list
    kafka-topics.sh --bootstrap-server kafka1:9092 --describe --topic kraft-test
    ```
    * kafka1:9092 也可以替换为 宿主机ip:9092

6. 生产消费  

    生产者：
    ```sh
    kafka-console-producer.sh --broker-list kafka1:9092 --topic kraft-test
    ```
    消费者：
    ```sh
    kafka-console-consumer.sh --bootstrap-server kafka3:9094 --topic kraft-test
    ```

7. 通过 metadata shell 查看 metadata

    * 首先找到 metadata 的位置，`/kafka/data/1/@metadata-0`, 可以通过环境变量设置：`KAFKA_LOG_DIRS=/tmp/kraft-combined-logs`

    * 使用
        ```sh
        kafka-metadata-shell.sh  --snapshot /kafka/data/1/@metadata-0/00000000000000000000.log
        ```
        ```sh
        >>> ls brokers/
        >>> ls topics/
        >>> cat topics/kraft-test/0/data
        >>> cat metadataQuorum/leader
        ```

## Reference

1. [Docker-compose 构建 Zookeeper 集群管理 Kafka 集群](https://www.cnblogs.com/hellxz/p/docker_zookeeper_cluster_and_kafka_cluster.html)
2. [How to easily install kafka without zookeeper](https://adityasridhar.com/posts/how-to-easily-install-kafka-without-zookeeper)
3. [Going ZooKeeper-less With the Debezium Container Image for Apache Kafka](https://debezium.io/blog/2021/08/31/going-zookeeperless-with-debezium-container-image-for-apache-kafka/)
4. [Docker Networking Design Philosophy](https://www.docker.com/blog/docker-networking-design-philosophy/)

