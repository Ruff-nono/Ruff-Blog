---
title: "Golang 环境安装"
slug: "中文路径"
author: "Ruff"
date: 2022-9-12T10:52:44+08:00
lastmod: 2022-10-17T15:57:04+08:00
description: "window环境"
tags:
- "Golang"
showToc: true
---

## 安装Golang

在Golang官网网站即可完成下载，链接：[https://go.dev/dl/](https://go.dev/dl/)

Windows下可以使用 .msi 后缀(在下载列表中可以找到该文件，如`go1.19.2.windows-amd64.msi`)的安装包来安装。

安装完成后，在对应安装目录的 xxx\bin 目录下打开cmd, 输入`go version`命令<br/>返回go version go1.19 windows/amd64即表示安装完成。

## 配置环境变量

为方便使用，我们需要配置golang对应的环境变量

新建环境变量`GO_HOME: D:\GO\SDK`

编辑path变量，新增 `%GO_HOME%\bin`

## Goland安装

官网下载最新版本[https://www.jetbrains.com/go/download/](https://www.jetbrains.com/go/download/)

`有条件的可以支持正版`

无条件的参考破解攻略[https://www.bilibili.com/read/cv18308655](https://www.bilibili.com/read/cv18308655)

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
go env -w GOINSECURE gitlab.xxx.xxx
```

git配置 使用ssh
```
git config --global --add url."ssh://xxxxx/".insteadOf "http://xxxxxx/"
```
