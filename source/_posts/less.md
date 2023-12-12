---
title: less
categories:
  - 笔记
excerpt: 'less 简单语法'
description: ''
date: 2023-12-12 09:31:45
---

## 定义变量

```less
@color: #fff;
span {
  color: @color;
}
```

编译成 CSS 代码：

```css
span {
  color: #fff;
}
```

## 循环出内容

```less
.loop(@i) when (@i > 0) {
  .loop((@i - 1)); // 递归调用自身
  padding: (10px + 5 * @i);
}

.call {
  .loop(4); // 调用循环
}
```

编译成 CSS 代码：

```css
.call {
  padding: 15px;
  padding: 20px;
  padding: 25px;
  padding: 30px;
}
```

## 循环出类名

```less
.loop(@counter) when (@counter > 0) {
  .p-@{counter} {
    padding: (1px * @counter);
  }
  .loop((@counter - 1));
}

.loop(4);
```

```css
.p-1 {
  padding: 1px;
}
.p-2 {
  padding: 2px;
}
.p-3 {
  padding: 3px;
}
.p-4 {
  padding: 4px;
}
```

