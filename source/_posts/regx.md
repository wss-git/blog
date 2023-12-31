---
title: 常用正则
categories:
  - 笔记
excerpt: '常用正则'
description: '常用正则'
date: 2023-12-12 10:00:38
---

## 金钱最多两位小数

```js

/(^[1-9]([0-9]+)?(\.[0-9]{1,2})?$)|(^(0){1}$)|(^[0-9]\.[0-9]([0-9])?$)/
/^[0-9]\d{0,9}(\.\d{1,2})?$/
```

## 移动端输不上

```js
this.value = val.replace(/[^\d.]/g, ''); //清除"数字"和"."以外的字符
this.value = this.value.replace(/^\./g, ''); //验证第一个字符是数字而不是
this.value = this.value.replace(/\.{2,}/g, '.'); //只保留第一个. 清除多余的
this.value = this.value
  .replace('.', '$#$')
  .replace(/\./g, '')
  .replace('$#$', '.');
this.value = this.value.replace(/^(\-)*(\d+)\.(\d\d).*$/, '$1$2.$3'); //只能输入两个小数
```

## 验证码 6 位

```js
/^[a-zA-Z0-9]{1,6}$/;
```

## 输入汉字和字母

```js
/^([A-Za-z]|[\u4E00-\u9FA5]|[\·]){2,32}$/;
```

## 身份证

```js
/^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/;
```

## 任意字符

```js
/^.{4,60}$/;
```

## 表情符号

```js
/[\ud800-\udbff][\udc00-\udfff]/g;
```

