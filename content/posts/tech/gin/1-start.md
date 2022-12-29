---
title: "gin框架(一)、快速入门"
slug: ""
date: 2022-11-03T15:01:12+08:00
lastmod: 2022-11-03T15:01:12+08:00
author: ["路非非"]
tags: # 标签
- "go"
- "gin"
series:
- "go-gin"
description: ""
weight: 1
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

## 1、什么是gin
`Gin` 是一个用`Go (Golang)`编写的 开源`web` 框架。 目前在`GitHub Start 47.4K`, 它是一个类似于 `martini` 但拥有更好性能的 `API` 框架，
路由解析由于使用的是`httprouter`，速度提高了近 40 倍。

## 2、安装

``` {linenos=false} 
go get -u github.com/gin-gonic/gin
```
代码中导入
```go {linenos=false}
import "github.com/gin-gonic/gin"
//（可选）导入net/http. 例如，如果使用常量，例如http.StatusOK.
import "net/http"
```

## 3、启动服务

```go {linenos=false}
package main

import (
  "net/http"

  "github.com/gin-gonic/gin"
)

func main() {
  r := gin.Default()
  r.GET("/ping", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
      "message": "pong",
    })
  })
  r.Run() // listen and serve on 0.0.0.0:8080 (for windows "localhost:8080")
}
```

## 4、访问

```
D:\>curl localhost:8080/ping
{"message":"pong"}
```
