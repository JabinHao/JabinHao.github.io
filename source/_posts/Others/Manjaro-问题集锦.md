---
title: Manjaro 问题集锦
excerpt: Manjaro KDE 桌面系统使用过程中的一些问题及解决方案
tags:
  - manjaro
categories:
  - Others
banner_img: /img/banner/catiger.png
index_img: /img/index/code.jpg
category: Others
abbrlink: 3ca66d4b
date: 2021-11-12 19:41:31
updated: 2021-11-12 19:41:31
subtitle:
---

## 1. 系统相关

### 1.1 display and monitor
1. 问题：修改屏幕分辨率后黑屏，在设置里将分辨率改为75hz后，电脑黑屏，重启后一登录就会黑屏，随后显示器显示没有信号源 
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Manjarosettings.png)
2. 原因：显示器不支持75hz（不知道为什么manjaro系统设置里有75hz这个选项）
3. 解决思路：系统设置保存在配置文件中，只要能修改配置文件，重新将 refresh rate 改回 60 即可
4. 解决方案：
    * 通过安装系统时的启动盘进入系统， （没有的话可以用其他电脑临时搞一个）
    * 进入 `~/.local/share/kscreen` 目录，打开一个名字类似 `8c494211920eaf37f55b700388774cd4`的文件，记录下其 fresh 值
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Manjarorefresh-remote.png)
    * 打开Dolphin, 找到安装系统的硬盘， 进入  `/home/你的用户名/.local/share/kscreen` 目录下，该目录及其子目录下各有一个文件
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Manjarokscreen-dir.png)
    * 将上面两个文件中的 refresh 改为第二步中记录的值
    * 重启，登录后桌面一切如常

### 1.2 更改插件样式导致黑屏
1. 问题描述
   * 将 Network speed 设置中的 Display style 修改为 Particles Vortex 后桌面卡住，随后黑屏
   * 虽然黑屏，但是之前打开的软件还在，通过快捷键也能正常打开应用

2. 解决方案
   * 按 F12 打开 Yakuake 终端
   * 通过 vim 或 kate 打开 ~/.config/plasma-org.kde.plasma.desktop-appletsrc
        ![我出问题的是Network speed](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Manjaronetwork-widget.png)
    * 找到出问题的 widget 配置，将其删除即可（保险起见最好先将该文件备份）, 自定义的 widget 一般以 `notmart.ksysguard` 开头
    * 重启后桌面恢复正常，但出问题的 widget 应该还在，将其删掉重新添加即可

### 1.3 密码问题
1. 问题：输错密码三次自动锁定
2. 解决
   * 打开 `/etc/security/faillock.conf`
   * 增加一行 `deny=0`
        ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Manjaroself-lock.png)


## 2. 软件安装

### 2.1 解压
1. 问题：Windows 下 zip 使用的GBK编码，Manjaro 使用 unzip 解压压缩文件时，文件名中的中文会乱码
2. 解决
    * 安装 unzip-icon
        ```sh
        yay unzip-icon
        ```
    * 解压时指定编码
        ```sh
        unzip -O GBK *.zip
        ```

### 2.2 待续 ...

