---
title: Chapter12 IO流
excerpt: File类的使用、IO流使用、NIO.2中Path、Paths、File类的使用
tags:
  - java
categories:
  - Java笔记
  - java基础
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: aa5fd161
date: 2020-10-31 20:12:32
updated: 2020-11-01 17:40:32
subtitle:
---
## 12.1 `File`类的使用
### 1. 概述
1. `File`类的一个对象，代表一个文件或一个文件目录(俗称：文件夹)
2. `File`类声明在`java.io`包下
3. `File`类中涉及到关于文件或文件目录的创建、删除、重命名、修改时间、文件大小等方法，并未涉及到写入或读取文件内容 的操作。如果需要读取或写入文件内容，必须使用IO流来完成。
4. 后续`File`类的对象常会作为参数传递到流的构造器中，指明读取或写入的"终点".

### 2.如何创建`File`类的实例
`File`类构造器：
```java
File(String filePath)
File(String parentPath,String childPath)
File(File parentFile,String childPath)
```

### 3. File类常用方法
1. 常用文件方法——获取
   ```java
   public String getAbsolutePath()：获取绝对路径
   public String getPath() ：获取路径
   public String getName() ：获取名称
   public String getParent()：获取上层文件目录路径。若无，返回null
   public long length() ：获取文件长度（即：字节数）。不能获取目录的长度。
   public long lastModified() ：获取最后一次的修改时间，毫秒值
   ```
2. 文件目录方法
   ```java
   public String[] list() ：获取指定目录下的所有文件或者文件目录的名称数组
   public File[] listFiles() ：获取指定目录下的所有文件或者文件目录的File数组
   ```
3. 文件方法——判断
   ```java
   public boolean renameTo(File dest):把文件重命名为指定的文件路径
   ```
    要想保证返回true,需要当前文件在硬盘中是存在的，且`dest`不能在硬盘中存在。
   ```java
   public boolean isDirectory()：判断是否是文件目录
   public boolean isFile() ：判断是否是文件
   public boolean exists() ：判断是否存在
   public boolean canRead() ：判断是否可读
   public boolean canWrite() ：判断是否可写
   public boolean isHidden() ：判断是否隐藏
   ```
4. 文件方法——创建与删除
   ```java
   public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false
   public boolean mkdir() ：创建文件目录。如果此文件目录存在，则不创建，返回false
   public boolean mkdirs() ：递归创建文件目录。如果此文件目录存在，则不创建，返回false
   public boolean delete()：删除文件或者文件夹，目录下不能有子目录或文件
   ```

## 12.2 IO流原理及流的分类
### 1.流的分类：
1. 操作数据单位：字节流、字符流
2. 数据的流向：输入流、输出流
3. 流的角色：节点流、处理流
   (抽象基类)|字节流|字符流
   :-:|:-:|:-:
    输入流|InputStream|Reader
    输出流|OutputStream|Writer
### 2. 流的体系结构
抽象基类|节点流（或文件流）| 缓冲流（处理流的一种）
:-:|:-:|:-:
InputStream|FileInputStream   (read(byte[] buffer))     |   BufferedInputStream (read(byte[] buffer))
OutputStream |   FileOutputStream  (write(byte[] buffer,0,len) | BufferedOutputStream (write(byte[] buffer,0,len) / flush()
Reader       |   FileReader (read(char[] cbuf))     |     BufferedReader (read(char[] cbuf) / readLine())
Writer      |    FileWriter (write(char[] cbuf,0,len)     |      BufferedWriter (write(char[] cbuf,0,len) / flush()

## 12.3节点流
### 1. `FileReader`、`FileWriter` 基本操作
1. `FileReader`基本操作
   ```java
   FileReader fr = null;
   try {
      //1.实例化File类的对象，指明要操作的文件
      File file = new File("hello.txt");//相较于当前Module
      //2.提供具体的流
      fr = new FileReader(file);

      //3.数据的读入
      //read():返回读入的一个字符。如果达到文件末尾，返回-1
      int data;
      while((data = fr.read()) != -1){
          System.out.print((char)data);
      }
   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      //4.流的关闭操作
      if(fr != null){
          try {
              fr.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
   }
   ```
2. `FileReader` 中使用 `read(char[] cbuf)`读入数据  
   对`read()`操作升级：使用`read`的重载方法
   ```java
   //3.读入的操作
   //read(char[] cbuf):返回每次读入cbuf数组中的字符的个数。如果达到文件末尾，返回-1
   char[] cbuf = new char[5];
   int len;
   while((len = fr.read(cbuf)) != -1){
       //方式一：
       //错误的写法, cbuf.length恒等于5
   //                for(int i = 0;i < cbuf.length;i++){
   //                    System.out.print(cbuf[i]);
   //                }
       //正确的写法
      for(int i = 0;i < len;i++){
          System.out.print(cbuf[i]);
      }
       //方式二：
       //错误的写法,对应着方式一的错误的写法
   //                String str = new String(cbuf);
   //                System.out.print(str);
       //正确的写法
       String str = new String(cbuf,0,len);
       System.out.print(str);
   ```
3. `FileWriter` 写出数据的操作
   ```java
   FileWriter fw = null;
   try {
      //1.提供File类的对象，指明写出到的文件
      File file = new File("hello1.txt");

      //2.提供FileWriter的对象，用于数据的写出
      fw = new FileWriter(file,false);

      //3.写出的操作
      fw.write("I have a dream!\n");
      fw.write("you need to have a dream!");
   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      //4.流资源的关闭
      if(fw != null){
          try {
              fw.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
   }
   ```
4. 使用 `FileReader` 和 `FileWriter` 实现文本文件的复制
   ```java
   FileReader fr = null;
   FileWriter fw = null;
   try {
      //1.创建File类的对象，指明读入和写出的文件
      File srcFile = new File("hello.txt");
      File destFile = new File("hello2.txt");

      //2.创建输入流和输出流的对象
      fr = new FileReader(srcFile);
      fw = new FileWriter(destFile);

      //3.数据的读入和写出操作
      char[] cbuf = new char[5];
      int len;//记录每次读入到cbuf数组中的字符的个数
      while((len = fr.read(cbuf)) != -1){
          //每次写出len个字符
          fw.write(cbuf,0,len);
      }
   } catch (IOException e) {
      e.printStackTrace();
   } finally {

      //4.关闭流资源
      try {
          if(fw != null)
              fw.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
      try {
          if(fr != null)
              fr.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
   }
   ```

### 2. `FileInputStream` 和 `FileOutputStream` 读写非文本文件
1. `FileInputStream`读取文件
   ```java
   FileInputStream fis = null;
   try {
      //1. 造文件
      File file = new File("hello.txt");

      //2.造流
      fis = new FileInputStream(file);

      //3.读数据
      byte[] buffer = new byte[5];
      int len;//记录每次读取的字节的个数
      while((len = fis.read(buffer)) != -1){

          String str = new String(buffer,0,len);
          System.out.print(str);

      }
   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      if(fis != null){
          //4.关闭资源
          try {
              fis.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
   }
   ```
2. 使用字节流实现对图片的复制操作
   ```java
   FileInputStream fis = null;
   FileOutputStream fos = null;
   try {
      //
      File srcFile = new File("pic.jpg");
      File destFile = new File("pic2.jpg");

      //
      fis = new FileInputStream(srcFile);
      fos = new FileOutputStream(destFile);

      //复制的过程
      byte[] buffer = new byte[1024];
      int len;
      while((len = fis.read(buffer)) != -1){
          fos.write(buffer,0,len);
      }

   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      if(fos != null){
          //
          try {
              fos.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      if(fis != null){
          try {
              fis.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
   }
   ```

### 3. 结论：
1. 使用字节流FileInputStream处理文本文件，可能出现乱码。
2. 对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
3. 对于非文本文件(.jpg,.mp3,.mp4,.avi,.doc,.ppt,...)，使用字节流处理

## 12.4 处理流之一：缓冲流的使用
1. 缓冲流：  
   ```java
   BufferedInputStream
   BufferedOutputStream
   BufferedReader
   BufferedWriter
   ```
2. `BufferedInputStream`与`BufferedOutputStream`实现非文本文件的复制 
   ```java
   BufferedInputStream bis = null;
   BufferedOutputStream bos = null;

   try {
      //1.造文件
      File srcFile = new File("pic.jpg");
      File destFile = new File("pic3.jpg");
      //2.造流
      //2.1 造节点流
      FileInputStream fis = new FileInputStream((srcFile));
      FileOutputStream fos = new FileOutputStream(destFile);
      //2.2 造缓冲流
      bis = new BufferedInputStream(fis);
      bos = new BufferedOutputStream(fos);

      //3.复制的细节：读取、写入
      byte[] buffer = new byte[10];
      int len;
      while((len = bis.read(buffer)) != -1){
          bos.write(buffer,0,len);

   //       bos.flush();//刷新缓冲区

      }
   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      //4.资源关闭
      //要求：先关闭外层的流，再关闭内层的流
      if(bos != null){
          try {
              bos.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
      if(bis != null){
          try {
              bis.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
   //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
   //    fos.close();
   //    fis.close();
   }
   ```
3. `BufferedReader`、`BufferedWriter`实现文件复制
   ```java
   BufferedReader br = null;
   BufferedWriter bw = null;
   try {
      //创建文件和相应的流
      br = new BufferedReader(new FileReader(new File("dbcp.txt")));
      bw = new BufferedWriter(new FileWriter(new File("dbcp1.txt")));

      //读写操作
      //方式一：使用char[]数组
   //   char[] cbuf = new char[1024];
   //   int len;
   //   while((len = br.read(cbuf)) != -1){
   //      bw.write(cbuf,0,len);
   //            }

      //方式二：使用String
      String data;
      while((data = br.readLine()) != null){
          //方法一：
   //       bw.write(data + "\n");//data中不包含换行符
          //方法二：
          bw.write(data);//data中不包含换行符
          bw.newLine();//提供换行的操作

      }


   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      //关闭资源
      if(bw != null){

          try {
              bw.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      if(br != null){
          try {
              br.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
   }
   ```

## 12.5 处理流之二：转换流的使用
1. 转换流：属于字符流
   * `InputStreamReader`：将一个字节的输入流转换为字符的输入流
   * `OutputStreamWriter`：将一个字符的输出流转换为字节的输出流
2. 作用：提供字节流与字符流之间的转换
3. 编码与解码
   * 解码：字节、字节数组  --->字符数组、字符串
   * 编码：字符数组、字符串 ---> 字节、字节数组
4. 字符集
   * ASCII：美国标准信息交换码。用一个字节的7位可以表示。
   * ISO8859-1：拉丁码表。欧洲码表，用一个字节的8位表示。
   * GB2312：中国的中文编码表。最多两个字节编码所有字符
   * GBK：中国的中文编码表升级，融合了更多的中文文字符号。最多两个字节编码
   * Unicode：国际标准码，融合了目前人类使用的所有字符。为每个字符分配唯一的字符码。所有的文字都用两个字节来表示。
   * UTF-8：变长的编码方式，可用1-4个字节来表示一个字符。
5. 综合使用`InputStreamReader`和`OutputStreamWriter`
   ```java
   //1.造文件、造流
   File file1 = new File("dbcp.txt");
   File file2 = new File("dbcp_gbk.txt");

   FileInputStream fis = new FileInputStream(file1);
   FileOutputStream fos = new FileOutputStream(file2);

   InputStreamReader isr = new InputStreamReader(fis,"utf-8");
   OutputStreamWriter osw = new OutputStreamWriter(fos,"gbk");

   //2.读写过程
   char[] cbuf = new char[20];
   int len;
   while((len = isr.read(cbuf)) != -1){
    //  String str = new String(cbuf,0,len);
    //    System.out.print(str);
      osw.write(cbuf,0,len);
   }

   //3.关闭资源
   isr.close();
   osw.close();
   ```

## 12.6 标准输入、输出流
1. 标准的输入、输出流
   * System.in:标准的输入流，默认从键盘输入
   * System.out:标准的输出流，默认从控制台输出
   * System类的setIn(InputStream is)
   * setOut(PrintStream ps)方式重新指定输入和输出的流。
  
   使用输入流从键盘读取字符串：
   ```java
   BufferedReader br = null;
   try {
      InputStreamReader isr = new InputStreamReader(System.in);
      br = new BufferedReader(isr);

      while (true) {
          System.out.println("请输入字符串：");
          String data = br.readLine();
          if ("e".equalsIgnoreCase(data) || "exit".equalsIgnoreCase(data)) {
              System.out.println("程序结束");
              break;
          }
      }
   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      if (br != null) {
          try {
              br.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
   }
   ```

## 12.7 打印流
1. `PrintStream` 和 `PrintWriter`
2. 提供了一系列重载的 `print()` 和 `println()`
3. 利用打印流将输出存到文件
   ```java
   PrintStream ps = null;
   try {
      FileOutputStream fos = new FileOutputStream(new File("D:\\IO\\text.txt"));
      // 创建打印输出流,设置为自动刷新模式(写入换行符或字节 '\n' 时都会刷新输出缓冲区)
      ps = new PrintStream(fos, true);
      if (ps != null) {// 把标准输出流(控制台输出)改成文件
          System.setOut(ps);
      }

      for (int i = 0; i <= 255; i++) { // 输出ASCII字符
          System.out.print((char) i);
          if (i % 50 == 0) { // 每50个数据一行
              System.out.println(); // 换行
          }
      }

   } catch (FileNotFoundException e) {
      e.printStackTrace();
   } finally {
      if (ps != null) {
          ps.close();
      }
   }
   ```

## 12.8 数据流
1. `DataInputStream` 和 `DataOutputStream`
2. 作用：用于读取或写出基本数据类型的变量或字符串
3. 将内存中的字符串、基本数据类型的变量写出到文件中
   ```java
   //1.
   DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.txt"));
   //2.
   dos.writeUTF("刘建辰");
   dos.flush();//刷新操作，将内存中的数据写入文件
   dos.writeInt(23);
   dos.flush();
   dos.writeBoolean(true);
   dos.flush();
   //3.
   dos.close();
   ```
4. 将文件中存储的基本数据类型变量和字符串读取到内存中，保存在变量中。  
      注意点：读取不同类型的数据的顺序要与当初写入文件时，保存的数据的顺序一致！
   ```java
   //1.
   DataInputStream dis = new DataInputStream(new FileInputStream("data.txt"));
   //2.
   String name = dis.readUTF();
   int age = dis.readInt();
   boolean isMale = dis.readBoolean();

   System.out.println("name = " + name);
   System.out.println("age = " + age);
   System.out.println("isMale = " + isMale);

   //3.
   dis.close();
   ```

## 12.9 处理流之六：对象流
### 1. 对象流概述
1. `ObjectInputStream` 和 `ObjectOutputStream`
2. 作用：用于存储和读取基本数据类型数据或对象的处理流。它的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。
3. 要想一个java对象是可序列化的，需要满足相应的要求。
   * 需要实现接口：`Serializable`
   * 当前类提供一个全局常量：`serialVersionUID`，用来表明类的不同版本间的兼容性。
   * 除了当前`Person`类需要实现`Serializable`接口之外，还必须保证其内部所有属性也必须是可序列化的。（默认情况下，基本数据类型可序列化）
4. `ObjectOutputStream`和`ObjectInputStream`不能序列化`static`和`transient`修饰的成员变量
5. 序列化机制：  
    对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。当其它程序获取了这种二进制流，就可以恢复成原来的Java对象。

### 2. 代码示例
1. 序列化
   * 将内存中的java对象保存到磁盘中或通过网络传输出去
   * 使用ObjectOutputStream实现

   ```java
   ObjectOutputStream oos = null;

   try {
      //1.
      oos = new ObjectOutputStream(new FileOutputStream("object.dat"));
      //2.
      oos.writeObject(new String("我爱北京天安门"));
      oos.flush();//刷新操作

      oos.writeObject(new Person("王铭",23));
      oos.flush();

   } catch (IOException e) {
      e.printStackTrace();
   } finally {
      if(oos != null){
          //3.
          try {
              oos.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
   }
   ```

2. 反序列化
   * 将磁盘文件中的对象还原为内存中的一个java对象
   * 使用ObjectInputStream来实现

   ```java
   ObjectInputStream ois = null;
   try {
      ois = new ObjectInputStream(new FileInputStream("object.dat"));

      Object obj = ois.readObject();
      String str = (String) obj;

      Person p = (Person) ois.readObject();

      System.out.println(str);
      System.out.println(p);

   } catch (IOException e) {
      e.printStackTrace();
   } catch (ClassNotFoundException e) {
      e.printStackTrace();
   } finally {
      if(ois != null){
          try {
              ois.close();
          } catch (IOException e) {
              e.printStackTrace();
          }

      }
   }
   ```

## 12.10 随机存取文件流`RandomAccessFile`的使用`
### 1. 概述
1. RandomAccessFile直接继承于java.lang.Object类，实现了DataInput和DataOutput接口
2. RandomAccessFile既可以作为一个输入流，又可以作为一个输出流
3. 如果RandomAccessFile作为输出流时:
   * 写出到的文件如果不存在，则在执行过程中自动创建。
   * 如果写出到的文件存在，则会对原有文件内容进行覆盖。（默认情况下，从头覆盖）
4. 可以通过相关的操作，实现RandomAccessFile“插入”数据的效果
### 2. 使用
1. 构造器
   ```java
   public RandomAccessFile(File file, String mode)
   public RandomAccessFile(String name, String mode)
   ```
2. mode 参数
   * r: 以只读方式打开
   * rw：打开以便读取和写入
   * rwd:打开以便读取和写入；同步文件内容的更新
   * rws:打开以便读取和写入； 同步文件内容和元数据的更新
  
    如果模式为rw读写。如果文件不存在则会去创建文件，如果存在则不会创建
3. 代码示例：复制图片
```java
RandomAccessFile raf1 = null;
RandomAccessFile raf2 = null;
try {
   //1.
   raf1 = new RandomAccessFile(new File("pic.jpg"),"r");
   raf2 = new RandomAccessFile(new File("pic1.jpg"),"rw");
   //2.
   byte[] buffer = new byte[1024];
   int len;
   while((len = raf1.read(buffer)) != -1){
       raf2.write(buffer,0,len);
   }
} catch (IOException e) {
   e.printStackTrace();
} finally {
   //3.
   if(raf1 != null){
       try {
           raf1.close();
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
   if(raf2 != null){
       try {
           raf2.close();
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
```
4. 使用 `RandomAccessFile` 实现数据的插入效果
   ```java
   RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");

   raf1.seek(3);//将指针调到角标为3的位置
   //保存指针3后面的所有数据到StringBuilder中
   StringBuilder builder = new StringBuilder((int) new File("hello.txt").length());
   byte[] buffer = new byte[20];
   int len;
   while((len = raf1.read(buffer)) != -1){
      builder.append(new String(buffer,0,len)) ;
   }
   //调回指针，写入“xyz”
   raf1.seek(3);
   raf1.write("xyz".getBytes());

   //将StringBuilder中的数据写入到文件中
   raf1.write(builder.toString().getBytes());

   raf1.close();
   ```

## 12.11 NIO.2中`Path`、`Paths`、`Files`类的使用
### 1. NIO概述
1. Java NIO (New IO， Non-Blocking IO)是从Java 1.4版本开始引入的一套新的IO API，可以替代标准的Java IO API。 NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同， NIO支持面向缓冲区的(IO是面向流的)、基于通道的IO操作。 
2. jdk7后，Java对NIO进行了极大的扩展，增强了文件处理和文件系统特性的支持，以至于我们称他们为 NIO.2
### 2. 常用方法
1. Path 常用方法：
   ```java
   String toString() ： 返回调用 Path 对象的字符串表示形式
   boolean startsWith(String path) : 判断是否以 path 路径开始
   boolean endsWith(String path) : 判断是否以 path 路径结束
   boolean isAbsolute() : 判断是否是绝对路径
   Path getParent() ：返回Path对象包含整个路径，不包含 Path 对象指定的文件路径
   Path getRoot() ：返回调用 Path 对象的根路径
   Path getFileName() : 返回与调用 Path 对象关联的文件名
   int getNameCount() : 返回Path 根目录后面元素的数量
   Path getName(int idx) : 返回指定索引位置 idx 的路径名称
   Path toAbsolutePath() : 作为绝对路径返回调用 Path 对象
   Path resolve(Path p) :合并两个路径，返回合并后的路径对应的Path对象
   File toFile(): 将Path转化为File类的对象
   ```

2. `java.nio.file.Files`常用方法：
   ```java
   Path copy(Path src, Path dest, CopyOption … how) : 文件的复制
   Path createDirectory(Path path, FileAttribute<?> … attr) : 创建一个目录
   Path createFile(Path path, FileAttribute<?> … arr) : 创建一个文件
   void delete(Path path) : 删除一个文件/目录，如果不存在，执行报错
   void deleteIfExists(Path path) : Path对应的文件/目录如果存在，执行删除
   Path move(Path src, Path dest, CopyOption…how) : 将 src 移动到 dest 位置
   long size(Path path) : 返回 path 指定文件的大小
   ```
   ```java
   //Files常用方法：用于判断
   boolean exists(Path path, LinkOption … opts) : 判断文件是否存在
   boolean isDirectory(Path path, LinkOption … opts) : 判断是否是目录
   boolean isRegularFile(Path path, LinkOption … opts) : 判断是否是文件
   boolean isHidden(Path path) : 判断是否是隐藏文件
   boolean isReadable(Path path) : 判断文件是否可读
   boolean isWritable(Path path) : 判断文件是否可写
   boolean notExists(Path path, LinkOption … opts) : 判断文件是否不存在
   
   //Files常用方法：用于操作内容
   SeekableByteChannel newByteChannel(Path path, OpenOption…how) : 获取与指定文件的连接， how 指定打开方式。
   DirectoryStream<Path> newDirectoryStream(Path path) : 打开 path 指定的目录
   InputStream newInputStream(Path path, OpenOption…how):获取 InputStream 对象
   OutputStream newOutputStream(Path path, OpenOption…how) : 获取 OutputStream 对象
   ```


