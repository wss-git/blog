---
title: Git 常用的命令
categories:
  - git
excerpt: 'Git 常用的命令'
description: 'Git 常用的命令'
date: 2023-12-12 09:24:03
---

## 设置用户名与邮箱

```
git config --global user.name "xxxxx"
git config --global user.email "xxxxxx@126.com"
```

## 添加密钥

```shell
ssh-keygen -t rsa -C "xxxxx@126.com"
```

## git commit 撤销

```
git reset --soft HEAD^
```

## git 回退到某个 commit

```
git reset --hard <commit_id>
```

## 删除远程分支

```shell
git push origin --delete <branchName>
```

## 代码找回

```shell
git reflog show --date=iso # 输出提交日志

git checkout -b feat-honey b592f8e # 找到提交记录切出分支

git diff-tree -r --no-commit-id --name-only f4710c4a32975904b00609f3145c709f31392140 | xargs tar -rf update_201800001.tar # 将某次 commit 修改的文件导出
```

## 当前仓库 设置 用户名和邮箱

优先级由上至下 依次降低

- local 项目级
- global 当前用户级
- system 系统级

```
git config --local user.name "xxxxx"
git config --local user.email "xxxx@126.com"
```

## http 方式 clone 记住密码

```shell
git config credential.helper store
```

## 忽略某文件

```shell
git update-index --assume-unchanged <filename>
```

## 重新添加已忽略的文件

```shell
git add -f <filename>
```

## 使用.gitignore 无效的解决方法

```shell
git rm -r --cached .

git add .
```

## 删除无效的远端分支

使用 remote show 查看远程仓库信息
```bash
git remote show origin
```

```log
* remote origin
  Fetch URL: https://gitee.com/wss-gitee/blog.git
  Push  URL: https://gitee.com/wss-gitee/blog.git
  HEAD branch: main
  Remote branches:
    main                           tracked
    refs/remotes/origin/docusaurus stale (use 'git remote prune' to remove)
    refs/remotes/origin/dumi       stale (use 'git remote prune' to remove)
    refs/remotes/origin/hexo       stale (use 'git remote prune' to remove)
    refs/remotes/origin/mkdocs     stale (use 'git remote prune' to remove)
  Local branch configured for 'git pull':
    main merges with remote main
  Local ref configured for 'git push':
    main pushes to main (up to date)
```


使用 remote prune 将本地有，但是远端不存在分支删除
```bash
git remote prune origin
```