---
title: Chapter8 文件上传与下载
excerpt: JavaWeb文件操作，包括文件的上传与下载、不同浏览器的编码问题等
tags:
  - java
  - JavaWeb
  - Servlet
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: f67e4aa
date: 2020-12-02 18:00:38
updated: 2020-12-02 18:00:38
subtitle:
---
## 8.1 文件的上传
### 8.1.1 概述
1. 文件上传过程
   * 一个 form 标签， method=post 请求
   * form 标签的 encType 属性值必须为 multipart/form-data 值
   * 在 form 标签中使用 input type=file 添加上传的文件
   * 编写服务器代码（Servlet 程序） 接收， 处理上传的数据。
2. 代码示例
   ```java
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("文件上传成功");
        ServletInputStream inputStream = req.getInputStream();

        byte[] buffer = new byte[1024000];
        int read = inputStream.read(buffer);
        System.out.println(new String(buffer,0,read));
    }
   ```

   ```html
   <body>
   <form action="http://localhost:8080/08_servlet/upload" method="post" enctype="multipart/form-data">
       用户名：<input type="text" name="username"/> <br>
       头像： <input type="file" name="photo"/><br>
       <input type="submit" value="上传">
   </form>
   ```
3. 说明
   * encType=multipart/form-data 表示提交的数据， 以多段（每一个表单项一个数据段） 的形式进行拼接 然后以二进制流的形式发送给服务器

### 8.1.2 commons-fileupload.jar 常用 API 
1. 使用
   * commons-fileupload.jar是第三方包，需要导入
   * commons-fileupload.jar 需要依赖 commons-io.jar 这个包，所以两个包都要引入。
2. 常用类
   * `ServletFileUpload` 类， 用于解析上传的数据。
   * `FileItem` 类， 表示每一个表单项。
   * `boolean ServletFileUpload.isMultipartContent(HttpServletRequest request)` 判断当前上传的数据格式是否是多段的格式。
   * `public List<FileItem> parseRequest(HttpServletRequest request)` 解析上传的数据
   * `boolean FileItem.isFormField()` 判断当前这个表单项， 是否是普通的表单项。 还是上传的文件类型。
     * `true` 表示普通类型的表单项
     * `false` 表示上传的文件类型
   * `String FileItem.getFieldName()` 获取表单项的 `name` 属性值
   * `String FileItem.getString()` 获取当前表单项的值。
   * `String FileItem.getName()` 获取上传的文件名
   * `void FileItem.write(file)` 将上传的文件写到 参数 file 所指向抽硬盘位置
3. 示例：接收文件并保存
   ```java
   //UploadServelt.java
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 1. 判断是否为多段数据（只有多段才为文件上传）
        if (ServletFileUpload.isMultipartContent(req)) {
            //创建FileItemFactory工厂实现类对象
            FileItemFactory fileItemFactory = new DiskFileItemFactory();
            // 创建用于解析上传数据的ServletFileUpload工具类对象
            ServletFileUpload servletFileUpload = new ServletFileUpload(fileItemFactory);
            try {
                // 解析上传数据
                List<FileItem> list = servletFileUpload.parseRequest(req);
                // 循环判断，每个表单项是普通类型还是文件
                for (FileItem fileItem: list) {
                    if (fileItem.isFormField()) {
                        // 普通表单
                        System.out.println("表单name属性值："+fileItem.getFieldName());
                        // 参数乱码问题
                        System.out.println("表单value属性值："+fileItem.getString("utf-8"));
                    } else {
                        //上传的文件
                        System.out.println("表单name属性值："+ fileItem.getFieldName());
                        System.out.println("上传文件名"+ fileItem.getName());

                        fileItem.write(new File("e:\\" + fileItem.getName()));
                    }
                }
            } catch (FileUploadException e) {
                e.printStackTrace();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
   ```
   upload.jsp
   ```jsp
    <body>
    <form action="http://localhost:8080/08_servlet/upload" method="post" enctype="multipart/form-data">
        用户名：<input type="text" name="username"/> <br>
        头像： <input type="file" name="photo"/><br>
        <input type="submit" value="上传">
    </form>
    </body>
   ```

## 8.2 文件的下载
### 8.2.1 概述
1. 客户端发送请求给服务器
2. 服务器：
   * 或获取要下载的文件名
   * 读取要下载的文件内容
   * 返回数据给客户端
   * 响应头中说明
     * 返回的数据类型
     * 返回的数据用于下载

### 8.2.2 常用API
1. API
   * `response.getOutputStream()`: 获取响应的输出流
   * `servletContext.getResourceAsStream()`: 读取文件
   * `servletContext.getMimeType()`：设置返回客户端的数据类型
   * `response.setContentType()`：

2. 代码示例
   ```java
   // DownloadServlet.java
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 1. 获取下载文件名
        String downloadFilename = "cat.jpg";
        // 2. 读取要下载的文件内容
        ServletContext servletContext = getServletContext();
        // 获取要下载的文件类型
        String mimetype = servletContext.getMimeType("/file/"+downloadFilename);
        // 告诉客户端数据类型
        resp.setContentType(mimetype);
        // 告诉客户的收到的数据用于下载, Content-Disposition响应头表示收到的数据如何处理，attachment表示附件，下载使用；filename为指定下载的文件名(即客户端获取到的文件的名字，可以更改)
        resp.setHeader("Content-Disposition","attachment; filename="+downloadFilename);
        // /file被服务器解析为http://ip:port/工程名/file/ 映射到web的file目录
        InputStream resourceAsStream = servletContext.getResourceAsStream("/file/" + downloadFilename);
        // 获取响应的输出流
        OutputStream outputStream = resp.getOutputStream();
        // 读取输入流中的数据，复制给输出流，输出给客户端
        IOUtils.copy(resourceAsStream,outputStream);
    }
   ```

### 8.2.3 中文乱码问题
1. 问题引入
   * `response.setHeader("Content-Disposition", "attachment; fileName=1.jpg")`
   *  attachment 表示附件， 也就是下载的一个文件。 fileName表示下载的文件名（返回给客户端的文件名）
   *  默认在响应头中， 不能包含有中文字符， 只能包含 ASCII 码。

2. 解决方案一：
   * 适用于谷歌和IE浏览器
   * 使用 `URLEncoder` 类先对中文名进行 UTF-8 的编码
   * IE 浏览器和谷歌浏览器收到含有编码后的字符串后会以 UTF-8 字符集进行解码显示

   ```java
   resp.setHeader("Content-Disposition","attachment; filename="+ URLEncoder.encode("猫.jpg", "utf-8"));
   ```

3. 解决方案二：
   * 适用于火狐浏览器
   * 对中文名进行 BASE64 的编码
   * BASE64编码有三种方法
     * 早期使用JDK里sun.misc套件下的BASE64Encoder和BASE64Decoder两个类，效率低下
     * Apache Commons Codec有提供Base64的编码与解码功能，会使用到org.apache.commons.codec.binary套件下的Base64类，需要引用Apache Commons Codec，很麻烦
     * Java 8的java.util套件中，新增了Base64的类别，可以用来处理Base64的编码与解码
         ```java
         Base64.Decoder decoder = Base64.getDecoder();
         Base64.Encoder encoder = Base64.getEncoder();
         String content = "这是需要Base64编码的内容";
         // 编码
         String encodeText = encoder.encodeToString(content.getBytes("UTF-8"));
         System.out.println(encodeText);
         // 解码
         byte[] decodeBytes = decoder.decode(encodeText);
         String decodeText = new String(decodeBytes);
         System.out.println(decodeText);
         ```
         输出：
         ```
         6L+Z5piv6ZyA6KaBQmFzZTY057yW56CB55qE5YaF5a65
         这是需要Base64编码的内容
         ```
   * 此时需要把请求头 `Content-Disposition: attachment; filename=中文名`编码成为： `Content-Disposition: attachment; filename==?charset?B?xxxxx?=`
     * `=?charset?B?xxxxx?=` 现在我们对这段内容进行一下说明。
     * `=?` 表示编码内容的开始
     * `charset` 表示字符集
     * B 表示 BASE64 编码
     * xxxx 表示文件名 BASE64 编码后的内容
     * ?= 表示编码内容的结束
     * 示例：
         ```java
         String head = "attachment; filename="+ "=?utf-8?B?" + Base64.getEncoder().encodeToString("猫.jpg".getBytes("UTF-8"))+"?=";
         resp.setHeader("Content-Disposition",head);
         ```

4. 通过User-Agent请求头动态切换
   ```java
   if (req.getHeader("User-Agent").contains("Firefox")) {
        // 创建BASE64编码器
        String head = "attachment; filename="+ "=?utf-8?B?" + Base64.getEncoder().encodeToString("猫.jpg".getBytes("UTF-8"))+"?=";
        resp.setHeader("Content-Disposition",head);
    }else {
        resp.setHeader("Content-Disposition","attachment; filename="+ URLEncoder.encode("猫.jpg", "utf-8"));
    }
   ```




