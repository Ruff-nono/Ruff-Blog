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

# 坑
首次`ssh`到服务器时，直接将代码`pull`到了当前用户目录，在`nginx`中配置路由到用户目录下的代码静态资源后，发现访问403。无论是怎么修改目录权限都仍是403。

原因是当`ssh`到服务器时，我们一般在当前用户目录，即`/home/{user}`。 我们可以在`/home`目录下执行`ls -l` 查看权限，发现用户目录权限为
```shell
$ ls -l
total 12
drwxr-x--- 2 lighthouse lighthouse 4096 Nov 10 15:06 lighthouse
drwxr-x--- 9 ubuntu     ubuntu     4096 Nov 13 16:25 ubuntu
drwxr-x--- 2 usertest   test       4096 Nov 10 14:45 usertest
```
在`linux`文件系统中，该权限字符串解释为
* 第一个字符 (d)： 表示文件类型。在这里，d 表示这是一个目录。如果是文件，这个位置将显示为 -。

* 后面的九个字符（rwxr-x---）： 这是权限字符串，分为三组，每组三个字符。每组表示文件的所有者、文件所属组的用户、以及其他用户的权限。

* 第一组 (rwx)： 表示文件所有者的权限。在这里，rwx 表示读、写、执行权限都被授予给了文件的所有者。

* 第二组 (r-x)： 表示文件所属组的用户的权限。在这里，r-x 表示只有读和执行权限被授予给了文件所属组的用户。

* 第三组 (---)： 表示其他用户的权限。在这里，--- 表示其他用户没有任何权限。

* `usertest`   `test` 分别表示用户和用户组。

在`nginx.conf`配置中一般默认用户为`uid=33(www-data) gid=33(www-data) groups=33(www-data)`。因此`nginx`无法对用户目录进行资源访问。

解决方法：

所以我们通常可以将网站文件放在`/var/www/`目录下。或者修改用户目录权限，允许其他组用户执行并读取目录。
```shell
sudo chmod 755 /home/usertest
```


