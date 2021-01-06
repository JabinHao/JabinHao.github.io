---
title: Chapter1 通道与缓冲区
excerpt: 摘要
tags:
  - java
  - NIO
categories:
  - Java笔记
  - NIO
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 659b75d3
date: 2020-12-06 01:32:06
updated: 2020-12-06 01:32:06
subtitle:
---
## 1.1 缓冲区
### 1.1.1 概述
1. 定义  
   缓冲区（Buffer）：在 Java NIO 中负责数据的存取。缓冲区就是数组。用于存储不同数据类型的数据

2. 根据数据类型不同（boolean 除外），提供了相应类型的缓冲区：
   * `ByteBuffer`
   * `CharBuffer`
   * `ShortBuffer`
   * `IntBuffer`
   * `LongBuffer`
   * `FloatBuffer`
   * `DoubleBuffer`

    上述缓冲区的管理方式几乎一致，通过 `allocate()` 获取缓冲区:
    ```java
    ByteBuffer buf = ByteBuffer.allocate(1024);
    ```

### 1.1.2 缓冲区中的四个核心属性：
1. `capacity` : 容量，表示缓冲区中最大存储数据的容量。一旦声明不能改变。
2. `limit` : 界限，表示缓冲区中可以操作数据的大小。（limit 后数据不能进行读写）
3. `position` : 位置，表示缓冲区中正在操作数据的位置。
4. `mark` : 标记，表示记录当前 position 的位置。可以通过 `reset()` 恢复到 `mark` 的位置  
0 <= mark <= position <= limit <= capacity

### 1.1.3 方法
1. 常用方法
   * `put()` : 存入数据到缓冲区中
   * `get()` : 获取缓冲区中的数据
   * `flip()`：切换读取模式
   * `rewind()`：重复读(将指针回拨到起始位置)
   * `clear()`：清空缓冲区（数据还在，但处于被遗忘状态）
   * `reset()`：回到mark的位置
2. 代码示例
    ```java
    public void test1(){
        String str = "abcde";
        // 1. 分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        System.out.println("----------------allocate()-----------------");
        System.out.println(buf.position());//0
        System.out.println(buf.limit());//1024
        System.out.println(buf.capacity());//1024

        // 2. 使用put()存入数据
        buf.put(str.getBytes());
        System.out.println("--------------------put()-------------------");
        System.out.println(buf.position());//5
        System.out.println(buf.limit());//1024
        System.out.println(buf.capacity());//1024

        // 3. 切换读取数据模式
        buf.flip();
        System.out.println("--------------------flip()-------------------");
        System.out.println(buf.position());//0
        System.out.println(buf.limit());//5
        System.out.println(buf.capacity());//1024

        // 4. 利用get()读取数据
        byte[] dst = new byte[buf.limit()];
        buf.get(dst);
        System.out.println("--------------------get()-------------------");
        System.out.println(buf.position()); //5
        System.out.println(buf.limit()); //5
        System.out.println(buf.capacity()); //1024
        System.out.println(new String(dst,0,dst.length)); // abcde

        // 5. rewind() ：可重复读
        buf.rewind();
        System.out.println("--------------------rewind()-------------------");
        System.out.println(buf.position()); //0
        System.out.println(buf.limit()); //5
        System.out.println(buf.capacity()); //1024

        // 6. clear()：清空缓冲区（数据还在，但出于被遗忘状态）
        buf.clear();
        System.out.println("--------------------clear()-------------------");
        System.out.println(buf.position()); //0
        System.out.println(buf.limit()); //1024
        System.out.println(buf.capacity()); //1024
        System.out.println((char) buf.get()); //a
    }
    ```
    ```java
    public void test2(){
        String str = "abcde";
        ByteBuffer buf = ByteBuffer.allocate(1024);
        buf.put(str.getBytes());
        buf.flip();
        byte[] dst = new byte[buf.limit()];
        buf.get(dst,0,2);
        System.out.println(buf.position());//2
        buf.mark(); //标记
        buf.get(dst,2,2);
        System.out.println(buf.position());//4
        buf.reset(); // 恢复到 mark 的位置
        System.out.println(buf.position()); //2
        if (buf.hasRemaining()){
            System.out.println(buf.remaining());//3
        }
    }
    ```

### 1.1.4 直接缓冲区与非直接缓冲区：
1. 非直接缓冲区：通过 `allocate()` 方法分配缓冲区，将缓冲区建立在 JVM 的内存中
2. 直接缓冲区：通过 `allocateDirect()` 方法分配直接缓冲区，将缓冲区建立在物理内存中。可以提高效率
3. `isDirect()`方法：判断是否为直接缓冲区
4. 代码
    ```java
    public void teat3(){
        ByteBuffer buf = ByteBuffer.allocateDirect(1024);
        System.out.println(buf.isDirect()); //true
    }
    ```

## 1.2 通道
### 1.2.1 概述
1. 简介
   * 通道Channel是一个接口，定义在`java.nio.channels`包下
   * 通道用于源节点与目标节点的连接，本身不能直接访问数据
   * IO中流可以访问+传输数据，NIO中，缓冲区读取数据，通道负责传输

2. Channel的实现类
   * `FileChannel`：读取、写入、映射和操作文件通道
   * `DatagramChannel`：通过UDP读写网络中的数据通道
   * `SocketChannel`：通过TCP读写网络中的数据
   * `ServerSocketChannel`：监听新进来的TCP连接，对每个新进来的连接都会创建一个 SocketChannel
3. `FileChannel`常用方法
   * `public abstract int read(ByteBuffer dst)`：从Channel中读取数据到ByteBuffer
   * `public final long read(ByteBuffer[] dsts)`：将Channel中的数据“分散”到`ByteBuffer[]`
   * `public abstract int write(ByteBuffer src)`：将`ByteBuffer`中的数据写入到Channel
   * `public final long write(ByteBuffer[] srcs)`：将`ByteBuffer[]`中的数据“聚集”到Channel

### 1.2.2 获取通道
1. 通过调用支持通道的对象的`getChannel()`方法，支持通道的类有：
   * 本地IO
      * `FileInputStream`
      * `FileOutputStream`
      * `RandomAccessFile`
    * 网络IO
      * `Socket`
      * `ServerSocket`
      * `DatagramSocket`
2. JDK1.7中的NIO.2针对各个通道(即Channel实现类)提供了静态方法 `open()`
3. JDK1.7中的NIO.2的Files工具类的静态方法 `newByteChannel()`

### 1.2.3 使用通道
1. 使用非直接缓冲区
   ```java
    FileInputStream fis = new FileInputStream("img/cat.jpg");
    FileOutputStream fos = new FileOutputStream("cat.jpg");

    // 1. 获取通道
    FileChannel inChannel = fis.getChannel();
    FileChannel outChannel = fos.getChannel();

    // 2. 分配指定大小的缓冲区
    ByteBuffer buf = ByteBuffer.allocate(1024);

    // 3. 将通道中的数据存入缓冲区
    while (inChannel.read(buf) != -1){
        buf.flip(); // 切换模式
        outChannel.write(buf); // 将缓冲区中的数据写入
        buf.clear(); // 清空缓冲区
    }
    // 4. 关闭通道（省略try-catch）
    outChannel.close();
    inChannel.close();
    fos.close();
    fis.close();
   ```
2. 使用直接缓冲区
    ```java
    FileChannel inChannel = FileChannel.open(Paths.get("img/cat.jpg"), StandardOpenOption.READ);
    FileChannel outChannel = FileChannel.open(Paths.get("img/copy_cat.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE, StandardOpenOption.READ);
    
    // 内存映射文件
    MappedByteBuffer inMappedBuf = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
    MappedByteBuffer outMappedBuf = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());

    // 直接对缓冲区数据进行读写操作
    byte[] dst = new byte[inMappedBuf.limit()];
    inMappedBuf.get(dst);
    outMappedBuf.put(dst);

    inChannel.close();
    outChannel.close();   
    ```

### 1.2.4 通道间的数据传输
1. 方法
   * `public abstract long transferTo(long position, long count,WritableByteChannel target)`：将数据从源通道传输到其他Channel中
   * `public abstract long transferFrom(ReadableByteChannel src, long position, long count)` ：将数据从源通道传输到其他Channel中

2. 示例（直接缓冲区）
    ```java
    FileChannel inChannel = FileChannel.open(Paths.get("img/cat.jpg"), StandardOpenOption.READ);
    FileChannel outChannel = FileChannel.open(Paths.get("img/copy2_cat.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE, StandardOpenOption.READ);

    inChannel.transferTo(0,inChannel.size(),outChannel);

    inChannel.close();
    outChannel.close();
    ```

### 1.2.5 分散与聚集
1. 概念
   * 分散读取（Scattering Reads）是指从Channel中读取的数据“分散”到多个Buffer中。
   * 聚集写入（Gathering Writes）是指将多个Buffer中的数据“聚集”到Channel
2. 示例
    ```java
    RandomAccessFile raf1 = new RandomAccessFile("files/1.txt","rw");
    // 1. 获取通道
    FileChannel channel1 = raf1.getChannel();

    // 分配缓冲区
    ByteBuffer buf1 = ByteBuffer.allocate(100);
    ByteBuffer buf2 = ByteBuffer.allocate(1023);

    // 3. 分散读取
    ByteBuffer[] bufs = {buf1,buf2};
    channel1.read(bufs);

    for (ByteBuffer buf : bufs) buf.flip();
    System.out.println(new String(bufs[0].array(),0,bufs[0].limit()));
    System.out.println(new String(bufs[1].array(),0,bufs[1].limit()));

    // 4. 聚集写入
    RandomAccessFile raf2 = new RandomAccessFile("files/5.txt","rw");
    FileChannel channel2 = raf2.getChannel();
    channel2.write(bufs);
    ```

### 1.2.6 字符集 Charset
1. 概念
   * 编码：字符串 -> 字节数组
   * 字节数组 -> 字符串

2. 获取所有格式
    ```java
    Map<String, Charset> map = Charset.availableCharsets();
    Set<Map.Entry<String, Charset>> set = map.entrySet();

    for (Map.Entry<String, Charset> entry : set) {
        System.out.println(entry.getKey()+ "=" + entry.getValue());
    }
    ```

3. 示例
    ```java
    Charset cs1 = Charset.forName("GBK");
    Charset cs12= StandardCharsets.UTF_8;

    // 获取编码器
    CharsetEncoder ce = cs1.newEncoder();
    // 获取解码器
    CharsetDecoder cd = cs1.newDecoder();
    //创建缓冲区
    CharBuffer cBuf = CharBuffer.allocate(1024);
    cBuf.put("尚硅谷威武！");
    cBuf.flip();
    //编码
    ByteBuffer bBuf = ce.encode(cBuf);
    for (int i = 0; i < 12; i++) System.out.println(bBuf.get());
    //解码
    bBuf.flip();
    CharBuffer cBuf2 = cd.decode(bBuf);
    System.out.println(cBuf2.toString());
    //换种解码方式
    bBuf.flip();
    CharBuffer cBuf3 = cs12.decode(bBuf);
    System.out.println(cBuf3.toString());
    ```


