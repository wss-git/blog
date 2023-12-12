---
title: 重构console输出时区
categories:
  - 笔记
excerpt: '在 linux 环境下，console 输出时区总是 UTC，需要手动设置时区。在一些场景无法配置，可以重写 console 实现自定义时区。'
description: ''
date: 2023-12-12 09:56:57
---


```javascript
'use strict';

var util = require('util');

function _writeToStdout(level, msg) {
  const now = new Date();
  // 自定义格式
  const timeStr = `${now.getFullYear()}-${
    now.getMonth() + 1
  }-${now.getDate()}T${now.getHours()}:${now.getMinutes()}:${
    (now.getSeconds() * 1000 + now.getMilliseconds()) / 1000
  }Z`;
  // const timeStr = now.toLocaleString();
  let requestID =
    process._fc && process._fc.requestId ? process._fc.requestId : '';
  let logMsg = `${timeStr} ${requestID} [${level}] ${msg}`;
  logMsg = logMsg.replace(/\n/g, '\r');
  process.stdout.write(logMsg + '\n');
}

var log = function (level, msg, ...params) {
  var logMsg = util.format(msg, ...params);
  _writeToStdout(level, logMsg);
};

console.log = function (msg, ...params) {
  log('verbose', msg, ...params);
};
console.info = function (msg, ...params) {
  log('info', msg, ...params);
};
console.warn = function (msg, ...params) {
  log('warn', msg, ...params);
};
console.error = function (msg, ...params) {
  log('error', msg, ...params);
};
console.debug = function (msg, ...params) {
  log('debug', msg, ...params);
};

exports.handler = (event, context, callback) => {
  // const eventObj = JSON.parse(event.toString());
  console.log('hello world');
  callback(null, 'hello world');
};
```

