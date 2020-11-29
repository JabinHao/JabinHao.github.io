---
title: Chapter4 jQuery
excerpt: 摘要
tags:
  - java
  - JavaWeb
  - JavaScript
  - jQuery
  - 前端
categories:
  - Java笔记
  - JavaWeb
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
date: 2020-11-28 23:03:31
updated: 2020-11-29 21:54:31
subtitle:
---
## 4.1 jQuery概述
### 4.1.1 背景
1. jQuery， 顾名思义， 也就是 JavaScript 和查询（Query） ， 它就是辅助 JavaScript 开发的 js 类库。
2. jQuery是开源免费的
3. 使用前需要先引入

### 4.1.2 jQuery与Dom
1. jQuery的核心函数`$()`
   * 参数为函数时，表示页面加载完成之后。 相当于 `window.onload = function(){}`
        ```js
        $(function{
            // 内容
        })
        ```
   * 传入参数为 [ HTML 字符串 ] 时：会对我们创建这个 html 标签对象
   * 传入参数为 [ 选择器字符串 ] 时：
     * `$("#id");` id 选择器， 根据 id 查询标签对象
     * `$(“标签名”);` 标签名选择器， 根据指定的标签名查询标签对象
     * `$(".class");` 类型选择器， 可以根据 class 属性查询标签对象
   * 传入参数为 [ DOM 对象 ] 时：会把这个 dom 对象转换为 jQuery 对象
2. `Dom` 对象
   * 通过 `getElementById()`查询出来的标签对象是 Dom 对象
   * 通过 `getElementsByName()`查询出来的标签对象是 Dom 对象
   * 通过 `getElementsByTagName()`查询出来的标签对象是 Dom 对象
   * 通过 `createElement()` 方法创建的对象， 是 Dom 对象

    **DOM 对象 Alert 出来的效果是：`object HTML 标签名 Element`**
3. `jQuery` 对象
   * 通过 JQuery 提供的 API 创建的对象， 是 JQuery 对象
   * 通过 JQuery 包装的 Dom 对象， 也是 JQuery 对象
   * 通过 JQuery 提供的 API 查询到的对象， 是 JQuery 对象

    **jQuery 对象 Alert 出来的效果是：`object Object`**
4. jQuery对象的本质  
   * jQuery 对象是 dom 对象的数组 + jQuery 提供的一系列功能函数
   * jQuery 对象和 DOM 对象不能互相使用彼此的属性和方法
5. Dom 对象和 jQuery 对象互转
   * dom 对象转化为 jQuery 对象
     1. 先有 DOM 对象
     2. `$( DOM 对象 )` 就可以转换成为 jQuery 对象
   * jQuery 对象转为 dom 对象
     1. 先有 jQuery 对象
     2. `jQuery对象[下标]`取出相应的 DOM 对象
   * 示例
        ```html
        <script type="text/javascript" src="./jquery-1.7.2.js"></script>
        <script type="text/javascript">
            $(function (){
                var $btnObj = $("#btnId");
                alert($btnObj[0]);
                $btnObj.click(function (){
                    alert("jQuery的单击事件");
                });
            });
        </script>
        ```

## 4.2 jQuery选择器
### 4.2.1 基本选择器
1. 选择器
   * `#ID` 选择器： 根据 id 查找标签对象
   * `.class` 选择器： 根据 class 查找标签对象
   * `element` 选择器： 根据标签名查找标签对象
   * `*` 选择器： 表示任意的， 所有的元素
   * `selector1, selector2` 组合选择器： 合并选择器 1，选择器 2 的结果并返回
   * `p.myClass`：`p`标签且`class`类型为`myClass`
2. 代码示例
   ```html
    <script type="text/javascript">
        
            $(function () {
                //1.选择 id 为 one 的元素 "background-color","#bbffaa"
                $("#btn1").click(function () {
                    // css() 方法 可以设置和获取样式
                    $("#one").css("background-color", "#005500");
                });


                //2.选择 class 为 mini 的所有元素
                $("#btn2").click(function () {
                    $(".mini").css("background-color","#bbffaa");
                });

                //3.选择 元素名是 div 的所有元素
                $("#btn3").click(function () {
                    $("div").css("background-color","#bbffaa");
                });

                //4.选择所有的元素
                $("#btn4").click(function () {
                    $("*").css("background-color","#bbffaa");
                });

                //5.选择所有的 span 元素和id为two的元素
                $("#btn5").click(function () {
                    $("span,#two").css("background-color","#bbffaa");
                });

            });

    </script>
   ```

### 4.2.2 层级选择器
1. 选择器
   * `ancestor descendant` 后代选择器 ： 在给定的祖先元素下匹配所有的后代元素
   * `parent > child` 子元素选择器： 在给定的父元素下匹配所有的子元素（不包含孙子元素）
   * `prev + next` 相邻元素选择器： 匹配所有紧接在 prev 元素后的 next 元素
   * `prev ~ sibings` 之后的兄弟元素选择器： 匹配 prev 元素之后的所有 siblings 元素
2. 代码示例
   ```html
	<script type="text/javascript">
        $(document).ready(function(){
            //1.选择 body 内的所有 div 元素
            $("#btn1").click(function(){
                $("body div").css("background", "#bbffaa");
            });

            //2.在 body 内, 选择div子元素  
            $("#btn2").click(function(){
                $("body > div").css("background", "#bbffaa");
            });

            //3.选择 id 为 one 的下一个 div 元素 
            $("#btn3").click(function(){
                $("#one+div").css("background", "#bbffaa");
            });

            //4.选择 id 为 two 的元素后面的所有 div 兄弟元素
            $("#btn4").click(function(){
                $("#two~div").css("background", "#bbffaa");
            });
        });
    </script>
   ```

### 4.2.3 过滤选择器
1. 基本过滤器
   * `:first` 获取第一个元素
   * `:last` 获取最后个元素
   * `:not(selector)` 去除所有与给定选择器匹配的元素
   * `:even` 匹配所有索引值为偶数的元素， 从 0 开始计数
   * `:odd` 匹配所有索引值为奇数的元素， 从 0 开始计数
   * `:eq(index)` 匹配一个给定索引值的元素
   * `:gt(index)` 匹配所有大于给定索引值的元素
   * `:lt(index)` 匹配所有小于给定索引值的元素
   * `:header` 匹配如 h1, h2, h3 之类的标题元素
   * `:animated` 匹配所有正在执行动画效果的元素 
    
    ```html
    <script type="text/javascript">
        $(document).ready(function(){
            //1.选择第一个 div 元素  
            $("#btn1").click(function(){
                $("div:first").css("background", "#bbffaa");
            });

            //2.选择最后一个 div 元素
            $("#btn2").click(function(){
                $("div:last").css("background", "#bbffaa");
            });

            //3.选择class不为 one 的所有 div 元素
            $("#btn3").click(function(){
                $("div:not(.one)").css("background", "#bbffaa");
            });

            //4.选择索引值为偶数的 div 元素
            $("#btn4").click(function(){
                $("div:even").css("background", "#bbffaa");
            });

            //5.选择索引值为奇数的 div 元素
            $("#btn5").click(function(){
                $("div:odd").css("background", "#bbffaa");
            });

            //6.选择索引值为大于 3 的 div 元素
            $("#btn6").click(function(){
                $("div:gt(3)").css("background", "#bbffaa");
            });

            //7.选择索引值为等于 3 的 div 元素
            $("#btn7").click(function(){
                $("div:eq(3)").css("background", "#bbffaa");
            });

            //8.选择索引值为小于 3 的 div 元素
            $("#btn8").click(function(){
                $("div:lt(3)").css("background", "#bbffaa");
            });

            //9.选择所有的标题元素
            $("#btn9").click(function(){
                $(":header").css("background", "#bbffaa");
            });

            //10.选择当前正在执行动画的所有元素
            $("#btn10").click(function(){
                $(":animated").css("background", "#bbffaa");
            });
            //11.选择没有执行动画的最后一个div
            $("#btn11").click(function(){
                $("div:not(:animated):last").css("background", "#bbffaa");
            });
        });
    </script>
    ```

2. 内容过滤器
   * :contains(text) 匹配包含给定文本的元素
   * :empty 匹配所有不包含子元素或者文本的空元素
   * :parent 匹配含有子元素或者文本的元素
   * :has(selector) 匹配含有选择器所匹配的元素的元素

    ```html
    <script type="text/javascript">
        $(document).ready(function(){
            //1.选择 含有文本 'di' 的 div 元素
            $("#btn1").click(function(){
                $("div:contains('di')").css("background", "#bbffaa");
            });
            //2.选择不包含子元素(或者文本元素) 的 div 空元素
            $("#btn2").click(function(){
                $("div:empty").css("background", "#bbffaa");
            });
            //3.选择含有 class 为 mini 元素的 div 元素
            $("#btn3").click(function(){
                $("div:has(.mini)").css("background", "#bbffaa");
            });
            //4.选择含有子元素(或者文本元素)的div元素
            $("#btn4").click(function(){
                $("div:parent").css("background", "#bbffaa");
            });
        });
    </script>
    ```

3. 属性过滤器
   * `[attribute]` 匹配包含给定属性的元素。
   * `[attribute=value]` 匹配给定的属性是某个特定值的元素
   * `[attribute!=value]` 匹配所有不含有指定的属性， 或者属性不等于特定值的元素。
   * `[attribute^=value]` 匹配给定的属性是以某些值开始的元素
   * `[attribute$=value]` 匹配给定的属性是以某些值结尾的元素
   * `[attribute*=value]` 匹配给定的属性是以包含某些值的元素
   * `[attrSel1][attrSel2][attrSelN]` 复合属性选择器， 需要同时满足多个条件时使用。

    ```html
    <script type="text/javascript">
        $(function() {
            //1.选取含有 属性title 的div元素
            $("#btn1").click(function() {
                $("div[title]").css("background", "#bbffaa");
            });
            //2.选取 属性title值等于'test'的div元素
            $("#btn2").click(function() {
                $("div[title='test']").css("background", "#bbffaa");
            });
            //3.选取 属性title值不等于'test'的div元素(*没有属性title的也将被选中)
            $("#btn3").click(function() {
                $("div[title!='test']").css("background", "#bbffaa");
            });
            //4.选取 属性title值 以'te'开始 的div元素
            $("#btn4").click(function() {
                $("div[title^='te']").css("background", "#bbffaa");
            });
            //5.选取 属性title值 以'est'结束 的div元素
            $("#btn5").click(function() {
                $("div[title$='est']").css("background", "#bbffaa");
            });
            //6.选取 属性title值 含有'es'的div元素
            $("#btn6").click(function() {
                $("div[title*='es']").css("background", "#bbffaa");
            });
            
            //7.首先选取有属性id的div元素，然后在结果中 选取属性title值 含有'es'的 div 元素
            $("#btn7").click(function() {
                $("div[id][title*='es']").css("background", "#bbffaa");
            });
            //8.选取 含有 title 属性值, 且title 属性值不等于 test 的 div 元素
            $("#btn8").click(function() {
                $("div[title][title!='test']").css("background", "#bbffaa");
            });
        });
    </script>
    ```

4. 表单过滤器
   * `:input` 匹配所有 input, textarea, select 和 button 元素
   * `:text` 匹配所有 文本输入框
   * `:password` 匹配所有的密码输入框
   * `:radio` 匹配所有的单选框
   * `:checkbox` 匹配所有的复选框
   * `:submit` 匹配所有提交按钮
   * `:image` 匹配所有 img 标签
   * `:reset` 匹配所有重置按钮
   * `:button` 匹配所有 `input type=button <button>`按钮
   * `:file` 匹配所有 `input type=file` 文件上传
   * `:hidden` 匹配所有不可见元素 `display:none` 或 `input type=hidden`

5. 表单对象属性过滤器
   * `:enabled` 匹配所有可用元素
   * `:disabled` 匹配所有不可用元素
   * `:checked` 匹配所有选中的单选，复选，和下拉列表中选中的 option 标签对象
   * `:selected` 匹配所有选中的 option
    
    ```html
    <script type="text/javascript">
        $(function(){	
            //1.对表单内 可用input 赋值操作
            $("#btn1").click(function(){
                // val()可以操作表单项的value属性值
                // 它可以设置和获取
                $(":text:enabled").val("我是万能的程序员");
            });
            //2.对表单内 不可用input 赋值操作
            $("#btn2").click(function(){
                $(":text:disabled").val("管你可用不可用，反正我是万能的程序员");
            });
            //3.获取多选框选中的个数  使用size()方法获取选取到的元素集合的元素个数
            $("#btn3").click(function(){
                alert( $(":checkbox:checked").length );
            });
            //4.获取多选框，每个选中的value值
            $("#btn4").click(function(){
                // 获取全部选中的复选框标签对象
                var $checkboies = $(":checkbox:checked");
                // 老式遍历
                // for (var i = 0; i < $checkboies.length; i++){
                // 	alert( $checkboies[i].value );
                // }
                // each方法是jQuery对象提供用来遍历元素的方法
                // 在遍历的function函数中，有一个this对象，这个this对象，就是当前遍历到的dom对象
                $checkboies.each(function () {
                    alert( this.value );
                });
            });
            //5.获取下拉框选中的内容  
            $("#btn5").click(function(){
                // 获取选中的option标签对象
                var $options = $("select option:selected");
                // 遍历，获取option标签中的文本内容
                $options.each(function () {
                    // 在each遍历的function函数中，有一个this对象。这个this对象是当前正在遍历到的dom对象
                    alert(this.innerHTML);
                });
            });
        })	
    </script>
    ```

### 4.2.4 `jQuery` 元素筛选
1. 筛选方法
   * `eq()` 获取给定索引的元素，功能跟 `:eq()` 一样
   * `first()` 获取第一个元素，功能跟 `:first` 一样
   * `last()` 获取最后一个元素，功能跟 `:last` 一样
   * `filter(exp)` 留下匹配的元素
   * `is(exp)` 判断是否匹配给定的选择器， 只要有一个匹配就返回， true
   * `has(exp)` 返回包含有匹配选择器的元素的元素，功能跟 `:has` 一样
   * `not(exp)` 删除匹配选择器的元素 功能跟 `:not` 一样
   * `children(exp)` 返回匹配给定选择器的子元素，功能跟 `parent>child` 一样
   * `find(exp)` 返回匹配给定选择器的后代元素，功能跟 `ancestor descendant` 一样
   * `next()` 返回当前元素的下一个兄弟元素，功能跟 `prev + next` 功能一样
   * `nextAll()` 返回当前元素后面所有的兄弟元素，功能跟 `prev ~ siblings` 功能一样
   * `nextUntil()` 返回当前元素到指定匹配的元素为止的后面元素
   * `parent()` 返回父元素
   * `prev(exp)` 返回当前元素的上一个兄弟元素（`exp`可选）
   * `prevAll()` 返回当前元素前面所有的兄弟元素
   * `prevUnit(exp)` 返回当前元素到指定匹配的元素为止的前面元素
   * `siblings(exp)` 返回所有兄弟元素
   * `add()` 把 add 匹配的选择器的元素添加到当前 jquery 对象中

    ```html
    <script type="text/javascript">
        $(document).ready(function(){
            //(1)eq()  选择索引值为等于 3 的 div 元素
            $("#btn1").click(function(){
                $("div").eq(3).css("background-color","#bfa");
            });
            //(2)first()选择第一个 div 元素
                $("#btn2").click(function(){
                    //first()   选取第一个元素
                $("div").first().css("background-color","#bfa");
            });
            //(3)last()选择最后一个 div 元素
            $("#btn3").click(function(){
                //last()  选取最后一个元素
                $("div").last().css("background-color","#bfa");
            });
            //(4)filter()在div中选择索引为偶数的
            $("#btn4").click(function(){
                //filter()  过滤   传入的是选择器字符串
                $("div").filter(":even").css("background-color","#bfa");
            });
                //(5)is()判断#one是否为:empty或:parent
            //is用来检测jq对象是否符合指定的选择器
            $("#btn5").click(function(){
                alert( $("#one").is(":empty") );
            });
            
            //(6)has()选择div中包含.mini的
            $("#btn6").click(function(){
                //has(selector)  选择器字符串    是否包含selector
                $("div").has(".mini").css("background-color","#bfa");
            });
            //(7)not()选择div中class不为one的
            $("#btn7").click(function(){
                //not(selector)  选择不是selector的元素
                $("div").not('.one').css("background-color","#bfa");
            });
            //(8)children()在body中选择所有class为one的div子元素
            $("#btn8").click(function(){
                //children()  选出所有的子元素
                $("body").children("div.one").css("background-color","#bfa");
            });
            //(9)find()在body中选择所有class为mini的div元素
            $("#btn9").click(function(){
                //find()  选出所有的后代元素
                $("body").find("div.mini").css("background-color","#bfa");
            });
            //(10)next() #one的下一个div
            $("#btn10").click(function(){
                //next()  选择下一个兄弟元素
                $("#one").next("div").css("background-color","#bfa");
            });
            //(11)nextAll() #one后面所有的span元素
            $("#btn11").click(function(){
                //nextAll()   选出后面所有的元素
                $("#one").nextAll("span").css("background-color","#bfa");
            });
            //(12)nextUntil() #one和span之间的元素
            $("#btn12").click(function(){
                //
                $("#one").nextUntil("span").css("background-color","#bfa")
            });
            //(13)parent() .mini的父元素
            $("#btn13").click(function(){
                $(".mini").parent().css("background-color","#bfa");
            });
            //(14)prev() #two的上一个div
            $("#btn14").click(function(){
                //prev()  
                $("#two").prev("div").css("background-color","#bfa")
            });
            //(15)prevAll() span前面所有的div
            $("#btn15").click(function(){
                //prevAll()   选出前面所有的元素
                $("span").prevAll("div").css("background-color","#bfa")
            });
            //(16)prevUntil() span向前直到#one的元素
            $("#btn16").click(function(){
                //prevUntil(exp)   找到之前所有的兄弟元素直到找到exp停止
                $("span").prevUntil("#one").css("background-color","#bfa")
            });
            //(17)siblings() #two的所有兄弟元素
            $("#btn17").click(function(){
                //siblings()    找到所有的兄弟元素，包括前面的和后面的
                $("#two").siblings().css("background-color","#bfa")
            });
            //(18)add()选择所有的 span 元素和id为two的元素
            $("#btn18").click(function(){
                //   $("span,#two,.mini,#one")
                $("span").add("#two").add("#one").css("background-color","#bfa");
            });
        });
    </script>
    ```

## 4.3 jQuery属性操作
### 4.3.1 常见方法一
1. 方法
   1. `html()` 它可以设置和获取起始标签和结束标签中的内容。 跟 dom 属性 `innerHTML` 一样。
   2. `text()` 它可以设置和获取起始标签和结束标签中的文本。 跟 dom 属性 `innerText` 一样。
   3. `val()` 它可以设置和获取表单项的 value 属性值。 跟 dom 属性 `value` 一样
2. 示例
   ```html
   <head>
        <script>
            $(function (){
                // alert($("div").html()); //获取
                // $("div").html("<h1>我是div标签中的标题</h1>")； //设置
                // alert($("div").text()); //获取
                // $("div").text("<h1>我是div标签中的标题</h1>"); //设置
                // 批量操作单选
                // $(":radio").val(["radio2"]);
                // // 批量操作筛选框选中状态
                // $(":checkbox").val(["checkbox2","checkbox3"]);
                // // 下拉多选
                // $("#multiple").val(["mul1","mul3"]);
                // // 下拉单选
                // $("#single").val(["sin2"])
                //组合选择
                $(":radio,:checkbox,#multiple,#single").val(["radio1","checkbox2","checkbox3","mul1","sin2","mul3"]);
            })
        </script>
    </head>
    <body>
        <!-- <div>我是div标签 <span>我是span标签</span></div> -->
        单选：
        <input type="radio" name="radio" id="" value="radio1"/> radio1 
        <input type="radio" name="radio" id="" value="radio2"/> radio2 
        <br/>
        多选：
        <input type="checkbox" name="checkbox" id="" value="checkbox1"/>checkbox1
        <input type="checkbox" name="checkbox" id="" value="checkbox2"/>checkbox2
        <input type="checkbox" name="checkbox" id="" value="checkbox3"/>checkbox3
        <br/>
        下拉列表：
        <select name="multiple" id="multiple" multiple="multiple" size="4">
            <option value="mul1">mul1</option>
            <option value="mul2">mul2</option>
            <option value="mul3">mul3</option>
            <option value="mul4">mul4</option>
        </select>
        <br/>
        下拉单选：
        <select name="" id="single">
            <option value="sin1">sin1</option>
            <option value="sin2">sin2</option>
            <option value="sin3">sin3</option>
        </select>
        <br/>
    </body>
   ```

### 4.3.2 常见方法二
1. 方法
   1. `attr()` 可以设置和获取属性的值， 不推荐操作 `checked`、 `readOnly`、 `selected`、 `disabled` 等等，attr 方法还可以操作非标准的属性。 比如自定义属性： abc,bbj
   2. `prop()` 可以设置和获取属性的值,只推荐操作 `checked`、 `readOnly`、 `selected`、 `disabled` 等等
2. 示例
   ```html
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <script src="./jquery-1.7.2.js"></script>
        <script>
            $(function(){
                // attr
                // alert($(":checkbox:first").attr("name")); //获取
                // $(":checkbox:first").attr("name","abc"); //设置name为abc
                // alert($(":checkbox:last").attr("checked")); //未选中则undefined，选中为checked
                // alert( $(":checkbox").prop("checked") ); //选中为true，否则为false
                // alert( $(":checkbox").prop("checked", "false") ); //设置 
            })
        </script>
    </head>
    <body>
        <br/>
        多选：
        <input type="checkbox" name="checkbox" value="checkbox1" />checkbox1
        <input type="checkbox" name="checkbox" value="checkbox2" checked="checked"/>checkbox2
    </body>
   ```

## 4.4 DOM的增删改
### 4.4.1 方法
1. 内部插入
   * `appendTo()`： a.appendTo(b) 把 a 插入到 b 子元素末尾， 成为最后一个子元素
   * `prependTo()`： a.prependTo(b) 把 a 插到 b 所有子元素前面， 成为第一个子元素
2. 外部插入
   * `insertAfter()` ：a.insertAfter(b) 得到 ba
   * `insertBefore()`：a.insertBefore(b) 得到 ab
3. 替换
   * `replaceWith()`： a.replaceWith(b) 用 b 替换掉 a
   * `replaceAll()`： a.replaceAll(b) 用 a 替换掉所有 b
4. 删除
   * `remove()`： a.remove(); 删除 a 标签
   * `empty()`： a.empty(); 清空 a 标签里的内容

### 4.4.2 示例
```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="./jquery-1.7.2.js"></script>
    <script>
        $(function(){
            $("<h1>标题1</h1>").appendTo("div");
            $("<h2>标题2</h2>").prependTo("div");
            $("<h3>标题3</h3>").insertBefore("#div2");
            $("span").replaceWith("<p>p1</p>");
            $("<h4>h4</h4>").replaceAll("a");
        })
    </script>
</head>
<body>
    <div id="div1">div1</div>
    <div id="div2">div2</div>
    <span>span1</span><br/>
    <span>span2</span>
    <a>a1</a>
    <a>a2</a>
</body>
```

## 4.5 CSS样式操作
### 4.5.1 方法
1. `addClass()`： 添加样式
2. `removeClass()`： 删除样式
3. `toggleClass()`：有就删除， 没有就添加样式。
4. `offset()`：获取和设置元素的坐标。

### 4.5.2 示例

```html
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
<style type="text/css">	
	div{
		width:100px;
		height:260px;
	}	
	div.border{
		border: 2px white solid;
	}	
	div.redDiv{
		background-color: red;
	}	
	div.blackDiv{
		border: 5px blue solid;
	}	
</style>
<script type="text/javascript" src="script/jquery-1.7.2.js"></script>
<script type="text/javascript">	
	$(function(){
		/*
CSS
css(name|pro|[,val|fn])       读写匹配元素的样式属性。 
								a.css('color')取出a元素的color
								a.css('color',"red")设置a元素的color为red
CSS 类
addClass(class|fn) 			为元素添加一个class值;<div class="mini big">
removeClass([class|fn]) 	删除元素的class值；传递一个具体的class值，就会删除具体的某个class
							a.removeClass()：移除所有的class值
**/	
		var $divEle = $('div:first');
		
		$('#btn01').click(function(){
			//addClass() - 向被选元素添加一个或多个类
			$divEle.addClass("redDiv blackDiv");
		});
		
		$('#btn02').click(function(){
			//removeClass() - 从被选元素删除一个或多个类 
			$divEle.removeClass()
		});

		$('#btn03').click(function(){
			//toggleClass() - 对被选元素进行添加/删除类的切换操作 
			//切换就是如果具有该类那么删除，如果没有那么添加上
			$divEle.toggleClass("redDiv");
		});
		
		$('#btn04').click(function(){
			//offset() - 返回第一个匹配元素相对于文档的位置。
			var os = $divEle.offset();
			//注意通过offset获取到的是一个对象，这个对象有两个属性top表示顶边距，left表示左边距
			alert("顶边距："+os.top+" 左边距："+os.left);
			
			//调用offset设置元素位置时，也需要传递一个js对象，对象有两个属性top和left
			//offset({ top: 10, left: 30 });
			 $divEle.offset({
				 top:50,
				 left:60
			 }); 
		});
		
	})
</script>
</head>
<body>

	<table align="center">
		<tr>
			<td>
				<div class="border">
				</div>
			</td>
			
			<td>
				<div class="btn">
					<input type="button" value="addClass()" id="btn01"/>
					<input type="button" value="removeClass()" id="btn02"/>
					<input type="button" value="toggleClass()" id="btn03"/>
					<input type="button" value="offset()" id="btn04"/>
				</div>
			</td>
		</tr>
	</table>
		
	<br /> <br />
</body>
```

## 4.6 jQuery 动画
### 4.6.1 基本动画
1. 函数
   1. `show()` 将隐藏的元素显示
   2. `hide()` 将可见的元素隐藏。
   3. `toggle()` 可见就隐藏， 不可见就显示
2. 参数：以上动画方法都可以添加参数。
   * 第一个参数是动画 执行的时长， 以毫秒为单位
   * 第二个参数是动画的回调函数 (动画完成后自动调用的函数)

### 4.6.2 淡入淡出动画
1. 函数
   1. `fadeIn()` 淡入（慢慢可见）
   2. `fadeOut()` 淡出（慢慢消失）
   3. `fadeTo()` 在指定时长内慢慢的将透明度修改到指定的值。 0 透明， 1 完成可见， 0.5 半透明
   4. `fadeToggle()` 淡入/淡出 切换
2. 参数  
   * 124同上
   * 3的参数为：时长,透明的,回调函数

## 4.7 jQuery 事件操作
### 4.7.1 jQuery与原生js

```js
$( function(){} ) 
和 
window.onload = function(){} 
的区别
```
1. 触发时间
   * jQuery 的页面加载完成之后是浏览器的内核解析完页面的标签创建好 DOM 对象之后就会马上执行。
   * 原生 js 的页面加载完成之后， 除了要等浏览器内核解析完标签创建好 DOM 对象， 还要等标签显示时需要的内容加载完成。

2. 触发顺序
   * jQuery 页面加载完成之后先执行
   * 原生 js 的页面加载完成之后

3. 执行的次数
   * 原生 js 的页面加载完成之后， 只会执行最后一次的赋值函数。
   * jQuery 的页面加载完成之后是全部把注册的 function 函数， 依次顺序全部执行

### 4.7.2 jQuery 中其他的事件处理方法
1. 方法
   * `click()` 它可以绑定单击事件， 以及触发单击事件（无参数则触发，参数为函数时为绑定）
   * `mouseover()` 鼠标移入事件
   * `mouseout()` 鼠标移出事件
   * `bind()` 可以给元素一次性绑定一个或多个事件。
   * `one()` 使用上跟 bind 一样。 但是 one 方法绑定的事件只会响应一次。
   * `unbind()` 跟 bind 方法相反的操作， 解除事件的绑定
   * `live()` 也是用来绑定事件。 它可以用来绑定选择器匹配的所有元素的事件。 哪怕这个元素是后面动态创建出来的也有效
2. 事件的冒泡
    * 父子元素同时监听同一个事件。 当触发子元素的事件的时候， 同一个事件也被传递到了父元素的事件里去响应。
    * 在子元素事件函数体内， return false; 可以阻止事件的冒泡传递

### 4.7.3 javaScript 事件对象
1. 定义  
   事件对象， 是封装有触发的事件信息的一个 javascript 对象。
2. 获取  
   * 在给元素绑定事件的时候， 在事件的 function( event ) 参数列表中添加一个参数， 这个参数名， 我们习惯取名为 event。
   * 这个 event 就是 javascript 传递参事件处理函数的事件对象。
3. 示例
   * 原生 javascript 获取 事件对象
        ```js
        window.onload = function () {
            document.getElementById("areaDiv").onclick = function (event) {
                console.log(event);
            }
        }
        ```
   * jQuery 代码获取 事件对象
        ```js
        $(function () {
            $("#areaDiv").click(function (event) {
                console.log(event);
            });
        });
        ```
   * 使用 bind 同时对多个事件绑定同一个函数。 怎么获取当前操作是什么事件
        ```js
        $("#areaDiv").bind("mouseover mouseout",function (event) {
            if (event.type == "mouseover") {
                console.log("鼠标移入");
            } else if (event.type == "mouseout") {
                console.log("鼠标移出");
            }
        });
        ```


