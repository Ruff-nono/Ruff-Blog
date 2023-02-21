---
title: "Ansible"
slug: ""
date: 2023-02-02T15:46:45+08:00
lastmod: 2023-02-02T15:46:45+08:00
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

# 前言
历史遗留了一批远古巨兽（虚拟机部署，双集群30+28）。通过ansible-playbook进行部署管理，学习一下，留个笔记。

# ansible介绍
`Ansible`默认通过 SSH 协议管理机器.

管理主机支持Red Hat, Debian, CentOS, OS X, BSD等系统的各种版本 (windows系统不可以做控制主机)，只要机器上安装了 Python 2.6 或 Python 2.7 就可以运行
ansible。

远程被管理主机不需要安装任何软件。

> Python 3 与 Python 2 是稍有不同的语言,大多数Python程序还不能在 Python 3 中正确运行.一些Linux发行版(Gentoo, Arch)没有默认安装 Python 2.X 解释器.在这些系统上,你需要安装一个 Python 2.X 解释器,并在 inventory (详见 Inventory文件) 中设置 ‘ansible_python_interpreter’ 变量指向你的 2.X Python.你可以使用 ‘raw’ 模块在托管节点上远程安装Python 2.X.
例如：ansible myhost --sudo -m raw -a "yum install -y python2 python-simplejson" 这条命令可以通过远程方式在托管节点上安装 Python 2.X 和 simplejson 模块.
Red Hat Enterprise Linux, CentOS, Fedora, and Ubuntu 等发行版都默认安装了 2.X 的解释器,包括几乎所有的Unix系统也是如此.

# 安装管理主机

...

# 

