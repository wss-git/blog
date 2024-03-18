---
title: mac-not-permitted
categories:
  - Linux
excerpt: 'mac 终端启动报错 chown: /usr/local/bin/corepack: Operation not permitted'
description: ''
date: 2024-03-18 20:08:02
---

## 报错信息

重装了一下 node 版本，每次启动终端都报错:

![alt text](/image/mac-not-permitted/image.png)

## 解决方案

运行命令 `sudo chown -R $(whoami) /usr/local/*`, 然后输入电脑开机密码，重新启动即可。