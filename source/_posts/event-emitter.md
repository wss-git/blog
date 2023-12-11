---
title: EventEmitter
categories:
  - 笔记
excerpt: 'Node.js 中的 EventEmitter 是一个核心模块，用于实现事件机制。它提供了一种简单的方式来注册、触发和处理事件。'
description: ''
date: 2023-12-11 10:15:19
---

以下是使用 EventEmitter 实现事件机制的基本步骤：

1. 创建 EventEmitter 实例。

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
```

这个代码片段中，我们使用 require('events') 来引入 EventEmitter 模块，并创建了一个 emitter 实例。

2. 注册事件监听器。

```js
emitter.on(eventName, listener);
```

其中，eventName 表示事件的名称，listener 是一个回调函数，用于处理事件。

例如，我们可以这样注册一个名为 myEvent 的事件监听器：

```js
emitter.on('myEvent', () => {
  console.log('Event occurred.');
});
```

3. 触发事件。

```js
emitter.emit(eventName, [args]);
```

其中，eventName 表示需要触发的事件名称，args 是一个可选的数组，表示传递给事件监听器的参数。

例如，我们可以使用以下代码触发 myEvent 事件：

```js
emitter.emit('myEvent');
```

这个代码片段将触发 myEvent 事件，并执行注册的事件监听器。如果我们在注册事件监听器时使用了参数，可以在调用 emit 方法时传递参数：

```js
emitter.on('myEvent', (arg1, arg2) => {
  console.log(`Event occurred with args: ${arg1}, ${arg2}.`);
});

emitter.emit('myEvent', 'foo', 'bar');
```

这个示例中，我们在注册事件监听器时使用了两个参数 arg1 和 arg2，并在触发事件时传递了这些参数。

完整示例代码：

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();

emitter.on('myEvent', () => {
  console.log('Event occurred.');
});

emitter.emit('myEvent');
```

这个示例将创建一个 emitter 实例，注册一个 myEvent 事件监听器，并在调用 emit 方法时触发该事件。
