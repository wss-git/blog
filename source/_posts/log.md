---
title: 修改文件后运行报错 env:node No such file or directory
categories:
  - 项目问题积累
excerpt: '在 mac 下，修改 node_modules 里面的文件保存后报错 env: node\r: No such file or directory'
description: '在 mac 下，修改 node_modules 里面的文件保存后报错 env: node\r: No such file or directory'
date: 2024-01-15 14:32:40
---


## 问题排查

原因是在文件开头，有一行注释 `#!/usr/bin/env node`

## 解决方案

将文件格式由 CRLF 修改为 LF