---
layout:     post
title:      "CentOS 7 下 yum 安装 Docker CE"
subtitle:   "Install Docker"
date:       2019-09-13 20:45:43
author:     "TyurinTsien"
header-img: "img/post-bg-docker.jpg"
catalog: true
tags:
    - Docker
    - Centos
---

# 前言
- Docker 使用越来越多，安装也很简单，本次记录一下基本的步骤。
- Docker 目前支持 CentOS 7 及以后的版本，内核要求至少为 3.10。
- Docker 官网有安装步骤，本文只是记录一下，您也可以参考 [Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
## 环境要求
CentOS 7
```
$ cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core)
```
# 准备工作

## 操作系统要求
- CentOS 7 以后都可以安装Docker了。
```
$ uname -a
Linux ecs-7087.novalocal 3.10.0-957.el7.x86_64 #1 SMP Thu Nov 8 23:39:32 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
- Docker 需要用到 <font color=Crimson>centos-extra</font> 这个源，如果您关闭了，需要重启启用，可以参考 [Available Repositories for CentOS](https://wiki.centos.org/AdditionalResources/Repositories)。

## 卸载旧版本
- 旧版本的 Docker 被叫做 <font color=Crimson>docker</font> 或 <font color=Crimson>docker-engine</font>，如果您安装了旧版本的 Docker ，您需要卸载掉它。
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
- 旧版本的内容在 <font color=Crimson>/var/lib/docker</font> 下，目录中的镜像(images), 容器(containers), 存储卷(volumes), 和 网络配置（networks）都可以保留。

- Docker CE 包，目前的包名为 <font color=Crimson>docker-ce</font>

# 安装

## 安装准备
- 为了方便添加软件源，支持 devicemapper 存储类型，安装如下软件包
```
$ sudo yum update
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
## 添加 yum 软件源
- 添加 Docker 稳定版本的 yum 软件源
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
## 安装 Docker
- 更新一下 yum 软件源的缓存，并安装 Docker。
```
$ sudo yum update
$ sudo yum install docker-ce
```
- 如果弹出 GPG key 的接收提示，请确认是否为 <font color=Crimson>060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35</font>，如果是，可以接受并继续安装。

- 至此，Docker 已经安装完成了，Docker 服务是没有启动的，操作系统里的 docker 组被创建，但是没有用户在这个组里。

> <font color=Crimson size=5>注意</font>
> - 默认的 docker 组是没有用户的（也就是说需要使用 sudo 才能使用 docker 命令）。
> - 您可以将用户添加到 docker 组中（此用户就可以直接使用 docker 命令了）。

- 加入 docker 用户组命令
```
$ sudo usermod -aG docker USER_NAME
```
- 用户更新组信息后，重新登录系统即可生效。

## 安装指定版本
- 如果想安装指定版本的 Docker，可以查看一下版本并安装。
```
yum list docker-ce --showduplicates | sort -r
docker-ce.x86_64            3:19.03.2-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:19.03.2-3.el7                    @docker-ce-stable
docker-ce.x86_64            3:19.03.1-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:19.03.0-3.el7                    docker-ce-stable 
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable 
Available Packages
```
- 可以指定版本安装,版本号可以忽略 <font color=Crimson>:</font> 和 <font color=Crimson>el7</font>，如 <font color=Crimson>docker-ce-18.09.1</font>
```
$ sudo yum install docker-ce-<VERSION STRING>
```
- 至此，指定版本的 Docker 也安装完成，同样，操作系统内 docker 服务没有启动，只创建了 docker 组，而且组里没有用户。

## 启动 Docker
- 如果想添加到开机启动
```
$ sudo systemctl enable docker
```
- 启动 docker 服务
```
$ sudo systemctl start docker
```

## 验证安装
- 验证 Docker CE 安装是否正确，可以运行 hello-world 镜像
```
$ sudo docker run hello-world
```

# 更新和卸载 Docker
- 使用 yum 管理，更新和卸载都很方便。

##更新 Docker CE
```
$ sudo yum update docker-ce
```
##卸载 Docker CE
```
$ sudo yum remove docker-ce
```
##删除本地文件
- 注意，docker 的本地文件，包括镜像(images), 容器(containers), 存储卷(volumes)等，都需要手工删除。默认目录存储在 <font color=Crimson>/var/lib/docker</font>/var/lib/docker。
```
$ sudo rm -rf /var/lib/docker
```

# 修改 Docker 默认存储位置
- 默认情况下Docker的存放位置为：/var/lib/docker

- 可以通过下面命令查看具体位置：
```
$ sudo docker info | grep "Docker Root Dir"
```
- 解决这个问题，最直接的方法当然是挂载分区到这个目录，但是我的数据盘还有其他东西，这肯定不好管理，所以采用修改镜像和容器的存放路径的方式达到目的。
## 方法一: 软链接
- 这个方法里将通过软连接来实现。

- 首先停掉Docker服务：
```
$ systemctl restart docker或者service docker stop
```
- 然后移动整个/var/lib/docker目录到目的路径：
```
$ mv /var/lib/docker /root/data/docker
$ ln -s /root/data/docker /var/lib/docker
```
- 这时候启动Docker时发现存储目录依旧是/var/lib/docker，但是实际上是存储在数据盘的，你可以在数据盘上看到容量变化。

## 其他方法
其他方法，详见[四个修改Docker默认存储位置的方法](https://blog.51cto.com/forangela/1949947)

# 参考资料
- [Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
- [Available Repositories for CentOS](https://wiki.centos.org/AdditionalResources/Repositories)
- [四个修改Docker默认存储位置的方法](https://blog.51cto.com/forangela/1949947)