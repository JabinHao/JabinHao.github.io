---
title: Chapter1&2 HTML与CSS
excerpt: 摘要
tags:
  - java
  - JavaWeb
  - html
  - css
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 8abc1fba
date: 2020-11-26 20:39:51
updated: 2020-11-29 21:51:51
subtitle:
---
# 1. html基础

## 1.1 html书写规范

1. 格式

   ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>标题</title>
    </head>
    <body>
        页面内容
    </body>
    </html>
   ```

2. 注释

    ```html
   <!-- 注释内容 -->
   ```

## 1.2 HTML标签

1. 标签的格式

    ```html
    <标签名>内容</标签名> <!--双标签-->
    <标签名/> <!--单标签-->
    ```

2. 规则
    * 标签名大小写不敏感
    * 标签分为单标签与双标签，常见单标签：
        * `br`：换行
        * `hr`：水平线

3. 标签的属性
    * 基本属性：可以修改简单的样式效果 `bgcolor="red"`
    * 事件属性：可以直接设置事件响应后的代码 `onclick="alert('你好！');"`

## 1.3 常用标签

1. font字体标签
    * color属性：颜色
    * face属性：字体
    * size属性：文本大小

    ```html
    <font color="red" face="宋体" size="5">我是字体标签</font>
    ```

    效果：  
    <font color="red" face="宋体" size="5">我是字体标签</font>

2. 特殊字符

    效果|描述|格式|编号
    :-:|:-:|:-:|:-:
    &nbsp;|空格|\&nbsp;|\&#160;
    <|小于号|\&lt;|\&#60;
    &gt;|大于号|\&gt;|\&#62;
    &|和号|\&amp;|\&#38;
    "|引号|\&quot;|\&#34;
    '|撇号|\&apos;|\&#39;

3. 标题
    * `h1`-`h6`，`h1`最大
    * `align`属性：对齐，`left`、`center`、`right`

    ```html
    <h1 align=center>标题</h1>
    ```

    <h1 align=center>标题</h1>

4. 超链接
    * `<a>`标签代表超链接
    * `href`属性设置链接地址
    * `target`属性设置跳转方式
        * `_self` ：当前页面
        * `_blank` ：打开新页面

    ```html
    <a href="" target="_self">个人博客</a><br/>
    ```

    <a href="" target="_self">个人博客</a><br/>

5. 列表标签
    * 无序列表 `<ul>`
    * 有序列表 `<ol>`
6. img标签  
    * 显示图片
    * src属性设置图片路径
        * 相对路径：`.`和`..`
        * 绝对路径：`http://ip:port/工程名/资源路径`
    * `width`、`height`属性设置宽度和高度
    * `border`设置边框(数字，像素数)
    * `alt` 显示出错时的文字提示

    ```html
    <img src="omg.png" wight=200 height="300" border="1" alt="omg">
    ```

7. 表格
    * `table`标签
        * `border`属性：边框
        * `width` 与 `height`：宽与高
        * `align`：设置表格相对于页面的位置
        * `cellspacing` 设置单元格间距(0则无间距)
    * \<tr>标签：行
    * \<td>标签：单元格
    * \<b>标签：加粗
    * \<th>标签：格式化的单元格（加粗、居中）
    * 代码示例及效果

        ```html
        <table border="1" width=300 height=200 cellspacing="0">
            <tr>
                <th>1.1</th>
                <th>1.2</th>
                <th>1.3</th>
            </tr>
            <tr>
                <td align=center>2.1</td>
                <td>2.1</td>
                <td>2.1</td>
            </tr>
        </table>
        ```

        <table border="1" width=300 height=200 cellspacing="0">
            <tr>
                <th>1.1</th>
                <th>1.2</th>
                <th>1.3</th>
            </tr>
            <tr>
                <td align=center>2.1</td>
                <td>2.1</td>
                <td>2.1</td>
            </tr>
        </table>

8. 跨行跨列表格
    * \<td>或\<th>标签中使用
    * 跨行：`colspan`属性，等于几则跨几行
    * 跨列：`rowspan`属性

9. `iframe`标签
    * 页面上开辟小区域，显示单独的页面
    * 与\<a>标签组合使用
        1. 在`iframe`标签中使用`name`属性定义名称
        2. `a`标签的`target`属性设置为`iframe`的`name`属性值
        3. 示例

            ```html
            <iframe src="3.标题标签.html" width="500" height="400" name="abc"></iframe>
            <br/>

            <ul>
                <li><a href="0-标签语法.html" target="abc">0-标签语法.html</a></li>
                <li><a href="1.font标签.html" target="abc">1.font标签.html</a></li>
                <li><a href="2.特殊字符.html" target="abc">2.特殊字符.html</a></li>
            </ul>
            ```

10. 表单
    * `form`标签，用于收集用户信息的所有元素集合
    * `input`标签：文本输入框, 根据`type`属性：
        * `type`属性为`text`：文本输入框，`value`属性设置默认值
        * `type`属性为`password`：密码输入框
        * `type="radio"`：单选框，`name`设置属性用于分组，`checked="checked"`默认选中当前选项
        * `type="checkbox"`：复选框，`checked="checked"`默认选中
        * `type="select"`：下拉列表框，`option`标签设置下拉列表内容，`selected="selected"`属性设置默认选中
        * `type="textarea"`：多行文本输入框
            * `row`属性：显示的行高
            * `cols`属性：显示几个字符宽度
            * 标签内容即为默认值
        * `type="reset"`：重置按钮，`value`属性修改其名称（默认重置）
        * `type=submit`：提交按钮，`value`·····
        * `type=button`：按钮，····
        * `type=file`：文件上传域
        * `type=hidden`：隐藏域，发送某些信息而不需要用户看到时使用

    * 表单格式化：  
        一般将表单内容放到表格中
11. 表单的提交
    * `action`属性设置提交的服务器地址
    * `method`属性设置提交的方式`GET`（默认）或`POST`
    * 表单提交时，数据没有发送的原因：
        * 表单项没有`name`属性
        * 单选、复选、下拉列表的`option`选项没有添加`value`属性
        * 表单项不在`form`标签中
    * GET请求的特点
        * 浏览器地址栏中的地址是：action属性 + [? + 请求参数]，请求参数的格式为：name=value&name=value
        * 不安全
        * 数据长度有限制
    * POST请求的特点
        * 地址栏中只有action属性
        * 相对于GET方法更加安全
        * 理论上没有数据长度的限制

12. 其他标签
    * `div`：默认独占一行
    * `span`：长度为封装数据的长度
    * `p`：段落标签，默认在其上下方各空出一行（如果存在则不再空）

# 2. css

## 2.1 css语法规则

1. 格式

    ```css
    选择器 {
        属性:值;
    }
    ```

2. 说明
    * 浏览器根据选择器选择css样式，选择器名字是用户定义的
    * 属性是要设置的样式名，属性与值组成样式声明，多个声明间用分号隔开，最后一个可以不加

## 2.2 CSS 和 HTML 的结合方式

1. 标签属性中设置  
    * 在html标签的style属性上设置"key: value value;"
    * 可读性差，没有复用性
    * 示例

        ```html
        <div style="border: 1px solid red;">div标签</div>
        ```

2. head中设置
    * head中使用style标签，为各种标签设置css格式
    * style中遵循css语法，注释为 `/*  */`
    * 只能在当前页面复用，维护不方便
    * 示例

        ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Document</title>
            <style>
                div{
                    border: 1px solid red;
                }
                span{
                    border: 2px solid blue;
                }
            </style>
        </head>
        <body>
            <div>div标签</div>
        <div>div标签</div>
        <span>span标签</span>
        </body>
        </html>
        ```

3. 单独的css文件
    * 在.css文件中定义css样式
    * 在html文件的头部通过link标签引入

    ```html
    <link rel="stylesheet" type="text/css" href="style.css">
    ```

## 2.3 css选择器

1. 标签名选择器
    * 格式
        ```css
        标签名{
            属性:值
        }
        ```
    * 引入后自动应用到相应的标签
2. `id`选择器
    * 格式
        ```css
        #idxxx{
            属性:值;
        }
        ```
    * xxx为任意数字
    * 需要在标签中定义`id`属性：`id="id002"`
3. class 选择器（类选择器）
    * 格式
        ```css
        .classXX{
            属性:值
        }
        ```
    * 要在标签中加上class属性：`class="class01"`
4. 组合选择器
    * 格式
        ```css
        选择器1, 选择器2, 选择器n{
            属性:值
        }
        ```
    * 可以让多个选择器共享样式
    * 可以是不同类型的选择器

## 2.4 常用样式
1. 颜色
    * color: ;
    * 可以是名称black、red、blue等
    * 可以是rgb值：rgb(255,255,0)
    * 也可以是十六进制值：#0000ff
2. 宽度和高度
    * 要先设置边框才能看到
    * `width`与`height`属性
    * 可以用像素值px或百分比
3. 背景颜色
    * `background-color:#0F2D4C`
4. 字体样式
    ```css
    color： #FF0000； 字体颜色
    font-size： 20px; 字体大小
    ```
5. 边框
    * `border: 1px solid red;`
    * 三个值分别代表粗细、形状、颜色
    * 形状有实线(solid)、虚线(dashed)等
6. div居中
    * 使整个div快块相对于页面居中
    * 通过页边距来设置
    * margin-left: auto;
    * margin-right: auto;
7. 文本居中
    * `text-align: center;`
8. 超链接去下划线
    * `text-decoration:none`
9. 表格细线
    * 即合并单元格与表格的边框
    ```css
    table{
        border: 2px solid red; // /*设置边框*/
        border-collapse: collapse;  /*将边框合并*/
    }
    td{
    border: 2px solid red;
    }
    ```
10. 列表去除修饰
    * 即去除前面的点或序号
    ```css
    ul {
        list-style: none;
    }
    ```




