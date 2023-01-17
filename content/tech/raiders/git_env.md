---
title: "Git_env"
description: "---"
tags: []
date: 2022-10-12T15:32:21+08:00
draft: true
---


如果首次克隆仓库及其模块，使用：
```shell
git clone --recursive 仓库地址
```

对于仓库首次拉取模块，可以使用:
```shell
git submodule update --init --recursive

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


