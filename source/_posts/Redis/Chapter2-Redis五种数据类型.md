---
title: Chapter2 Redis五种数据类型
excerpt: 五种数据类型、相关的操作命令
tags:
  - redis
categories:
  - Redis
banner_img: /img/post/banner/Sasha.jpg
index_img: /img/post/redis_logo.png
category: Redis
abbrlink: 86f5a7ca
date: 2021-02-02 22:15:54
updated: 2021-02-03 16:05:48
subtitle:
---
## 2.1  字符串类型 string

### 2.1.1 简介

1. Redis 中最基本的数据结构
2. 可以存储任何类型的数据，包括二进制数据，序列化后的数据， JSON 化的对象甚至是一张图片
3. 单个value最大 512M

### 2.1.2 常用命令

1. 添加键值对

    ```
    set key value
    ```

2. 查询对应键值

    ```
    get key
    ```

3. 追加数据到value末尾

    ```
    append key value
    ```

4. 获取value长度

    ```
    strlen key
    ```

5. key不存在则设置值

    ```sh
    setnx key value
    # 或
    set key value nx
    ```

6. value值加减

    ```
    incr key
    decr key
    incrby key 步长
    decrby key 步长
    ```
    * 只能对数字值操作
    * 若value为空则新增为1

7. 设置多个

    ```
    mset key1 value1 key2 value2 ...
    ```

8. 获取多个

    ```
    mget key1 key2 ...
    ```

9. msetnx

    ```
    msetnx key1 value1 key2 value2 .... 
    ```
    * 当且仅当所有给定键都不存在时， 为所有给定键设置值

10. 对值进行切片

    ```
    getrange key start end
    ```
    * 前后都包括，闭集
    * apple-red：`getrange apple 1 2` - ed

11. 赋值

    ```
    setrange key offset value
    ```

12. 设置带过期时间的值

    ```sh
    setex key seconds value
    # 或
    set key value EX seconds
    set key value PX milliseconds
    ```

13. 获取并重新赋值

    ```
    getset key value
    ```

## 2.2 列表类型 list

### 2.2.1 简介

1. 列表是简单的字符串列表，按照插入顺序排序，元素可以重复。
2. 可以添加一个元素到列表的头部（左边）或者尾部（右边） ,底层是个双向链表结构

### 2.2.2 常用命令

1. lpush 与 rpush

    ```
    lpush key value1 value2 ....
    ```
    * 将一个或多个值 value 插入到列表 key 的最左边（表头）/最右边（表尾）， 各个value 值依次插入到表头位置。
    * 插入后 valueN 在最前面
    * 返回值： 插入之后的列表的长度

2. lpop 与 rpop

    ```
    lpop key
    ```
    * 移除并返回列表 key 头部第一个/最后一个元素， 即列表左侧/右侧的第一个元素。
    * 返回值： 列表左侧第一个元素的值；列表 key 不存在，返回 nil

3. rpoplpush

    ```
    rpoplpush key1 key2
    ```
    * key1右侧弹出并插入到key2左侧
    * 返回值：被操作的value

4. lrange

    ```
    lrange key startIndex endIndex
    ```
    * 获取列表 key 中指定下标区间内的元素， 下标从 0 开始，到列表长度-1； 
    * 下标也可以是负数，表示列表从后往前取，-1 表示倒数第一个元素
    * startIndex 和 endIndex 超出范围不会报错。
    *  返回值： 获取到的元素列表

5. lindex

    ```
    lindex key index
    ```
    * 获取列表 key 中下标为指定 index 的元素， 列表元素不删除， 只是查询。 
    * 0 表示列表的第一个元素， 1 表示列表的第二个元素； index 也可以负数的下标， -1 表示列表的最后一个元素。
    * 返回值： key 存在时，返回指定元素的值；Key 不存在时，返回 nil

6. llen

    ```
    llen key
    ```
    * 获取列表 key 的长度
    * 返回值： 数值，列表的长度； key 不存在返回 0

7. linsert

    ```
    linsert key before/after pivot value
    ```
    * 将值 value 插入到列表 key 当中位于值 pivot 之前或之后的位置。 key 不存在或者 pivot 不在列表中，不执行任何操作。
    * 返回值： 命令执行成功，返回新列表的长度。没有找到 pivot 返回 -1， key 不存在返回 0

8. lrem

    ```
    lrem key count value
    ```
    * 根据参数 count 的值，移除列表中与参数 value 相等的元素，
        * count >0 ， 从列表的左侧向右开始移除；
        * count < 0 从列表的尾部开始移除；
        * count = 0 移除表中所有与 value 相等的值。
    * 返回值： 数值，移除的元素个数

9. ltrim

    ```
    ltrim key start stop
    ```
    * 让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除
    * 返回值：OK

10. lset

    ```
    lset key index value
    ```
    * 将列表 key 下标为 index 的元素的值设置为 value。
    * 设置成功返回 ok ; key 不存在或者 index 超出范围返回错误信息

## 2.3 集合类型 set

### 2.3.1 简介

1. Set 是 string 类型的无序不重复集合。
2. 底层是一个value为null的hash表

### 2.3.2 常用命令

1. sadd

    ```
    sadd key member [member...]
    ```
    * 将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略，不会再加入。
    * 返回值： 加入到集合的新元素的个数(不包括被忽略的元素)

2. smembers: 获取集合 key 中的所有成员元素

    ```
    smembers key
    ```
3. sismember

    ```
    sismember key member
    ```
    * 判断 member 元素是否是集合 key 的元素
    * 返回值： member 是集合成员返回 1，其他返回 0

4. scard: 获取集合里面的元素个数

    ```
    scard key
    ```

5. srem

    ```
    srem key member [member...]
    ```
    * 移除集合中一个或多个元素， 不存在的元素被忽略。
    * 返回值： 数字，成功移除的元素个数， 不包括被忽略的元素。

6. srandmember

    ```
    srandmember key [count]
    ```
    * 只提供 key，随机返回集合中一个元素，元素不删除，依然在集合中；
    * 提供了 count 时
        * count 正数, 返回包含 count 个数元素的集合，集合元素各不重复。 
        * count 负数，返回一个 count 绝对值的长度的集合，集合中元素可能会重复多次。
    * 返回值： 一个元素或者多个元素的集合

7. spop

    ```
    spop key
    ```
    * 随机从集合中删除一个或 count 个元素。
    * 返回值： 被删除的元素， key 不存在或空集合返回 nil

8. smove 

    ```
    smove src dest member
    ```
    * 将 member 元素从 src 集合移动到 dest 集合
        * member 不存在， smove 不执行操作，返回 0
        * 如果 dest 存在 member，则仅从 src 中删除 member。
        * 可以对同一个集合操作
    * 返回值： 成功返回 1 ，其他返回 0

9. sinter
    
    ```
    sinter key1 key2 [key...]
    ```
    * 返回值： 交集元素组成的集合，如果没有则返回空集合
  
10. sdiff

    ```
    sdiff key1 key2 [key...]
    ```
    * 返回值： 返回第一个集合中有而后边集合中都没有的元素组成的集合，如果第一个集合中的元素在后边集合中都有则返回空集合

11. sunion

    ```
    sunion key1 key2 [key...]
    ```
    * 返回值： 返回所有集合元素组成的大集合，如果所有 key 都不存在，返回空集合。


## 2.4 哈希类型 hash

### 2.4.1 简介

1. Redis 的 hash 是一个 string 类型的 key 和 value 的映射表 
2. hash的 value 是一系列的键值对， hash 特别适合用于存储对象

### 2.4.2 常用命令

1. hset

    ```
    hset key field value [field value...]
    ```
    * 将键值对 field-value 设置到哈希列表 key 中
        * 如果 key 不存在，则新建哈希列表，然后执行赋值
        * 如果 key下的 field 已经存在，则 value 值覆盖。
    * 返回值： 返回设置成功的键值对个数

2. hget

    ```
    hget key field
    ```
    * 获取哈希表 key 中给定域 field 的值。
    * 返回值： field 域的值，如果 key 不存在或者 field 不存在返回 nil

3. hmset
    * 语法、作用与hset相同
    * 返回 OK 或 nil

4. hexists

    ```
    hexists key field
    ```
    * 查看哈希表 key 中，给定域 field 是否存在
    * 返回值： 如果 field 存在，返回 1，其他返回 0
5. hkeys

    ```
    hkeys key
    ```
    * 查看哈希表 key 中的所有 field 域列表
    * 返回值： 包含所有 field 的列表， key 不存在返回空列表

6. hvals key

    ```
    hvals key
    ```
    * 返回哈希表 中所有域的值列表
    * 返回值： 包含哈希表所有域值的列表， key 不存在返回空列表
  
7. hgetall

    ```
    hgetall key
    ```
    * 获取哈希表 key 中所有的域和值
    * 返回值： 以列表形式返回 hash 中域和域的值， key 不存在，返回空 hash

8. hincrby

    ```
    hincrby key field increment
    ```
    * 给哈希表 key 中的 field 域增加 int
    * 返回值： 返回增加之后的 field 域的值

9. hsetnx

    ```
    hsetnx key field value
    ```
    * 将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在的时候才设置，否则不设置。
    * 返回值： 设值成功返回 1，其他返回 0

10. hdel

    ```
    hdel key field [field]
    ```
    * 删除哈希表 key 中的一个或多个指定域 field，不存在 field 直接忽略。
    * 返回值： 成功删除的 field 的数量

11. hlen

    ```
    hlen key 
    ```
    * 获取哈希表 key 中域 field 的个数
    * 返回值： 数值， field 的个数。 key 不存在返回 0


## 2.5 有序集合类型 zset （sorted set）

### 2.5.1 简介

1. 有序集合zset和集合set一样也是string类型元素的集合，且不允许重复的成员。
2. 不同的是 zset 的每个元素都会关联一个分数（score可以重复）， redis 通过分数来为集合中的成员进行从小到大的排序

### 2.5.2 常用命令

1. zadd

    ```
    zadd key score member [score member...]
    ```
    * 将一个或多个 member 元素及其 score 值加入到有序集合 key 中，如果 member 存在集合中，则覆盖原来的值； score 可以是整数或浮点数.
    * 返回值： 数字，新添加的元素个数.

2. zrange

    ```
    zrange key startIndex endIndex [WITHSCORES]
    ```
    * 查询有序集合，指定区间的内的元素。 集合成员按 score 值从小到大来排序； 
        * startIndex 和 endIndex 都是从0 开始表示第一个元素
        * startIndex 和 endIndex 都可以取负数-1 表示倒数第一个元素； 
        * WITHSCORES 选项让 score 和 value 一同返回。
    * 返回值： 指定区间的成员组成的集合

3. zrangebyscore

    ```
    zrangebyscore key min max [WITHSCORES] [LIMIT offset count]
    ```
    * 获取有序集 key 中，所有 score 值介于 min 和 max 之间（包括 min 和 max）的成员，有序成员是按递增（从小到大）排序；
        * 使用符号”(“ 表示包括 min 但不包括 max；
        * withscores 显示 score 和 value；
        * limit 用来限制返回结果的数量和区间，在结果集中从第 offset 个开始，取 count 个
    * 返回值： 指定区间的集合数据
4. zrevrange

    ```
    zrevrange key startIndex endIndex [WITHSCORES]
    ```
    * 查询有序集合，指定区间的内的元素。集合成员按 score 值从大到小来排序

5. zrevrangebyscore
   
    ```
    zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]
    ```
    * 成员按递减（从大到小）排序

6. zincrby

    ```
    zincrby key increment value
    ```
    * 为元素的score加上增量
    * 返回值：增加后的score

7. zrem

    ```
    zrem key member [member...]
    ```
    * 删除有序集合 key 中的一个或多个成员，不存在的成员被忽略。
    * 返回值： 被成功删除的成员数量，不包括被忽略的成员

8. zcard

    ```
    zcard key
    ```
    * 获取有序集 key 的元素成员的个数。
    * 返回值： key 存在， 返回集合元素的个数； key 不存在，返回 0
9. zcount 

    ```
    zcount key min max
    ```
    * 返回有序集 key 中， score 值在 min 和 max 之间(包括 score 值等于 min 或 max )的成员的数量。
    * 返回值： 指定有序集合中分数在指定区间内的元素数量

10. zrank

    ```
    zrank key member
    ```
    * 获取有序集 key 中成员 member 的排名，有序集成员按 score 值从小到大顺序排列， 从 0 开始排名， score最小的是 0 。
    * 返回值： 指定元素在有序集合中的排名；如果指定元素不存在，返回 nil。

11. zrevrank

    ```
    zrevrank key member
    ```
    * 有序集成员按 score 值从大到小顺序排列














