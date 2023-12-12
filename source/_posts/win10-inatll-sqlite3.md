---
title: win10-inatll-sqlite3
categories:
  - 默认
excerpt: ''
description: ''
date: 2023-12-12 10:36:33
---

## 下载包

打开[官网](https://www.sqlite.org/download.html)下载 sqlite3，根据操作系统下载`32位`或`64位`，**必须**下载 `sqlite-tools-xxxx.zip`那个包

## 解压包

解压好之后有这五个文件
![win10-install-sqlite2-unzip.png](/image/win10-install-sqlite3/unzip.png)

## 配置环境变量

打开系统环境变量，在 PATH 变量下新增一条数据，填写`C:\Program Files\sqlite` (因为这次配置如上图，是将文件都复制到了 `C:\Program Files\sqlite`)。

## 验证

打开 cwd 终端，输入 `sqlite3 --version`
