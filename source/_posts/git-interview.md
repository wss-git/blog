---
title: git 常遇到的问题
categories:
  - git
excerpt: '如何添加多个仓库、游离分支、git log 和 git reflog 有什么区别等'
description: ''
date: 2023-12-12 15:14:59
---


## git 如何添加多个仓库

```bash
# 新增远程仓库
git remote add <alias> <url>

# remote 其他高频命令
git remote rename <old-alias> <new-alias>
git remote remove <alias>
```

## git 游离分支

> [参考链接](https://juejin.cn/post/6930097627589509127)

当我们切换到指定的某一次提交，HEAD 就会处于「detached」状态，也就是游离状态。

**好处**：HEAD 处于游离状态时，开发者可以很方便地在历史版本之间互相切换。
**弊端**：若在该基础上进行了提交，则会新开一个「匿名分支」；也就是说我们的提交是无法可见保存的，一旦切换到别的分支，原游离状态以后的提交就不可追溯了。(如何解决:在切换到游离状态的时候应该新建一个分支，然后我们所有的操作修改和提交都会保存到该分支，HEAD 也就指向了该分支最新提交的 commit id 处，而不会再处于游离状态。)

```BASH
# 方法1：创建新的临时分支
git checkout -b test
git checkout main
git pull
git branch -D test # 不可恢复操作，确认test不再需要后执行

# 方法2：无条件切回mian
git fetch origin
git reset --hard origin/main
```

## git log 和 git reflog 有什么区别

git reflog 和 git log 的区别在于它们记录的内容不同。

git reflog 记录了你本地仓库中所有的 HEAD 和分支的移动。它能够帮助你找回已经被删除的分支或者丢失的提交。

git log 记录了提交历史。它按时间顺序列出所有的提交，包括提交的作者、提交的时间、提交的信息等。

## git 如何删除本地仓库未跟踪的文件

```bash
git clean -fn  # 如果需要删除 ignore 的文件，可以加上 -x
```

> 更多内容点击查看[相关笔记](/Git/git-shell/)

