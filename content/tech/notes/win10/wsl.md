---
title: "Wsl启动无响应"
date: 2023-11-17T16:41:35+08:00
lastmod: 2023-11-17T16:41:35+08:00
author: ["Ruff"]
keywords: 
- 
categories: # 没有分类界面可以不填写
- 
tags: # 标签
- 
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

## 问题

运行 `wsl` 时无响应，无奈只能 `CTRL+C`
![11](wsl.png)

执行 `sc query LxssManager`, 发现状态为STOP PENDING， 而正常状态应为 RUNNING
![22](query.png)

首先通过 `tasklist /svc /fi "imagename eq svchost.exe" | findstr LxssManager` 来获取运行 `LxssManager` 的svchost.exe的PID
![33](pid.png)

运行任务管理器，在详细信息选项卡中，找到对应 PID 的 svchost.exe，右键结束进程树

通过 `net start LxssManager` 重新启动

此时可以正常执行 `wsl -l -v` 等相关命令。

## LxssManager 简介

服务管理器中对 `LxssManager` 的描述
> LXSS Manager 服务支持运行本机 ELF 二进制文件。该服务提供在 Windows 上运行 ELF 二进制文件所需的基础结构。如果停止或禁用该服务，这些二进制文件将不再运行。

LxssManager的作用
>Windows Subsystem for Linux是一个独立的环境子系统，是NT内核固有机制，之前具有OS/2子系统和POSIX子系统，后来Windows发展中删除了。每个子系统通常有一个子系统服务进程和一个内核态驱动组成，Windows子系统则是CSRSS.exe和Win32k.sys。
>
>WSL子系统则是服务进程LxssManager，服务管理器中可以看到这个服务。LxssManager的核心代码是一个DLL，它位于system32\lxss子目录，这个DLL运行在svchost.exe宿主进程中。内核空间运行的Linux系统驱动有lxss.sys和LxCore.sys两个。
