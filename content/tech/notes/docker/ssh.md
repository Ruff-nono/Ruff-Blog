---
title: "在 Docker Build 时使用 SSH 私钥进行认证"
date: 2023-12-08T10:28:26+08:00
lastmod: 2023-12-08T10:28:26+08:00
author: ["Ruff"]
keywords: 
- 
series: # 系列
- Docker 攻略
tags: # 标签
- 
description: "本文主要介绍如何在 Docker Build 时使用 SSH 私钥进行认证，比如需要拉取私有仓库场景。包括18.09版本之前的使用多阶段构建方式，以及 18.09版本后的 --ssh 方式。"
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

# 前言

在工作中，Build Docker 镜像时，经常会碰到需要在镜像内使用 SSH Private Key 的场景。如期望在容器内对代码进行编译构建，那么不可避免的需要从 GitHub、GitLab 的私有库下载依赖代码。

如果直接把自己的 SSH Private Key 打包到 Docker 镜像中的话，会存在很大安全风险的。那么如何解决这个问题呢？

# 多阶段构建
通过参数将私钥传递到容器里，同时配合多阶段构建以解决直接把私钥打包进容器带来的安全风险。

使用多阶段构建，只要私钥不出现在最后一阶段，都是比较安全的，中间过程的镜像只会存放在本机，不会公开，因此问题也不大。

```dockerfile
# 复制 SSH 私钥到容器中
# syntax=docker/dockerfile:1
FROM golang:1.21-alpine as sources

COPY deploy-ssh-key /root/.ssh/id_rsa
RUN chmod 600 /root/.ssh/id_rsa
RUN ssh-keyscan private.domain >> ~/.ssh/known_hosts

WORKDIR /app
COPY . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o dist/main .

FROM debian:11

WORKDIR /app

COPY --from=build /workspace/dist/ .
CMD ["sh", "-c", "/app/main"]
```

# SSH mount type
Docker 在 18.09 版本后，推出了 BuildKit 的 SSH mount type，我们也可以用这个特性来解决该问题。

```shell
--ssh=default|<id>[=<socket>|<key>[,<key>]]
```
`--ssh` 向构建公开 SSH 密钥，并配合使用 `RUN --mount=type=ssh`。

```shell
# syntax=docker/dockerfile:1

WORKDIR /app
COPY . .

RUN --mount=type=ssh mkdir -p -m 0700 ~/.ssh && ssh-keyscan private.domain >> ~/.ssh/known_hosts
RUN --mount=type=ssh,id=ssh_key go mod download
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o dist/main .

FROM debian:11

WORKDIR /app
COPY --from=build /workspace/dist/ .
CMD ["sh", "-c", "/app/main"]
```

最后，即可使用如下命令来 Build Docker 镜像了：
```shell
docker build --ssh ssh_key=/root/.ssh/id_ed25519 -f Dockerfile --rm --no-cache -t demo-image:1.0.0 .
```

这种方式，既能正常使用上 SSH Private Key，又能使其在镜像中不留痕迹。完美！


参考
[docker buildx build](https://docs.docker.com/engine/reference/commandline/buildx_build/#progress)



