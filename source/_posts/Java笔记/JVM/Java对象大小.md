---
title: Java对象大小
excerpt: Java对象模型，Java对象占内存大小分析
tags:
  - jvm
categories:
  - Java笔记
  - JVM
banner_img: /img/banner/limbo.jpg
index_img: /img/index/pacman.png
category: Java笔记/JVM
abbrlink: 933c65ac
date: 2021-09-01 18:43:37
updated: 2021-09-03 18:43:37
subtitle:
---
### 1.1 对象模型
1. 对象组成
    * 对象头
    * 实例数据
    * 对齐填充

    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/RabbitMQ/20210901023420.png)

2. 对象头
    * MarkWord：存储对象运行时的数据，好比 HashCode、锁状态标志、GC分代年龄等，8字节
    * 类型指针：指向类元数据，8字节，压缩后4字节
    * 数组长度（数组对象专有）：4字节

3. 实例数据
    * 8种基本数据类型
    * 实例变量：引用大小 4字节

4. 对齐填充
    * Java对象大小必须是8的倍数，因此需要填充
    * 由于 CPU 进行内存访问时，一次寻址的指针大小是 8 字节，正好也是 L1 缓存行的大小。如果不进行内存对齐，则可能出现跨缓存行的情况，这叫做 缓存行污染

5. 压缩原理
    * 对象大小是8的倍数，即二进制数最后三位都是0
    * 堆中32位，实际可以表示35位（存入时 >> 3, 取出到寄存器时 << 3）(寄存器要为64位)
    * $2^{32}$ = 4G，$2^{35}$ = 4 * 8 = 32G
    * 命令： -XX:+UseCompressedOops

6. 对象大小
    * 空对象：16字节（压缩则有4字节padding）
    * 有属性对象：
        * 对象头 4字节
        * 基本数据类型累加
        * 引用数据类型：4字节（压缩）

### 1.2 分析
1. jol-core库

    ```xml
    <dependency>
        <groupId>org.openjdk.jol</groupId>
        <artifactId>jol-core</artifactId>
        <version>0.16</version>
    </dependency>
    ```

2. 空对象

    ```java
    class ObjA{}

    public class ObjSizeTest {

        public static void main(String[] args) {
            ObjA obj = new ObjA();
            ClassLayout classLayout = ClassLayout.parseInstance(obj);
            System.out.println(classLayout.toPrintable());
        }
    }
    ```
    ```
    com.olivine.ObjA object internals:
    OFF  SZ   TYPE DESCRIPTION               VALUE
    0   8        (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
    8   4        (object header: class)    0xf800c143
    12   4        (object alignment gap)    
    Instance size: 16 bytes
    Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
    ```

3. 有属性对象

    ```java
    class ObjA{
        int id;

        long time;

        Map<String, String> map;
    }

    public class ObjSizeTest {

        public static void main(String[] args) {
            ObjA obj = new ObjA();
            ClassLayout classLayout = ClassLayout.parseInstance(obj);
            System.out.println(classLayout.toPrintable());
        }
    }
    ```
    ```
    com.olivine.ObjA object internals:
    OFF  SZ            TYPE DESCRIPTION               VALUE
    0   8                 (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
    8   4                 (object header: class)    0xf800c143
    12   4             int ObjA.id                   0
    16   8            long ObjA.time                 0
    24   4   java.util.Map ObjA.map                  null
    28   4                 (object alignment gap)    
    Instance size: 32 bytes
    Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
    ```