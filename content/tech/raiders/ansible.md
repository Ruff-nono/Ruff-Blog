---
title: "Ansible 安装管理"
slug: ""
date: 2023-02-16T10:26:50+08:00
lastmod: 2023-02-16T10:26:50+08:00
author: ["路非非"]
tags: # 标签
- ansible
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

# 安装管理主机

## 从源码运行
从项目的checkout中可以很容易运行Ansible,Ansible的运行不要求root权限,也不依赖于其他软件,不要求运行后台进程,也不需要设置数据库.
因此我们社区的许多用户一直使用Ansible的开发版本,这样可以利用最新的功能特性,也方便对项目做贡献.因为不需要安装任何东西,跟进Ansible的开发版相对于其他开源项目要容易很多.

从源码安装的步骤
```shell
$ git clone git://github.com/ansible/ansible.git --recursive
$ cd ./ansible
```

使用 Bash:
```shell
$ source ./hacking/env-setup
```

使用 Fish:
```shell
$ . ./hacking/env-setup.fish
```

If you want to suppress spurious warnings/errors, use:
```shell
$ source ./hacking/env-setup -q
```

如果没有安装pip, 请先安装对应于你的Python版本的pip:
```shell
$ sudo easy_install pip
```

以下的Python模块也需要安装 [1]_:
```shell
$ sudo pip install paramiko PyYAML Jinja2 httplib2 six
```

**注意,当更新ansible版本时,不只要更新git的源码树,也要更新git中指向Ansible自身模块的 “submodules” (不是同一种模块)**
```shell
$ git pull --rebase
$ git submodule update --init --recursive
```
一旦运行env-setup脚本,就意味着Ansible从源码中运行起来了.默认的inventory文件是 /etc/ansible/hosts.inventory文件也可以另行指定 (详见 Inventory文件) :
```shell
$ echo "127.0.0.1" > ~/ansible_hosts
$ export ANSIBLE_HOSTS=~/ansible_hosts
```

你可以在手册的后续章节阅读更多关于 inventory 文件的使用,现在让我们测试一条ping命令:

```shell
$ ansible all -m ping --ask-pass
```

你也可以使用命令 “sudo make install”

## 通过yum安装
```shell
# install the epel-release RPM if needed on CentOS, RHEL, or Scientific Linux
$ sudo yum install ansible
```