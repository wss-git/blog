---
title: Node unzip 解压失败
categories:
  - 项目问题积累
excerpt: '通过 http 请求将远端的 zip 包下载下来，然后通过 node 解压偶现 `end of central directory record signature not found`'
description: ''
date: 2023-12-12 09:45:36
---

## 问题复现

通过 http 请求将远端的 zip 包下载下来，然后通过 node 解压偶现 `end of central directory record signature not found`

## 问题解决

在解压阶段多尝试几次就好了

## 问题猜测

程序通过 http 请求数据，然后通过文件流的形式输出一个 zip 包。很可能是 http 请求已经结束了，但是文件还没有写完然后这时候就开始解压了，导致没有文件尾。
