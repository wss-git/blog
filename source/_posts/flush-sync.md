---
title: react 异步任务转同步
categories:
  - 基础知识
excerpt: 'react 异步任务转同步'
description: 'react 异步任务转同步'
date: 2023-12-12 09:21:41
---

## 抛出问题

由于 react 的批处理，会合并为一次 render

```javascript
function handleClick() {
  setLoading(true);
  setCount(1);
  setLoading(false);
}
```

## 解决问题

React 中的批处理简单来说就是将多个状态更新合并为一次重新渲染，以获得更好的性能。如果要一个请求的 loading 效果怎么实现呢？

### 方式一: 利用 Promise、setTimeout

```javascript
function handleClick() {
  setTimeout(() => {
    setCount(2);
    setLoading(false);
  });
  setLoading(true);
}
```

### 方式二: 利用 await

```javascript
async function handleClick() {
  setLoading(true);
  await setCount(2);
  setLoading(false);
}
```

### 方式三: 利用 react17 新出的 flushSync

> flushSync 会以函数为作用域，函数内部的多个 setState 仍然为批量更新，这样可以精准控制哪些不需要的批量更新

```javascript
import React, { useState } from 'react';
import { flushSync } from 'react-dom';

export default () => {
  const [count, setCount] = useState(1);
  const click = async () => {
    flushSync(() => setLoading(true));
    setCount(2);
    setLoading(false);
  };
  return <button onClick={click}> count </button>;
};
```

