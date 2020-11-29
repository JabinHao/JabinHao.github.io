---
title: Chapter13 网络编程
excerpt: IP和端口号、TCP、UDP
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: ae2dd0d4
date: 2020-11-01 18:03:47
updated: 2020-11-01 18:03:47
subtitle:
---
## 13.1 网络编程概述
1. 网络编程的目的：  
   直接或间接地通过网络协议与其它计算机实现数据交换，进行通讯。
2. 网络编程中有两个主要的问题：
 * 如何准确地定位网络上一台或多台主机；定位主机上的特定的应用
 * 找到主机后如何可靠高效地进行数据传输

## 13.2 网络通信要素概述
1. IP和端口号
2. 网络通信协议: TCP/IP参考模型（应用层、传输层、网络层、物理+数据链路层）

## 13.3 通信要素1：IP和端口号
### 1. IP相关
1. IP:唯一的标识 Internet 上的计算机（通信实体）
2. 在Java中使用`InetAddress`类代表IP
3. IP分类：IPv4 和 IPv6 ; 万维网 和 局域网
4. 域名:   www.baidu.com   www.mi.com  www.sina.com  www.jd.com  www.vip.com
5. 本地回路地址：127.0.0.1 对应着：`localhost`
6. 如何实例化InetAddress：
   * 实例化 `getByName(String host)`、 `getLocalHost()`
   * 常用方法：`getHostName()` 、`getHostAddress()`

### 2. 端口号
1. 端口号：正在计算机上运行的进程。
   * 要求：不同的进程有不同的端口号
   * 范围：被规定为一个 16 位的整数 0~65535。
2. 端口分类：
   * 公认端口： 0~1023。被预先定义的服务通信占用（如： HTTP占用端口80， FTP占用端口21， Telnet占用端口23）
   * 注册端口： 1024~49151。分配给用户进程或应用程序。（如： Tomcat占用端口8080， MySQL占用端口3306， Oracle占用端口1521等） 。
   * 动态/私有端口： 49152~65535
3. 端口号与IP地址的组合得出一个网络套接字：Socket

## 13.4 通信要素2：网络协议
**TCP/IP协议簇**
1. 传输层协议中有两个非常重要的协议：
   * 传输控制协议TCP(Transmission Control Protocol)
   * 用户数据报协议UDP(User Datagram Protocol)
2. TCP协议：
   * 使用TCP协议前，须先建立TCP连接，形成传输数据通道
   * 传输前，采用“三次握手” 方式，点对点通信， 是可靠的
   * TCP协议进行通信的两个应用进程：客户端、 服务端。
   * 在连接中可进行大数据量的传输
   * 传输完毕，需释放已建立的连接， 效率低
3. UDP协议：
   * 将数据、源、目的封装成数据包， 不需要建立连接
   * 每个数据报的大小限制在64K内
   * 发送不管对方是否准备好，接收方收到也不确认， 故是不可靠的
   * 可以广播发送
   * 发送数据结束时无需释放资源，开销小，速度快

## 13.5 TCP网络编程
1. 客户端
   * 创建 Socket
   * 打开连接到 Socket 的输入/出流： 使用 getInputStream()方法获得输入流，getOutputStream()方法获得输出流
   * 按照一定的协议对 Socket 进行读/写操作
   * 关闭 Socket： 断开客户端到服务器的连接，释放线路
2. 服务器
   * 调用 ServerSocket(int port) 
   * 调用 accept()
   * 调用 该Socket类对象的 getOutputStream() 和 getInputStream ()
   * 关闭ServerSocket和Socket对象
3. 代码示例
   ```java
   //1. 客户端发送信息给服务端，服务端将数据显示在控制台上
   //客户端
   Socket socket = null;
   OutputStream os = null;
   try {
      //1.创建Socket对象，指明服务器端的ip和端口号
      InetAddress inet = InetAddress.getByName("192.168.14.100");
      socket = new Socket(inet,8899);
      //2.获取一个输出流，用于输出数据
      os = socket.getOutputStream();
      //3.写出数据的操作
      os.write("你好，我是客户端mm".getBytes());
   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      //4.资源的关闭
      if(os != null){
          try {
              os.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      if(socket != null){
          try {
              socket.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
   }
   //服务器
   ServerSocket ss = null;
   Socket socket = null;
   InputStream is = null;
   ByteArrayOutputStream baos = null;
   try {
      //1.创建服务器端的ServerSocket，指明自己的端口号
      ss = new ServerSocket(8899);
      //2.调用accept()表示接收来自于客户端的socket
      socket = ss.accept();
      //3.获取输入流
      is = socket.getInputStream();
      //4.读取输入流中的数据
      baos = new ByteArrayOutputStream();
      byte[] buffer = new byte[5];
      int len;
      while((len = is.read(buffer)) != -1){
          baos.write(buffer,0,len);
      }
      System.out.println(baos.toString());
      System.out.println("收到了来自于：" + socket.getInetAddress().getHostAddress() + "的数据");

   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      if(baos != null){
          //5.关闭资源
          try {
              baos.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
       // 下面三个同上
       is.close();
       socket.close();
       ss.close();
   ```
   ```java
   //2. 客户端发送文件给服务端，服务端将文件保存在本地,并返回“发送成功”给客户端。
   Socket socket = new Socket(InetAddress.getByName("127.0.0.1"),9090);
   //2.
   OutputStream os = socket.getOutputStream();
   //3.
   FileInputStream fis = new FileInputStream(new File("beauty.jpg"));
   //4.
   byte[] buffer = new byte[1024];
   int len;
   while((len = fis.read(buffer)) != -1){
      os.write(buffer,0,len);
   }
   //关闭数据的输出
    socket.shutdownOutput();

    //5.接收来自于服务器端的数据，并显示到控制台上
    InputStream is = socket.getInputStream();
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    byte[] bufferr = new byte[20];
    int len1;
    while((len1 = is.read(buffer)) != -1){
    baos.write(buffer,0,len1);
    }

   System.out.println(baos.toString());
   //6.try-catch
   fis.close();
   os.close();
   socket.close();
   baos.close();
   }

   //server
   //这里涉及到的异常，应该使用try-catch-finally处理
   //1.
   ServerSocket ss = new ServerSocket(9090);
   //2.
   Socket socket = ss.accept();
   //3.
   InputStream is = socket.getInputStream();
   //4.
   FileOutputStream fos = new FileOutputStream(new File("beauty1.jpg"));
   //5.
   byte[] buffer = new byte[1024];
   int len;
   while((len = is.read(buffer)) != -1){
      fos.write(buffer,0,len);
   }
    //6.服务器端给予客户端反馈
    OutputStream os = socket.getOutputStream();
    os.write("发送成功！".getBytes());
   //7.
   fos.close();
   is.close();
   socket.close();
   ss.close();
   os.close()
   ```

## 13.6 UDP网络编程
1. 流程
   1. DatagramSocket与DatagramPacket
   2. 建立发送端，接收端
   3. 建立数据包
   4. 调用Socket的发送、 接收方法
   5. 关闭Socket
2. 代码示例
   ```java
   //发送端
   DatagramSocket socket = new DatagramSocket();



   String str = "我是UDP方式发送的导弹";
   byte[] data = str.getBytes();
   InetAddress inet = InetAddress.getLocalHost();
   DatagramPacket packet = new DatagramPacket(data,0,data.length,inet,9090);

   socket.send(packet);

   socket.close();

   //接收端
   DatagramSocket socket = new DatagramSocket(9090);

   byte[] buffer = new byte[100];
   DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length);

   socket.receive(packet);

   System.out.println(new String(packet.getData(),0,packet.getLength()));

   socket.close();
   ```

## 13.7 URL编程
1. 概述  
  URL(Uniform Resource Locator)：统一资源定位符，它表示 Internet 上某一资源的地址。
2. 格式：
   * http://localhost:8080/examples/beauty.jpg?username=Tom
   *  协议   主机名    端口号  资源地址           参数列表
3. 代码示例
   ```java

   ```