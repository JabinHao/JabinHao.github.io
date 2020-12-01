---
title: Chapter5 XML
excerpt: 摘要
tags:
  - java
  - JavaWeb
  - xml
  - 前端
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 3e26f358
date: 2020-11-29 16:28:37
updated: 2020-11-29 21:55:37
subtitle:
---
## 5.1 XML 简介
### 5.1.1 xml是什么
xml 是可扩展的标记性语言

### 5.1.2 xml 的作用
    1. 用来保存数据， 而且这些数据具有自我描述性
    2. 它还可以做为项目或者模块的配置文件
    3. 还可以做为网络传输数据的格式（现在 JSON 为主）

## 5.2 xml 语法
### 5.2.1 文档声明
1. 代码
   ```xml
    <?xml version="1.0" encoding="UTF-8"?>
   ```
2. 属性
   * version 是版本号
   * encoding 是 xml 的文件编码
   * standalone="yes/no" 表示这个 xml 文件是否是独立的 xml 文件

### 5.2.2 xml注释
1. 格式
   ```xml
   <!-- xml注释 -->
   ```
2. xml注释与html相同

### 5.2.3 元素

1. 元素是指从开始标签到结束标签的内容。
2. XML 元素命名规则
   * 名称可以含字母、 数字以及其他的字符
   * 名称不能以数字或者标点符号开始
   * 名称不能包含空格

### 5.2.4 xml属性
1. xml属性与html类似
2. 属性必须使用引号引起来， 不引会报错示例代码

### 5.2.5 语法规则
1. 所有 XML 元素都须有关闭标签
2. XML 中的元素（ 标签） 也 分成 单标签和双标签
3. XML 标签对大小写敏感
4. XML 文档必须有根元素
   * 根元素就是顶级元素
   * 没有父标签的元素， 叫顶级元素。
   * 根元素是唯一一个才行
5. XML 的属性值须加引号
6. XML 中的特殊字符：与HTML相同（`&lt;`）
7. 文本区域（ CDATA 区）
   * CDATA 语法可以告诉 xml 解析器， 我 CDATA 里的文本内容， 只是纯文本， 不需要 xml 语法解析 
   * CDATA 格式
        ```xml
        <![CDATA[  
            这里可以把你输入的字符原样显示，不会解析 xml 
        ]]>
        ```

## 5.3 XML 解析
### 5.3.1 解析技术
1. 不管是 html 文件还是 xml 文件它们都是标记型文档， 都可以使用 w3c 组织制定的 dom 技术来解析
2. 早期 JDK 为我们提供了两种 xml 解析技术 DOM 和 Sax 简介（已经过时， 但需要知道）
3. 第三方的解析：
   * jdom 在 dom 基础上进行了封装 、
   * `dom4j` 又对 jdom 进行了封装。
   * pull 主要用在 Android 手机开发， 是在跟 sax 非常类似都是事件机制解析 xml 文件。

### 5.3.2 dom4j 解析技术
1. dom4j类库的使用
   * dom4j是第三方技术，需要下载相应的jar包
   * dom4j是开源的：[github](https://dom4j.github.io/)
2. dom4j 编程步骤：
   * 先加载 xml 文件创建 Document 对象
   * 通过 Document 对象拿到根元素对象
   * 通过`根元素.elelemts(标签名);`可以返回一个集合，这个集合里放着所有你指定的标签名的元素对象
   * 找到你想要修改、删除的子元素，进行相应在的操作
   * 保存到硬盘上 

3. 获取 document 对象
   * 需要先导入jar包
   * books.xml
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <books>
            <book sn="SN12341232">
                <name>辟邪剑谱</name>
                <price>9.9</price>
                <author>班主任</author>
            </book>
            <book sn="SN12341231">
                <name>葵花宝典</name>
                <price>99.99</price>
                <author>班长</author>
            </book>
        </books>
        ```
   * Book.class
        ```java
        package com.atguigu.javaweb;

        import java.math.BigDecimal;

        public class Book {
            private String sn;
            private String name;
            private BigDecimal price;
            private String author;

            public Book(String sn, String name, BigDecimal price, String author) {
                this.sn = sn;
                this.name = name;
                this.price = price;
                this.author = author;
            }

            public Book() {
            }

            public String getSn() {
                return sn;
            }

            public void setSn(String sn) {
                this.sn = sn;
            }

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            public BigDecimal getPrice() {
                return price;
            }

            public void setPrice(BigDecimal price) {
                this.price = price;
            }

            public String getAuthor() {
                return author;
            }

            public void setAuthor(String author) {
                this.author = author;
            }

            @Override
            public String toString() {
                return "Book{" +
                        "sn='" + sn + '\'' +
                        ", name='" + name + '\'' +
                        ", price=" + price +
                        ", author='" + author + '\'' +
                        '}';
            }
        }


        ```
   * Dom4jTest.class
        ```java
        package com.atguigu.javaweb;

        import org.dom4j.*;

        import org.dom4j.io.SAXReader;
        import org.junit.Test;

        import java.math.BigDecimal;
        import java.util.List;

        public class Dom4jTest {

            @Test
            public void test1() {
                // 1. 创建一个SaxReader输入流，去读取xml配置文件，生成Document对象
                SAXReader saxReader = new SAXReader();
                Document document = null;
                try {
                    document = saxReader.read("src/books.xml"); // Junit测试中，相对路径从模块名开始
                } catch (DocumentException e) {
                    e.printStackTrace();
                }

                // 2. 通过Document对象获取根元素
                assert document != null;
                Element rootElement = document.getRootElement();

                // 3. 通过根元素获取book标签对象
                String s = "book";
                List<Element> books = rootElement.elements(s);// element()和elements()可以通过标签名查找子元素

                // 4. 遍历，处理每个book标签转换为Book类
                for (Element book:books) {
                    Element nameElement = book.element("name");
                    // System.out.println(nameElement.asXML()); // asXML()把标签对象转换为标签字符串
                    String nameText = nameElement.getText(); // getText() 可以获取标签中的文本内容
                    String priceText = book.elementText("price"); // 直接获取文本内容
                    String authorText = book.elementText("author");
                    String snValue = book.attributeValue("sn");
                    System.out.println(new Book(snValue,nameText, BigDecimal.valueOf(Double.parseDouble(priceText)),authorText));
                }
            }

        }

        ```

