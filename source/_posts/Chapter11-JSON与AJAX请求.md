---
title: Chapter11 JSON与AJAX请求
excerpt: JSON简介及其在JavaScript和Java中的使用、原生Ajax与jQuery中的Ajax请求
tags:
  - java
categories:
  - Java笔记
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 481964b9
date: 2020-12-03 21:41:08
updated: 2020-12-03 21:41:08
subtitle:
---
## 11.1 JSON
### 11.1.1 JSON概述
1. 什么是JSON  
   JSON (JavaScript Object Notation) 是一种轻量级的数据交换格式，易于人阅读和编写，同时也易于机器解析和生成。
2. JSON的轻量级指的是跟 xml 做比较
3. 数据交换指的是客户端和服务器之间业务数据的传递格式


### 11.1.2 JSON在JavaScript中的使用
1. json的格式
   * json 是由键值对组成， 并且由花括号（大括号） 包围
   * 每个键由引号引起来， 键和值之间使用冒号进行分隔，多组键值对之间进行逗号进行分隔
   * 示例：
      ```javascript
      var jsonObj = {
          "key1":12,
          "key2":"abc",
          "key3": true,
          "key4":[11,"arr",false],
          "key5":{
              "key5_1":551,
              "key5_2":"key5_2_value"
          },
          "key6":[{
              "key6_1_1":6611,
              "key6_1_2":"key6_1_2_value"
          },{
              "key6_2_1":6621,
              "key6_2_2":"key6_2_2_value"
          }]
      };
      ```
2. json的访问
   * json本身是一个Object对象
   * json 中的 key 我们可以理解为是对象中的一个属性。
   * json 中的 key 访问就跟访问对象的属性一样： json 对象.key
   * 示例：
        ```js
        alert(jsonObj.key1); //12
        alert(jsonObj.key4); //数组 11,"arr",false
        alert(jsonObj.key5.key5_1); //551
        for (var i=0; i<jsonObj.key4.length; i++) {
            alert(jsonObj.key4[i]);
        };
        var jsonItem = jsonObj.key6[0]; // JSON对象
        ```
3. json 的两个常用方法
   * json有两种形式：
     * json对象：以对象的形式存在，操作 json 中的数据的时候使用
     * json字符串：以字符串形式存在，客户端和服务器之间进行数据交换的时候使用
   * 转换
      * `JSON.stringify()` 把 json 对象转换成为 json 字符串
      * `JSON.parse()` 把 json 字符串转换成为 json 对象
      * 示例
         ```js
         var jsonObjString = JSON.stringify(jsonObj);
		 alert(jsonObjString);
		 var jsonObj2 = JSON.parse(jsonObjString);
		 alert(jsonObj2.key1); //12
         ```

### 11.1.3 JSON在Java中的使用
需要用到第三方包 gson-2.2.4.jar  

1. javaBean 和 json 的互转
   * `public String toJson(Object src)` 将Java对象转化为json字符串
   * `public <T> T fromJson(String json, Class<T> classOfT)` 将json字符串转换为Java对象
   * 示例
      ```java
      public void test1(){
          Person person = new Person(1,"you bear lai");
          // 创建Gson对象实例
          Gson gson = new Gson();
          // 将person对象转换为json字符串
          String personJsonString = gson.toJson(person);
          System.out.println(personJsonString); //{"id":1,"name":"you bear lai"}
  
          // 转回去
          Person person1 = gson.fromJson(personJsonString, Person.class);
          System.out.println(person1); // Person{id=1, name='you bear lai'}
      }
      ```
2. List 与 json互转
   * `public String toJson(Object src)` 将List转化为json字符串列表
   * `public <T> T fromJson(String json, Type typeOfT)` 将json字符串列表转换为List
     * typeOfT的获取：需要一个`TypeToken`的子类对象，一般使用匿名子类
     * 泛型需指定为要转换的Java对象列表：`List<对象>`或`ArrayList<对象>`
     * 调用匿名子类对象的getType()方法
   * 示例
      ```java
      public void test2(){
          List<Person> personList = new ArrayList<>();
          personList.add(new Person(1,"马保国"));
          personList.add(new Person(2,"窃格瓦拉"));
          Gson gson = new Gson();
          
          //List转化为json列表
          String personListJsonString = gson.toJson(personList);
        System.out.println(personListJsonString); //[{"id":1,"name":"马保国"},{"id":2,"name":"窃  格瓦拉"}]
          
          // json字符串转为List
        List<Person> list = gson.fromJson(personListJsonString, new TypeToken<ArrayList<Person>>  () {
          }.getType());
        System.out.println(list); // [Person{id=1, name='马保国'}, Person{id=2, name='窃格瓦拉'}  ]
          System.out.println(list.get(0)); //Person{id=1, name='马保国'}
      }
      ```
3. map 和 json 的互转
   * `public String toJson(Object src)` 将Map转化为json字符串列表
   * `public <T> T fromJson(String json, Type typeOfT)` 将json字符串列表转换为Map，具体使用同上
      ```java
      public void test3(){
          Map<Integer, Person> personMap = new HashMap<>();
          personMap.put(1, new Person(1, "Harry Potter"));
          personMap.put(2, new Person(2, "Hermione Granger"));
          Gson gson = new Gson();
  
          // Map转为json
          String personMapString = gson.toJson(personMap);
          System.out.println(personMapString); // {"1":{"id":1,"name":"Harry Potter"},"2":{"id":2,"name":"Hermione Granger"}}
  
          // json转为Map
          Map<Integer,Person> personMap2 = gson.fromJson(personMapString, new TypeToken<HashMap<Integer, Person>>() {}.getType());
          System.out.println(personMap2); // {1=Person{id=1, name='Harry Potter'}, 2=Person{id=2, name='Hermione Granger'}}
          System.out.println(personMap2.get(1)); // Person{id=1, name='Harry Potter'}
      }
      ```

## 11.2 AJAX请求
### 11.2.1 概述
1. AJAX 即“Asynchronous Javascript And XML”（异步 JavaScript 和 XML） ， 是指一种创建交互式网页应用的网页开发技术。
2. ajax 是一种浏览器通过 js 异步发起请求， 局部更新页面的技术。
3. Ajax 请求的局部更新， 浏览器地址栏不会发生变化，局部更新不会舍弃原来页面的内容

### 11.2.2 原生AJAX请求
1. 步骤
   * 创建`XMLHttpRequest` 对象
   * 调用`open`方法设置请求参数
   * 调用`send`方法发送请求
   * 在`send`方法前绑定`onreadystatechange`事件，处理请求完成后的操作。
2. 示例
   
    ```html
    <!-- ajax.html -->
      <script type="text/javascript">
        function ajaxRequest() {
     			// 1、我们首先要创建XMLHttpRequest 
          var xmlhttprequest = new XMLHttpRequest();

     			//2、调用open方法设置请求参数
          xmlhttprequest.open("GET","http://localhost:8080/11_servlet/ajaxServlet?action=javaScriptAjax",true)

          // 4、在send方法前绑定onreadystatechange事件，处理请求完成后的操作。
          xmlhttprequest.onreadystatechange = function (){
            // 要先判断 
            if (xmlhttprequest.readyState == 4 && xmlhttprequest.status == 200){
              var jsonObj = JSON.parse(xmlhttprequest.responseText);
              //把响应数据显示在页面上
              document.getElementById("div01").innerHTML = "编号："+jsonObj.id+"姓名："+jsonObj.names;
            }
          }

         // 3、调用send方法发送请求
          xmlhttprequest.send();

        }
      </script>
     ```

    ```java
    //AjaxServlet.java
    protected void javaScriptAjax(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
        System.out.println("Ajax请求过来了");
        Person person = new Person(1,"Jacob");
        // json字符串
        Gson gson = new Gson();
        String personJsonString = gson.toJson(person);
        resp.getWriter().write(personJsonString);
    }
    ```

### 11.2.3 jQuery 中的 AJAX 请求

1. `$.ajax` 方法：通过 HTTP 请求加载远程数据。
   * url 表示请求的地址
   * type 表示请求的类型 GET 或 POST 请求
   * async :Boolean类型，默认值: true，即所有请求均为异步请求。
   * data 表示发送给服务器的数据, 格式有两种：
     * 一： name=value&name=value
     * 二： {key:value}
   * success 请求成功， 响应的回调函数
   * dataType 响应的数据类型,，常用的数据类型有：
     * text 表示纯文本
     * xml 表示 xml 数据
     * json 表示 json 对象
   * 示例

      ```js
      <script type="text/javascript">
          $(function(){
            // ajax请求
            $("#ajaxBtn").click(function(){

              $.ajax({
                url: "http://localhost:8080//11_servlet/ajaxServlet",
                data: "action=jQueryAjax", // AjaxServlet类中方法名
                type: "GET",
                success: function (data){
                  $("#msg").html("编号："+data.id+", 姓名："+ data.name);
                },
                dataType: "json"
              });

            });
          })
      </script>
      ```
      ```java
      // AjaxServlet.java
      protected void jQueryAjax(HttpServletRequest req, HttpServletResponse resp) throws IOException {
          System.out.println("jQueryAjax");
          Person person = new Person(1,"Jacob");
          // json字符串
          Gson gson = new Gson();
          String personJsonString = gson.toJson(person);
          resp.getWriter().write(personJsonString);
      }
      ```

2. `$.get` 方法和`$.post` 方法

   * `get()` 方法通过远程 HTTP GET 请求载入信息。
   * 语法：`$(selector).get(url,data,success(response,status,xhr),dataType)`
   * 参数
     * url 请求的 url 地址
     * data 发送的数据
     * callback 成功的回调函数
     * type 返回的数据类型
   * 示例

      ```javascript
      // ajax--get请求
	    $("#getBtn").click(function(){
				$.get("http://localhost:8080//11_servlet/ajaxServlet","action=jQueryGet", function (data){$("#msg").html("get编号："+data.id+"姓名："+data.name);},"json");
			});
      ```

3. `$.getJSON` 方法
   * 通过 HTTP GET 请求载入 JSON 数据
   * 语法：`jQuery.getJSON(url,data,success(data,status,xhr))`
   * 参数
     * url 请求的 url 地址
     * data 发送给服务器的数据
     * callback 成功的回调函数
   * 示例

      ```js
      // ajax--getJson请求
      $("#getJSONBtn").click(function(){
        // 调用
        $.getJSON("http://localhost:8080//11_servlet/ajaxServlet","action=jQuerygetJSON", function (data){$("#msg").html("getJSON编号："+data.id+"姓名："+data.name);})
      });
      ```

4. 表单序列化 `serialize()`
   * serialize()可以把表单中所有表单项的内容都获取到， 并以 name=value&name=value 的形式进行拼接
   * 语法：`$(selector).serialize()`
   * 示例

    ```java
    //AjaxServlet.java
    protected void jQuerySerialize(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
        System.out.println("jQuerySerialize");
        System.out.println("用户名："+req.getParameter("username"));
        System.out.println("密码："+req.getParameter("password"));
        Person person = new Person(1,"Jacob");
        // json字符串
        Gson gson = new Gson();
        String personJsonString = gson.toJson(person);
        resp.getWriter().write(personJsonString);
    }
    ```

    ```js
    // ajax请求
    $("#submit").click(function(){
      // 把参数序列化
      alert($("#form01").serialize());
      $.getJSON("http://localhost:8080//11_servlet/ajaxServlet","action=jQuerySerialize&"+$("#form01").serialize(), function (data){$("#msg").html("jQuerySerialize编号："+data.id+"姓名："+data.name);})
    });
    ```
