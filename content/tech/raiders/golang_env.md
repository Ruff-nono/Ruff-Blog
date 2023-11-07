---
title: "Golang 环境安装"
slug: ""
date: 2022-9-12T10:52:44+08:00
lastmod: 2022-9-12T10:52:44+08:00
author: ["路非非"]
tags: # 标签
- golang
series:
- 
description: ""
weight:
draft: false # 是否为草稿
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

## 安装Golang

在Golang官网网站即可完成下载，链接：[https://go.dev/dl/](https://go.dev/dl/)

Windows下可以使用 .msi 后缀(在下载列表中可以找到该文件，如`go1.19.2.windows-amd64.msi`)的安装包来安装。

linux下安装示例
```shell
wget https://go.dev/dl/go1.19.4.linux-amd64.tar.gz

tar -C ~/Ruff/ -xzf go1.19.4.linux-amd64.tar.gz
```

安装完成后，在对应安装目录的 xxx\bin 目录下打开cmd, 输入`go version`命令<br/>返回go version go1.19 windows/amd64即表示安装完成。

## 配置环境变量

为方便使用，我们需要配置golang对应的环境变量

新建环境变量`GO_HOME: D:\GO\SDK`

编辑path变量，新增 `%GO_HOME%\bin`

linux下
编辑 ~/.bash_profile 或者 /etc/profile
```
export PATH=$PATH:/usr/local/go/bin
```
添加后需要执行
```shell
source ~/.bash_profile
或
source /etc/profile
```
## Goland安装

官网下载最新版本[https://www.jetbrains.com/go/download/](https://www.jetbrains.com/go/download/)

## Go 私有仓库配置

go env设置

国内仓库地址
```
go env -w GOPROXY=https://goproxy.cn,direct
```  
私有仓库地址
```
go env -w GOPRIVATE=gitlab.xxx.xxx
```  
若私仓为http则设置，否则CA证书验证不通过报错
```
go env -w GOINSECURE=gitlab.xxx.xxx
```

git配置 使用ssh
```
git config --global --add url."ssh://xxxxx/".insteadOf "http://xxxxxx/"
```
