---
title: express路由访问问题
categories:
  - 项目问题积累
excerpt: '记录项目实战中一次关于 express 路由的问题'
description: ''
date: 2023-12-12 09:10:49
---


## 问题简述

按照下面的问题复现，为什么能够请求到 `/init`, 但是请求不到 `initialize`呢？如何才能请求到此路径呢？

## 问题复现

1. 安装 express

```shell
npm install express
```

2. 创建 index.js

```javascript
const express = require('express');
const router = require('express').Router();
const app = express();
const PORT = 9000;
const HOST = '0.0.0.0';

app.post('/init', async (_req, res) => {
  console.log('请求到了 init');
  res.send('app init');
});

router.post('/initialize', async (_req, res) => {
  console.log('请求到了 initialize');
  res.send('router initialize');
});

router.get('/', async (_req, res) => {
  console.log('请求到了 /');
  res.send('router /');
});

app.use('/*', router);

const server = app.listen(PORT, HOST);
console.debug(`Running on http://${HOST}:${PORT}`);

server.timeout = 0; // never timeout
server.keepAliveTimeout = 0; // keepalive, never timeout
```

3. 创建 fetch 文件

```javascript
const http = require('http');

const fetch = (options) => {
  const req = http.request(options, (res) => {
    console.log(`状态码: ${res.statusCode}`);

    res.on('data', (d) => {
      process.stdout.write(d);
    });
  });
  req.on('error', (error) => {
    console.error(error);
  });

  req.end();
};

// 成功运行
fetch({
  hostname: '0.0.0.0',
  port: 9000,
  path: '/init',
  method: 'POST',
});

// 成功运行
fetch({
  hostname: '0.0.0.0',
  port: 9000,
  path: '/',
  method: 'GET',
});

// 请求不到结果
fetch({
  hostname: '0.0.0.0',
  port: 9000,
  path: '/initialize',
  method: 'POST',
});
```

4. 启动一个终端运行 `node index`，另起一个终端运行 `node fetch`。得到结果 `init` 请求成功，`initialize` 请求失败

## 问题解决

```javascript
const express = require('express');
const router = require('express').Router();
const app = express();
const PORT = 9000;
const HOST = '0.0.0.0';

app.post('/init', async (_req, res) => {
  console.log('请求到了 init');
  res.send('app init');
});

router.post('/initialize', async (_req, res) => {
  console.log('请求到了 initialize');
  res.send('router initialize');
});

router.get('/*', async (_req, res) => {
  console.log('请求到了 /');
  res.send('router /');
});

app.use(router);

const server = app.listen(PORT, HOST);
console.debug(`Running on http://${HOST}:${PORT}`);

server.timeout = 0; // never timeout
server.keepAliveTimeout = 0; // keepalive, never timeout
```

