---
title: "Helm 快速入门指南"
date: 2023-11-30T13:45:49+08:00
lastmod: 2023-11-30T13:45:49+08:00
author: ["Ruff"]
keywords: 
- 
series: # 系列
- k8s 学习
tags: # 标签
- k8s
description: ""
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

## 先决条件
想成功和正确地使用Helm，需要以下前置条件。

1. 一个 Kubernetes 集群 
2. 确定你安装版本的安全配置 
3. 安装和配置Helm。

安装或者使用现有的Kubernetes集群
* 使用Helm，需要一个Kubernetes集群。对于Helm的最新版本，我们建议使用Kubernetes的最新稳定版， 在大多数情况下，它是倒数第二个次版本。
您也应该有一个本地的 kubectl.
* 查看Helm和对应支持的Kubernetes版本，您可以参考 [Helm 版本支持策略](https://helm.sh/zh/docs/topics/version_skew/)。

```shell
# 查询 helm 版本
root@xx:~# helm version
version.BuildInfo{Version:"v3.13.2", GitCommit:"2a2fb3b98829f1e0be6fb18af2f6599e0f4e8243", GitTreeState:"clean", GoVersion:"go1.20.10"}
# k8s 版本
root@xx:~# kubectl version
Client Version: v1.28.4
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3
```

## 安装
以 Debian/Ubuntu 为例

Helm社区成员贡献了针对Apt的一个 Helm包，包通常是最新的。

```shell
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

更多安装见[安装helm](https://helm.sh/zh/docs/intro/install/)

## 初始化
当您已经安装好了Helm之后，您可以添加一个chart 仓库。从 Artifact Hub中查找有效的Helm chart仓库。

>"Chart 仓库" 指 Helm Chart 的存储库，用于定义和管理 Kubernetes 应用的包。这些仓库包含了 Helm Charts 的模板、描述文件以及相关信息，使得用户能够轻松地分享、发布和使用这些 Charts。
> 
> Bitnami Charts 是由 Bitnami 组织提供的 Helm Charts 集合。提供了许多常见应用程序的 Helm Charts，包括数据库（如 MySQL、PostgreSQL）
> 、Web 服务器（如 Apache、NGINX）、消息队列（如 RabbitMQ）、监控工具（如 Prometheus）、日志记录工具（如 Elasticsearch、Fluentd、Kibana）等。
> 这些 Charts 包含了一些预定义的配置选项，使得用户可以通过简单的 Helm 部署命令将这些应用程序部署到 Kubernetes 集群中。
> 
> 除了使用公共仓库外，一些组织或项目也可能设置自己的私有 Chart 仓库。允许组织自己管理和分发 Helm Charts，而无需依赖外部仓库。


```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```

添加完成后，可以查询charts列表
```shell
helm search repo bitnami
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/airflow                                 16.1.6          2.7.3           Apache Airflow is a tool to express and execute...
bitnami/apache                                  10.2.3          2.4.58          Apache HTTP Server is an open-source HTTP serve...
bitnami/apisix                                  2.2.7           3.7.0           Apache APISIX is high-performance, real-time AP...
bitnami/appsmith                                2.1.7           1.9.48          Appsmith is an open source platform for buildin...
bitnami/argo-cd                                 5.2.8           2.9.2           Argo CD is a continuous delivery tool for Kuber...
bitnami/argo-workflows                          6.1.4           3.5.1           Argo Workflows is meant to orchestrate Kubernet...
bitnami/aspnet-core                             5.0.1           8.0.0           ASP.NET Core is an open-source framework for we...
... and many more
```

## 安装Chart示例
您可以通过helm install 命令安装chart。 Helm可以通过多种途径查找和安装chart， 但最简单的是安装官方的bitnami charts。
```shell
$ helm repo update              # 确定我们可以拿到最新的charts列表
$ helm install bitnami/mysql --generate-name
NAME: mysql-1701325660
LAST DEPLOYED: Thu Nov 30 14:27:42 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 9.14.4
APP VERSION: 8.0.35
...
```
在上面的例子中，bitnami/mysql这个chart被发布，名字是 mysql-1701325660。 

通过执行 `helm show chart bitnami/mysql` 命令简单的了解到这个chart的基本信息。 

每次执行 helm install 的时候，都会创建一个新的发布版本。 所以一个chart在同一个集群里面可以被安装多次，每一个都可以被独立的管理和升级。 

## 查看版本发布
通过Helm
```shell
$ helm ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mysql-1701325660        default         1               2023-11-30 14:27:42.4610191 +0800 CST   deployed        mysql-9.14.4    8.0.35
```
helm list (或 helm ls) 命令会列出所有可被部署的版本。

查询pods, 发现已经被部署
```shell
$ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
mysql-1701325660-0   1/1     Running   0          5m
```

## 卸载版本
您可以使用 `helm uninstall` 命令卸载你的版本

```shell
$ helm uninstall mysql-1701325660
release "mysql-1701325660" uninstalled
```

该命令会从Kubernetes卸载 mysql-1612624192， 它将删除和该版本相关的所有相关资源（service、deployment、 pod等等）甚至版本历史。

执行 `helm uninstall --keep-history <release-name>` 选项， Helm将会保存版本历史。 您可以通过命令查看该版本的信息。

```shell
$ helm status mysql-1701327004
NAME: mysql-1701327004
LAST DEPLOYED: Thu Nov 30 14:50:07 2023
NAMESPACE: default
STATUS: uninstalled
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 9.14.4
APP VERSION: 8.0.35
```
使用 `helm history <release-name>` 查看历史版本, 并通过 `helm rollback <release-name> <revision-number>` 回滚到指定版本.

```shell
$ helm history mysql-1701327004
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Thu Nov 30 14:50:07 2023        uninstalled     mysql-9.14.4    8.0.35          Uninstallation complete

$ helm rollback mysql-1701327004 1
Rollback was a success! Happy Helming!

$ helm history mysql-1701327004
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Thu Nov 30 14:50:07 2023        uninstalled     mysql-9.14.4    8.0.35          Uninstallation complete
2               Thu Nov 30 15:00:24 2023        deployed        mysql-9.14.4    8.0.35          Rollback to 1
```

## 参考
[Helm快速入门指南](https://helm.sh/zh/docs/intro/quickstart/)