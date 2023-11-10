---
title: "Commands"
slug: ""
date: 2023-03-17T11:30:09+08:00
lastmod: 2023-03-17T11:30:09+08:00
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

# 回滚

## 使用 git checkout 撤销本地修改

即放弃对本地已修改但尚未提交的文件的修改，还原其到未修改前的状态。  
注意： 已 add/ commit 的文件不适用个方法，应该用本文提到的第二种方法。  
命令如下：
```shell
  git checkout .      # 撤销对所有已修改但未提交的文件的修改，但不包括新增的文件
  git checkout [filename]     # 撤销对指定文件的修改，[filename]为文件名
 ```

## 使用 git reset 回退项目版本

可以回退到任意已经提交过的版本。已 add / commit 但未 push 的文件也适用。

命令如下：
```shell
git reset --hard [commit -hashcode]
#[commit -hashcode]是某个 commit 的哈希值，可以用 git log 查看
```
因此一般用 `git log` 查看历史版本的 `commit` 哈希值，然后 `reset` 到对应版本。

## 查看文件及文件数量
```shell
git ls-files
git ls-files | wc -l
```

```shell
git branch -a
git checkout -b master remotes/origin/master
```




