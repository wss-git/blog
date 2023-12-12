---
title: npm install -g 报错 ENOTEMPTY
categories:
  - 项目问题积累
excerpt: ''
description: ''
date: 2023-12-12 09:47:50
---

<img src="/image/npm-install-error-ENOTEMPTY/not-empty.png" />

## 问题定位

ENOTEMPTY is an error code that indicates a directory is not empty and cannot be removed using the rmdir command. This error occurs when a directory contains files or other directories within it. The directory must be empty before it can be removed.

## 解决

删除目录重新安装
