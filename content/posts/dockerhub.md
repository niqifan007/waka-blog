---
title: Docker Hub 镜像加速器汇总
subtitle: Docker Hub 镜像加速器汇总
date: 2023-07-25T22:00:57+08:00
draft: false
author:
  name: waka
  link:
  email:
  avatar: /images/avatar.jpg
description: Docker Hub 镜像加速器
keywords: Docker
license:
comment: false
weight: 0
tags:
  - Docker
categories:
  - Docker
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: false
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---
# Docker Hub 镜像加速器

国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务。

> Dockerized 实践 https://github.com/y0ngb1n/dockerized

## 配置加速地址

> Ubuntu 16.04+、Debian 8+、CentOS 7+

创建或修改 `/etc/docker/daemon.json`：

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Docker Hub 镜像加速器列表

镜像加速器 | 镜像加速器地址 | 专属加速器[？](# "需登录后获取平台分配的专属加速器") | 其它加速[？](# "支持哪些镜像来源的镜像加速")
--- | --- | --- | ---
~~[Docker 中国官方镜像](https://docker-cn.com/registry-mirror)~~ | ~~`https://registry.docker-cn.com`~~ | | ~~Docker Hub~~（[已关闭](https://github.com/docker/docker.github.io/issues/8793)）
[DaoCloud 镜像站](https://github.com/DaoCloud/public-image-mirror) | `https://docker.m.daocloud.io` | |  Docker Hub、GCR、K8S、GHCR、Quay、NVCR 等
[Azure 中国镜像](https://github.com/Azure/container-service-for-azure-china/blob/master/aks/README.md#22-container-registry-proxy) | `https://dockerhub.azk8s.cn` | 仅供内部访问 | Docker Hub、GCR、Quay
[科大镜像站](https://mirrors.ustc.edu.cn/help/dockerhub.html) | `https://docker.mirrors.ustc.edu.cn` | [仅供内部访问](https://mirrors.ustc.edu.cn/help/dockerhub.html) | Docker Hub、[GCR](https://github.com/ustclug/mirrorrequest/issues/91)、[Quay](https://github.com/ustclug/mirrorrequest/issues/135)
[阿里云](https://cr.console.aliyun.com) | `https://<your_code>.mirror.aliyuncs.com` | 需登录，系统分配 | Docker Hub
~~[七牛云](https://kirk-enterprise.github.io/hub-docs/#/user-guide/mirror)~~ | ~~`https://reg-mirror.qiniu.com`~~ | | ~~Docker Hub、GCR、Quay~~
[网易云](https://c.163yun.com/hub) | `https://hub-mirror.c.163.com` | | Docker Hub
[腾讯云](https://cloud.tencent.com/document/product/457/9113) | `https://mirror.ccs.tencentyun.com` | 仅供内部访问 | Docker Hub
[Docker 镜像代理](https://dockerproxy.com) | `https://dockerproxy.com` | | Docker Hub、GCR、K8S、GHCR
[百度云](https://cloud.baidu.com/doc/CCE/s/Yjxppt74z#%E4%BD%BF%E7%94%A8dockerhub%E5%8A%A0%E9%80%9F%E5%99%A8) | `https://mirror.baidubce.com` | | Docker Hub
[南京大学镜像站](https://doc.nju.edu.cn/books/35f4a) | `https://docker.nju.edu.cn` | | Docker Hub、GCR、GHCR、Quay、NVCR 等
[上海交大镜像站](https://mirrors.sjtug.sjtu.edu.cn/) | `https://docker.mirrors.sjtug.sjtu.edu.cn` | | Docker Hub、GCR 等

> ⚠️ 以下镜像站存在未同步最新源镜像问题，请按需选用
> - 阿里云

## 检查加速器是否生效

命令行执行 `docker info`，如果从结果中看到了如下内容，说明配置成功。

```
Registry Mirrors:
 [...]
 https://docker.m.daocloud.io
```

## Docker Hub 镜像测速

使用镜像前后，可使用 `time` 统计所花费的总时间。测速前先移除本地的镜像！ 

```
$ docker rmi node:latest
$ time docker pull node:latest
Pulling repository node
[...]

real   1m14.078s
user   0m0.176s
sys    0m0.120s
```

## 更新日志

---

## 参考链接

+ https://docs.docker.com/registry/recipes/mirror/
+ https://github.com/yeasy/docker_practice/blob/master/install/mirror.md
+ https://github.com/moby/moby/blob/d409b05970e686993e343d226fae5b463d872082/docs/articles/registry_mirror.md
+ https://www.fengbohello.top/archives/docker-registry-mirror
+ https://www.ilanni.com/?p=14534
+ https://github.com/Azure/container-service-for-azure-china/blob/master/aks/README.md#22-container-registry-proxy
+ https://moelove.info/2020/09/20/突破-DockerHub-限制全镜像加速服务/

<!--more-->
