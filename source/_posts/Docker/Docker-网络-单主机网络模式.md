---
title: 'Docker 网络: 单主机网络模式'
excerpt: Docker 网络理论以及 Docker 单主机网络配置
tags:
  - Docker
categories:
  - Docker
banner_img: /img/banner/docker.jpg
index_img: /img/index/docker.png
category: Docker
abbrlink: 28007f80
date: 2021-10-28 12:32:51
updated: 2021-10-29 23:25:10
subtitle:
---

## 1. Docker 网络

### 1.1 网络理论

1. 容器网络模型，包含三个概念
    * network： 网络模型
        * 单主机网络模式（none、host、bridge，joined container）
        * 多主机网络模式（overlay、macvlan、flannel）
    * sandbox：沙盒，它定义了容器内的虚拟网卡、DNS和路由表，是network namespace的一种实现，是容器的内部网络栈
    * endpoint：端点，用于连接sandbox和network

2. network namespace
   * network namespace 是实现网络虚拟化的重要功能，它能创建多个隔离的网络空间，它们有独自的网络栈信息
   * Linux 使用 veth pair 在不同的 network namespace 之间通信，中文称为虚拟网卡接口。它总是成对出现，像一个管道连接两个网络
   * Linux Bridge，即 Linux 网桥，是Linux提供的一种虚拟网络设备之一。其工作方式非常类似于物理的网络交换机设备。

3. Docker 网络
    * Docker 提供了四种可用的网络模式，但都比较简单，实际应用中一般要另外配置（pipework、weave）
    * 使用 docker run 命令时可以用 `--net=***` 配置，docker-compose 中使用 networks 指定

4. 查看 Docker 网络

    ```docker
    docker network ls
    ```
    Docker 自动创建的三个网络：
    ```
    NETWORK ID     NAME      DRIVER    SCOPE
    c9022131324d   bridge    bridge    local
    fca0d094c188   host      host      local
    cfeabbb7ad91   none      null      local
    ```

### 1.2 四种模式

1. bridge模式，使用--net=bridge指定，是docker默认设置。
2. host模式，使用--net=host指定。容器使用宿主机的IP和端口。
3. container模式，使用--net=container:NAME_or_ID指定。使用场景不多，两个容器必须跑在同一台docker服务器上。
4. none模式，使用--net=none指定。在这种模式下，Docker容器拥有自己的Network，但没有网卡、IP、路由等信息。需要我们自己为Docker容器添加网卡、配置IP等。

## 2. 四种模式详解

### 2.1 host 模式

1. 简介
    * host模式类似于Vmware的桥接模式，与宿主机在同一个网络中，但没有独立IP地址
    * 和宿主机共用一个Network Namespace，共享 ip 和端口
    * 该模式只能在 Linux 下使用

2. 使用

    ```docker
    docker run --name=nginx --net=host -d nginx
    ```
    无需端口映射，可以直接访问 127.0.0.1:80

### 2.2 container 模式

1. 简介
   * 创建容器时，和已经存在的某个容器共享一个 Network Namespace
   * 两个容器共享 ip 和端口
2. 示例
    * 创建 redis 容器，采用默认的 bridge 模式
        ```docker
        docker run -d --name server redis redis-server
        ```
    * 创建ubuntu容器，桥连到 server
        ```docker
        docker run -idt --net container:server --name client ubuntu
        ```
    * 进入 client， 查看网络端口
        ```docker
        docker exec -it client bash
        ```
        ```sh
        netstat -natp
        ```
        可以得到
        ```
        Active Internet connections (servers and established)
        Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
        tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      -
        tcp        0      0 127.0.0.1:6379          127.0.0.1:56242         ESTABLISHED -
        ```
        安装客户端并连接server
        ```sh
        apt update
        apt install redis-tools
        redis-cli
        ```

### 2.3 bridge 模式

1. 简介
    * Docker安装时会创建一个名为 docker0 的虚拟网桥，也可以自己定义
    * docker0 对宿主机来说相当于单独的网卡，对容器来说相当于交换机
    * 容器为 bridge 模式时，会创建一个名为 veth 的网卡，连接到 docker0
    * docker 默认为该模式

2. 手动配置
    * 创建
        ```docker
        docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 mynet2
        ```
        * --subnet 指定子网、IP地址范围
        * --gateway 指定网关
    * 查看

        ```docker
        docker network inspect mynet2
        ```
    * 查看宿主机网桥信息
        ```sh
        jphao:~/ $ brctl show                                                                                       
        bridge name     bridge id               STP enabled     interfaces
        br-af8226dcae24         8000.02422f96741c       no
        docker0         8000.02428ffad316       no
        ```


### 2.4 none 模式

1. 简介
    * 该模式关闭了容器的网络功能，不分配子网跟ip
    * 挂在这个网络下的容器除了 lo，没有其他任何网卡。容器创建时，可以通过 --network=none 指定使用 none 网络
    * 一般用在对安全性要求较高且不需要网络连接的情况下

2. 示例
    * 创建容器

        ```docker
        # 以下四种方式皆可
        # --net none、--net=none、--network none、--network=none
        docker run -idt --name none --net none ubuntu
        docker exec -it none bash
        ```
    * 查看
        ```
        root@8e587ebe7ea3:/# ifconfig
        lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
                inet 127.0.0.1  netmask 255.0.0.0
                loop  txqueuelen 1000  (Local Loopback)
                RX packets 0  bytes 0 (0.0 B)
                RX errors 0  dropped 0  overruns 0  frame 0
                TX packets 0  bytes 0 (0.0 B)
                TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        ```


## Reference

1. [Docker 官方文档](https://docs.docker.com/)
2. [Free AI-Hub](https://www.freeaihub.com/)
3. [飞污熊博客](https://www.xncoding.com/)


