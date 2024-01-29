---
title: "Note"
date: 2023-11-27T15:42:14+08:00
lastmod: 2023-11-27T15:42:14+08:00
author: ["Ruff"]
keywords: 
- 
series: # 系列
- 
tags: # 标签
- 
description: ""
weight:
slug: ""
draft: true # 是否为草稿
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

Kubernetes 的宗旨就是在应用之间共享机器。 通常来说，共享机器需要两个应用之间不能使用相同的端口，但是在多个应用开发者之间 去大规模地协调端口是件很困难的事情，尤其是还要让用户暴露在他们控制范围之外的集群级别的问题上。

动态分配端口也会给系统带来很多复杂度 - 每个应用都需要设置一个端口的参数， 而 API 服务器还需要知道如何将动态端口数值插入到配置模块中，服务也需要知道如何找到对方等等。 与其去解决这些问题，Kubernetes 选择了其他不同的方法。