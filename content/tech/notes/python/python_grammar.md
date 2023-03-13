---
title: "Python语法"
slug: ""
date: 2022-10-12T14:46:43+08:00
lastmod: 2022-10-12T14:46:43+08:00
author: ["路非非"]
tags: # 标签
- python
series:
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

# 注释
- 单行注释 #
- 多行注释 在注释内容的开始和结尾分别使用三个单引号或双引号

```python {linenos=true}
print("hello world")

def print(self, *args, sep=' ', end='\n', file=None): # known special case of print
    """
    print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
    
    Prints the values to a stream, or to sys.stdout by default.
    Optional keyword arguments:
    file:  a file-like object (stream); defaults to the current sys.stdout.
    sep:   string inserted between values, default a space.
    end:   string appended after the last value, default a newline.
    flush: whether to forcibly flush the stream.
    """
```

# 格式化方法

有时候我们会想要从其他信息中构建字符串。这正是 format() 方法大有用武之地的地方。

```python {linenos=true}
age = 20
name = 'Swaroop'
print('{0} was {1} years old when he wrote this book'.format(name, age))
print('Why is {0} playing with that python?'.format(name))
```

```
Swaroop was 20 years old when he wrote this book
Why is Swaroop playing with that python?
```