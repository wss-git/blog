---
title: 包引用位置不同instanceof不符合预期
categories:
  - 项目问题积累
excerpt: '本地开发两个包：a、b 和 c，b 依赖 a 包，a 和 b 都依赖 c 包。发布了 a 包之后发现有些问题，将 a 包软链到本地调试。然后问题就来了，之前写的 instanceOf 报错了'
description: ''
date: 2023-12-12 09:26:53
---

## 现象描述

本地开发两个包：a、b 和 c，b 依赖 a 包，a 和 b 都依赖 c 包。发布了 a 包之后发现有些问题，将 a 包软链到本地调试。然后问题就来了，之前写的 instanceOf 报错了。伪代码如下：

c:

```js
module.exports = class C {};
```

a:

```js
const C = require('c');
const a = new C();

return a;
```

b:

```js
const a = require('a');
const C = require('c');

console.log(a instanceOf C);
```

## 问题定位

经排查，问题是软链接带来的问题。使用软链接之后形成了 a 引入自己项目依赖的 c 文件，b 也引入自己目录的 c 文件，导致了代码一直，但是其实引入了不同的文件。

