---
title: 通过函数计算管理阿里云NAS
categories:
  - 默认
excerpt: '通过函数计算管理阿里云NAS'
description: '通过函数计算管理阿里云NAS'
date: 2024-04-07 18:19:41
---

## 通过 kodbox 管理

通过应用直接[创建](https://fcnext.console.aliyun.com/applications/create?template=start-fc-kodbox)，然后修改配置管理NAS

## 简单管理

1. [创建函数](https://fcnext.console.aliyun.com/cn-beijing/functions/create)
 
2. 配置 VPC 和 NAS，NAS 目录配置推荐使用 `/`。
![配置项](/image/fc-nas/config.jpg)

3. 调用函数

4. 登陆函数实例，通过 linux 命令管理文件