---
title: "Kubectl"
slug: ""
date: 2023-02-08T11:18:27+08:00
lastmod: 2023-02-08T11:18:27+08:00
author: ["路非非"]
tags: # 标签
-
series:
-
description: ""
weight:
draft: true # 是否为草稿
comments: true # 本页面是否显示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" #图片路径例如：posts/tech/123/123.png
  caption: "" #图片底部描述
  alt: ""
  relative: false
---

# 在 Linux 系统中安装并设置 kubectl

## 准备
kubectl 版本和集群版本之间的差异必须在一个小版本号内。 例如：v1.26 版本的客户端能与 v1.25、 v1.26 和 v1.27 版本的控制面通信。 用最新兼容版的 kubectl 有助于避免不可预见的问题。

## 用 curl 在 Linux 系统中安装 kubectl

1. 用以下命令下载最新发行版：

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

>说明：<br/>
>如需下载某个指定的版本，请用指定版本号替换该命令的这一部分： `$(curl -L -s https://dl.k8s.io/release/stable.txt)`。
>
>例如，要在 Linux 中下载 v1.26.0 版本，请输入：
>
>```shell
>curl -LO https://dl.k8s.io/release/v1.26.0/bin/linux/amd64/kubectl

2. 验证该执行文件

下载 kubectl 校验和文件：
```shell
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
基于校验和文件，验证 kubectl 的可执行文件：
```shell
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```
验证通过时，输出为：
```
kubectl: OK
```
验证失败时，sha256 将以非零值退出，并打印如下输出：
```
kubectl: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
```
>说明：<br/>
>下载的 kubectl 与校验和文件版本必须相同。

3. 安装 kubectl
```shell
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
>说明：<br/>
>即使你没有目标系统的 root 权限，仍然可以将 kubectl 安装到目录 ~/.local/bin 中：
>```
>chmod +x kubectl
>mkdir -p ~/.local/bin
>mv ./kubectl ~/.local/bin/kubectl
># 之后将 ~/.local/bin 附加（或前置）到 $PATH
>```

4. 执行测试，以保障你安装的版本是最新的：
```shell
kubectl version --client
```
或者使用如下命令来查看版本的详细信息：
```shell
kubectl version --client --output=yaml
```
输出为
```
clientVersion:
  buildDate: "2023-01-18T15:58:16Z"
  compiler: gc
  gitCommit: 8f94681cd294aa8cfd3407b8191f6c70214973a4
  gitTreeState: clean
  gitVersion: v1.26.1
  goVersion: go1.19.5
  major: "1"
  minor: "26"
  platform: linux/amd64
kustomizeVersion: v4.5.7
```

## 验证 kubectl 配置
为了让 kubectl 能发现并访问 Kubernetes 集群，你需要一个 kubeconfig 文件， 该文件在 kube-up.sh 创建集群时，或成功部署一个 Minikube 集群时，均会自动生成。 通常，kubectl 的配置信息存放于文件 ~/.kube/config 中。

通过获取集群状态的方法，检查是否已恰当地配置了 kubectl：