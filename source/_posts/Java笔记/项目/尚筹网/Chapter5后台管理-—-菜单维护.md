---
title: Chapter5后台管理 — 菜单维护
excerpt: 菜单维护：页面显示树形结构、菜单增删改
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
abbrlink: dd32bff4
date: 2021-01-09 01:16:29
updated: 2021-01-10 01:01:36
subtitle:
---
## 5.1 页面显示树形结构
### 5.1.1 树形结构
1. 树形结构示意
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/5-1-5.png)
   * 约定：整个树形结构节点的层次最多只能有 3 级。
2. 数据库中节点关联：通过pid字段关联到父节点id
3. Java中树形结构
   * 实现：在 Menu 类中使用 List<Menu> children 属性存储当前节点的子节点
   * 属性（配合zTree）
      * pid：父节点
      * name：节点名称
      * icon：节点图标
      * open：节点是否默认打开
      * url：节点跳转位置
4. 不同级别节点规则
   * level0：根节点
     * 添加子节点
   * level1：分支节点
     * 修改
     * 添加子节点
     * 删除：有子节点时允许
   * level2：叶节点
     * 修改
     * 删除

### 5.1.2 目标思路
1. 目标：将数据库中查询得到的数据到页面上显示出来
2. 思路：  
   数据库查询全部 -> Java对象组装 -> 页面上使用 zTree 显示

### 5.1.3 准备工作
1. 数据库表
    ```mysql
    # 创建菜单表
    DROP TABLE IF EXISTS t_menu;
    CREATE TABLE t_menu
    (
        id      INT(11) NOT NULL AUTO_INCREMENT,
        pid     INT(11),
        name    VARCHAR(200),
        url     VARCHAR(200),
        icon    VARCHAR(200),
        PRIMARY KEY (id)
    );
    ```
    ```mysql
    # 插入数据
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('1',NULL,'系统权限菜单','glyphicon glyphicon-th-list',NULL);
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('2','1',' 控 制 面 板 ','glyphicon glyphicon-dashboard','main.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('3','1','权限管理','glyphicon glyphicon glyphicon-tasks',NULL);
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('4','3',' 用 户 维 护 ','glyphicon glyphicon-user','user/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('5','3',' 角 色 维 护 ','glyphicon glyphicon-king','role/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('6','3',' 菜 单 维 护 ','glyphicon glyphicon-lock','permission/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('7','1',' 业 务 审 核 ','glyphicon glyphicon-ok',NULL);
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('8','7',' 实 名 认 证 审 核 ','glyphicon glyphicon-check','auth_cert/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('9','7',' 广 告 审 核 ','glyphicon glyphicon-check','auth_adv/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('10','7',' 项 目 审 核 ','glyphicon glyphicon-check','auth_project/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('11','1',' 业 务 管 理 ','glyphicon glyphicon-th-large',NULL);
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('12','11',' 资 质 维 护 ','glyphicon glyphicon-picture','cert/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('13','11',' 分 类 管 理 ','glyphicon glyphicon-equalizer','certtype/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('14','11',' 流 程 管 理 ','glyphicon glyphicon-random','process/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('15','11',' 广 告 管 理 ','glyphicon glyphicon-hdd','advert/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('16','11',' 消 息 模 板 ','glyphicon glyphicon-comment','message/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('17','11',' 项 目 分 类 ','glyphicon glyphicon-list','projectType/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('18','11',' 项 目 标 签 ','glyphicon glyphicon-tags','tag/index.htm');
    INSERT INTO `t_menu` (`id`, `pid`, `name`, `icon`, `url`) VALUES ('19','1',' 参 数 管 理 ','glyphicon glyphicon-list-alt','param/index.htm');
    ```
2. 逆向工程
   * 修改generatorConfig.xml
        ```xml
        <!--  数据库表名字和我们的 entity 类对应的映射指定 -->
        <table tableName="t_menu" domainObjectName="Menu" />
        ```
   * mybatis-generator:generate
3. 给Menu类增加属性、getter、setter等
    ```java
    public class Menu {

        // 主键
        private Integer id;

        // 父节点id
        private Integer pid;

        // 节点名称
        private String name;

        // 节点附带url地址，点击菜单项时要跳转的地址
        private String url;

        // 节点图标样式
        private String icon;

        // 存储子节点的集合，初始化以避免空指针异常
        private List<Menu> children = new ArrayList<>();

        // 控制节点是否默认展开，设置为true表示默认打开
        private Boolean open = true;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public Integer getPid() {
            return pid;
        }

        public void setPid(Integer pid) {
            this.pid = pid;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name == null ? null : name.trim();
        }

        public String getUrl() {
            return url;
        }

        public void setUrl(String url) {
            this.url = url == null ? null : url.trim();
        }

        public String getIcon() {
            return icon;
        }

        public void setIcon(String icon) {
            this.icon = icon == null ? null : icon.trim();
        }

        public List<Menu> getChildren() {
            return children;
        }

        public Boolean getOpen() {
            return open;
        }

        public void setChildren(List<Menu> children) {
            this.children = children;
        }

        public void setOpen(Boolean open) {
            this.open = open;
        }

        public Menu() {
        }

        public Menu(Integer id, Integer pid, String name, String url, String icon, List<Menu> children, Boolean open) {
            this.id = id;
            this.pid = pid;
            this.name = name;
            this.url = url;
            this.icon = icon;
            this.children = children;
            this.open = open;
        }

        @Override
        public String toString() {
            return "Menu{" +
                    "id=" + id +
                    ", pid=" + pid +
                    ", name='" + name + '\'' +
                    ", url='" + url + '\'' +
                    ", icon='" + icon + '\'' +
                    ", children=" + children +
                    ", open=" + open +
                    '}';
        }
    }
    ```
4. 资源归位：将生成的四个类移动到相应的模块中
5. 新建service  
    MenuService
    ```java
    public interface MenuService {
        
    }
    ```
    MenuServiceImpl
    ```java
    @Service
    public class MenuServiceImpl implements MenuService {

        @Autowired
        private MenuMapper menuMapper;
    }
    ```
6. handler
    ```java
    @Controller
    public class MenuHandler {

        @Autowired
        private MenuService menuService;

    }
    ```

### 5.1.4 代码
1. handler
    ```java
    @ResponseBody
    @RequestMapping("/menu/get/whole/tree.do")
    public ResultEntity<Menu> getWholeTreeNew() {

        // 1.查询全部的Menu对象
        List<Menu> menuList = menuService.getAll();

        // 2.声明一个变量来存储找到的根节点
        Menu root = null;

         // 3.创建 Map 对象用来存储 id 和 Menu 对象的对应关系便于查找父节点
        Map<Integer,Menu> menuMap = new HashMap<>();

         // 4.遍历 menuList 填充 menuMap
        for (Menu menu: menuList) {

            Integer id = menu.getId();

            menuMap.put(id, menu);
        }

        // 5.再次遍历 menuList 查找根节点、组装父子节点
        for (Menu menu: menuList) {

            // 5.1 获取当前对象pid
            Integer pid = menu.getPid();

            // 5.2 检查pid是否为null
            if (pid == null) {
                // 将当前对象赋给root
                root = menu;
                continue;
            }
            // 5.3 如果pid不为null，说明当前节点有父节点，找到父节点就可以进行组装
            menuMap.get(pid).getChildren().add(menu);
        }
        return ResultEntity.successWithData(root);
    }
    ```

2. service
    ```java
    List<Menu> getAll();
    ```
    ```java
    @Override
    public List<Menu> getAll() {

        List<Menu> menuList = menuMapper.selectByExample(new MenuExample());
        return menuList;
    }
    ```
3. mvc配置文件
    ```xml
    <mvc:view-controller path="/menu/to/page.do" view-name="menu-page"/>
    ```
4. 新建menu-page.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <!DOCTYPE html>
    <html lang="zh-CN">
    <%@include file="include-head.jsp" %>
    <link rel="stylesheet" href="ztree/zTreeStyle.css"/>
    <script type="text/javascript" src="ztree/jquery.ztree.all-3.5.min.js"></script>
    <script type="text/javascript">
        $(function (){

            // 1.创建JSON对象用于存储对zTree所做的设置
            var setting = {};

            // 2.准备生成树形结构的JSON数据
            var zNodes =[
                { name:"父节点1 - 展开", open:true,
                    children: [
                        { name:"父节点11 - 折叠",
                            children: [
                                { name:"叶子节点111"},
                                { name:"叶子节点112"},
                                { name:"叶子节点113"},
                                { name:"叶子节点114"}
                            ]},
                        { name:"父节点12 - 折叠",
                            children: [
                                { name:"叶子节点121"},
                                { name:"叶子节点122"},
                                { name:"叶子节点123"},
                                { name:"叶子节点124"}
                            ]},
                        { name:"父节点13 - 没有子节点", isParent:true}
                    ]},
                { name:"父节点2 - 折叠",
                    children: [
                        { name:"父节点21 - 展开", open:true,
                            children: [
                                { name:"叶子节点211"},
                                { name:"叶子节点212"},
                                { name:"叶子节点213"},
                                { name:"叶子节点214"}
                            ]},
                        { name:"父节点22 - 折叠",
                            children: [
                                { name:"叶子节点221"},
                                { name:"叶子节点222"},
                                { name:"叶子节点223"},
                                { name:"叶子节点224"}
                            ]},
                        { name:"父节点23 - 折叠",
                            children: [
                                { name:"叶子节点231"},
                                { name:"叶子节点232"},
                                { name:"叶子节点233"},
                                { name:"叶子节点234"}
                            ]}
                    ]},
                { name:"父节点3 - 没有子节点", isParent:true}
            ];

            // 3.初始化树形结构
            $.fn.zTree.init($("#treeDemo"), setting, zNodes);


        })
    </script>

    <body>
    <%@include file="include-nav.jsp" %>
    <div class="container-fluid">
        <div class="row">
            <%@include file="include-sidebar.jsp" %>
            <div class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">

                <div class="panel panel-default">
                    <div class="panel-heading"><i class="glyphicon glyphicon-th-list"></i> 权限菜单列表 <div style="float:right;cursor:pointer;" data-toggle="modal" data-target="#myModal"><i class="glyphicon glyphicon-question-sign"></i></div></div>
                    <div class="panel-body">
                        <!-- zTree动态生成的标签所依附的静态节点 -->
                        <ul id="treeDemo" class="ztree"></ul>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </body>
    </html>
    ```
5. 修改include-sidebar.jsp
    ```jsp
    <a href="menu/to/page.do"><i class="glyphicon glyphicon-lock"></i> 菜单维护</a>
    ```
6. 修改menu-page.jsp的js代码
    ```jsp
    <script type="text/javascript">
        $(function (){
            // 1.准备生成树形结构的JSON数据，数据来源是发送Ajax请求得到
            $.ajax({
                url: "menu/get/whole/tree.do",
                type: "post",
                dataType: "json",
                success: function (resp) {
                    var result = resp.result;
                    if (result === "SUCCESS") {

                        // 2.创建json对象用于存储对zTree所做的设置
                        var setting = {
                            view:{
                                addDiyDom: myAddDiyDom
                            },
                            data: {
                                key:{
                                    url: "nothing", // url值改为不存在的属性名，则点击页面响应菜单时不会跳转
                                }
                            }
                        };

                        // 3.从响应体中获取用来生成树形结构的JSON数据
                        var zNodes = resp.data;

                        // 4.初始化树形结构
                        $.fn.zTree.init($("#treeDemo"), setting, zNodes);
                    }
                    if (result === "FAILED") {
                        layer.msg(resp.message);
                    }
                },
            });

        })
    </script>
    ```

7. 新建 my-menu.js 文件
    ```js
    function myAddDiyDom(treeId, treeNode) {

        // treeId是整个树形结构附着的ul标签的id
        console.log("treeId="+treeId);

        // 当前树形节点的全部数据、
        console.log(treeNode);

    /*    zTree 生成 id 的规则
        例子： treeDemo_7_ico
        解析： ul 标签的 id_当前节点的序号_功能
        提示： “ul 标签的 id_当前节点的序号” 部分可以通过访问 treeNode 的 tId 属性得到
        根据 id 的生成规则拼接出来 span 标签的 id
    */
        var spanId = treeNode.tId + "_ico";

        // 根据控制图标的span标签的id找到这个span标签
        // 删除旧的class
        // 添加新的class
        $("#"+spanId).removeClass().addClass(treeNode.icon);
    }
    ```
8. menu-page.jsp头部引入
    ```jsp
    <html lang="zh-CN">
    <%@include file="include-head.jsp" %>
    <link rel="stylesheet" href="ztree/zTreeStyle.css"/>
    <script type="text/javascript" src="ztree/jquery.ztree.all-3.5.min.js"></script>
    <script type="text/javascript" src="js/my-menu.js"></script>
    ```

### 5.1.5 动态效果
1. 说明
   * 鼠标移到节点上时，显示对应节点拥有权限的按钮（增删等）
   * 鼠标移开时自动消失
2. my-menu.js中添加函数
    ```js
    // 鼠标移入节点范围时添加按钮组
    function myAddHoverDom(treeId, treeNode) {

        // 按钮组的标签结构： <span><a><i></i></a><a><i></i></a></span>
        // 按钮组出现的位置： 节点中 treeDemo_n_a 超链接的后面

        // 为了在需要移除按钮组的时候能够精确定位到按钮组所在 span， 需要给 span 设置有规律的 id
        var btnGroupId = treeNode.tId + "_btnGrp";
        // 判断一下以前是否已经添加了按钮组
        if($("#"+btnGroupId).length > 0) {
            return ;
        }
        var addBtn = "<a id='"+treeNode.id+"' class='btn btn-info dropdown-toggle btn-xs' style='margin-left:10px;padding-top:0px;' href='#' title='添加子节点'>&nbsp;&nbsp;<i class='fa fa-fw fa-plus rbg '></i></a>";
        var removeBtn = "<a id='"+treeNode.id+"' class='btn btn-info dropdown-toggle btn-xs' style='margin-left:10px;padding-top:0px;' href='#' title=' 删 除 节 点 '>&nbsp;&nbsp;<i class='fa fa-fw fa-times rbg '></i></a>";
        var editBtn = "<a id='"+treeNode.id+"' class='btn btn-info dropdown-toggle btn-xs' style='margin-left:10px;padding-top:0px;' href='#' title=' 修 改 节 点 '>&nbsp;&nbsp;<i class='fa fa-fw fa-edit rbg '></i></a>";

        // 获取当前节点的级别数据
        var level = treeNode.level;
        // 声明变量存储拼装好的按钮代码
        var btnHTML = "";
        // 判断当前节点的级别
        if(level === 0) {
        // 级别为 0 时是根节点， 只能添加子节点
            btnHTML = addBtn;
        }
        if (level === 1) {
            // 级别为 1 时是分支节点， 可以添加子节点、 修改
            btnHTML = addBtn + " " + editBtn;
            // 获取当前节点的子节点数量
            var length = treeNode.children.length;
            // 如果没有子节点， 可以删除
            if(length === 0) {
                btnHTML = btnHTML + " " + removeBtn;
            }
        }
        if (level === 2) {
            // 级别为 2 时是叶子节点， 可以修改、 删除
            btnHTML = editBtn + " " + removeBtn;
        }
        // 找到附着按钮组的超链接
        var anchorId = treeNode.tId + "_a";
        // 执行在超链接后面附加 span 元素的操作
        $("#"+anchorId).after("<span id='"+btnGroupId+"'>"+btnHTML+"</span>");
    }

    // 鼠标离开节点范围时删除按钮组
    function myRemoveHoverDom(treeId, treeNode) {

        // 拼接按钮组的 id
        var btnGroupId = treeNode.tId + "_btnGrp";
        // 移除对应的元素
        $("#"+btnGroupId).remove();
    }
    ```

3. menu-page.jsp中调用
    ```js
    $.ajax({
        url: "menu/get/whole/tree.do",
        type: "post",
        dataType: "json",
        success: function (resp) {
            var result = resp.result;
            if (result === "SUCCESS") {

                // 2.创建json对象用于存储对zTree所做的设置
                var setting = {
                    view: {
                        addDiyDom: myAddDiyDom,
                        addHoverDom: myAddHoverDom,
                        removeHoverDom: myRemoveHoverDom
                    },
                    data: {
                        key:{
                            url: "nothing", // url值改为不存在的属性名，则点击页面响应菜单时不会跳转
                        }
                    }
                };

                // 3.从响应体中获取用来生成树形结构的JSON数据
                var zNodes = resp.data;

                // 4.初始化树形结构
                $.fn.zTree.init($("#treeDemo"), setting, zNodes);
            }
            if (result === "FAILED") {
                layer.msg(resp.message);
            }
        },
    });
    ````

### 5.1.6 封装
1. 将 menu-page.jsp 中的js代码封装到my-role.js的函数中
    ```js
    // 生成树形结构的函数
    function generateTree() {
        // 1.准备生成树形结构的JSON数据，数据来源是发送Ajax请求得到
        $.ajax({
            url: "menu/get/whole/tree.do",
            type: "post",
            dataType: "json",
            success: function (resp) {
                var result = resp.result;
                if (result === "SUCCESS") {

                    // 2.创建json对象用于存储对zTree所做的设置
                    var setting = {
                        view: {
                            addDiyDom: myAddDiyDom,
                            addHoverDom: myAddHoverDom,
                            removeHoverDom: myRemoveHoverDom
                        },
                        data: {
                            key:{
                                url: "nothing", // url值改为不存在的属性名，则点击页面响应菜单时不会跳转
                            }
                        }
                    };

                    // 3.从响应体中获取用来生成树形结构的JSON数据
                    var zNodes = resp.data;

                    // 4.初始化树形结构
                    $.fn.zTree.init($("#treeDemo"), setting, zNodes);
                }
                if (result === "FAILED") {
                    layer.msg(resp.message);
                }
            },
        });
    }
    ```
2. 在 menu-page.jsp 中调用
    ```jsp
    <script type="text/javascript">
        $(function (){

            // 调用专门封装好的函数初始化树形结构
            generateTree();
        })
    </script>
    ```

## 5.2 添加子节点
### 5.2.1 目标思路
1. 给当前节点添加子节点， 保存到数据库并刷新树形结构的显示，仍通过模态框来实现
2. 思路  
   ![图片：添加子节点-思路](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/5-2-1.png)
### 5.2.2 前端代码
1. my-role.js：给按钮加上class
    ```js
    var addBtn = "<a id='"+treeNode.id+"' class='addBtn btn btn-info dropdown-toggle btn-xs' style='margin-left:10px;padding-top:0px;' href='#' title='添加子节点'>&nbsp;&nbsp;<i class='fa fa-fw fa-plus rbg '></i></a>";
    var removeBtn = "<a id='"+treeNode.id+"' class='removeBtn btn btn-info dropdown-toggle btn-xs' style='margin-left:10px;padding-top:0px;' href='#' title=' 删 除 节 点 '>&nbsp;&nbsp;<i class='fa fa-fw fa-times rbg '></i></a>";
    var editBtn = "<a id='"+treeNode.id+"' class='editBtn btn btn-info dropdown-toggle btn-xs' style='margin-left:10px;padding-top:0px;' href='#' title=' 修 改 节 点 '>&nbsp;&nbsp;<i class='fafa-fw fa-edit rbg '></i></a>";
    ```
2. 新建文件 modal-menu-add.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div id="menuAddModal" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal"
                            aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                    <h4 class="modal-title">尚筹网系统弹窗</h4>
                </div>
                <form>
                    <div class="modal-body">
                        请输入节点名称：<input type="text" name="name" /><br />
                        请输入URL地址：<input type="text" name="url" /><br />
                        <i class="glyphicon glyphicon-th-list"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-th-list" />&nbsp;

                        <i class="glyphicon glyphicon-dashboard"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-dashboard" /> &nbsp;

                        <i class="glyphicon glyphicon glyphicon-tasks"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon glyphicon-tasks" /> &nbsp;

                        <i class="glyphicon glyphicon-user"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-user" /> &nbsp;

                        <i class="glyphicon glyphicon-king"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-king" /> &nbsp;

                        <i class="glyphicon glyphicon-lock"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-lock" /> &nbsp;

                        <i class="glyphicon glyphicon-ok"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-ok" /> &nbsp;

                        <i class="glyphicon glyphicon-check"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-check" /> &nbsp;

                        <i class="glyphicon glyphicon-th-large"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-th-large" /> <br />

                        <i class="glyphicon glyphicon-picture"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-picture" /> &nbsp;

                        <i class="glyphicon glyphicon-equalizer"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-equalizer" /> &nbsp;

                        <i class="glyphicon glyphicon-random"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-random" /> &nbsp;

                        <i class="glyphicon glyphicon-hdd"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-hdd" /> &nbsp;

                        <i class="glyphicon glyphicon-comment"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-comment" /> &nbsp;

                        <i class="glyphicon glyphicon-list"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-list" /> &nbsp;

                        <i class="glyphicon glyphicon-tags"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-tags" /> &nbsp;

                        <i class="glyphicon glyphicon-list-alt"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-list-alt" /> &nbsp;
                        <br />

                    </div>
                    <div class="modal-footer">
                        <button id="menuSaveBtn" type="button" class="btn btn-default"><i class="glyphicon glyphicon-plus"></i> 保存</button>
                        <button id="menuResetBtn" type="reset" class="btn btn-primary"><i class="glyphicon glyphicon-refresh"></i> 重置</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
    ```
3. menu-page.jsp 中给按钮绑定单击响应函数
    ```jsp
    <script type="text/javascript">
        $(function (){

            // 调用专门封装好的函数初始化树形结构
            generateTree();

            // 给添加子节点按钮绑定单击响应函数
            $("#treeDemo").on("click",".addBtn",function(){

                // 将当前节点的id，作为新节点的pid保存到全局变量
                window.pid = this.id;

                // 打开模态框
                $("#menuAddModal").modal("show");

                return false;
            });

            // 给添加子节点的模态框中的保存按钮绑定单击响应函数
            $("#menuSaveBtn").click(function(){

                // 收集表单项中用户输入的数据
                var name = $.trim($("#menuAddModal [name=name]").val());
                var url = $.trim($("#menuAddModal [name=url]").val());

                // 单选按钮要定位到“被选中”的那一个(没有checked则会选择第一个)
                var icon = $("#menuAddModal [name=icon]:checked").val();

                // 发送Ajax请求
                $.ajax({
                    "url":"menu/save.do",
                    "type":"post",
                    "data":{
                        "pid": window.pid,
                        "name":name,
                        "url":url,
                        "icon":icon
                    },
                    "dataType":"json",
                    "success":function(response){
                        var result = response.result;

                        if(result == "SUCCESS") {
                            layer.msg("操作成功！");

                            // 重新加载树形结构，注意：要在确认服务器端完成保存操作后再刷新
                            // 否则有可能刷新不到最新的数据，因为这里是异步的
                            generateTree();
                        }

                        if(result == "FAILED") {
                            layer.msg("操作失败！"+response.message);
                        }
                    },
                    "error":function(response){
                        layer.msg(response.status+" "+response.statusText);
                    }
                });

                // 关闭模态框
                $("#menuAddModal").modal("hide");

                // 清空表单
                // jQuery对象调用click()函数，里面不传任何参数，相当于用户点击了一下
                $("#menuResetBtn").click();
            });

        })
    </script>
    ```
4. menu-page.jsp 页面引入模态框
    ```jsp
    <%@include file="/WEB-INF/modal-menu-add.jsp"%>
    <%@include file="/WEB-INF/modal-menu-edit.jsp"%>
    <%@include file="/WEB-INF/modal-menu-confirm.jsp"%>
    </body>
    ```

### 5.2.3 后端代码
1. handler
    ```java
    @ResponseBody
    @RequestMapping("/menu/save.do")
    public ResultEntity<String> saveMenu(Menu menu) {

        menuService.saveMenu(menu);

        return ResultEntity.successWithoutData();
    }
    ```

2. service
    ```java
    void saveMenu(Menu menu);
    ```
    ```java
    @Override
    public void saveMenu(Menu menu) {
        menuMapper.insert(menu);
    }
    ```

## 5.3 更新节点
### 5.3.1 目标思路
1. 目标：修改当前节点的基本属性。 不更换父节点。
2. 思路  
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/5-3-1.png)

### 5.3.1 前端代码
1. menu-page.jsp绑定单击响应函数
    ```js
    // 给编辑按钮绑定单击响应函数
    $("#treeDemo").on("click",".editBtn",function(){

        // 将当前节点的id保存到全局变量
        window.id = this.id;

        // 打开模态框
        $("#menuEditModal").modal("show");

        // 获取zTreeObj对象
        var zTreeObj = $.fn.zTree.getZTreeObj("treeDemo");

        // 根据id属性查询节点对象
        // 用来搜索节点的属性名
        var key = "id";

        // 用来搜索节点的属性值
        var value = window.id;

        var currentNode = zTreeObj.getNodeByParam(key, value);

        // 回显表单数据
        $("#menuEditModal [name=name]").val(currentNode.name);
        $("#menuEditModal [name=url]").val(currentNode.url);

        // 回显radio可以这样理解：被选中的radio的value属性可以组成一个数组，
        // 然后再用这个数组设置回radio，就能够把对应的值选中
        $("#menuEditModal [name=icon]").val([currentNode.icon]);

        return false;
    });

    // 给更新模态框中的更新按钮绑定单击响应函数
    $("#menuEditBtn").click(function(){

        // 收集表单数据
        var name = $("#menuEditModal [name=name]").val();
        var url = $("#menuEditModal [name=url]").val();
        var icon = $("#menuEditModal [name=icon]:checked").val();

        // 发送Ajax请求
        $.ajax({
            "url":"menu/update.do",
            "type":"post",
            "data":{
                "id": window.id,
                "name":name,
                "url":url,
                "icon":icon
            },
            "dataType":"json",
            "success":function(response){
                var result = response.result;

                if(result === "SUCCESS") {
                    layer.msg("操作成功！");

                    // 重新加载树形结构，注意：要在确认服务器端完成保存操作后再刷新
                    // 否则有可能刷新不到最新的数据，因为这里是异步的
                    generateTree();
                }

                if(result === "FAILED") {
                    layer.msg("操作失败！"+response.message);
                }
            },
            "error":function(response){
                layer.msg(response.status+" "+response.statusText);
            }
        });

        // 关闭模态框
        $("#menuEditModal").modal("hide");

    });
    ```
2. 新建 modal-edit-menu.jsp
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div id="menuEditModal" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal"
                        aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                    <h4 class="modal-title">尚筹网系统弹窗</h4>
                </div>
                <form>
                    <div class="modal-body">
                        请输入节点名称：<input type="text" name="name" /><br />
                        请输入URL地址：<input type="text" name="url" /><br />
                        <i class="glyphicon glyphicon-th-list"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-th-list" />&nbsp;
                        
                        <i class="glyphicon glyphicon-dashboard"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-dashboard" /> &nbsp;
                        
                        <i class="glyphicon glyphicon glyphicon-tasks"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon glyphicon-tasks" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-user"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-user" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-king"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-king" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-lock"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-lock" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-ok"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-ok" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-check"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-check" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-th-large"></i>
                        <input type="radio" name="icon" value="glyphicon glyphicon-th-large" /> <br /> 
                        
                        <i class="glyphicon glyphicon-picture"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-picture" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-equalizer"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-equalizer" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-random"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-random" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-hdd"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-hdd" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-comment"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-comment" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-list"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-list" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-tags"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-tags" /> &nbsp;
                        
                        <i class="glyphicon glyphicon-list-alt"></i> 
                        <input type="radio" name="icon" value="glyphicon glyphicon-list-alt" /> &nbsp;
                        <br />
                        
                    </div>
                    <div class="modal-footer">
                        <button id="menuEditBtn" type="button" class="btn btn-default"><i class="glyphicon glyphicon-edit"></i> 更新</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
    ```

### 5.3.2 后端代码
1. handler
    ```java
    @ResponseBody
    @RequestMapping("/menu/update.do")
    public ResultEntity<String> updateMenu(Menu menu) {

        menuService.updateMenu(menu);
        return ResultEntity.successWithoutData();
    }
    ```
2. service
    ```java
    void updateMenu(Menu menu);
    ```
    ```java
    @Override
    public void updateMenu(Menu menu) {
        menuMapper.updateByPrimaryKeySelective(menu);
    }
    ```

## 5.4 删除节点
### 5.4.1 目标思路
1. 目标：删除当前节点
2. 思路  
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/crowd_funding/5-4-1.png)

### 5.4.2 前端代码
1. menu-page.jsp绑定单击响应函数
    ```js
    // 给 "x" 按钮绑定单击响应函数
    $("#treeDemo").on("click",".removeBtn",function(){
        // 将当前节点的 id 保存到全局变量
        window.id = this.id;
        // 打开模态框
        $("#menuConfirmModal").modal("show");
        // 获取 zTreeObj 对象
        var zTreeObj = $.fn.zTree.getZTreeObj("treeDemo");
        // 根据 id 属性查询节点对象
        // 用来搜索节点的属性名
        var key = "id";
        // 用来搜索节点的属性值
        var value = window.id;
        var currentNode = zTreeObj.getNodeByParam(key, value);
        $("#removeNodeSpan").html("【<i class='"+currentNode.icon+"'></i>"+currentNode.name+"】");
        return false;
    });
    ```
    ```js
    // 给确认模态框中的 OK 按钮绑定单击响应函数
    $("#confirmBtn").click(function(){
        $.ajax({
            "url":"menu/remove.do",
            "type":"post",
            "data":{
                "id":window.id
            },
            "dataType":"json",
            "success":function(response){
                var result = response.result;
                if(result === "SUCCESS") {
                    layer.msg("操作成功！ ");

                    // 重新加载树形结构， 注意： 要在确认服务器端完成保存操作后再刷新
                    // 否则有可能刷新不到最新的数据， 因为这里是异步的
                    generateTree();
                } if
                (result === "FAILED") {
                    layer.msg("操作失败！ "+response.message);
                }
            },
            "error":function(response){
                layer.msg(response.status+" "+response.statusText);
            }
        });

        // 关闭模态框
        $("#menuConfirmModal").modal("hide");
    });
    ```
2. 新建modal-menu-confirm.jsp文件
    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <div id="menuConfirmModal" class="modal fade" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal"
                        aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                    <h4 class="modal-title">尚筹网系统弹窗</h4>
                </div>
                <form>
                    <div class="modal-body">
                        您真的要删除<span id="removeNodeSpan"></span>这个节点吗？
                    </div>
                    <div class="modal-footer">
                        <button id="confirmBtn" type="button" class="btn btn-danger"><i class="glyphicon glyphicon-ok"></i> OK</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
    ```

### 5.4.3 后端代码
1. handler
    ```java
    @ResponseBody
    @RequestMapping("/menu/remove.do")
    public ResultEntity<String> removeMenu(@RequestParam("id") Integer id) {

        menuService.removeMenu(id);
        return ResultEntity.successWithoutData();
    }
    ```

2. service
    ```java
    void removeMenu(Integer id);
    ```
    ```java
    @Override
    public void removeMenu(Integer id) {
        menuMapper.deleteByPrimaryKey(id);
    }
    ```


## 5.5 RestController注解
### 5.5.1 @ResponseBody
1. MenuHandler中每个方法上都使用了`@ResponseBody`注解，所以可以把它提取到类上
2. 代码：
    ```java
    @Controller
    @ResponseBody
    public class MenuHandler {
        ....
    ```

### 5.5.2 @RestController
1. 可以使用 `@RestController` 注解代替上面两个注解
2. 代码
    ```java
    @RestController
    public class MenuHandler {
        ...
    ```
3. RoleController 同样


## 5.6 遇到的问题
1. 乱码问题
   * 问题描述引入自己新建的js文件（my-role.js等）后页面显示乱码
   * 解决方案：使用其它编辑器软件打开该文件，以 UTF-8 with BOM 格式保存



