---
title: Node 执行子程序输出到主程序
categories:
  - 笔记
excerpt: 'Node 执行子程序输出到主程序'
description: ''
date: 2023-12-12 10:04:41
---

```js
const { spawnSync } = require('child_process');

spawnSync('ls -al', { encoding: 'utf8', stdio: 'inherit', shell: true, });
```
