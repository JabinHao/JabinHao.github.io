---
title: git笔记
category: git
tags:
  - git
index_img: /img/post/git.png
excerpt: git学习笔记，记录git基本操作及使用中遇到的问题
abbrlink: d9283bc6
date: 2020-10-08 18:08:00
---
# git基本操作
## 1. git基本命令
### 1.1 本地仓库管理
1. 初始化一个Git仓库(以/home/gitee/test文件夹为例)  
```
$ cd /home/gitee/test    #进入git文件夹
$ git init               #初始化一个Git仓库
```
2. 将文件添加到Git的暂存区  
```
$ git add "readme.txt" 
```
注：使用git add -A或git add . 可以提交当前仓库的所有改动。

3. 查看仓库当前文件提交状态（A：提交成功；AM：文件在添加到缓存之后又有改动）  
```
$ git status -s
```
4. 从Git的暂存区提交版本到仓库，参数-m后为当次提交的备注信息  
```
$ git commit -m "1.0.0"  
```
5. 将本地的Git仓库信息推送上传到服务器  
```
$ git push https://gitee.com/***/test.git
```
6. 查看git提交的日志  
```
$ git log
```

### 1.2 远程仓库管理
1. 修改仓库名  
在执行clone或者其他操作时，默认仓库名都是 origin，如果说想改名字，比如改为 test 那么就要在仓库目录下执行命令:
```
git remote rename origin test
```
这样远程仓库名字就改成了test，以后推送或拉取时执行的命令就不再是 git push origin master 而是 git push test master
2. 添加一个仓库  
在不执行克隆操作时，如果想将一个远程仓库添加到本地的仓库中，可以执行
```
git remote add origin  仓库地址
```
注意:
   * origin是你的仓库的别名，可以随便改，但不能与已有的仓库别名冲突 
   * 仓库地址一般来讲支持 http/https/ssh/git协议，其他协议地址请勿添加  
3. 查看当前仓库对应的远程仓库地址  
```
git remote -v 
```
这条命令能显示你当前仓库中已经添加了的仓库名和对应的仓库地址，通常来讲，会有两条一模一样的记录，分别是fetch和push，其中fetch是用来从远程同步 push是用来推送到远程
4. 修改仓库对应的远程仓库地址  
```
git remote set-url origin 仓库地址
```

## 2. git常用操作
### 2.1 创建仓库并推送到github/gitee
1. 创建本地仓库
```sh
mkdir test
cd test
git init
touch README
git add .
git commit -m "git init"
```
2. 创建远程仓库  
在github或gitee上新建一个仓库，注意不要添加README、LICENSE等文件，一定要是空仓库。
将远程仓库的ssh地址复制下来，注意是git开头的，例如：
```
git@gitee.com:user/test.git
```
3. 将远程仓库地址添加到本地
在本地仓库内执行：
```
git remote add origin git@gitee.com:user/test.git
```
4. 添加SSH KEY  
如果是初次使用git，需要在远程仓库上添加ssh key来进行加密传输。  
首先在本地创建密钥对：
```git
git ssh-keygen -t rsa -C "youremail@mail.com"
```
然后一路回车就好了，完成后~目录下（win10在用户目录下）会多出一个.ssh的**隐藏**文件夹，进入文件夹，打开id_rsa.pub文件，全选并复制。  
在github或gitee的设置中找到SSH keys，点击新建，标题可以随便填，内容把刚刚复制的id_rsa.pub中的内容贴上去。
5. 将本地内容推送到远程仓库  
```git
git push -u origin master
```
新的仓库第一次推送需要`-u`参数，后面就不需要了。
6. 问题
如果第二步不小心点了使用README初始化仓库，在推送的时候会报错，需要合并一下仓库，在本地仓库下执行：
```git
git pull --rebase origin master
```

### 2.2 clone远程仓库到本地
    相当简单

### 2.3 从远程仓库复制某些文件/文件夹
1）复制文件  
```sh
wget <url>
example:
        wget https://gitee.com/hao_ji_peng/super_computing/raw/master/test/test.sh
```
2）复制文件夹
Git1.7.0以后加入了Sparse Checkout模式，因此以下操作需要git版本1.7.0以上。
具体实现如下：    
1. 首先创建空仓库：
```
mkdir new_folder 
cd new_folder 
git init 
git remote add -f origin <url>
```
2. 将远程Git Server URL加入到Git Config文件中：
```
git config core.sparsecheckout true # 也可直接修改 .git/config文件  
```
3. 将要Check Out的文件或文件夹添加到 .git/info/sparse-checkout 文件中（文件不存在的话就创建一个）
```
echo your_dir >> .git/info/sparse-checkout 

# your_dir为要下载的文件或文件夹，要填写完整路径，如 LSTHF/final_structure/1.CONTCAR.vasp
```
4. 从想要的分支中将项目拉下来：
```
git pull origin master
```
相关链接：[git官网-Sparse checkout](https://git-scm.com/docs/git-sparse-checkout#_sparse_checkout)
***
### 2.4 添加.gitignore
如果本地仓库中有些文件/文件夹不想推送到远程，可以将它们添加到.gitignore文件中。  
1. 在.git同级目录中新建.gitignore文件：
```sh
touch .gitignore
```
2. 将文件/文件夹加入到.gitignore中  
这里要注意，一定要添加完整路径，如：
```
/input/
/output/vasp.out
```
3. 删除已推送文件
如果添加.gitignore的时候，git里面已经上传了很多不需要的文件，则需要清除缓存：
```
git rm -r --cached 文件名
```

## 3. git常见问题
### 3.1 push到远程分支被拒绝
错误提示：
```
 ! [rejected]        master -> master (non-fast-forward)
error: 无法推送一些引用到 'git@gitee.com:hao_ji_peng/super_computing.git'
提示：更新被拒绝，因为您当前分支的最新提交落后于其对应的远程分支。
提示：再次推送前，先与远程变更合并（如 'git pull'）。详见
提示：'git push --help' 中的 'Note about fast-forwards' 小节。
```
原因一般是远程分支修改，与本地不匹配，解决方法如下：
```
git fetch origin
git merge origin/master
```
### 3.2 fatal: 远程 origin 已经存在。
原因: git远程地址配错，执行以下命令删除并重新配置
```
git remote rm origin
git remote add origin <url>
```
### 3.3 error failed to push some refs to https:*******
远程与本地不匹配，需要合并仓库：
```git
git pull --rebase origin master
```

```python
print('hello, world')
for i in range 10:
    continue
```