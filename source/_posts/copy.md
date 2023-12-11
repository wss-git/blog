---
title: 前端实现复制
categories:
  - 笔记
excerpt: '前端实现复制效果'
description: '前端实现复制效果'
date: 2023-12-11 10:12:38
---


```javascript
// 创建假元素
const createFakeElement = (value) => {
  const fakeElement = document.createElement('textarea');

  fakeElement.style.border = '0';
  fakeElement.style.padding = '0';
  fakeElement.style.margin = '0';

  fakeElement.style.position = 'absolute';
  fakeElement.style.left = '-999px';
  fakeElement.style.top = `${
    window.pageYOffset || document.documentElement.scrollTop
  }px`;
  fakeElement.setAttribute('readonly', '');
  fakeElement.value = value;
  return fakeElement;
};

const copyText = (value) => {
  const element = createFakeElement(value);
  document.body.appendChild(element);

  // 选中元素
  element.focus();
  element.select();
  element.setSelectionRange(0, element.value.length);

  document.execCommand('copy');
  document.body.removeChild(element);
  Message.success('Copy 成功');
};
```

