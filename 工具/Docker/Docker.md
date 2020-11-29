> Docker 学习
>
> - docker概述
> - docker安装
> - docker命令
>   - 镜像命令
>   - 容器命令
>   - 操作命令
> - docker 镜像
> - 容器数据卷
> - dockerFile
> - Docker网络原理
> - idea整合docker
> - docker compose
> - docker swarm
> - CI/CD Jenkins
>
> Docker官方文档: https://docs.docker.com



# Docker概述

## Docker为什么出现?

部署一个环境会很费时费力, Docker可以将环境和项目一起打成一个包(镜像), 运维可以直接运行



## Docker可以做什么

> **DevOps (开发,运维)**

### 应用更快度的交付和部署

传统: 一堆文档, 安装程序等.

Docker: 打包镜像,发布测试, 一键运行

### 更便捷的升级和扩容等

### 更简单的系统运维

### 更高效的计算机资源利用



# Docker安装

## Docker名词

### 镜像(image)

Docker镜像就好像一个模板, 可以通过这个模板来创建容器服务.

通过镜像可以创建多个容器



### 容器(container)

Docker 通过容器技术, 独立运行一个或者多个应用. 容器通过镜像创建. 就好比一个简易的Linux系统



### 仓库(repository)

repository就是存放镜像的地方. 分为公有仓库和私有仓库



