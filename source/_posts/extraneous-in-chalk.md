---
title: 运行报错extraneous-in-chalk
categories:
  - 项目问题积累
excerpt: '使用 `command-line-usage` 包做 help 的输出，一直抛出异常 `Error: Found extraneous } in Chalk template literal`'
description: ''
date: 2023-12-12 09:13:02
---

> command-line-usage 用于在命令行中美化输出信息

## 问题记录

使用 `command-line-usage` 包做 help 的输出，示例如下。调用方式 1 一直抛出异常 `Error: Found extraneous } in Chalk template literal`，经[查询](https://github.com/chalk/chalk-template/issues/4)需要通过方式 2 或者方式 3 调用

```typescript
import commandLineUsage, { Section } from 'command-line-usage';

function help(sections: Section | Section[]) {
  console.log(commandLineUsage(sections));
}

// 调用方式1
help([
  {
    header: 'Examples with Yaml',
    content: [`$ --route-policy '{}' --weight 20`],
  },
]);
// 调用方式2
help([
  {
    header: 'Examples with Yaml',
    content: [`$ --route-policy '\\{\\}' --weight 20`],
  },
]);
// 调用方式3
help([
  {
    header: 'Examples with Yaml',
    content: [String.raw`$ --route-policy '\{\}' --weight 20`],
  },
]);
```
