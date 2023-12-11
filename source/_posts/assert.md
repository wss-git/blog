---
title: assert
categories:
  - 笔记
excerpt: '在 Node.js 中，断言是一种用于检查代码实现的正确性的机制。Node.js 自带了一个断言模块 assert，可以用来实现断言。'
description: ''
date: 2023-12-11 09:27:54
---

在 Node.js 中，断言是一种用于检查代码实现的正确性的机制。Node.js 自带了一个断言模块 assert，可以用来实现断言。

使用断言需要先引入 assert 模块：

```js
const assert = require('assert');
```

然后可以使用 assert 模块的不同方法来进行断言。

常用的方法有：

- assert.equal(actual, expected[, message]) - 判断实际值 actual 是否等于期望值 expected，如果不等于，则抛出一个错误。如果指定了 message 参数，则在抛出错误时将其作为错误信息。
- assert.strictEqual(actual, expected[, message]) - 判断实际值 actual 是否严格等于期望值 expected（即类型和值都要相等），如果不等于，则抛出一个错误。
- assert.deepEqual(actual, expected[, message]) - 判断实际值 actual 是否深度等于期望值 expected，即检查对象和数组的内容是否相等。如果不相等，则抛出一个错误。
- assert.ok(value[, message]) - 判断给定的值 value 是否为真值（即不是 false、0、null、undefined、NaN），如果不是，则抛出一个错误。
  以下是一些示例：

```js
assert.equal(2 + 2, 4, '2 + 2 应该等于 4');
assert.strictEqual(2 + 2, '4', '2 + 2 不应该等于字符串 "4"');
assert.deepEqual([1, 2], [1, 2], '两个数组内容应该相等');
assert.ok(true, 'true 应该是真值');
```

## 完整示例

```js
const assert = require('assert');

// 判断函数 add 是否正确实现
function add(a, b) {
  return a + b;
}

assert.strictEqual(add(2, 4), 6, '2 + 4 应该等于 6');
assert.strictEqual(add(-2, 3), 1, '-2 + 3 应该等于 1');

// 判断对象是否相等
const obj1 = { a: 1, b: 2 };
const obj2 = { a: 1, b: 2 };
const obj3 = { a: 1, b: 3 };

assert.deepEqual(obj1, obj2, '两个对象内容应该相等');
assert.notDeepEqual(obj1, obj3, '两个对象内容不应该相等');

// 判断数组是否相等
const arr1 = [1, 2, 3];
const arr2 = [1, 2, 3];
const arr3 = [2, 3, 4];

assert.deepEqual(arr1, arr2, '两个数组内容应该相等');
assert.notDeepEqual(arr1, arr3, '两个数组内容不应该相等');

// 判断布尔值是否为真值
assert.ok(true, 'true 应该是真值');
assert.ok(1, '1 应该是真值');
assert.ok('hello world', '"hello world" 应该是真值');
assert.ok([], '[] 应该是真值');
assert.ok({}, '{} 应该是真值');

// 判断布尔值是否为假值
assert.ok(!false, 'false 应该是假值');
assert.ok(!0, '0 应该是假值');
assert.ok(!'', '"" 应该是假值');
assert.ok(!null, 'null 应该是假值');
assert.ok(!undefined, 'undefined 应该是假值');
assert.ok(isNaN(NaN), 'NaN 应该是假值');
```
