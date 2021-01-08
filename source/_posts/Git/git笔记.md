---
title: git笔记
excerpt: git基本操作
tags:
  - git
categories:
  - git
banner_img: /img/dog.png
index_img: /img/post/git.png
abbrlink: 518e617c
date: 2020-12-25 15:33:00
updated: 2020-12-26 23:08:57
subtitle:
---
## 1. 基本操作
### 1.1 设置签名
1. 全局
    ```sh
    git config --global user.name "用户名"
    git config --global user.email "邮箱"
    ```

2. 项目级别
    ```sh
    git config user.name "用户名"
    git config user.email "邮箱"
    ```

### 1.2 查看日志
1. 查看详细信息
   ```sh
    git log
   ```
   ![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/git-1-1.png)
   * 多屏显示时：空格翻页，b向上翻页，q退出  

2. 单行显示
    ```sh
    git log --pretty=oneline
    ```
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/git-1-2.png)

3. 简略显示
    ```sh
    git log --oneline
    ```
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/git-1-3.png)

4. 显示步数
    ```sh
    git reflog
    ```
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/git-1-4.png)

### 1.3 前进后退
1. 基于索引值
    ```sh
    git reset --hard 局部索引值
    ```
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/img/git-1-5.png)

2. 单步回退
    ```sh
    git reset --hard HEAD^
    ```
    * 一个^表示后退一步，以此类推

3. 多步回退
    ```sh
    git reset --hard~n
    ```
    * n表示回退n步
4. reset参数
   * --soft：仅在本地库移动指针
   * --mixed：在本地库移动指针并重置暂存区
   * --hard：在本地库移动指针，重置暂存区与工作区
### 1.4 比较文件差异
1. 与暂存区文件比较
    ```sh
    git diff 文件名
    ```

2. 与本地库中文件比较
    ```sh
    git diff 本地库历史版本 文件名
    git diff HEAD^ README.md # 与上一版本进行比较
    ```

3. 比较多个文件
    ```sh
    git diff
    ```

### 1.5 拉取与推送
1. 拉取
   * pull
        ```sh
        git pull 远程仓库 分支
        ```

   * fetch
        ```sh
        git fetch 远程仓库 分支
        ```

   * 区别：fetch只拉取，不合并，pull=fetch+merge

2. 推送
    ```sh
    git push 
    # 或
    git push 远程仓库名 分支名
    ```


## 2. 仓库管理
### 2.1 本地仓库
1. 仓库创建
   * [新建仓库文件夹]，在仓库文件夹下打开git终端
   * 初始化
        ```sh
        git init
        ```

   * 将文件添加到暂存区
        ```sh
        git add 文件名
        # 可以使用 git add . 或 git add -A 添加所有
        ```

   * 查看状态
        ```sh
        git status
        # 或
        git status -s # 简化显示结果，A：提交成功；AM：文件在添加到缓存之后又有改动
        ```

   * 提交到本地仓库
        ```sh
        git commit -m "备注信息"
        ```

   * 推送到远程服务器
        ```sh
        git push 远程仓库名或地址
        ```
2. 仓库日常操作
   * 正常开发，创建、修改、删除文件
   * 将修改提交到暂存区
        ```sh
        git add .
        ```

   * 提交到本地库
        ```sh
        git commit -m "备注信息"
        ```

   * 推送到远程库
        ```sh
        git push 远程仓库
        ```
3. 其他操作
   * 克隆远程仓库
        ```sh
        git clone 仓库地址
        ```

   * 拉取远程仓库到本地（同步）
        ```sh
        git pull 
        # 或
        git pull 远程仓库名 分支名
        ```
   * 推送
        ```sh
        git push origin master
        ```
     * 如果远程仓库存在你本地仓库没有的更新，则在推送前你需要先进行一次同步
     * 如果不想更新可以使用 -f 选项，强制推送
        ```sh
        git push origin master -f
        ```

### 2.2 远程仓库
1. 查看远程仓库
    ```sh
    git remote -v
    ```

2. 添加远程仓库
    ```sh
    git remote add 仓库名 仓库地址 
    # 仓库名一般用origin
    # 仓库地址支持 http/https/ssh/git协议
    ```
    
3. 修改远程仓库名
    ```sh
    git remote rename 原名 新名
    ```

4. 修改远程仓库地址
    ```sh
    git remote set-url 仓库名 仓库地址
    ```

## 3. 分支操作
### 3.1 基本操作
1. 查看分支
    ```git
    git branch -v
    ```

2. 创建分支
    ```sh
    git branch 分支名
    ```

3. 切换分支
    ```git
    git checkout 分支名
    ```
4. 删除分支
    ```sh
    git branch -d 分支名
    ```

### 3.2 高级操作
1. 合并分支
    ```sh
    # 1. 切换到被合并分支
    git checkout 分支名
    # 2. 合并到主分支
    git merge 主分支名
    ```

2. 解决冲突
   * 编辑文件，删除提示符号，并根据实际情况修改冲突内容
   * 添加到缓存区
        ```git
        git add 冲突的文件名
        ```
   * 提交文件
        ```sh
        git commit -m "信息" # 这里不能加文件名
        ```

## 4. 其它操作
### 4.1 
1. 修改上次提交
   ```sh
   git commit --amend
   git commit --amend -m ""
   ```
2. 


















