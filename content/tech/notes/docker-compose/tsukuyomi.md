---
title: "使用docker-compose组合一套应用运行环境"
slug: ""
date: 2023-11-03T17:58:21+08:00
lastmod: 2023-11-03T17:58:21+08:00
author: ["路非非"]
tags: # 标签
-
series:
-
description: "介绍以下docker compose 顺便记录一些常用的中间件镜像引用"
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
在日常开发中，如果你经常遇到希望本地测试接口，但是又没有合适的数据库或者第三方接口环境可以使用，又或者流程测试时，人员共享数据导致的各种脏数据的困扰。

如果希望有一个干净的整体环境，每次干净的启动，干净的退出，正如人生一样。那么本文应该对你是有用的。

# 记录每一个容器的诞生过程

## mysql
```yaml
version: '3.9'
services:
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: your-root-password
      MYSQL_USER: your-username
      MYSQL_PASSWORD: your-password
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    restart: always
```



