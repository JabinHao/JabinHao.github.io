---
title: Chapter7 Jedis操作Redis
excerpt: 在java中通过Jedis操作Redis
tags:
  - redis
categories:
  - Redis
banner_img: /img/post/banner/kangni.png
index_img: /img/post/redis_logo.png
category: Redis
abbrlink: '52316885'
date: 2021-02-04 02:00:55
updated: 2021-02-04 03:06:00
subtitle:
---
## 7.1 简介

1. 使用 Redis 官方推荐的 Jedis，在 java 应用中操作 Redis。 
2. Jedis 几乎涵盖了 Redis 的所有命令。
3. 操作 Redis 的命令在Jedis 中以方法的形式出现


## 7.2 使用

### 7.2.1 准备工作

1. 修改 bind 为 0.0.0.0 ，启动 redis
2. 新建 maven 工程

    ```xml
    <dependencies>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.5.1</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.7.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    ```

### 7.2.2 测试

1. 新建测试类

    ```java
    package com.bjnode.jedis;

    import org.junit.jupiter.api.Test;

    public class JedisKeyTest {


    }
    ```

2. 测试：key

    ```java
    @Test
    public void testKey() {

        // 连接 redis
        Jedis jedis = new Jedis("34.92.2.255", 6379);

        // 使用jedis对象操作redis服务

        Set<String> keys = jedis.keys("*");
        for (String key : keys) {
            System.out.println(key);
        }
    }
    ```

3. 测试：zset

    ```java
    @Test
    public void testKey() {

        // 连接 redis
        Jedis jedis = new Jedis("34.92.2.255", 6379);

        // 使用jedis对象操作redis服务
        Set<String> zsets = jedis.zrange("zset1", 0, -1);
        for (String zset : zsets) {
            System.out.println(zset);
        }

        System.out.println("**************zset1*******************");
        Set<Tuple> zset1 = jedis.zrangeWithScores("zset1", 0, -1);
        for (Tuple tuple : zset1) {
            System.out.print((int) tuple.getScore());
            System.out.println(": "+ tuple.getElement());
        }
    }
    ```

4. 测试 set

    ```java
    @Test
    public void testSet() {

        // 连接 redis
        Jedis jedis = new Jedis("34.92.2.255", 6379);

        // 使用jedis对象操作redis服务
        SetParams params = SetParams.setParams();
        System.out.println(jedis.set("t1", "v1", params.ex(20).nx()));
    }
    ```

5. 测试 事务

    ```java
     @Test
    public void testTrans() {

        // 连接 redis
        Jedis jedis = new Jedis("34.92.2.255", 6379);

        // 使用jedis对象操作redis服务
        Transaction tran = jedis.multi();
        tran.set("t2", "v2", SetParams.setParams().ex(20).nx());
        Map<String, String> map = new HashMap<>();
        map.put("f1","v1");
        map.put("f2","v2");
        map.put("f3","v3");
        map.put("f4","v4");
        tran.hset("hset",map);
        tran.exec();
    }   
    ```
