---
title: "测试测试测试"
slug: ""
date: 2023-03-13T14:51:03+08:00
lastmod: 2023-03-13T14:51:03+08:00
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


```mermaid
classDiagram
  class Engine
  Engine : +[]methodTree trees
  Engine : +...
  Engine : +deposit(amount)
  Engine : +withdrawal(amount)
```

``` mermaid
classDiagram 
classA <|-- classB   
classC *-- classD   
classE  o -- classF  
classG <-- classH   
classI -- classJ   
classK <.. classL   
classM <|.. classN   
classO .. class
```

```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    Animal <|-- Zebra
    Animal : +int age
    Animal : +String gender
    Animal: +isMammal()
    Animal: +mate()
    class Duck{
        +String beakColor
        +swim()
        +quack()
    }
    class Fish{
        -int sizeInFeet
        -canEat()
    }
    class Zebra{
        +bool is_wild
        +run()
    }
```

{{ if .Page.Store.Get "hasMermaid" }}
  <script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
  <script>
    mermaid.initialize({ startOnLoad: true });
  </script>
{{ end }}