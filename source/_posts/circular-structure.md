---
title: 解决循环引用的问题
categories:
  - 项目问题积累
excerpt: '解决循环引用的问题'
description: ''
date: 2023-12-11 09:44:49
---

## 问题描述

代码复现

```js
var mycar ={}
mycar.a = mycar
JSON.stringify(mycar)
```

核心的报错内容如下

```log
- Converting circular structure to JSON
    --> starting at object with constructor 'DerivedLogger'
    |     property '_readableState' -> object with constructor 'ReadableState'
    |     property 'pipes' -> object with constructor 'Console'
    --- property 'parent' closes the circle
```

## 解决问题

1. 去除依赖，尝试删除在代码中创建的任何循环依赖项。
2. 使用扁平包，示例代码：

```
const {parse, stringify} = require('flatted/cjs');

const mycar = {};
mycar.a = mycar;

stringify(mycar);
console.log(mycar)
```

3. 删除 console.logs

