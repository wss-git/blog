---
title: husky
categories:
  - 轮子应用
excerpt: '项目中使用 git hooks：husky'
description: ''
date: 2023-12-12 09:25:33
---

## git 常用钩子

- pre-commit
- commit-msg

## 使用 husky@^8.0.1

1. install

```
npm install husky --save-dev
```

2. 启用 husky

```
npx husky install
npm set-script prepare "husky install"
```

3. 添加一个钩子

```
npx husky add .husky/pre-commit 'npm run xxx'
```

4. 更新钩子

修改 `.husky/pre-commit` 文件内容

```
# 全局检测文件文档中写的是这个位置
if [ -f ~/.git-hooksPath/hooks/pre-commit ]; then
  ~/.git-hooksPath/hooks/pre-commit
fi
```
