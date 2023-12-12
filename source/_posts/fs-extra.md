---
title: fs-extra
categories:
  - 轮子应用
excerpt: 'fs-extra 是 Node.js 内置模块 fs 的扩展，提供了更多的文件系统操作方法，并且支持 Promise 和 async/await 语法。它是一个轻量级的模块，可以用于替代 Node.js 原生的 fs 模块。

在 fs-extra 中，所有方法都是异步的，但是仍然可以使用回调函数进行处理。此外，还提供了很多有用的方法，如复制、移动、删除、创建目录、迭代目录等等。

本文将介绍 fs-extra 的常用方法，以及如何使用它们来处理文件和目录。'
description: ''
date: 2023-12-12 09:22:37
---

## 安装

首先，在使用 fs-extra 之前，需要先将其安装到项目中。可以使用 npm 或 yarn 进行安装，示例代码如下：

```sh
# 使用 npm 安装
npm install --save fs-extra

# 使用 yarn 安装
yarn add fs-extra
```

安装完成后，可以在代码中引入该模块：

```js
const fs = require('fs-extra');
```

## 常用方法

### readFile 读取文件

要读取文件的内容，可以使用 fs.readFile() 方法。在 fs-extra 中，该方法可以使用 Promise 和 async/await 语法进行处理。示例代码如下：

```js
// 使用 Promise 处理
fs.readFile('/path/to/file')
  .then((data) => {
    console.log(data.toString());
  })
  .catch((err) => {
    console.error(err);
  });

// 使用 async/await 处理
try {
  const data = await fs.readFile('/path/to/file');
  console.log(data.toString());
} catch (err) {
  console.error(err);
}
```

其中，fs.readFile() 方法将返回一个 Promise 对象，可以使用 .then() 和 .catch() 方法进行处理。使用 async/await 语法时，需要将该方法放在 try...catch 语句中，以处理可能出现的错误。

### writeFile 写入文件

要写入文件，可以使用 fs.writeFile() 方法。在 fs-extra 中，该方法也支持 Promise 和 async/await 语法。示例代码如下：

```js
// 使用 Promise 处理
fs.writeFile('/path/to/file', 'Hello world')
  .then(() => {
    console.log('写入文件成功');
  })
  .catch((err) => {
    console.error(err);
  });

// 使用 async/await 处理
try {
  await fs.writeFile('/path/to/file', 'Hello world');
  console.log('写入文件成功');
} catch (err) {
  console.error(err);
}
```

在上面的示例中，fs.writeFile() 方法的第一个参数是文件路径，第二个参数是要写入的内容。当写入文件成功时，将会返回一个 Promise 对象，可以使用 .then() 方法进行处理。

### copy 复制文件

要复制文件，可以使用 fs.copy() 方法。该方法将源文件复制到目标文件，可以使用 Promise 和 async/await 语法进行处理。示例代码如下：

```js
// 使用 Promise 处理
fs.copy('/path/to/source', '/path/to/destination')
  .then(() => {
    console.log('复制文件成功');
  })
  .catch((err) => {
    console.error(err);
  });

// 使用 async/await 处理
try {
  await fs.copy('/path/to/source', '/path/to/destination');
  console.log('复制文件成功');
} catch (err) {
  console.error(err);
}
```

在上面的示例中，fs.copy() 方法的第一个参数是源文件路径，第二个参数是目标文件路径。当复制文件成功时，将会返回一个 Promise 对象，可以使用 .then() 方法进行处理。

### move 移动文件

要移动文件，可以使用 fs.move() 方法。该方法将源文件移动到目标文件，可以使用 Promise 和 async/await 语法进行处理。示例代码如下：

```js
// 使用 Promise 处理
fs.move('/path/to/source', '/path/to/destination')
  .then(() => {
    console.log('移动文件成功');
  })
  .catch((err) => {
    console.error(err);
  });

// 使用 async/await 处理
try {
  await fs.move('/path/to/source', '/path/to/destination');
  console.log('移动文件成功');
} catch (err) {
  console.error(err);
}
```

在上面的示例中，`fs.move()` 方法的第一个参数是源文件路径，第二个参数是目标文件路径。当移动文件成功时，将会返回一个 Promise 对象，可以使用 `.then()` 方法进行处理。

### remove 删除文件或目录

要删除文件或目录，可以使用 `fs.remove()` 方法。该方法将递归地删除目录及其子目录中的所有文件和目录。可以使用 Promise 和 async/await 语法进行处理。示例代码如下：

```javascript
// 使用 Promise 处理
fs.remove('/path/to/file_or_directory')
  .then(() => {
    console.log('删除文件或目录成功');
  })
  .catch((err) => {
    console.error(err);
  });

// 使用 async/await 处理
try {
  await fs.remove('/path/to/file_or_directory');
  console.log('删除文件或目录成功');
} catch (err) {
  console.error(err);
}
```

在上面的示例中，fs.remove() 方法的参数是要删除的文件或目录的路径。当删除文件或目录成功时，将会返回一个 Promise 对象，可以使用 .then() 方法进行处理。

### ensureDir 创建目录

要创建目录，可以使用 fs.ensureDir() 方法。该方法将递归地创建目录，如果目录已经存在则不会进行任何操作。可以使用 Promise 和 async/await 语法进行处理。示例代码如下：

```javascript
// 使用 Promise 处理
fs.ensureDir('/path/to/directory')
  .then(() => {
    console.log('创建目录成功');
  })
  .catch((err) => {
    console.error(err);
  });

// 使用 async/await 处理
try {
  await fs.ensureDir('/path/to/directory');
  console.log('创建目录成功');
} catch (err) {
  console.error(err);
}
```

在上面的示例中，fs.ensureDir() 方法的参数是要创建的目录的路径。当创建目录成功时，将会返回一个 Promise 对象，可以使用 .then() 方法进行处理。

### 迭代目录

要迭代目录中的所有文件和子目录，可以使用 fs.readdir() 方法。该方法将返回指定目录中所有文件和子目录的名称，可以使用 Promise 和 async/await 语法进行处理。示例代码如下：

```js
// 使用 Promise 处理
fs.readdir('/path/to/directory')
  .then((files) => {
    console.log(files);
  })
  .catch((err) => {
    console.error(err);
  });

// 使用 async/await 处理
try {
  const files = await fs.readdir('/path/to/directory');
  console.log(files);
} catch (err) {
  console.error(err);
}
```

在上面的示例中，fs.readdir() 方法的参数是要迭代的目录的路径。当迭代目录成功时，将会返回一个 Promise 对象，其中包含指定目录中所有文件和子目录的名称。

### 其他方法

emptyDir 清空目录  
mkdirp 递归创建目录  
readJson 读取 JSON 文件  
writeJson 写 JSON 文件

## 判断文件的 755

可以使用 fs-extra 模块的 statSync()方法获取文件的权限信息，然后通过判断文件的权限是否为 755 来判断文件是否为 755，示例代码如下：

```js
const fs = require('fs-extra');

// 获取文件的权限信息
const stats = fs.statSync('/path/to/file');

// 判断文件的权限是否为755
const is755 = (stats.mode & 0o755) === 0o755;
if (!is755) {
  console.log('文件权限不是755');
}
```

## 判断文件可运行

可以使用 fs-extra 模块的 statSync()方法获取文件的权限信息，然后通过判断文件的权限是否包含可执行权限（即可读且可执行权限）来判断文件是否可运行，示例代码如下：

```js
const fs = require('fs-extra');

// 获取文件的权限信息
const stats = fs.statSync('/path/to/file');

// 判断文件是否可运行
const isExecutable = (stats.mode & 0o111) !== 0;
if (isExecutable) {
  console.log('文件可运行');
}
```

其中，0o111 代表可执行权限，0o100 代表可读权限，0o010 代表可写权限。因此，判断文件是否可读可执行可以使用(stats.mode & 0o111) !== 0 的方式。

## 判断文件是不是软链接

可以使用 lstat() 方法来判断文件是否为软链接。lstat() 方法与 stat() 方法类似，都可以用于获取文件的状态信息，但是 lstat() 方法会返回软链接本身的信息，而不是软链接指向的文件的信息。如果文件是软链接，则 lstat() 方法返回的 stats 对象的 isSymbolicLink() 方法会返回 true，示例代码如下：

```js
const fs = require('fs-extra');

try {
  const stats = await fs.lstat('/path/to/file');

  if (stats.isSymbolicLink()) {
    console.log('文件是软链接');
  } else {
    console.log('文件不是软链接');
  }
} catch (err) {
  console.error(err);
}
```

在上面的示例中，fs.lstat() 方法的参数是要获取状态信息的文件路径。当获取状态信息成功时，将会返回一个 stats 对象，可以使用 isSymbolicLink() 方法判断文件是否为软链接。

除了 lstat() 方法，还可以使用 stat() 方法来获取文件的状态信息，但是该方法会返回软链接指向的文件的信息，而不是软链接本身的信息。要判断软链接本身是否存在，需要使用 lstat() 方法。

