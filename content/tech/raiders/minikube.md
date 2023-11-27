---
title: "在linux中安裝Minikube"
date: 2023-11-20T16:42:54+08:00
lastmod: 2023-11-20T16:42:54+08:00
author: ["Ruff"]
keywords: 
- 
series: # 没有分类界面可以不填写
- k8s 学习
tags: # 标签
- k8s
description: "Minikube 是一种轻量级的Kubernetes 实现，可在本地计算机上创建VM 并部署仅包含一个节点的简单集群。 Minikube 可用于Linux、macOS 和Windows 系统。 Minikube CLI 提供了用于引导集群工作的多种操作， 包括启动、停止、查看状态和删除。"
weight:
slug: ""
draft: false # 是否为草稿
comments: false # 本页面是否显示评论
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---

minikube 是本地 Kubernetes，专注于让 Kubernetes 易于学习和开发。

您所需要的只是 Docker（或类似兼容的）容器或虚拟机环境，只需一条命令即可使用 Kubernetes：minikube start。

## 先决条件
* 2 CPU 以上
* 2 GB 可用内存
* 20 GB 可用磁盘空间
* 容器管理器 如 Docker、Hyper-V等

## 安装
`linux` `x86-64` 下使用安装命令
```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
运行 `minikube`后，提示不应以 `root` 权限运行
```shell
minikube start
😄  minikube v1.32.0 on Ubuntu 20.04
✨  Using the docker driver based on existing profile
🛑  The "docker" driver should not be used with root privileges. If you wish to continue as root, use --force.
💡  If you are running minikube within a VM, consider using --driver=none:
📘    https://minikube.sigs.k8s.io/docs/reference/drivers/none/
💡  Tip: To remove this root owned cluster, run: sudo minikube delete
```
可以赋予给一个普通用户`minikube` 和 `docker` 权限，因只是学习需要，所以可以强制 `root` 启动
```shell
minikube start --force
```

## 测试
如果已经安装了kubectl, 可以通过它来进行访问集群
```shell
kubectl get po -A
```
或者通过 `minikube` 中的 `kubectl`使用
```shell
minikube kubectl -- get po -A
```
这种方式可以通过创建命令别名, 来做到类似`kubectl`的效果
```shell
alias kubectl="minikube kubectl --"
```
如果要取消别名
```shell
unalias kubectl
```

打开仪表盘
```shell
minikube dashboard
```

## 部署
在 Kubernetes 中创建一个部署（deployment）和一个 NodePort 服务，并将它们绑定到一个映像（image）。
```shell
#创建一个名为 hello-minikube 的部署。
#使用 kicbase/echo-server:1.0 镜像作为容器镜像。
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
#通过 kubectl expose 命令为 hello-minikube 部署创建一个服务。
#--type=NodePort 指定服务类型为 NodePort，这意味着服务将通过节点的一个高端口公开。
#--port=8080 指定服务的集群内端口为 8080。
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```
通过 `minikube` 加载 web 服务
```shell
minikube service hello-minikube
# 或
minikube service --url hello-minikube
```
或者使用 `kubectl` 转发端口, 可以通过 `http://localhost:7080` 进行访问
```shell
kubectl port-forward service/hello-minikube 7080:8080
```

## kubectl 安装
{{< innerlink src="tech/raiders/kubectl.md" >}}

## 参考
[minikube Start](https://minikube.sigs.k8s.io/docs/start/)

[minikube 手册](https://minikube.sigs.k8s.io/docs/handbook/)

