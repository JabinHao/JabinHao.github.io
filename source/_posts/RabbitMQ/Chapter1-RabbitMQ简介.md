---
title: Chapter1 RabbitMQ简介
excerpt: 什么是消息队列，RabbitMQ是什么，如何在CentOS 8 上安装 RabbitMQ
tags:
  - RabbitMQ
categories:
  - RabbitMQ
banner_img: /img/banner/doggie.png
index_img: /img/index/RabbitMQ.png
category: RabbitMQ
abbrlink: a86f2271
date: 2021-06-18 01:14:07
updated: 2021-06-18 20:28:30
subtitle:
---
## 1.1 消息队列

### 1.1.1 概念

1. 什么是MQ  
   * MQ(message queue)，本质是个队列， FIFO 先入先出，只不过队列中存放的内容是message
   * 是一种跨进程的通信机制，用于上下游传递消息。
   * 在互联网架构中， MQ 是一种非常常见的上下游“逻辑解耦+物理解耦”的消息通信服务。使用了 MQ 之后，消息发送上游只需要依赖 MQ，不用依赖其他服务

2. 作用
   * 流量消峰
   * 应用解耦
   * 异步处理

### 1.1.2 常见MQ

1. ActiveMQ
2. Kafka：高吞吐
3. RocketMQ：Alibaba，为互联网金融而生
4. RabbitMQ

## 1.2 RabbitMQ

### 1.2.1 概念

1. 什么是RabbitMQ  
   消息中间件，接受并转发消息
2. 四大核心概念
   * 生产者：产生数据发送消息
   * 交换机：接收并处理消息
   * 队列：本质上是一个大的消息缓冲区
   * 消费者：等待接收消息的程序

### 1.2.2 六大模式

1. Hello World 简单模式
2. Work queues 工作模式
3. Public/Subscribe 发布/订阅
4. Routing     路由
5. Topics      主题
6. Publisher Confirms 发布确认

### 1.2.3 名词解释

![RabbitMQ 原理图](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/1-1.png)


1. Broker：接收和分发消息的应用， RabbitMQ Server 就是 Message Broker
2. Virtual host：
    * 出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。
    * 当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／ queue 等
3. Connection： producer／ consumer 和 broker 之间的 TCP 连接
4. Channel：
    * Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯
    * AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。 
    * Channel 作为轻量级的Connection 极大减少了操作系统建立 TCP connection 的开销
5. Exchange： 
    * message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。
    * 常用的类型有： direct (point-to-point), topic (publish-subscribe) and fanout (multicast)
6. Queue： 消息最终被送到这里等待 consumer 取走
7. Binding： 
    * exchange 和 queue 之间的虚拟连接， binding 中可以包含 routing key
    * Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

## 1.3 安装（CentOS 8）

### 1.3.1 在线安装

1. 安装 erlang

    ```sh
    # 安装 RabbitMQ 提供的精简版本
        curl -1sLf \
    'https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/setup.rpm.sh' \
    | sudo -E bash
    ```

2. 设置 repository
   
    ```sh
    curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
    ```

3.  更新

    ```sh
    yum update -y
    yum -q makecache -y --disablerepo='*' --enablerepo='rabbitmq_erlang' --enablerepo='rabbitmq_server'
    # 这里需要把 rabbitmq_erlang、rabbitmq_server改成自己的，在 /etc/yum.repo.d/目录下
    # 可以通过 ls /etc/yum.repos.d/ 命令查看
    ```

4. 安装依赖

    ```sh
    yum install socat logrotate -y
    ```

5. 安装 RabbitMQ

    ```sh
    yum install --repo rabbitmq_erlang --repo rabbitmq_server erlang rabbitmq-server -y
    ```

### 1.3.2 命令

1. 设置开机启动

    ```sh
    chkconfig rabbitmq-server on
    ```

2. 启动

    ```sh
    /sbin/service rabbitmq-server start
    ```

3. 查看状态

    ```sh
    /sbin/service rabbitmq-server status
    ```

4. 停止

    ```sh
    /sbin/service rabbitmq-server stop
    ```

### 1.3.3 远程登录

1. 开启web管理

    ```sh
    rabbitmq-plugins enable rabbitmq_management
    ```

2. 创建账户

    ```sh
    rabbitmqctl add_user admin password    
    ```
3. 分配角色

    ```sh
    rabbitmqctl set_user_tags admin administrator
    ```

4. 设置权限

    ```sh
    rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
    ```

5. 浏览器访问

    ```sh
    http://IP:15672/
    ```

