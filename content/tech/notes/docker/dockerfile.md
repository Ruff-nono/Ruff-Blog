---
title: "Dockerfile"
slug: ""
date: 2023-01-13T11:19:49+08:00
lastmod: 2023-01-13T11:19:49+08:00
author: ["路非非"]
tags: # 标签
-
series:
-
description: ""
weight:
draft: true # 是否为草稿
comments: true # 本页面是否显示评论
showToc: true # 显示目录;'
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
# 常见的 Dockerfile 命令
FROM: 用于指定基础镜像，即您的新镜像将基于哪个现有镜像构建。

RUN: 用于在构建过程中运行命令，例如安装软件包、更新包管理系统等。

CMD: 用于指定容器运行时的默认命令和参数。如果在 Dockerfile 中指定了多个 CMD，只有最后一个会生效。

ENTRYPOINT: 用于设置容器的入口点，类似于 CMD，但在运行容器时不可覆盖，而是作为默认命令。

EXPOSE: 用于指定容器在运行时监听的端口。

ENV: 用于设置环境变量，这些变量将在容器内部可用。

WORKDIR: 用于设置容器内的工作目录。

COPY 和 ADD: 用于将文件和目录从主机复制到容器内。

VOLUME: 用于创建一个挂载点，允许容器和主机之间共享数据。

USER: 用于指定在容器内执行命令时要切换到的用户。

HEALTHCHECK: 用于定义容器的健康检查指令，以检查容器的健康状态。

ARG: 用于定义构建时参数，这些参数可以在构建过程中传递给 Dockerfile。



