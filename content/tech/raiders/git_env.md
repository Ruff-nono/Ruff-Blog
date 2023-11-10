---
title: "Git 环境安装"
slug: ""
date: 2022-09-15T10:57:49+08:00
lastmod: 2022-09-15T10:57:49+08:00
author: ["路非非"]
tags: # 标签
- git
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

如果首次克隆仓库及其模块，使用：
```shell
git clone --recursive 仓库地址
```

对于仓库首次拉取模块，可以使用:
```shell
git submodule update --init --recursive
git submodule update --remote themes/PaperMod

```
更新子模块(适用于git 1.8.2及以上版本)
```shell
git submodule update --recursive --remote
```
更新子模块(适用于git 1.7.3及以上版本)
```shell
git submodule update --recursive
```

或者
```shell
git pull --recurse-submodules
```


