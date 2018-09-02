---
title: 07 Spring Boot 之Docker
date: 2018-09-04
categories: Spring Boot
tags: [Spring Boot]
---

## 1 核心概念

docker主机（host）：安装了Docker程序的机器，Docker是直接安装在操作系统之上的。

docker客户端（Client）：连接Docker主机进行操作。

docker仓库（Registry）：用来保存各种打包好的软件镜像。

docker镜像（images）：软件打包好的镜像。

docker容器（Container）：镜像启动后的实例，称为一个容器。

使用Docker步骤：

1. 安装Docker。
2. 去Docker仓库找到需要的镜像。
3. 使用Docker运行这个镜像，这个镜像就会生成一个容器。
4. 对容器的操作就是对软件的操作。

<!-- more -->

## 2 安装、启动、停止

### 2.1 linux下安装docker

以centOS为例。

> 注意：linux的内核版本必须是3.10以上版本。使用`uname -r`查看。

一、【安装】：

```shell
yum install docker
```

二、【启动】：

```shell
systemctl start docker
```

可以使用`docker -v`查看版本号。

开机启动docker：

```shell
systemctl enable docker
```

三、【停止】：

```
systemctl stop docker
```

## 3 常用操作

### 3.1 镜像操作

一、【搜索镜像】：

```
docker search 镜像名
```

二、【下载镜像】：

```
docker pull 镜像名:tag
```

> docker镜像的版本通过标签（tag）来区分。
>
> 不加tag标签，默认下载最新的版本（latest）。

三、【查看所有的镜像】：

```
docker images
```

四、【删除镜像】：

```
docker rmi 镜像id（image-id）
```

### 3.2 容器操作

> 镜像 -> 运行镜像 -> 产生一个容器（正在运行时的软件）

一、【运行镜像】：

```
docker run [--name container-name] -d image-name
```

> `--name`：定义容器名称。
>
> `container-name`：容器名称。
>
> `-d`：后台运行。
>
> `image-name`：指定的镜像名称。

 

二、【查看运行的容器列表】：

```
docker ps
```

查看所有的容器：（包含未启动的）

```
docker ps -a
```

三、【停止容器】：

```
docker stop container-id/container-name
```

四、【启动容器】：

```
docker start container-id
```

五、【删除容器】：

```
docker rm container-id
```

**六、【端口映射】：**

再运行容器时加上`-p`参数，进行端口映射。

```
docker run [--name container-name] -d -p 主机端口:容器端口 image-name
```

> `-p`：主机端口映射到容器内部的端口。

七、【查看容器日志】：

```
docker logs container-id 
```

