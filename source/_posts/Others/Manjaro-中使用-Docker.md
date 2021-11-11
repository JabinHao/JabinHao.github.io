---
title: Manjaro 中使用 Docker
excerpt: manjaro 安装docker，配置镜像加速及系统代理
tags:
  - manjaro
categories:
  - Others
banner_img: /img/banner/catiger.png
index_img: /img/index/code.jpg
category: Others
date: 2021-11-02 22:05:18
updated: 2021-11-02 22:05:18
subtitle:
---

##  1. 安装 

### 1.1 安装启动

1. 安装

    ```sh
    sudo pacman -S docker
    ```

2. 启动

    ```sh
    sudo systemctl start docker.service
    ```
3. 开机自启动

    ```sh
    sudo systemctl enable docker.service
    ```

### 1.2 用户配置

1. 启动后会发现 docker 命令无法执行，这是因为Docker 默认只能通过 root 权限执行操作
2. 将用户添加到 docker 用户组

    ```sh
    sudo usermod -aG docker jphao
    ```

## 2. 设置

### 2.1 镜像仓库

1. 打开或创建 /etc/docker/daemon.json

    ```sh
    sudo touch /etc/docker/daemon.json
    kate /etc/docker/daemon.json
    ```

2. 在文件中添加镜像

    ```json
        {
            "registry-mirrors": [
                "https://registry.docker-cn.com",
                "https://dockerhub.azk8s.cn"
            ]
        }
    ```

3. 重启服务

    ```sh
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

4. 查看是否添加成功

    ```sh
    docker info
    ```
    ![](https://raw.githubusercontent.com/JabinHao/mihs/master/blog/Docker/docker-mirrors.png)

### 2.2 系统代理

设置完镜像仓库后，拉取仍然很慢，尽管 docker info 显示已经成功设置，但很明显并没有起作用，此时只能通过另一种方法：**clash + 代理** 了

1. dockerd 代理
    * 拉取镜像时，由守护进程 dockerd 执行，此环境的代理配置
        ```sh
        # 创建配置文件
        sudo mkdir -p /etc/systemd/system/docker.service.d
        sudo touch /etc/systemd/system/docker.service.d/proxy.conf
        # 打开配置文件
        kate /etc/systemd/system/docker.service.d/proxy.conf
        ```
    * 在文件中添加以下内容
        ```conf
        [Service]
        Environment="HTTP_PROXY=http://127.0.0.1:7890/"
        Environment="HTTPS_PROXY=http://127.0.0.1:7890/"
        Environment="NO_PROXY=localhost,127.0.0.1/8, ::1"
        ```
    * 重启服务
        ```
        sudo systemctl daemon-reload
        sudo systemctl restart docker.service
        ```
        此时再拉取镜像，速度已经很快了

2. container 代理
    * 容器运行阶段若要设置代理，则要通过 **~/.docker/config.json** 文件进行配置，详见[官网](https://docs.docker.com/network/proxy/#configure-the-docker-client)
    * 打开 ~/.docker/config.json 文件，添加以下内容
        ```json
        {
            "proxies": {
                "default": {
                    "httpProxy": "http://172.17.0.1:7890",
                    "httpsProxy": "http://172.17.0.1:7890",
                    "noProxy": "localhost,127.0.0.1/8, ::1"
                }
            }
        }
        ```
        这里要用 172.17.0.1，因为容器通过docker0网桥访问宿主机，127.0.0.1其实是容器内部
    * 重启服务（同上）


