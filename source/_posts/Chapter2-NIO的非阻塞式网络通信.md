---
title: Chapter2 NIO的非阻塞式网络通信
excerpt: 摘要
tags:
  - java
  - NIO
categories:
  - Java笔记
  - NIO
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: acf49072
date: 2020-12-06 01:34:41
updated: 2020-12-06 01:34:41
subtitle:
---
## 2.1 概述
1. 传统IO
   * 传统的IO流都是阻塞式的
   * 当一个线程调用read()或write()时，该线程被阻塞，直到有一些数据被读取或写入，该线程在此期间不能执行其他任务。
   * 网络通信进行IO操作时，服务器端必须为每个客户端都提供一个独立的线程进行处理，当服务器端需要处理大量客户端时，性能急剧下降。
2. NIO
   * Java NIO是非阻塞模式的。
   * 线程通常将非阻塞IO的空闲时间用于在其他通道上执行IO操作，所以单独的线程可以管理多个输入和输出通道。
   * NIO可以让服务器端使用一个或有限几个线程来同时处理连接到服务器端的所有客户端。
3. NIO网络通信的三个核心
   * 通道（Channel）负责连接
      * `java.nio.channels.Channel`接口
        * `SelectableChannel`实现类（抽象类）
          * `AbstractSelectableChannel`子类（抽象类）
            * `DatagramChannel`：UDP
            * `ServerSocketChannel`：TCP
            * `SocketChannel`：TCP
   * 缓冲区（Buffer）：数据存取
   * 选择器（Selector）：`SelectableChannel`的多路复用器，用于监控`SelectableChannel`的IO状况
## 2.2 选择器（Selector）
### 2.1 什么是选择器
1. 选择器（ Selector） `SelectableChannel`的多路复用器，用于监控`SelectableChannel`的IO状况
2. 利用 Selector可使一个单独的线程管理多个 Channel。 Selector 是非阻塞 IO 的核心

### 2.2 选择器的使用
1. 步骤
   * 创建 Selector ：通过调用 Selector.open() 方法创建一个 Selector
   * 向选择器注册通道： SelectableChannel.register(Selector sel, int ops)
     * ops参数指定对通道的监听事件
     * 监听类型可通过抽象类SelectionKey的四个静态常量表示
       * 读 : `SelectionKey.OP_READ` 
       * 写 : `SelectionKey.OP_WRITE` 
       * 连接 : `SelectionKey.OP_CONNECT` 
       * 接收 : `SelectionKey.OP_ACCEPT` 
   * 若注册时不止监听一个事件，则可以使用“位或”操作符 `|` 连接

2. `SelectionKey`类
   * 表示 `SelectableChannel` 和 `Selector` 之间的注册关系
   * 每次向选择器注册通道时就会选择一个事件(选择键)
   * 常用方法
     * `boolean isReadable()`：检测 Channal 中读事件是否就绪
     * `boolean isWritable()`：检测 Channal 中写事件是否就绪
     * `boolean isConnectable()`：检测 Channel 中连接是否就绪
     * `boolean isAcceptable()`：检测 Channel 中接收是否就绪

3. 阻塞通信案例
   * 客户端上传图片到服务器

      ```java
      public class BlockingNIOTest {
         //客户端
         @Test
         public void client() throws IOException {
            //1. 获取通道
            SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9898));
            FileChannel inChannel = FileChannel.open(Paths.get("img/cat.jpg"), StandardOpenOption.READ);
            //2. 分配指定大小的缓冲区
            ByteBuffer buf = ByteBuffer.allocate(1024);
            //3. 读取本地文件并发送到服务端
            while (inChannel.read(buf)!=-1){
                  buf.flip();
                  sChannel.write(buf);
                  buf.clear();
            }
            //4. 关闭通道
            inChannel.close();
            sChannel.close();
         }

         // 服务端
         @Test
         public void server() throws IOException {
            //1. 获取通道
            ServerSocketChannel ssChannel = ServerSocketChannel.open();
            FileChannel outChannel = FileChannel.open(Paths.get("img/iocopy_cat2.jpg"),StandardOpenOption.WRITE,StandardOpenOption.CREATE);
            // 2. 绑定连接
            ssChannel.bind(new InetSocketAddress(9898));
            // 3. 获取客户端连接通道
            SocketChannel sChannel = ssChannel.accept();
            //4. 分配指定大小的缓冲区
            ByteBuffer buf = ByteBuffer.allocate(1024);
            //5. 接收客户端数据，保存到本地
            while (sChannel.read(buf)!=-1){
                  buf.flip();
                  outChannel.write(buf);
                  buf.clear();
            }
            ssChannel.close();
            sChannel.close();
            outChannel.close();
         }
      }

      ```

   * 客户端上传图片到服务器，服务器接收完毕后发送反馈
      ```java
      public class BlockingNIOTest2 {
         //客户端
         @Test
         public void client() throws IOException {
            SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("114.214.224.116",9898));
            FileChannel inChannel = FileChannel.open(Paths.get("img/dog.png"), StandardOpenOption.READ);
            ByteBuffer buf = ByteBuffer.allocate(1024);
            while (inChannel.read(buf)!=-1){
                  buf.flip();
                  sChannel.write(buf);
                  buf.clear();
            }
            sChannel.shutdownOutput();//没有这一句则会阻塞，因为客户端不知道服务器是否已经接收完毕
            // 接收服务端反馈
            int len;
            while ((len = sChannel.read(buf))!=-1) {
                  buf.flip();
                  System.out.println(new String(buf.array(),0,len));
                  buf.clear();
            }
            inChannel.close();
            sChannel.close();
         }

         //服务端
         @Test
         public void server() throws IOException {
            ServerSocketChannel ssChannel = ServerSocketChannel.open();
            ssChannel.bind(new InetSocketAddress(9898));
            FileChannel outChannel = FileChannel.open(Paths.get("img/copy_dog.png"),StandardOpenOption.WRITE,StandardOpenOption.CREATE);
            ByteBuffer buf = ByteBuffer.allocate(1024);
            SocketChannel sChannel = ssChannel.accept();
            while (sChannel.read(buf)!=-1) {
                  buf.flip();
                  outChannel.write(buf);
                  buf.clear();
            }
            // 发送反馈到客户端
            buf.put("服务端接收数据成功".getBytes());
            buf.flip();
            sChannel.write(buf);
            sChannel.close();
            ssChannel.close();
            outChannel.close();
         }
      }
      ```
4. 非阻塞NIO通信
   ```java
   public class BlockingNIOTest3 {
      //客户端
      @Test
      public void client() throws IOException {
         // 1. 获取通道
         SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9898));
         // 2. 切换非阻塞模式
         sChannel.configureBlocking(false);
         // 3. 分配指定大小的缓冲区
         ByteBuffer buf = ByteBuffer.allocate(1024);
         // 4. 发送数据给服务端，可以像以前一样发送图片，也可以发送文字消息
         buf.put(LocalDateTime.now().toString().getBytes());
         buf.put("\n来自客户端的问候".getBytes());
         buf.flip();
         sChannel.write(buf);
         buf.clear();
         // 5. 关闭通道
         sChannel.close();

      }
      //服务端
      @Test
      public void server() throws IOException {
         // 1. 获取通道
         ServerSocketChannel ssChannel = ServerSocketChannel.open();
         // 2. 切换非阻塞模式
         ssChannel.configureBlocking(false);
         // 3. 绑定
         ssChannel.bind(new InetSocketAddress(9898));
         // 4. 获取选择器
         Selector selector = Selector.open();
         // 5. 将通道注册到选择器，并指定监听接收事件
         ssChannel.register(selector,SelectionKey.OP_ACCEPT);
         // 6. 轮询式地获取选择器上已经准备就绪的事件（select()>0说明已就绪）
         while (selector.select()>0){
               // 7. 获取当前选择器中所有注册的选择键（已就绪的监听事件）
               Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
               while (iterator.hasNext()) {
                  // 8. 获取准备就绪的事件
                  SelectionKey selectionKey = iterator.next();
                  // 9. 判断具体是什么事件准备就绪
                  if (selectionKey.isAcceptable()) {
                     // 10. 若接收就绪，则获取客户端连接
                     SocketChannel sChannel = ssChannel.accept();
                     // 11. 切换非阻塞模式
                     sChannel.configureBlocking(false);
                     // 12. 注册到选择器上
                     sChannel.register(selector,SelectionKey.OP_READ); // register()方法会返回一个SelectionKey对象，这个对象包含了一些你感兴趣的属性
                  }else if (selectionKey.isReadable()) { // 读就绪，开始读取
                     // 13. 获取当前选择器上读就绪状态的通道
                     SocketChannel sChannel = (SocketChannel) selectionKey.channel();
                     // 14. 读取数据
                     ByteBuffer buf = ByteBuffer.allocate(1024);
                     int len = 0;
                     while ((len=sChannel.read(buf))>0) {
                           buf.flip();
                           System.out.println(new String(buf.array(),0,len));
                           buf.clear();
                     }
                  }
                  // 15. 取消选择键(已经处理过的事件，就应该取消掉了)
                  iterator.remove();
               }
         }
      }
   }
   ```

5. UDP通信（DatagramChannel）
   ```java
   public class NonBlockingNIOTest {
      // 客户端
      @Test
      public void send() throws IOException {
         DatagramChannel datagramChannel = DatagramChannel.open();
         datagramChannel.configureBlocking(false);
         ByteBuffer buf = ByteBuffer.allocate(1024);
         Scanner scanner = new Scanner(System.in);
         while (scanner.hasNext()) {
               String str = scanner.next();
               buf.put((LocalDateTime.now().toString()+":\n"+str).getBytes());
               buf.flip();
               datagramChannel.send(buf,new InetSocketAddress("127.0.0.1",9898));
               buf.clear();
         }
         datagramChannel.close();
      }

      // 服务端
      @Test
      public void receive() throws IOException {
         DatagramChannel datagramChannel = DatagramChannel.open();
         datagramChannel.configureBlocking(false);
         datagramChannel.bind(new InetSocketAddress(9898));
         Selector selector = Selector.open();
         datagramChannel.register(selector, SelectionKey.OP_READ);
         while (selector.select()>0) {
               Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
               while (iterator.hasNext()) {
                  SelectionKey selectionKey = iterator.next();
                  if (selectionKey.isReadable()) {
                     ByteBuffer buf = ByteBuffer.allocate(1024);
                     datagramChannel.receive(buf);
                     buf.flip();
                     System.out.println(new String(buf.array(),0,buf.limit()));
                     buf.clear();
                  }
               }
               iterator.remove();
         }
      }
   }
   ```

## 2.3 管道
1. 说明
   * NIO 管道是2个线程之间的单向数据连接
   * Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取
2. 示例

   ```java
   // 1. 获取管道
   Pipe pipe = Pipe.open();
   // 2. 将缓冲区中的数据写入管道
   ByteBuffer buf = ByteBuffer.allocate(1024);
   Pipe.SinkChannel sinkChannel = pipe.sink();
   buf.put("通过单向管道发送数据".getBytes());
   buf.flip();
   sinkChannel.write(buf);
   // 3. 读取缓冲区中的数据
   Pipe.SourceChannel sourceChannel = pipe.source();
   buf.flip();
   int len = sourceChannel.read(buf);
   System.out.println(new String(buf.array(),0,len));
   ```

