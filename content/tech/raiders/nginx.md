---
title: "Ubuntu下配置Nginx"
slug: ""
date: 2023-01-17T15:32:12+08:00
lastmod: 2023-01-17T15:32:12+08:00
author: ["路非非"]
tags: # 标签
-
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
# 前言
Nginx 发音 “engine x” ,是一个高性能 HTTP 和反向代理服务器，用来在互联网上处理一些大型网站。它可以被用作独立网站服务器，负载均衡，内容缓存和针对 HTTP 和非 HTTP 的反向代理服务器。

本文描述如何在 Ubuntu 20.04上安装和管理 Nginx。

# 安装
Nginx 在默认的 Ubuntu 源仓库中可用。想要安装它，运行下面的命令：

```shell
sudo apt update
sudo apt install nginx
```

安装完成后验证
```shell
$ systemctl status nginx
```

```shell
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-11-08 10:46:04 CST; 2min 34s ago
       Docs: man:nginx(8)
 ...
```

# 配置防火墙

# 访问验证
浏览器输入服务器的公网ip，可观察到
![welcome](welcome.png)

# nginx 目录
`/etc/nginx/`：Nginx的主要配置文件和相关配置文件存放在这个目录中，包括：

* `/etc/nginx/nginx.conf`：Nginx的主配置文件。
* `/etc/nginx/conf.d/`：用于存放额外的配置文件。
* `/etc/nginx/sites-available/`：存放网站配置文件的目录。
* `/etc/nginx/sites-enabled/`：包含已启用的网站配置文件的符号链接。
* `/etc/nginx/snippets/`：可重复使用的配置段（例如，SSL配置、安全头等）可以存放在这里。
* `/etc/nginx/modules-enabled/`：包含Nginx模块的配置文件。
* `/usr/share/nginx/`：这个目录通常包含一些Nginx的静态资源文件，例如默认的错误页面、HTML文件等。

`/var/log/nginx/`：Nginx的日志文件存放在这个目录中，包括：

* `/var/log/nginx/access.log`：访问日志。
* `/var/log/nginx/error.log`：错误日志。
* `/var/www/`：这个目录通常是用来存放网站文件的根目录，但不是Nginx的默认根目录。您可以在Nginx配置中指定不同的根目录。

`/run/nginx/`：这个目录包含Nginx主进程的PID文件，通常是 /run/nginx/nginx.pid。

# 配置路由
```shell
sudo vim /etc/nginx/sites-available/your-config-file
```
添加路由配置
```nginx configuration
server {
    listen 80;
    server_name your_domain.com;

    location / {
        proxy_pass http://your_backend_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 可以根据需要添加其他路由配置
}

```
启用虚拟主机： 确保将这个配置文件（/etc/nginx/sites-available/）链接到/etc/nginx/sites-enabled/目录中，
以启用虚拟主机。可以使用ln -s命令创建符号链接
```shell
ln -s /etc/nginx/sites-available/your-config-file /etc/nginx/sites-enabled/
```
重启nginx
```shell
sudo nginx -t
sudo systemctl reload nginx
```