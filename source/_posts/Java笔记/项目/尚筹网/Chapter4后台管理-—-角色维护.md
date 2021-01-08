---
title: Chapter4后台管理 — 角色维护
excerpt: 角色列表分页、角色增删改查，单个删除与批量删除
tags:
  - java
  - ssm
categories:
  - Java笔记
  - 项目
  - 尚筹网
banner_img: /img/post/banner/mandao.png
index_img: /img/post/ssm.png
category: Java笔记/项目/尚筹网
abbrlink: 358bc598
date: 2021-01-06 18:32:51
updated: 2021-01-09 01:09:52
subtitle:
---
## 4.1 角色分页操作
### 4.1.1 说明
1. 效果：角色维护页面实现分页显示
2. 思路
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/4-1-1.png)
   
### 4.1.2 准备工作
1. 新建数据库表
    ```mysql
    CREATE TABLE `t_role` (
        id INT NOT NULL AUTO_INCREMENT,
        name CHAR(100),
        PRIMARY KEY (id)
    );
    ```

2. 逆向工程
   * 修改reverse模块下的generatorConfig.xml
        ```xml
        <!--  数据库表名字和我们的 entity 类对应的映射指定 -->
        <table tableName="t_role" domainObjectName="Role" />
        ```
   * 生成 mybatis-generator:generate
   * Role中新建有参、无参构造器、toString方法
   * 资源归位：将新生成的四个文件移动到相应的模块
3. service
   * RoleService接口
        ```java
        public interface RoleService {
            PageInfo<Role> getPageInfo(Integer pageNum, Integer pageSize, String keyword);
        }
        ```
   * RoleServiceImpl实现类
        ```java
        @Service
        public class RoleServiceImpl implements RoleService{

            @Autowired
            private RoleMapper roleMapper;
                @Override
            public PageInfo<Role> getPageInfo(Integer pageNum, Integer pageSize, String keyword) {
                // 1.开启分页功能
                PageHelper.startPage(pageNum,pageSize);
                // 2.执行查询
                List<Role> roleList = roleMapper.selectRoleByKeyword(keyword);
                // 3.封装为PageInfo对象返回
                return new PageInfo<>(roleList);
            }
        }
        ```
4. handler
   * RoleHandler类
        ```java
        @Controller
        public class RoleHandler {

        @Autowired
        private RoleService roleService;

        @RequestMapping("/role/get/page/info.do")
        public ResultEntity<PageInfo<Role>> getPageInfo(@RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum,
                                                        @RequestParam(value = "pageSize", defaultValue = "5") Integer pageSize,
                                                        @RequestParam(value = "keyword", defaultValue = "") String keyword) {
            // 调用service获取分页数据
            PageInfo<Role> pageInfo = roleService.getPageInfo(pageNum,pageSize,keyword);
            return ResultEntity.successWithData(pageInfo);
            }
        }
        ```
5. mapper
   * RoleMapper.xml新增方法
        ```xml
        <select id="selectRoleByKeyword" resultMap="BaseResultMap">
            select id, name from t_role
            where name like concat("%",#{keyword},"%)
        </select>
        ```
        可能需要将文件内的Role、RoleExample修改为全类名：`parameterType="com.atguigu.crowd.entity.RoleExample"`
   * RoleMapper.java
        ```java
        Role selectRoleByKeyword(String keyword);
        ```

### 4.1.3 主页面配置
1. mvc配置文件
    ```xml
    <mvc:view-controller path="/role/to/page.do" view-name="role-page"/>
    ```
2. role-page.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <!DOCTYPE html>
    <html lang="zh-CN">
    <%@include file="include-head.jsp" %>

    <body>
    <%@include file="include-nav.jsp" %>
    <div class="container-fluid">
        <div class="row">
            <%@include file="include-sidebar.jsp" %>
            <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
                <div class="panel panel-default">
                    <div class="panel-heading">
                        <h3 class="panel-title"><i class="glyphicon glyphicon-th"></i> 数据列表</h3>
                    </div>
                    <div class="panel-body">
                        <form class="form-inline" role="form" style="float:left;">
                            <div class="form-group has-feedback">
                                <div class="input-group">
                                    <div class="input-group-addon">查询条件</div>
                                    <input class="form-control has-success" type="text" placeholder="请输入查询条件">
                                </div>
                            </div>
                            <button type="button" class="btn btn-warning"><i class="glyphicon glyphicon-search"></i> 查询</button>
                        </form>
                        <button type="button" class="btn btn-danger" style="float:right;margin-left:10px;"><i class=" glyphicon glyphicon-remove"></i> 删除</button>
                        <button type="button" class="btn btn-primary" style="float:right;" onclick="window.location.href='form.html'"><i class="glyphicon glyphicon-plus"></i> 新增</button>
                        <br>
                        <hr style="clear:both;">
                        <div class="table-responsive">
                            <table class="table  table-bordered">
                                <thead>
                                <tr>
                                    <th width="30">#</th>
                                    <th width="30"><input type="checkbox"></th>
                                    <th>名称</th>
                                    <th width="100">操作</th>
                                </tr>
                                </thead>
                                <tbody>
                                <tr>
                                    <td>1</td>
                                    <td><input type="checkbox"></td>
                                    <td>PM - 项目经理</td>
                                    <td>
                                        <button type="button" class="btn btn-success btn-xs"><i class=" glyphicon glyphicon-check"></i></button>
                                        <button type="button" class="btn btn-primary btn-xs"><i class=" glyphicon glyphicon-pencil"></i></button>
                                        <button type="button" class="btn btn-danger btn-xs"><i class=" glyphicon glyphicon-remove"></i></button>
                                    </td>
                                </tr>
                                </tbody>
                                <tfoot>
                                <tr>
                                    <td colspan="6" align="center">
                                        <ul class="pagination">
                                            <li class="disabled"><a href="#">上一页</a></li>
                                            <li class="active"><a href="#">1 <span class="sr-only">(current)</span></a></li>
                                            <li><a href="#">2</a></li>
                                            <li><a href="#">3</a></li>
                                            <li><a href="#">4</a></li>
                                            <li><a href="#">5</a></li>
                                            <li><a href="#">下一页</a></li>
                                        </ul>
                                    </td>
                                </tr>

                                </tfoot>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </body>
    </html>
    ```
3. 修改include-bar.jsp
    ```jsp
    <a href="role/to/page.do"><i class="glyphicon glyphicon-king"></i> 角色维护</a>
    ```

### 4.1.4 分页操作
1. 新建js/my-role.js文件
    ```js
    // 执行分页操作
    function generatePage() {
        // 1.获取分页数据
        var pageInfo = getPageInfoRemote();
        // 2.填充表格
        fillTableBody(pageInfo);

    }

    // 远程访问服务器端程序获取pageInfo数据
    function getPageInfoRemote() {

        console.log("pageNum="+window.pageNum);
        var ajaxResult = $.ajax({
            url: "role/get/page/info.do",
            type: "post",
            data: {
                "pageNum": window.pageNum,
                "pageSize": window.pageSize,
                "keyword": window.keyword
            },
            async: false,
            dataType: "json"
        });
        console.log(ajaxResult);
        // 判断当前响应状态码是否为200
        var statusCode = ajaxResult.status;
        // 如果当前响应状态码不是200，说明出现错误，显示提示信息，让当前函数停止执行
        if (statusCode != 200) {
            layer.msg("失败! 状态码="+statusCode+" 提示信息="+ajaxResult.statusText);
            return null;
        }
        // 如果响应码为200，说明请求处理成功，获取pageInfo
        var resultEntity = ajaxResult.responseJSON;
        // 从resultEntity属性中获取result属性
        var result = resultEntity.result;
        // 判断result是否成功
        if (result == "FAILED"){
            layer.msg(resultEntity);
            return null;
        }
        var pageInfo = resultEntity.data;
        return pageInfo;
    }

    // 填充表格
    function fillTableBody(pageInfo) {
        // 清楚原来的内容
        $("#rolePageBody").empty();
        $("#Pagination").empty();
        // 判断pageInfo是否有效
        if (pageInfo == null || pageInfo.list == null || pageInfo.list.length === 0){
            $("#rolePageBody").append("<tr><td colspan='4' align='center'>抱歉！没有查询到您搜索的数据</td></tr>");
            return;
        }
        // 使用pageInfo的list属性填充tBody
        for (var i = 0; i < pageInfo.list.length; i++) {

            var role = pageInfo.list[i];
            var roleId = role.id;
            var roleName = role.name;

            var numberTd = "<td>"+(i+1)+"</td>";
            var checkboxTd = "<td><input type='checkbox'></td>";
            var roleNameTd = "<td>"+roleName+"</td>";

            var checkBtn = "<button type=\"button\" class=\"btn btn-success btn-xs\"><i class=\" glyphicon glyphicon-check\"></i></button>";
            var pencilBtn = "<button type=\"button\" class=\"btn btn-primary btn-xs\"><i class=\" glyphicon glyphicon-pencil\"></i></button>";
            var removeBtn = "<button type=\"button\" class=\"btn btn-danger btn-xs\"><i class=\" glyphicon glyphicon-remove\"></i></button>";

            var buttonTd = "<td>"+checkBtn+" "+pencilBtn+" "+removeBtn+"</td>"
            var tr = "<tr>"+numberTd+checkboxTd+roleNameTd+buttonTd+"</tr>";

            $("#rolePageBody").append(tr);
        }
        // 生成分页导航条
        generateNavigator(pageInfo);
        console.log("生成分页导航条")
    }

    // 生成分页页码导航条
    function generateNavigator(pageInfo) {

        // 获取总记录数
        var totalRecord = pageInfo.total;

        // 声明相关属性
        var properties = {
            "num_edge_entries": 3, // 边缘页数
            "num_display_entries": 5, // 主体页数
            "callback": paginationCallBack,
            "items_per_page":pageInfo.pageSize, // 每页显示1项
            "current_page": pageInfo.pageNum - 1, // Pagination内部使用pageIndex来管理页码，从0开始，而pageNum从1开始
            "prev_text": "上一页",
            "next_text": "下一页"
        };

        // 调用pagination()函数
        $("#Pagination").pagination(totalRecord, properties);

    }

    // 翻页时的回调函数
    function paginationCallBack(pageIndex,jQuery) {
        // 根据pageIndex计算得到pageNum
        window.pageNum = pageIndex + 1;

        // 调用分页函数
        generatePage();

        // 由于每一个页码按钮都是超链接，所以在这个函数最后取消超链接的默认行为
        return false;
    }
    ```
2. 修改role-page.jsp
    ```jsp
    <%@include file="include-head.jsp" %>
    <link rel="stylesheet" href="css/pagination.css">
    <script type="text/javascript" src="jquery/jquery.pagination.js"></script>
    <script type="text/javascript" src="js/my-role.js"></script>
    <script type="text/javascript">
        $(function (){
            // 1.为分页操作准备初始化数据
            window.pageNum = 1;
            window.pageSize = 5;
            window.keyword = "";

            // 2. 调用分页函数，实现分页效果
            generatePage();
        });
    </script>
    ```
    ```jsp
    <tbody id="rolePageBody"></tbody>
    <tfoot>
    <tr>
        <td colspan="6" align="center">
            <div id="Pagination" class="pagination"><!--这里显示分页--></div>
        </td>
    </tr>
    </tfoot>
    ```

## 4.2 角色查询操作
### 4.2.1 思路

### 4.2.2 代码
1. 标记id
    ```jsp
    <form class="form-inline" role="form" style="float:left;">
        <div class="form-group has-feedback">
            <div class="input-group">
                <div class="input-group-addon">查询条件</div>
                <input id="keywordInput" class="form-control has-success" type="text" placeholder="请输入查询条件">
            </div>
        </div>
        <button id="searchBtn" type="button" class="btn btn-warning"><i class="glyphicon glyphicon-search"></i> 查询</button>
    </form>
    ```

2. jQuery中获取
    ```js
    // 3. 查询操作
    $("#searchBtn").click(function (){
        window.keyword = $("#keywordInput").val();
        generatePage();
    });
    ```

## 4.3 角色保存操作
### 4.3.1 说明
1. 目标：通过模态框实现新增并保存操作
2. 思路
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/4-3-1.png)
3. 模态框：bootstrap提供的 [javascript插件](https://v3.bootcss.com/javascript/#modals)，html代码：
    ```jsp
    <div class="modal fade" tabindex="-1" role="dialog">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
        <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
            <h4 class="modal-title">Modal title</h4>
        </div>
        <div class="modal-body">
            <p>One fine body&hellip;</p>
        </div>
        <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
            <button type="button" class="btn btn-primary">Save changes</button>
        </div>
        </div><!-- /.modal-content -->
    </div><!-- /.modal-dialog -->
    </div><!-- /.modal -->
    ```
    默认是隐藏的，一般放到页面最后

### 4.3.2 代码
1. 新建modal-role-add.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div id="addModal" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title">系统弹窗</h4>
                </div>
                <div class="modal-body">
                    <form class="form-signin" role="form">
                        <div class="form-group has-success has-feedback">
                            <input type="text" name="roleName" class="form-control" placeholder="请输入角色名称" autofocus>
                            <span class="glyphicon glyphicon-user form-control-feedback"></span>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button id="saveRoleBtn" type="button" class="btn btn-primary">保存</button>
                </div>
            </div><!-- /.modal-content -->
        </div><!-- /.modal-dialog -->
    </div><!-- /.modal -->
    ```

2. 修改role-page.jsp
    ```js
    // 4. 点击新增打开模态框
    $("#showAddModalBtn").click(function (){
        $("#addModal").modal("show");
    });
    // 5.模态框数据保存
    $("#saveRoleBtn").click(function (){
        // 1.获取用户在模态框中输入的角色名，trim去前后空格
        var roleName = $.trim($("#addModal [name=roleName]").val());

        console.log("打印：roleName"+roleName);
        // 发送ajax请求
        $.ajax({
            url: "role/save.do",
            type: "post",
            data: {
                roleName: roleName
            },
            dataType: "json",
            success: function (resp){
                let result = resp.result;
                if (result === "SUCCESS") {
                    layer.msg("已保存");

                    // 重新加载分页
                    window.pageNum = 99999999;
                    generatePage();
                }
                if (result === "FAILED") {
                    layer.msg("操作失败！"+resp.message);
                }
            },
            error: function (resp) {
                layer.msg(resp.status+""+resp.statusText);
            }
        });

        // 关闭模态框
        $("#addModal").modal("hide");

        // 清理模态框内容
        $("#addModal [name=roleName]").val("");

    });
    ```
    ```jsp
    <button type="button" id="showAddModalBtn" class="btn btn-primary" style="float:right;" ><i class="glyphicon glyphicon-plus"></i> 新增</button>

    ```
    ```jsp
    <%@include file="/WEB-INF/modal-role-add.jsp"%>
    </body>
    ```
3. handler
    ```java
    @ResponseBody
    @RequestMapping("/role/save.do")
    public ResultEntity<String> saveRole(@RequestParam("roleName") String roleName) {

        System.out.println(roleName);
        roleService.saveRole(new Role(null, roleName));

        return ResultEntity.successWithoutData();
    }
    ```

4. service
    ```java
    void saveRole(Role role);
    ```
    ```java
    @Override
    public void saveRole(Role role) {
        roleMapper.insert(role);
    }
    ```


## 4.4 角色更新操作
### 4.4.1 说明
1. 目标
2. 思路
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/4-4-1.png)

### 4.4.2 代码
1. 新建modal-role-edit.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div id="editModal" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title">系统弹窗</h4>
                </div>
                <div class="modal-body">
                    <form class="form-signin" role="form">
                        <div class="form-group has-success has-feedback">
                            <input type="text" name="roleName" class="form-control" placeholder="请输入角色名称" autofocus>
                            <span class="glyphicon glyphicon-user form-control-feedback"></span>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button id="updateRoleBtn" type="button" class="btn btn-success">更新</button>
                </div>
            </div><!-- /.modal-content -->
        </div><!-- /.modal-dialog -->
    </div><!-- /.modal -->
    ```
2. 修改role-page.jsp
    ```js
    // 6. 更新
    $("#rolePageBody").on("click",".pencilBtn", function (){
        // 打开模态框
        $("#editModal").modal("show");

        // 获取表格中当前行中的角色名称
        var roleName = $(this).parent().prev().text();

        // 获取当前角色id
        window.roleId = this.id;
        console.log("roleId:"+window.roleId)
        // 使用roleName的值设置模态框中的文本框
        $("#editModal [name=roleName]").val(roleName);
    });

    // 7.给更新模态框中的更新按钮绑定单机响应函数
    $("#updateRoleBtn").click(function (){
        // 从文本框中获取新的角色名称
        var roleName = $("#editModal [name=roleName]").val();

        // 发送ajax请求执行更新
        $.ajax({
            url: "role/update.do",
            type: "post",
            data: {
                id: window.roleId,
                name: roleName
            },
            dataType: "json",
            success: function (resp){
                let result = resp.result;
                if (result === "SUCCESS") {
                    layer.msg("更新成功！");

                    // 重新加载分页
                    generatePage();
                }
                if (result === "FAILED") {
                    layer.msg("操作失败！"+resp.message);
                }
            },
            error: function (resp) {
                layer.msg(resp.status+""+resp.statusText);
            }
        });
        // 关闭模态框
        $("#editModal").modal("hide");
    });
    ```
    ```jsp
    <button type="button" id="showAddModalBtn" class="btn btn-primary" style="float:right;" ><i class="glyphicon glyphicon-plus"></i> 新增</button>
    ```
    ```jsp
    <%@include file="/WEB-INF/modal-role-add.jsp"%>
    <%@include file="/WEB-INF/modal-role-edit.jsp"%>
    </body>
    ```

3. 修改 my-role.js，给更新指定id
    ```js
    var pencilBtn = "<button id='" + roleId + "'type='button' class='btn btn-primary btn-xs pencilBtn'><i class='glyphicon glyphicon-pencil'></i></button>";
    ```

4. handler
    ```java
    @ResponseBody
    @RequestMapping("/role/save.do")
    public ResultEntity<String> saveRole(@RequestParam("roleName") String roleName) {

        roleService.saveRole(new Role(null, roleName));

        return ResultEntity.successWithoutData();
    }

    @ResponseBody
    @RequestMapping("/role/update.do")
    public ResultEntity<String> updateRole(Role role) {

        roleService.updateRole(role);
        return ResultEntity.successWithoutData();
    }
    ```
5. service
    ```java
    void updateRole(Role role);
    ```
    ```java
    @Override
    public void updateRole(Role role) {
        roleMapper.updateByPrimaryKey(role);
    }
    ```


## 4.5 角色删除操作
### 4.5.1 说明
1. 前端的“单条删除” 和“批量删除” 在后端合并为同一套操作。 合并的依据是： 单条删除时 id 也放在数组中， 后端完全根据 id 的数组进行删除
2. 思路
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/4-5-1.png)

### 4.5.2 后端代码
1. handler
    ```java
    @ResponseBody
    @RequestMapping("/role/remove/by/role/id/array.do")
    public ResultEntity<String> removeByRoleIdArray(@RequestBody List<Integer> roleIdList) {

        roleService.removeRole(roleIdList);
        
        return ResultEntity.successWithoutData();
    }
    ```

2. service
    ```java
    void removeRole(List<Integer> roleIdLList);
    ```
    ```java
    @Override
    public void removeRole(List<Integer> roleIdLList) {
        RoleExample example = new RoleExample();

        RoleExample.Criteria criteria = example.createCriteria();

        criteria.andIdIn(roleIdLList);

        roleMapper.deleteByExample(example);
    }
    ```

### 4.5.3 前端代码 - 单个删除
1. 新建modal-role-confirm.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div id="confirmModal" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title">系统弹窗</h4>
                </div>
                <div class="modal-body">
                    <h4>请确认是否要删除下列角色</h4>
                    <div id="roleNameDiv" style="text-align: center;"></div>
                </div>
                <div class="modal-footer">
                    <button id="removeRoleBtn" type="button" class="btn btn-primary">确认删除</button>
                </div>
            </div><!-- /.modal-content -->
        </div><!-- /.modal-dialog -->
    </div><!-- /.modal -->
    ```

2. my-role.js定义函数
    ```js
    // 声明专门的函数，显示确认模态框
    function showConfirmModal(roleArray) {

        console.log(roleArray);
        // 打开模态框
        $("#confirmModal").modal("show");

        // 清除旧数据
        $("#roleNameDiv").empty();

        // 全局变量，存放角色id
        window.roleIdArray = [];

        // 遍历roleArray数组
        for (let i = 0; i < roleArray.length; i++) {
            var role = roleArray[i];
            var roleName = role.roleName;
            $("#roleNameDiv").append(roleName+"<br/>");

            var roleId = role.roleId;

            window.roleIdArray.push(roleId);
        }
    }
    ```

3. 修改my-role.js
    ```jsp
    var removeBtn = "<button id='" + roleId + "' type=\"button\" class=\"btn btn-danger btn-xs removeBtn\"><i class=\" glyphicon glyphicon-remove\"></i></button>";
    ```
4. role-page.jsp
    ```js
    // 8. 点击确认模态框中的确认删除按钮执行删除
    $("#removeRoleBtn").click(function (){

        let requestBody = JSON.stringify(window.roleIdArray);
        $.ajax({
            url: "role/remove/by/role/id/array.do",
            type: "post",
            data: requestBody,
            contentType: "application/json;charset=utf-8",
            dataType: "json",
            success: function (resp){
                let result = resp.result;
                if (result === "SUCCESS") {
                    layer.msg("删除成功！");

                    // 重新加载分页
                    generatePage();
                }
                if (result === "FAILED") {
                    layer.msg("操作失败！"+resp.message);
                }
            },
            error: function (resp) {
            layer.msg(resp.status+""+resp.statusText);
            }
        });
        // 关闭模态框
        $("#confirmModal").modal("hide");
    });

    // 9. 单条删除
    $("#rolePageBody").on("click", ".removeBtn", function (){

        // 从当前按钮出发获取角色名称
        let roleName = $(this).parent().prev().text();

        // 创建role对象存入数组
        var roleArray = [{
            roleId: this.id,
            roleName: roleName
        }];

        // 调用函数打开模态框
        showConfirmModal(roleArray);
    });

    // 10.给总的checkbox绑定单机响应函数
    $("#summaryBox").click(function (){

        // 获取当前多选框自身状态
        var currentStatus = this.checked;

        // 用当前多选框状态设置其它多选框
        $(".itemBox").prop("checked", currentStatus);
    });
    ```
    ```jsp
    <%@include file="/WEB-INF/modal-role-confirm.jsp"%>
    </body>
    ```

### 4.5.4 前端代码 - 批量删除
1. 修改my-role.js
    ```js
    var checkboxTd = "<td><input class='itemBox' id='" + roleId + "' type='checkbox'></td>";
    ```
2. role-page.jsp
    ```js
    // 10.给总的checkbox绑定单机响应函数
    $("#summaryBox").click(function (){

        // 获取当前多选框自身状态
        var currentStatus = this.checked;

        // 用当前多选框状态设置其它多选框
        $(".itemBox").prop("checked", currentStatus);
    });

    // 11.全选全不选的反向操作
    $("#rolePageBody").on("click", ".itemBox", function (){

        // 获取当前已经选中的.itemBox的数量
        var checkedBoxCount = $(".itemBox:checked").length;

        // 获取全部.itemBox的数量
        var totalBoxCount = $(".itemBox").length;

        // 使用两者的比较结果设置总的checkBox
        $("#summaryBox").prop("checked", checkedBoxCount === totalBoxCount)
    });

    // 12.给批量删除的按钮绑定单击响应函数
    $("#batchRemoveBtn").click(function (){

        // 创建数组对象用来存放后面获取到的角色对象
        var roleArray = [];

        // 遍历当前选中的多选框
        $(".itemBox:checked").each(function (){

            // 使用this引用当前遍历得到的多选框
            var roleId = this.id;
            console.log("roleId:"+roleId);

            // 通过DOM操作获取角色名称
            var roleName = $(this).parent().next().text();

            roleArray.push({
                roleId: roleId,
                roleName: roleName
            });
        });

        // 检查roleArray的长度是否为0
        if (roleArray.length === 0) {
            layer.msg("请至少选择一个执行删除");
            return;
        }

        // 调用专门的函数打开确认模态框
        showConfirmModal(roleArray);
    });
    ```
    ```jsp
    <button type="button" id="batchRemoveBtn" class="btn btn-danger" style="float:right;margin-left:10px;"><i class=" glyphicon glyphicon-remove"></i> 删除</button>
    ```


