---
title: "Git中一些使用场景下的命令"
date: 2023-03-17T11:30:09+08:00
lastmod: 2024-01-31T18:22:31+08:00
author: ["Ruff"]
keywords: 
- 
series: # 系列
- 
tags: # 标签
- 
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: false # 本页面是否显示评论
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
    caption: "" #图片底部描述
    alt: ""
    relative: false
---

# 子模块 git submodule
```shell
# 拉取远程仓库同时拉取子模块代码
git clone --recursive <主仓库URL>
```
如果已经执行过`git clone`, 需要拉取子模块时
```shell
# 进入主仓库目录 初始化并更新子模块
git submodule init
git submodule update
```
有时候会提示报错例如，说明引用的远程分支或提交不存在
```
fatal: remote error: upload-pack: not our ref fd311075d71a0536b0b876e158f62be8a45671d0
fatal: Fetched in submodule path 'themes/PaperMod', but it did not contain fd311075d71a0536b0b876e158f62be8a45671d0. Direct fetching of that commit failed.
fatal:
```
可以进入子模块拉取指定分支或提交
```shell
cd themes/PaperMod
git fetch 
git checkout master
git pull origin master
# 返回父仓库更新提交
cd ../..
git add themes/PaperMod
git commit -m "Update submodule to latest code"
```

# 回滚
**查看改动点**
```shell
# 忽略行末空格的差异
git diff --ignore-space-at-eol
# 暂存区的改动 
git diff --cached
# 或
git diff --staged
```
**使用 git checkout 撤销本地修改**
回滚未提交的代码
命令如下：
```shell
  git checkout .      # 撤销对所有已修改但未提交的文件的修改，但不包括新增的文件
  git checkout [filename]     # 撤销对指定文件的修改，[filename]为文件名
 ```

**回滚提交**
可以回退到任意已经提交过的版本。已 add / commit 但未 push 的文件也适用。
```shell
#[commit -hashcode]是某个 commit 的哈希值，可以用 git log 查看
# 最近的一次提交 (HEAD^)
# --soft 修改将会保留在暂存区，你可以重新调整暂存区并再次提交。
# --mixed 修改将会放回到你的工作目录中，但不会被添加到暂存区。你需要手动将它们添加到暂存区并提交
# --hard 删除最近一次的提交及其改动。慎用该命令，因为它会永久性地丢失你的改动
git reset --soft [commit -hashcode]
```

# 远程仓库
**推送私有服务器仓库**

1、登录服务器，创建远程仓库，并配置权限 
```shell
# --bare 创建没有工作目录的 Git 裸仓库
# 如果使用代码，不使用该参数，但无法推送到当前检出的分支
sudo git init --bare /Ruff/test.git
# 因为是根目录创建，所以要改变目录拥有者
sudo chown -R git:git /Ruff/test.git
```
2、配置ssh密钥，用于访问远程仓库
NOTE: 注意用户、密钥、端口等问题
```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```
如果不是标准的私钥文件名，可以配置`.ssh/config`
```config
Host example.com
    HostName example.com
    User your_username
    IdentityFile ~/.ssh/private.pem
    IdentitiesOnly yes
```
测试连接
```shell
ssh -T example.com
```
3、添加远程仓库`git remote add <name> <url>`
```shell
git remote add mine ssh://git@example.com/Ruff/test.git
```
推送仓库
```shell
git push mine main
```
NOTE: 子模块代码不会进行推送，如果要处理冲突问题，需要先拉取子模块代码

# 合并提交
尽可能地提交。频繁提交可以帮助您跟踪项目的进展并记录更改的历史，同时频繁提交还可以减少在需要时解决冲突的潜在复杂性。

然而太多的提交导致历史变得混乱，增加代码审查和版本控制的难度，记录会导致提交应该是有意义的、逻辑上相关的更改集。因此我们有时候需要合并提交。

**修正提交**
在不创建新提交的情况下修改先前的提交消息、添加漏掉的文件或者修改提交的内容。当执行这个命令时，Git 会将修改添加到先前的提交中，从而更新它的内容。适合对最近一次提交进行小的修改或补充。
```shell
git commit --amend
```

**合并提交**
```shell
git rebase -i HEAD~2
# 或者
git rebase -i <commit-hash>
```
在弹出的编辑器中，如
```
pick abc1234 Commit message 1
pick def5678 Commit message 2

# 如果您要编辑 commit message 或者 合并提交，请将 pick 改为 reword 或 squash
# pick, reword, edit, squash, fixup 都可以用作操作选项

# Rebase 1234567..def5678 onto 1234567 (2 commands)
#
# Commands:
# p, pick <commit> = 使用提交
# r, reword <commit> = 使用提交，并修改提交消息
# e, edit <commit> = 使用提交，并在重新应用时暂停以进行编辑
# s, squash <commit> = 将提交与前一个提交合并
# f, fixup <commit> = 将提交与前一个提交合并，但丢
```
将pick修改为`s`或`squash`，`git log` 查看提交已合并

然后强制推送到远程仓库
```shell
git push --force
```
