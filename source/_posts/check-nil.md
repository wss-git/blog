---
title: 空值验证
categories:
  - 笔记
excerpt: '个人在项目中常使用的一些空值验证'
description: ''
date: 2023-12-11 09:39:19
---


个人在项目中常使用的一些空值验证：

隐式转化：

```js
if (value) {
  // 此时变量 value 转化为 boolean:
  // Boolean(null) // false
  // Boolean(undefined) // false
  // Boolean('') // flase
  // Boolean(NaN) // flase
  // Boolean(0) // flase
  // 其他的均为 ture，需要注意的是下面这几个是转化为 true
  // Boolean([]) // true
  // Boolean({}) // true
  // Boolean(Infinity) // true
}
```

loadsh.isNil(value)：

> 如果 value 为 null 或 undefined，那么返回 true，否则返回 false

loadsh.isEmpty(value)：

> 检查 value 是否为一个空对象，集合，映射或者 set。 判断的依据是除非是有枚举属性的对象，length 大于 0 的 arguments object, array, string 或类 jquery 选择器。
> 对象如果被认为为空，那么他们没有自己的可枚举属性的对象。
> 类数组值，比如 arguments 对象，array，buffer，string 或者类 jQuery 集合的 length 为 0，被认为是空。类似的，map（映射）和 set 的 size 为 0，被认为是空。

```js
const _ = require('lodash');

_.isEmpty(null);
// => true

_.isEmpty(undefined);
// => true

// boolean
_.isEmpty(true);
// => true
_.isEmpty(false);
// => true

// number
_.isEmpty(1);
_.isEmpty(0);
// => true
// => true

_.isEmpty([]);
// => true
_.isEmpty([1, 2, 3]);
// => false

_.isEmpty({});
// => true
_.isEmpty({ a: 1 });
// => false
```
