****---
title: "SkyWalking Go 原理分析"
date: 2024-01-11T16:56:31+08:00
lastmod: 2024-01-11T16:56:31+08:00
author: ["Ruff"]
keywords: 
- 
series: # 系列
- skywalking-go
tags: # 标签
- skywalking
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


# 前言 
SkyWalking 是一个开源可观测平台，用于收集、分析、聚合和可视化来自服务和云原生基础设施的数据。****