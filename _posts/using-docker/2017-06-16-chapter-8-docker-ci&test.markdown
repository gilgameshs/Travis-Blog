---
layout:     post
title:      "Docker持续集成与测试"
date:       2017-06-16 23:00:00
author:     "Travis"
header-img: "img/post-bg-2015.jpg"
tags:
    - Docker
    - Jenkins
---

## Docker持续集成与测试

### Docker中测试
    利用docker的隔离特性，一些对环境有影响的测试可以放在容器中进行测试

### Docker持续集成
- 创建Jenkins容器，能够构建镜像
    + 将docker套接字从主机挂载到容器内，让jenkins创建“同级”容器
    docker套接字是客户端和守护进程之间通信的端点，默认情况，是在/var/run/docker.sock的IPC套接字，Docker也支持基于网络地址的TCP套接字和systemd形式的套接字

    + Docker-in-Docker（DinD）
    DinD详情，参见[jpetazzo项目](https://github.com/jpetazzo/dind)，以及他的[关于谨慎使用DinD的文章](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci)

    + 不要使用docker用户组，而是使用sudo
    + 启动jenkins容器前创建数据容器（基于相同的镜像）
