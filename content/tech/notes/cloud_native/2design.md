---
title: "【二】如何云原生"
slug: "design"
date: 2022-10-19T15:40:40+08:00
lastmod: 2022-10-19T15:40:40+08:00
author: ["路非非"]
tags: # 标签
- 

series:
- 云原生

description: ""
weight: 2
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

> 云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。
> 云原生的代表技术包括 容器、服务网格、微服务、不可变基础设施 和 声明式 API。
> 这些技术能够构建容错性好、易于管理和便于观察的松耦合系统。结合可靠的自动化手段，云原生技术使工程师能够轻松地对系统作出频繁和可预测的重大变更。——CNCF（云原生计算基金会）。

## 云原生的设计理念

* 面向分布式设计（Distribution）：容器、微服务、API 驱动的开发；
* 面向配置设计（Configuration）：一个镜像，多个环境配置；
* 面向韧性设计（Resistancy）：故障容忍和自愈；
* 面向弹性设计（Elasticity）：弹性扩展和对环境变化（负载）做出响应；
* 面向交付设计（Delivery）：自动拉起，缩短交付时间；
* 面向性能设计（Performance）：响应式，并发和资源高效利用；
* 面向自动化设计（Automation）：自动化的 DevOps；
* 面向诊断性设计（Diagnosability）：集群级别的日志、metric 和追踪；
* 面向安全性设计（Security）：安全端点、API Gateway、端到端加密；

## 云计算介绍

云原生借了云计算的东风，没有云计算，自然没有云原生，云计算是云原生的基础。

通俗的讲，云计算可以说是一种配置资源的方式，随着虚拟化技术，可持续交付，编排系统的快速发展及普及，应用上云已经是不可逆转的趋势。
云计算从宏观上按3层划分，即基础设施即服务(IaaS)、平台即服务(PaaS)、软件即服务(SaaS)为云原生提供了技术基础和方向指引，

* IaaS：这是为了想要建立自己的商业模式并进行自定义的客户，例如亚马逊的 EC2、S3 存储、Rackspace 虚拟机等都是 IaaS。
* PaaS：工具和服务的集合，对于想用它来构建自己的应用程序或者想快速得将应用程序部署到生产环境而不必关心底层硬件的用户和开发者来说是特别有用的，
比如 Cloud Foundry、Google App Engine、Heroku 等。
* SaaS：终端用户可以直接使用的应用程序。这个就太多，我们生活中用到的很多软件都是 SaaS 服务，只要基于互联网来提供的服务基本都是
SaaS 服务，有的服务是免费的，比如 Google Docs，还有更多的是根据我们购买的 Plan 和使用量付费，比如 GitHub、各种云存储。

真正的云化不仅仅是基础设施和平台的变化，应用也需要做出改变，摈弃传统的土方法，在架构设计、开发方式、部署维护等各个阶段和方面都基于云的特点，重新设计，从而建设全新的云化的应用，即云原生应用。

---

Kubernetes作为一个分布式集群管理系统，它的一个重要目标是：将适合的资源分配给适合的应用，满足对应用的QoS要求和获得最优的资源使用效率。