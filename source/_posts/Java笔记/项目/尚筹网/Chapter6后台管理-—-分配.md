---
title: Chapter6后台管理 — 分配
excerpt: 实现权限分配：给管理员分配角色，给角色分配权限
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
abbrlink: 5ea5d345
date: 2021-01-09 01:21:10
updated: 2021-01-11 14:41:05
subtitle:
---
## 6.1 说明
### 6.1.1 权限分配思路
1. 思路 
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/5-4-1.png)

### 6.2.1 实现
1. 权限分配通过 SpringSecurity 框架来实现
2. 我们需要给Admin分配好相应的Role

## 6.2 给Admin分配Role
### 6.2.1 目标思路
1. 目标：通过页面操作把 Admin 和 Role 之间的关联关系保存到数据库
2. 思路  
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/6-2-1.png)

### 6.2.2 后端代码
1. 数据库表
    ```mysql
    # 权限分配
    DROP TABLE IF EXISTS inner_admin_role;
    CREATE TABLE `inner_admin_role` (
        id          INT NOT NULL AUTO_INCREMENT,
        admin_id    INT,
        role_id     INT,
        PRIMARY KEY (id)
    );
    ```
2. 修改分配按钮的超链接 admin-page.jsp
    ```JSP
    <a href="assign/to/assign/role/page.do?adminId=${admin.id }&pageNum=${requestScope.pageInfo.pageNum}&keyword=${param.keyword}" class="btn btn-success btn-xs"><i class=" glyphicon glyphicon-check"></i></a>
    ```
3. AssignHandler
    ```java
    @Controller
    public class AssignHandler {

        @Autowired
        private AdminService adminService;

        @Autowired
        private RoleService roleService;

        @RequestMapping("/assign/to/assign/role/page.do")
        public String toAssignRolePage(@RequestParam("adminId") Integer adminId,
                                    ModelMap modelMap) {

            // 1.查询已分配的角色
            List<Role> assignedRoleList = roleService.getAssignedRole(adminId);

            // 2. 查询未分配的角色
            List<Role> unAssignedRoleList = roleService.getUnAssignedRole(adminId);

            // 3.存入模型
            modelMap.addAttribute("assignedRoleList", assignedRoleList);
            modelMap.addAttribute("unAssignedRoleList", unAssignedRoleList);

            return "assign-role";
        }
    }
    ```
4. RoleService
    ```java
    List<Role> getAssignedRole(Integer adminId);

    List<Role> getUnAssignedRole(Integer adminId);
    ```
    ```java
    @Override
    public List<Role> getAssignedRole(Integer adminId) {

        return roleMapper.selectAssignedRole(adminId);
    }

    @Override
    public List<Role> getUnAssignedRole(Integer adminId) {
        return roleMapper.selectUnAssignedRole(adminId);
    }
    ```
5. RoleMapper
    ```java
    List<Role> selectAssignedRole(Integer adminId);

    List<Role> selectUnAssignedRole(Integer adminId);
    ```
    ```xml
    <select id="selectAssignedRole" resultMap="BaseResultMap">
        select id, name from t_role where id in (select role_id from inner_admin_role where admin_id = #{adminId})
    </select>
    <select id="selectUnAssignedRole" resultMap="BaseResultMap">
        select id, name from t_role where id not in (select role_id from inner_admin_role where admin_id = #{adminId})
    </select>
    ```

### 6.2.3 前端代码
1. 修改agmin-page.jap
    ```html
    <%-- <button type="button" class="btn btn-success btn-xs"><i class=" glyphicon glyphicon-check"></i></button> --%>
    <a href="assign/to/assign/role/page.do?adminId=${admin.id }&pageNum=${requestScope.pageInfo.pageNum}&keyword=${param.keyword}" class="btn btn-success btn-xs"><i class=" glyphicon glyphicon-check"></i></a>
    ```
2. 新建 assign-role.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <!DOCTYPE html>
    <html lang="zh-CN">
    <%@include file="include-head.jsp" %>

    <body>
    <%@include file="include-nav.jsp" %>
    <div class="container-fluid">
        <div class="row">
            <%@include file="include-sidebar.jsp" %>
            <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">
                <ol class="breadcrumb">
                    <li><a href="#">首页</a></li>
                    <li><a href="#">数据列表</a></li>
                    <li class="active">分配角色</li>
                </ol>
                <div class="panel panel-default">
                    <div class="panel-body">
                        <form action="assign/do/role/assign.do" method="post" role="form" class="form-inline">
                            <input type="hidden" name="adminId" value="${param.adminId}">
                            <input type="hidden" name="pageNum" value="${param.pageNum}">
                            <input type="hidden" name="keyword" value="${param.keyword}">
                            <div class="form-group">
                                <label>未分配角色列表</label><br>
                                <select class="form-control" multiple="" size="10" style="width:100px;overflow-y:auto;">
                                    <c:forEach items="${requestScope.unAssignedRoleList}" var="role">
                                        <option value="${role.id}">${role.name }</option>
                                    </c:forEach>
                                </select>
                            </div>
                            <div class="form-group">
                                <ul>
                                    <li class="btn btn-default glyphicon glyphicon-chevron-right"></li>
                                    <br>
                                    <li class="btn btn-default glyphicon glyphicon-chevron-left" style="margin-top:20px;"></li>
                                </ul>
                            </div>
                            <div class="form-group" style="margin-left:40px;">
                                <label>已分配角色列表</label><br>
                                <select name="roleList" class="form-control" multiple="multiple" size="10" style="width:100px;overflow-y:auto;">
                                    <c:forEach items="${requestScope.assignedRoleList}" var="role">
                                        <option value="${role.id}">${role.name }</option>
                                    </c:forEach>
                                </select>
                            </div>
                            <br/><br><br>
                            <button type="submit" style="width: 150px;" class="btn btn-lg btn-success btn-block">提交</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </body>
    </html>
    ```

3. assign-role.jsp：实现角色左右移动
    ```js
    $(function () {
        // 1.左侧选择移到右侧列表
        $("#toRightBtn").click(function (){
            /*
            select 是标签选择器
            :eq(0)表示选择页面上的第一个
            :eq(1)表示选择页面上的第二个
            “>” 表示选择子元素
            :selected 表示选择“被选中的” option
            appendTo()能够将 jQuery 对象追加到指定的位置
            */
            $("select:eq(0)>option:selected").appendTo("select:eq(1)");
        });

        // 2.右侧选择移到左侧列表
        $("#toLeftBtn").click(function () {
            $("select:eq(1)>option:selected").appendTo("select:eq(0)");
        });
    })
    ```
    给两个箭头添加id
    ```html
    <li id="toRightBtn" class="btn btn-default glyphicon glyphicon-chevron-right"></li>
    <br>
    <li id="toLeftBtn" class="btn btn-default glyphicon glyphicon-chevron-left" style="margin-top:20px;"></li>
    ```

### 6.2.4 后端代码：实现分配
1. handler
    ```java
    @RequestMapping("/assign/do/role/assign.do")
    public String saveAdminRoleRelationship(@RequestParam("adminId") Integer adminId,
                                            @RequestParam("pageNum") Integer pageNum,
                                            @RequestParam("keyword") String keyword,
                                            //允许用户在页面上取消所有角色，故该参数可以不提交
                                            @RequestParam(value = "roleIdList", required = false) List<Integer> roleIdList)  {

        adminService.saveAdminRoleRelationship(adminId, roleIdList);

        return "redirect:/admin/get/page.do?pageNum="+pageNum+"&keyword="+keyword;
    }
    ```

2. AdminService
    ```java
    void saveAdminRoleRelationship(Integer adminId, List<Integer> roleIdList);
    ```
    ```java
    @Override
    public void saveAdminRoleRelationship(Integer adminId, List<Integer> roleIdList) {

        // 1. 根据adminId删除旧的关联关系数据
        adminMapper.deleteOldRelationship(adminId);

        // 2.根据roleIdList和adminId保存新的关联关系
        if (roleIdList != null && roleIdList.size()>0) {
            adminMapper.insertNewRelationship(adminId, roleIdList);
        }
    }
    ```

3. AdminMapper
    ```java
    void deleteOldRelationship(Integer adminId);

    void insertNewRelationship(@Param("adminId") Integer adminId, @Param("roleIdList") List<Integer> roleIdList);
    ```
    ```xml
    <delete id="deleteOldRelationship">
        delete from inner_admin_role where admin_id=#{adminId}
    </delete>
    <insert id="insertNewRelationship">
        insert into inner_admin_role(admin_id, role_id) VALUES
        <foreach collection="roleIdList" item="roleId" separator=",">(#{adminId},#{roleId})</foreach>
    </insert>
    ```

4. bug修正: assign-role.jsp
    ```js
    $("#submitBtn").click(function () {
        $("select:eq(1)>option").prop("selected","selected");

        // 为了看到上面代码的效果， 暂时不让表单提交
        // return false;
    });
    ```
    ```html
    <button id="submitBtn" type="submit" style="width: 150px;" class="btn btn-lg btn-success btn-block">提交</button>
    ```

## 6.3 给 Role 分配 Auth
### 6.3.1 目标思路
1. 目标：把角色和权限的关联关系保存到数据库
2. 思路
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/6-3-1.png)
3. 说明：
   * 权限管理菜单下应还有一个权限维护
   * 权限维护与菜单维护类似，树形结构显示，增删改查功能，为节省时间，本项目中省略

### 6.3.2 准备工作
1. 新建数据库表
    ```sql
    # t_auth
    CREATE TABLE `t_auth` (
                            `id` INT(11) NOT NULL AUTO_INCREMENT,
                            `name` VARCHAR(200) DEFAULT NULL,
                            `title` VARCHAR(200) DEFAULT NULL,
                            `category_id` INT(11) DEFAULT NULL,
                            PRIMARY KEY (`id`)
    );
    INSERT INTO t_auth(id,`name`,title,category_id) VALUES(1,'','用户模块',NULL);
    INSERT INTO t_auth(id,`name`,title,category_id) VALUES(2,'user:delete','删除',1);
    INSERT INTO t_auth(id,`name`,title,category_id) VALUES(3,'user:get','查询',1);
    INSERT INTO t_auth(id,`name`,title,category_id) VALUES(4,'','角色模块',NULL);
    INSERT INTO t_auth(id,`name`,title,category_id) VALUES(5,'role:delete','删除',4);
    INSERT INTO t_auth(id,`name`,title,category_id) VALUES(6,'role:get','查询',4);
    INSERT INTO t_auth(id,`name`,title,category_id) VALUES(7,'role:add','新增',4);
    ```
   * name 字段： 给资源分配权限或给角色分配权限时使用的具体值， 将来做权限验证也是使用 name 字段的值来进行比对。 建议使用英文。
   * title 字段： 在页面上显示， 让用户便于查看的值。 建议使用中文。
   * category_id 字段： 关联到当前权限所属的分类。 这个关联不是到其他表关联， 而是就在当前表内部进行关联， 关联其他记录。 所以说t_auth 表中是依靠 category_id 字段建立了“节点” 之间的父子关系。
2. 逆向工程：
   * generatorConfig.xml
        ```xml
        <!--  数据库表名字和我们的 entity 类对应的映射指定 -->
        <table tableName="t_auth" domainObjectName="Auth" />
        ```
   *  mybatis-generator:generate
3. 给Auth类添加无参构造器和有参构造器
4. 资源归位
5. 新建AuthService接口及其实现类
    ```java
    public interface AuthService {
    }
    ```
    ```java
    @Service
    public class AuthServiceImpl implements AuthService {

        @Autowired
        private AuthMapper authMapper;
    }
    ```
6. 在 AssignHandler中装配
    ```java
    @Autowired
    private AuthService authService;
    ```

### 6.3.3 过渡工作一 — 获取全部权限生成树形结构
1. 修改 my-role.js
    ```js
    var checkBtn = "<button id='" + roleId + "' type=\"button\" class=\"btn btn-success btn-xs checkBtn\"><i class=\" glyphicon glyphicon-check\"></i></button>";
    ```
2. 准备模态框：新建 modal-role-assign-auth.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div id="assignModal" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal"
                            aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                    <h4 class="modal-title">尚筹网系统弹窗</h4>
                </div>
                <div class="modal-body">
                    <ul id="authTreeDemo" class="ztree"></ul>
                </div>
                <div class="modal-footer">
                    <button id="assignBtn" type="button" class="btn btn-primary">好的，我设置好了！执行分配！</button>
                </div>
            </div>
        </div>
    </div>
    ```

3. 在 role-page.jsp中引入模态框并绑定单击响应函数
    ```jsp
    <link rel="stylesheet" href="ztree/zTreeStyle.css">
    <script type="text/javascript" src="ztree/jquery.ztree.all-3.5.min.js"></script>
    <script type="text/javascript" src="js/my-role.js"></script>
    ```
    ```js
    // 13.给分配权限按钮绑定单击响应函数
    $("#rolePageBody").on("click", ".checkBtn", function() {

        // 打开模态框
        $("#assignModal").modal("show");

        //  在模态框中装载Auth的树形结构数据
        fillAuthTree();
    });
    ```
    ```jsp
    <%@include file="modal-role-assign-auth.jsp"%>
    ```
4. my-role.js中声明函数
    ```js
    // 声明专门的函数用来在分配Auth的模态框中显示Auth的树形结构数据
    function fillAuthTree(){

        // a.发送ajax请求查询Auth数据
        var ajaxReturn = $.ajax({
            url: "assign/get/all/auth.do",
            type: "post",
            dataType: "json",
            async: false
        });

        if (ajaxReturn.status !== 200) {
            layer.msg("请求处理出错！响应状态码是："+ajaxReturn.status+" 说明："+ajaxReturn.statusText);
            return ;
        }

        // b.从响应结果中获取Auth的JSON数据
        // 从服务器端查询到的 list 不需要组装成树形结构， 这里我们交给 zTree 去组装
        var authList = ajaxReturn.responseJSON.data;

        // c.准备对 zTree 进行设置的 JSON 对象
        var setting = {
            data: {
                simpleData: {
                    // 开启简单JSON功能
                    enable: true,

                    // 使用 categoryId属性关联父节点，不用pid
                    pIdKey: "categoryId"
                },
                key: {
                    // 使用 title 属性显示节点名称， 不用默认的 name 作为属性名
                    name: "title"
                }
            },
            check: {
                enable: true
            }
        };

        // d.生成树形结构
        $.fn.zTree.init($("#authTreeDemo"), setting, authList);

        // 获取 zTreeObj 对象
        let zTreeObj = $.fn.zTree.getZTreeObj("authTreeDemo");

        // 调用 zTreeObj 对象的方法， 把节点展开
        zTreeObj.expandAll(true);

        // e. 查询已分配的Auth的id组成的数组

        // f. 根据 authIdArray 把树形结构中对应的节点勾选上
    }
    ```
5. AssignHandler
    ```java
    @ResponseBody
    @RequestMapping("/assign/get/all/auth.do")
    public ResultEntity<List<Auth>> getAllAuth() {

        List<Auth> authList = authService.getAll();
        return ResultEntity.successWithData(authList);
    }
    ```
6. service
    ```java
    List<Auth> getAll();
    ```
    ```java
    @Override
    public List<Auth> getAll() {
        return authMapper.selectByExample(new AuthExample());
    }
    ```

### 6.3.4 过渡工作二 — 获取角色拥有的权限
1. 创建角色到权限之间的关联表
    ```sql
    DROP TABLE IF EXISTS inner_role_auth;
    CREATE TABLE inner_role_auth
    (
        id INT AUTO_INCREMENT,
        role_id INT NULL ,
        auth_id INT NULL ,
        PRIMARY KEY (id)
    );
    ```
2. handler
    ```java
    @ResponseBody
    @RequestMapping("/assign/get/assigned/auth/id/by/role/id.do")
    public ResultEntity<List<Integer>> getAssignAuthIdByRoleId(@RequestParam("roleId") Integer roleId) {

        List<Integer> authIdList = authService.getAssignedAuthIdByRoleId(roleId);
        return ResultEntity.successWithData(authIdList);
    }
    ```
4. AuthService
    ```java
    List<Integer> getAssignedAuthIdByRoleId(Integer roleId);
    ```
    ```java
    @Override
    public List<Integer> getAssignedAuthIdByRoleId(Integer roleId) {
        return authMapper.selectAssignedAuthIdByRoleId(roleId);
    }
    ```
5. AuthMapper
    ```java
    List<Integer> selectAssignedAuthIdByRoleId(Integer roleId);
    ```
    ```xml
    <select id="selectAssignedAuthIdByRoleId" resultType="int">
        select auth_id from inner_role_auth where role_id=#{roleId}
    </select>
    ```

6. 先在role-page.jsp中配置全局变量`window.roleId`：
    ```js
    // 13.给分配权限按钮绑定单击响应函数
    $("#rolePageBody").on("click", ".checkBtn", function(){
        window.roleId = this.id;
        ...
    ```
7. my-role.js函数功能实现
    ```js
    // e.查询已分配的Auth的id组成的数组
    console.log("window.roleId="+window.roleId)
    ajaxReturn = $.ajax({
        url: "assign/get/assigned/auth/id/by/role/id.do",
        type: "post",
        data:{
            roleId: window.roleId
        },
        dataType: "json",
        async: false
    });
    if(ajaxReturn.status !== 200) {
        layer.msg(" 请 求 处 理 出 错 ！ 响 应 状 态 码 是 ： "+ajaxReturn.status+" 说 明 是 ："+ajaxReturn.statusText);
        return ;
    }
    // 从响应结果中获取 authIdArray
    var authIdArray = ajaxReturn.responseJSON.data;

    // f.根据 authIdArray 把树形结构中对应的节点勾选上
    // ①遍历 authIdArray
    for(var i = 0; i < authIdArray.length; i++) {
        var authId = authIdArray[i];
        // ②根据 id 查询树形结构中对应的节点
        var treeNode = zTreeObj.getNodeByParam("id", authId);
        // ③将 treeNode 设置为被勾选
        // checked 设置为 true 表示节点勾选
        var checked = true;
        // checkTypeFlag 设置为 false， 表示不“联动”， 不联动是为了避免把不该勾选的勾选上
        var checkTypeFlag = false;
        // 执行
        zTreeObj.checkNode(treeNode, checked, checkTypeFlag);
    }
    ```

### 6.3.5 执行分配
1. 给分配权限模态框中的“分配” 按钮绑定单击响应函数：role-page.jsp
    ```js
    // 14.给分配权限模态框中的“分配” 按钮绑定单击响应函数
    $("#assignBtn").click(function () {
        // ①收集树形结构的各个节点中被勾选的节点
        // [1]声明一个专门的数组存放 id
        var authIdArray = [];
        // [2]获取 zTreeObj 对象
        var zTreeObj = $.fn.zTree.getZTreeObj("authTreeDemo");
        // [3]获取全部被勾选的节点
        var checkedNodes = zTreeObj.getCheckedNodes();
        // [4]遍历 checkedNodes
        for (var i = 0; i < checkedNodes.length; i++) {
            var checkedNode = checkedNodes[i];
            var authId = checkedNode.id;
            authIdArray.push(authId);
        }
        // ②发送请求执行分配
        var requestBody = {
            "authIdArray": authIdArray,
            // 为了服务器端 handler 方法能够统一使用 List<Integer>方式接收数据， roleId 也存入数组
            "roleId": [window.roleId]
        };
        requestBody = JSON.stringify(requestBody);
        $.ajax({
            url: "assign/do/role/assign/auth.do",
            type: "post",
            data: requestBody,
            contentType: "application/json;charset=UTF-8",
            dataType: "json",
            success: function (response) {
                var result = response.result;
                if (result === "SUCCESS") {
                    layer.msg("操作成功！ ");
                }
                if
                (result === "FAILED") {
                    layer.msg("操作失败！ " + response.message);
                }
            },
            error: function (response) {
                layer.msg(response.status + " " + response.statusText);
            }
        });
        $("#assignModal").modal("hide");
    });
    ```
2. handler
    ```java
    @ResponseBody
    @RequestMapping("/assign/do/role/assign/auth.do")
    public ResultEntity<String>  saveRoleAuthRelathinship(@RequestBody Map<String, List<Integer>> map) {

        authService.saveRoleAuthRelationship(map);
        return ResultEntity.successWithoutData();
    }
    ```

3. service
    ```java
    void saveRoleAuthRelationship(Map<String, List<Integer>> map);
    ```
    ```java
    @Override
    public void saveRoleAuthRelationship(Map<String, List<Integer>> map) {

        // 1.获取roleId的值
        List<Integer> roleIdList = map.get("roleId");
        Integer roleId = roleIdList.get(0);

        // 2. 删除旧的关联数据
        authMapper.deleteOldRelationship(roleId);

        // 3.获取 authIdList
        List<Integer> authIdList = map.get("authIdArray");

        // 4.判断 authIdList是否有效
        if (authIdList != null && authIdList.size()>0) {
            authMapper.insertNewRelationship(roleId,authIdList);
        }
    }
    ```
4. mapper
    ```java
    void deleteOldRelationship(Integer roleId);

    void insertNewRelationship(@Param("roleId") Integer roleId, @Param("authIdList") List<Integer> authIdList);
    ```
    ```xml
    <delete id="deleteOldRelationship">
        delete from inner_role_auth where role_id=#{roleId}
    </delete>
    <insert id="insertNewRelationship">
        insert into inner_role_auth(role_id, auth_id) VALUES
        <foreach collection="authIdList" item="authId" separator=",">(#{roleId}, #{authId})</foreach>
    </insert>
    ```

### 6.4 给 Menu 分配 Auth
与上面类似，本项目中省略


