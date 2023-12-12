---
title: 文件输入流监听失效
categories:
  - 项目问题积累
excerpt: '在开发 node cli 时遇到一个奇葩的问题：如果存在 `inquirer` 的交互就会导致项目监听输入失败，如果没有交互则没有问题。'
description: '文件输入流监听失效'
date: 2023-12-12 10:16:25
---


## 问题描述

在开发 node cli 时遇到一个奇葩的问题：如果存在 `inquirer` 的交互就会导致项目监听输入失败，如果没有交互则没有问题。

测试代码

```js
const inquirer = require('inquirer');

(async function () {
  const s = await inquirer.prompt([
    {
      type: 'list',
      name: 'name',
      message: 'Please select a type:',
      choices: [
        { name: 'test add', value: 'add' },
        { name: 'test over', value: 'over' },
      ],
    },
  ]);
  console.log(s);

  console.log('开始输入:\n');
  process.stdin.setEncoding('ascii');
  process.stdin.setRawMode(true);
  process.stdin.on('data', (chunk) => {
    console.log("----- 输入 -----", chunk);
  });

  setTimeout(() => {
    console.log('======= setTimeout end =======')
    process.exit(0);
  }, 5000);
})()
```

## 案例分析

监听失败的原因是 `inquirer` 这个包会使 `process.stdin` 处于运行状态
http://nodejs.cn/api/stream.html#readableispaused

## 问题解决

```js
const inquirer = require('inquirer');

(async function () {
  const s = await inquirer.prompt([
    {
      type: 'list',
      name: 'name',
      message: 'Please select a type:',
      choices: [
        { name: 'test add', value: 'add' },
        { name: 'test over', value: 'over' },
      ],
    },
  ]);
  console.log(s);


  if (process.stdin.isPaused()) {
    process.stdin.resume(); // 将流切换到流动模式
  }

  console.log('开始输入:\n');
  process.stdin.setEncoding('ascii');
  process.stdin.setRawMode(true);
  process.stdin.on('data', (chunk) => {
    console.log("----- 输入 -----", chunk);
  });

  setTimeout(() => {
    console.log('======= setTimeout end =======')
    process.exit(0);
  }, 5000);
})()
```

