---
title: 被迫统计代码量
categories:
  - 默认
excerpt: '记录一次多仓库统计代码量'
description: ''
date: 2023-12-12 10:10:41
---


## 背景

2023-04-13 日公司稽核被重点照顾了，需要统计个人去年每月的代码量...甩了下面这几张图，可惜人事不认同啊。身为一个搬运工一共几十个仓库怎么可能人肉计算出来呢，只能抓紧学习一波了。

![git-log_2021_push_github](/image/statistical-code/git-log_2021_push_github.png)
![git-log_2022_push_github](/image/statistical-code/git-log_2022_push_github.png)

## 一些简单的命令

输出远端仓库地址

```shell
git remote get-url origin
```

输出提交日志

```shell
git log
```

输出某人的提交日志

```shell
git log --author="用户名或者邮箱"
```

通过一定格式输出日志

```shell
git log --since=2023-01-01 --until=2023-04-01 --pretty=tformat: --numstat
```

`--since`：开始时间  
`--until`：结束时间  
`--pretty=tformat:` ：表示以特定格式输出提交历史记录，具体格式可以在命令行中自定义。使用 tformat: 可以消除输出中的换行符，这样就可以方便地将输出转换为 CSV 格式。
`--numstat`：表示以文件的添加、修改和删除的行数的方式输出提交历史记录。大概格式如下：

```log
89      0       docs/blog/stdin.md
332     0       docs/blog/code.md
15      9       docs/blog/fc-cd.md
1       0       s.yaml
```

统计某人代码提交量

```shell
git log --author="用户名或者邮箱" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "新增行数: %s, 删除行数: %s, 总行数: %s\n", add ? add : 0, subs ? subs : 0, loc ? loc : 0 }'
```

统计提交数

```shell
git log --oneline | wc -l
```

## 统计

### 前言

只是做简单的统计，**代码不健壮**。只是为了记录实现思路。

由于项目过多，所以有个习惯就是将自己维护的项目都是放在一个文件里面，所以脚本文件就在这个文件夹的根目录。

里面有个问题，不知为什么运行 `git log | awk ***` 总是失败，所以引入了`shell`文件来解决

### 按照月份划分

**按照月份里面**分为了两个文件：
mouth 是各个月份各个项目代码变动情况
project 是这个一些总揽情况

```js
const { spawnSync } = require('child_process');
const fs = require('fs');
const path = require('path');

const outputMonthFile = path.join(__dirname, 'month.txt');
const outputFile = path.join(__dirname, 'project.txt');
const shellPath = path.join(__dirname, 'git-log.sh');

const times = [
  '2022-01-01',
  '2022-02-01',
  '2022-03-01',
  '2022-04-01',
  '2022-05-01',
  '2022-06-01',
  '2022-07-01',
  '2022-08-01',
  '2022-09-01',
  '2022-10-01',
  '2022-11-01',
  '2022-12-01',
];
const files = fs.readdirSync(__dirname);

times.reduce((since, until) => {
  spawnSync(`echo "${since} ～ ${until}" >> ${outputMonthFile}`, {
    stdio: 'inherit',
    shell: true,
  });
  spawnSync(`echo "执行月份：${since} ～ ${until}" >> ${outputFile}`, {
    stdio: 'inherit',
    shell: true,
  });

  files.forEach((file) => {
    const fullPath = path.join(__dirname, file); //获取文件的完整路径
    const stat = fs.statSync(fullPath); //获取文件的信息

    // 如果是文件夹，就运行脚本
    if (stat.isDirectory()) {
      fs.appendFileSync(outputFile, `项目名称是：${file}\n`);
      spawnSync(shellPath, {
        cwd: path.join(__dirname, file),
        env: {
          ...process.env,
          outputFile,
          outputMonthFile,
          since,
          until,
          file,
        },
        stdio: 'inherit',
        shell: true,
      });
    }
  });

  return until;
});

const result = {};
let month = '';
let allAdd = 0;
let allSubs = 0;
let allLoc = 0;

fs.readFileSync(outputFile, 'utf-8')
  .split('\n')
  .forEach((item) => {
    if (item.startsWith('执行月份')) {
      result[item] = {
        add: 0,
        subs: 0,
        loc: 0,
      };
      month = item;
    } else if (/^\d/.test(item)) {
      const r = item.split(',');
      const [add, subs, loc] = r;
      result[month].add += Number(add);
      result[month].subs += Number(subs);
      result[month].loc += Number(loc);
      allAdd += Number(add);
      allSubs += Number(subs);
      allLoc += Number(loc);
    }
  });

fs.unlinkSync(outputFile);
fs.writeFileSync(
  outputFile,
  `总新增: ${allAdd}
总删除: ${allSubs}
总计: ${allLoc}

细分:
${JSON.stringify(result, null, 2)}`,
);
```

### 按照项目划分

**按照项目划分**里面详细说明了哪个项目、项目的地址、这个项目每个月提交情况

`index.js`

```js
const { spawnSync } = require('child_process');
const fs = require('fs');
const path = require('path');

const outputFile = path.join(__dirname, 'project.txt');
const shellPath = path.join(__dirname, 'git-log.sh');

const times = [
  '2022-01-01',
  '2022-02-01',
  '2022-03-01',
  '2022-04-01',
  '2022-05-01',
  '2022-06-01',
  '2022-07-01',
  '2022-08-01',
  '2022-09-01',
  '2022-10-01',
  '2022-11-01',
  '2022-12-01',
];

const files = fs.readdirSync(__dirname);
files.forEach((file) => {
  const fullPath = path.join(__dirname, file); //获取文件的完整路径
  const stat = fs.statSync(fullPath); //获取文件的信息

  // 如果是文件夹，就运行脚本
  if (stat.isDirectory()) {
    fs.appendFileSync(outputFile, `项目名称是：${file}\n代码仓库地址：`);
    spawnSync(`git remote get-url origin >> ${outputFile}`, {
      cwd: path.join(__dirname, file),
      stdio: 'inherit',
      shell: true,
    });
    times.reduce((since, until) => {
      spawnSync(shellPath, {
        cwd: path.join(__dirname, file),
        env: {
          ...process.env,
          outputFile,
          since,
          until,
        },
        stdio: 'inherit',
        shell: true,
      });
      return until;
    });
    fs.appendFileSync(outputFile, `\n`);
  }
});
```

`git-log.sh`

```shell
#!/bin/bash

echo "$since ～ $until" >> $outputFile
git log --author="wss-git" --since=$since --until=$until --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "新增行数: %s, 删除行数: %s, 总行数: %s\n", add ? add : 0, subs ? subs : 0, loc ? loc : 0 }' >> $outputFile
```
