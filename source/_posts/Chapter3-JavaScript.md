---
title: Chapter3 JavaScript
excerpt: 摘要
tags:
  - java
  - JavaWeb
  - JavaScript
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 7ba5e8e6
date: 2020-11-27 20:59:31
updated: 2020-11-29 21:53:31
subtitle:
---
## 3.1 javascript简介
1. js是弱类型语言(即变量类型可变)，Java是强类型语言
2. 特点
   * 交互性
   * 安全性（不允许直接访问硬盘）
   * 跨平台性
## 3.2 JavaScript 和 html 代码的结合方式
1. 在html中直接使用
   * 使用`script`标签
   * 在body或head中均可以
2. 外部引入
   * 使用`script`标签的`src`属性引入
   * 可以是相对路径或绝对路径

## 3.3 变量
### 3.3.1 变量类型
1. 数值类型 number
2. 字符串：string
3. 对象：object
4. 布尔类型：boolean 
5. 函数类型：function
### 3.3.2 特殊值
* undefined：未定义，所有 js 变量未赋于初始值的时候， 默认值都是 undefined.
* null：空值
* NaN：全称是： Not a Number。 非数字。 非数值
### 3.3.3 定义变量
```javascript
var 变量名;
var 变量名 = 值;
```

## 3.4 关系运算（比较）
1. `==`：等于，简单的做字面值的比较
2. `===`：全等于，除了做字面值的比较之外， 还会比较两个变量的数据类型

## 3.5 逻辑运算
### 3.5.1 运算符
1. 且运算： &&
2. 或运算： ||
3. 取反运算： !

### 3.5.2 规则
1. 在 JavaScript 语言中， 所有的变量， 都可以做为一个 boolean 类型的变量去使用。
    * 0、null、undefined、""(空串) 都认为是 `false`
    * 其它为真
2. `&&` 且运算
   * 当表达式全为真的时候。 返回最后一个表达式的值。
   * 当表达式中， 有一个为假的时候。 返回第一个为假的表达式的值
3. `||` 或运算
   * 当表达式全为假时， 返回最后一个表达式的值
   * 只要有一个表达式为真。 就会把回第一个为真的表达式的值
4. 短路  
   * 且与或都有短路，即有结果后不再继续判断

## 3.6 数组
1. 定义
   ```javascript
   var 数组名 = [];
   var 数组名 = [1,'a',true];
   ```
2. 数组可以自动扩容
   ```javascript
   var a = [2];
   a[2] = 5;
   ```
3. 未赋值的数组值为`undefined`
4. 数组可以遍历
   ```javascript
    var b = [1,'c',null];
    for( i=0;i<4;i++){
        alert(b[i]);
    }
   ```

## 3.7 函数
### 3.7.1两种定义方式
1. 方式一
   ```javascript
   function 函数名(形参列表){
       函数体
   }
   ```
   * 有返回值的函数只需在函数体中直接使用`return`语句
   * 形参无需指定类型
2. 方式二
    ```javascript
    var 函数名 = function(形参列表){
        函数体
    }
    ```
3. 函数不能重载，会直接覆盖

### 3.7.2 函数的 `arguments` 隐形参数（只在 `function` 函数内）
1. 隐形参数：在 function 函数中不需要定义， 但却可以直接用来获取所有参数的变量。 
2. 类似于java中的可变参数：`public void fun(String args[])`
3. 隐形参数的操作也类似于数组
4. 使用`arguments`获取全部参数
   ```javascript
   arguments.length //参数个数
   arguments[0]; //第一个参数
   ```

## 3.8 JS中的自定义对象
1. js可以使用`Object`定义对象
2. 定义
   ```javascript
   var 变量名 = new Object(); //空对象
   变量名.属性 = 值; //定义属性
   变量名.函数 = function(){} // 定义函数
   ```
3. 对象的访问
   ```javascript
   变量名.属;
   变量名.函数();
   ```
4. 花括号定义对象
   ```javascript
   var 变量名 = {
       属性名:值; // 定义属性
       函数名:function(){} // 定义函数
   }
   ```

## 3.9 事件
### 3.9.1 简介
1. 定义  
   电脑输入设备与页面进行交互的响应称之为事件。
2. 常用事件
   * `onload`加载完成事件：页面加载完成之后， 常用于做页面 js 代码初始化操作
   * `onclick`单击事件：常用于按钮的点击响应操作
   * `onblur`失去焦点事件：常用用于输入框失去焦点后验证其输入内容是否合法。
   * `onchange`内容发生改变事件：常用于下拉列表和输入框内容发生改变后操作
   * `onsubmit`表单提交事件：常用于表单提交前， 验证所有表单项是否合法

### 3.9.2 事件的注册
1. 告诉浏览器， 当事件响应后要执行哪些操作代码， 叫事件注册或事件绑定
2. 静态注册事件： 通过 html 标签的事件属性直接赋于事件响应后的代码， 这种方式我们叫静态注册
3. 动态注册事件： 是指先通过 js 代码得到标签的 dom 对象， 然后再通过 dom 对象.事件名 = function(){} 这种形式赋于事件响应后的代码， 叫动态注册
4. 动态注册的基本步骤
   * 获取标签对象
   * 标签对象.事件名 = function()

### 3.9.3 事件注册示例
1. `onload`事件
   ```html
   <head>
        <meta charset="UTF-8">
        <script>
            function onloadFun(){
                alert("静态注册onload事件");
            }
        </script>
        <title>Document</title>
    </head>
    <body onload="onloadFun();">

    </body>
    ```
    `onload` 事件是浏览器解析完页面之后就会自动触发的事件，需要在`body`标签中调用函数，动态注册`onload`则无需任何操作:
    ```html
    <head>
        <meta charset="UTF-8">
        <script>
            function onloadFun(){
                alert("静态注册onload事件");
            }
            window.onload = function(){
                alert("动态注册的onload事件");
            }
        </script>
        <title>Document</title>
    </head>
    <body onload="onloadFun();">

    </body>
    ```

2. `onclick`事件
   ```html
    <head>
        <meta charset="UTF-8">
        <script>
            function onclickFun(){
                alert("静态注册onclick事件");
            }
        </script>
        <title>Document</title>
    </head>
    <body>
        <button onclick="onclickFun();">按钮1</button>
        <button>按钮2</button>
    </body>
   ```
   ```html
    <head>
        <meta charset="UTF-8">
        <script>
            window.onload = function(){
                //获取标签对象
                var btnObj = document.getElementById("btn01");
                
                //标签对象.事件名 = fucntion(){}
                btnObj.onclick = function(){
                    alert("动态注册的onclick事件");
                }
            }
        </script>
        <title>Document</title>
    </head>
    <body>
        <button>按钮1</button>
        <button id="btn01">按钮2</button>
    </body>
   ```

3. `onblur`失去焦点事件
   ```html
    <head>
        <meta charset="UTF-8">
        <script type="text/javascript">
            function onblurFun(){
                console.log("静态注册失去焦点事件");
            }
        </script>
        <title>Document</title>
    </head>
    <body>
        用户名：<input type="text"><br>
        密码：<input type="password" name="" id="" onblur="onblurFun();"><br>
    </body>
   ```
   `console` 是控制台对象， 是由 `JavaScript` 语言提供， 专门用来向浏览器的控制器打印输出， 用于测试使用，`log()` 是打印的方法
   ```html
    <head>
        <meta charset="UTF-8">
        <script type="text/javascript">
            window.onload = function(){
                var passwordObj = document.getElementById("password");
                passwordObj.onblur = function(){
                    console.log("动态注册失去焦点事件");
                }
            }
        </script>
        <title>Document</title>
    </head>
    <body>
        用户名：<input type="text"><br/>
        密码：<input type="password" id="password" ><br/>
    </body>
   ```

4. `onchange`事件
   ```html
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
        <script type="text/javascript">
            function onchangeFun(){
                alert("国籍已改变");
            }
            window.onload = function(){
                var selObj = document.getElementById("gender");
                alert(selObj);
                selObj.onchange = function(){
                    alert("gender has been changed!");
                }
            }
        </script>
    </head>
    <body>
        请选择国籍：
        <select onchange="onchangeFun();">
            <option value="none">---country---</option>
            <option value="cn">China</option>
            <option value="US">US</option>
            <option value="eu">Europe</option>
        </select><br><br>
        <select name="gender" id="gender">
            <option value="none">---gender---</option>
            <option value="man">man</option>
            <option value="woman">woman</option>
        </select>
    </body>
   ```

5. `onsubmit`事件
    ```html
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
        <script>
            function onsubmitFun(){
                alert("静态注册提交表单，发现不合法");
                return false;
            }

            window.onload = function(){
                subObj = document.getElementById("form02");
                subObj.onsubmit = function(){
                    alert("动态注册提交表单，发现不合法");
                    return false;
                }
            }
        </script>
    </head>
    <body>
        <form action="http://localhost:8080" method="GET" onsubmit="return onsubmitFun();"> <!-- 也可以直接return false -->
            <input type="text" name="text" id="01" value="静态注册"/><br/><br/>
            <input type="submit"/>
        </form>
        <br><br>
        <form action="http://localhost:8080" method="GET" id="form02">
            <input type="text" value="动态注册"/><br/><br/>
            <input type="submit"/>
        </form>
    </body>
    ```

## 3.10 DOM模型
>DOM全称 `Document Object Model`(文档对象模型)，即把文档中的标签，属性，文本，转换成为对象来管理

### 3.10.1 Document对象
1. Document对象的理解
   * Document 管理了所有的 `HTML` 文档内容
   * document 是一种树结构的文档，有层级关系
   * 我们把所有的标签都对象化
   * 可以通过 `document` 访问所有的标签对象
2. 模拟对象化
   ```java
    class Dom{
        private String id; // id 属性
        private String tagName; //表示标签名
        private Dom parentNode; //父节点
        private List<Dom> children; // 子结点
        private String innerHTML; // 起始标签和结束标签中间的内容
    }
   ```

### 3.10.2 RegExp对象
1. 简介  
    用于创建正则表达式对象
2. 语法
   ```javascript
   // 1. 创建正则表达式
   var patt = new RegExp(pattern, modifiers);
   //或
   var patt = /pattern/modifiers;
   // 2. 匹配
   patt.test(str);
   ```
   * pattern为正则表达式
   * modifiers描述了检索是否是全局，区分大小写等。
     * i：执行对大小写不敏感的匹配。
     * g：执行全局匹配（查找所有匹配而非在找到第一个匹配后停止）
     * m：执行多行匹配

3. 方括号
   * [abc]：匹配方括号内的任意字母
   * [^abc]：匹配任何不在方括号内的字母
   * [0-9]：任意数字
   * [a-z]：任意小写字母
   * [A-Z]：任意大写字母
   * [A-z]：任意字母

4. 元字符
   * .  ：任意单个字符，除了换行和结束符
   * \w ：任意单词字符：字母、数字、下划线
   * \W ：任意非单词字符
   * \d：数字
   * \D：非数字
   * \s：空白字符
   * \S：非空白字符
   * \b：匹配边界：\babc：以abc开头或结尾
   * \B：非边界
   * ^：开头
   * $：结尾

5. 量词
   * +：至少一个相应字符：`a+`至少一个`a`
   * *：零个或多个相应字符
   * ?：0个或一个
   * {n}：n个连续相应字符
   * {n,m}：n-m个连续相应字符
   * {n,}：n个及以上连续相应字符 

### 3.10.3 Document对象常用方法
1. `getElementById(elementId)`  
    通过标签的 id 属性查找标签 dom 对象， elementId 是标签的 id 属性值

    ```html
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
        <script>
            function onclickFun(){
                var usernameObj = document.getElementById("username");
                var usernametext = usernameObj.value;
                var patt = /^[A-Z]+\w{3,5}/
                var usernamespanObj = document.getElementById("usernamespan");
                if(patt.test(usernametext)){
                    usernamespanObj.innerHTML = "用户名合法";
                }
                else {
                    usernamespanObj.innerHTML = "用户名不合法";
                }
            }
        </script>
    </head>
    <body>
        用户名：<input type="text" id="username" value="jacob"/>
        <span id="usernamespan" style="color: red;"></span>
        <button onclick="onclickFun();">校验</button>
    </body>
    ```

2. `document.getElementsByName(elementName)`   
    * 通过标签的 name 属性查找标签 dom 对象， elementName 标签的 name 属性值
    * 返回的是多个对象（dom集合）
    ```html
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
        <script>
            function checkAll(){
                var hobbies = document.getElementsByName("hobby");
                for(i=0; i<hobbies.length; i++){
                    hobbies[i].checked = true;
                    //checked 表示复选框的选中状态。 如果选中是 true， 不选中是 false
                }
            }

            function checkNo(){
                var hobbies = document.getElementsByName("hobby");
                for(i=0; i<hobbies.length; i++){
                    hobbies[i].checked = false;
                }
            }

            function checkReverse(){
                var hobbies = document.getElementsByName("hobby");
                for(i=0; i<hobbies.length; i++){
                    hobbies[i].checked = !hobbies[i].checked;
                }
            }
        </script>
    </head>
    <body>
        兴趣爱好：
        <input type="checkbox" name="hobby" id="cpp">C++
        <input type="checkbox" name="hobby" id="java">Java 
        <input type="checkbox" name="hobby" id="python">python
        <input type="checkbox" name="hobby" id="go">go
        <br><br>
        <button onclick="checkAll();">全选</button>
        <button onclick="checkNo();">全不选</button>
        <button onclick="checkReverse();">反选</button>
    </body>
    ```

3. `getElementsByTagName(tagname)`  
    * 通过标签名查找标签 dom 对象，tagname 是标签名
    * 返回的是集合，操作类似数组
   
    ```html
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
        <script>
            function checkAll(){
                var inputs = document.getElementsByTagName("input");
                for (let index = 0; index < inputs.length; index++) {
                    inputs[index].checked=true;             
                }
            }
        </script>
    </head>
    <body>
        兴趣爱好：
        <input type="checkbox" name="hobby" id="cpp">C++
        <input type="checkbox" name="hobby" id="java">Java 
        <input type="checkbox" name="hobby" id="python">python
        <input type="checkbox" name="hobby" id="go">go
        <br><br>
        <button onclick="checkAll();">全选</button>
    </body>
    ```
4. `createElement( tagName)`  
    通过给定的标签名， 创建一个标签对象。 tagName 是要创建的标签名
    ```html
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
        <script>
            window.onload = function(){
                var divObj = document.createElement("div");
                divObj.innerHTML = "createElement";
                document.body.appendChild(divObj);
                // 方式二
                var textNodeObj = document.createTextNode("通过文字节点添加");
                divObj.appendChild(textNodeObj);
            }
        </script>
    </head>
    <body>  
    </body>
    ```

5. 注意事项
   * 使用优先级 `getElementById`> `getElementsByName ` > ` getElementsByTagName`
   * html按顺序执行，script标签中的脚本执行早于页面加载，若要读取可以在`window.onload = function(){}中执行`

### 3.10.4 节点的常用属性和方法
1. 节点即标签对象
2. 方法
   * 通过具体的元素节点调用：`getElementsByTagName()`， 获取当前节点的指定标签名孩子节点
   * `appendChild(oChildNode)`： 可以添加一个子节点， oChildNode 是要添加的孩子节点
3. 属性
   * `childNodes`：获取当前节点的所有子节点
   * `firstChild`：获取当前节点的第一个子节点
   * `lastChild`: 获取当前节点的最后一个子节点
   * `parentNode`: 获取当前节点的父节点
   * `nextSibling`: 获取当前节点的下一个节点
   * `previousSibling`属: 获取当前节点的上一个节点
   * `className`：用于获取或设置标签的 class 属性值
   * `innerHTML`：表示获取/设置起始标签和结束标签中的内容
   * `innerText`：表示获取/设置起始标签和结束标签中的文本

