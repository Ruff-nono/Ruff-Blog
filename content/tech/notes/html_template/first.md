---
title: "First"
slug: ""
date: 2023-03-20T10:17:43+08:00
lastmod: 2023-03-20T10:17:43+08:00
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

在Golang模板中，可以使用{{ if eq $variable "value" }}来判断一个变量是否等于特定值。其中$variable为变量名，"value"为要比较的值。在条件成立时会输出if语句中间的内容，反之则不会。

例如：

{{ if eq .Name "John" }}
Hello John!
{{ end }}
复制代码
这里的.Name是一个变量，如果它的值等于"John"，则会输出"Hello John!"

另外，还有ne（not equal）用于判断不等于，lt（less than）用于判断小于，le（less or equal）用于判断小于等于，gt（greater than）用于判断大于，ge（greater or equal）用于判断大于等于。