---
title: archiver
categories:
  - 轮子应用
excerpt: 'archiver 是一个用于创建归档文件（如 ZIP 和 TAR）的 Node.js 模块。它提供了一组简单的 API，使开发人员可以轻松地将文件和目录打包成归档文件，并可以选择多种压缩算法。'
description: 'archiver 压缩简单使用'
date: 2023-12-11 09:25:23
---

## 简介

archiver 是一个用于创建归档文件（如 ZIP 和 TAR）的 Node.js 模块。它提供了一组简单的 API，使开发人员可以轻松地将文件和目录打包成归档文件，并可以选择多种压缩算法。

## 安装

在使用 archiver 之前，需要先通过 npm 安装它：

```sh
npm install archiver
```

## 使用方法

以下是一个使用 archiver 创建 ZIP 压缩包的示例：

```js
const archiver = require('archiver');
const fs = require('fs');

// 创建输出流
const output = fs.createWriteStream('example.zip');

// 创建 archiver 实例
const archive = archiver('zip', {
  zlib: { level: 9 }, // 设置压缩级别
});

// 将文件添加到归档文件
archive.file('file1.txt', { name: 'file1.txt' });
archive.file('file2.txt', { name: 'file2.txt' });

// 将目录添加到归档文件
archive.directory('dir1', 'dir1');

// 完成归档文件的创建
archive.finalize();

// 将归档文件输出到磁盘
archive.pipe(output);
```

在上述示例中，我们首先创建了一个输出流 output，用于将归档文件输出到磁盘。然后，我们创建了一个 archiver 实例，并指定要创建的归档文件类型为 ZIP 格式。我们还通过 zlib 选项设置了压缩级别，以控制归档文件的大小和压缩速度。

接下来，我们添加了两个文件 file1.txt 和 file2.txt，以及一个目录 dir1 到归档文件中。对于文件，我们调用 archive.file() 方法，并指定文件名和在归档文件中的路径名。对于目录，我们调用 archive.directory() 方法，并指定要添加的目录和在归档文件中的路径名。

最后，我们调用 archive.finalize() 方法来完成归档文件的创建。完成后，我们将归档文件通过 pipe() 方法输出到磁盘。

## 压缩算法

archiver 支持多种压缩算法，包括 store、deflate、deflateRaw、bzip2 和 lzma。可以通过在创建 archiver 实例时指定 gzip、zlib 或 bzip2 属性来选择不同的压缩算法，例如：

```js
// 使用 gzip 压缩算法
const archive1 = archiver('tar', { gzip: true });

// 使用 zlib 压缩算法
const archive2 = archiver('zip', { zlib: { level: 9 } });

// 使用 bzip2 压缩算法
const archive3 = archiver('tar', { bzip2: true });
```

在上述示例中，我们分别使用 gzip、zlib 和 bzip2 压缩算法来创建归档文件。

## API 文档

archiver 提供了以下常用的 API：

```text
archiver(format, options)：创建一个 archiver 实例。format 参数指定要创建的归档文件格式（如 zip、tar 等），options 参数是可选的配置对象，用于指定压缩算法、压缩级别等选项。
archive.directory(dirpath, destpath)：将目录添加到归档文件中。dirpath 参数指定要添加的目录路径，destpath 参数指定在归档文件中的路径。
archive.file(filepath, options)：将文件添加到归档文件中。filepath 参数指定要添加的文件路径，options 参数是可选的配置对象，用于指定在归档文件中的路径、文件名等选项。
archive.glob(pattern, options)：将通配符匹配的文件添加到归档文件中。pattern 参数是用于匹配文件的通配符模式，options 参数是可选的配置对象，用于指定在归档文件中的路径、文件名等选项。

archive.append(source, options)：将源数据添加到归档文件中。source 参数可以是字符串、Buffer、流等数据类型，options 参数是可选的配置对象，用于指定在归档文件中的路径、文件名等选项。
archive.pipe(destination)：将归档文件输出到目标流。destination 参数指定目标输出流，可以是文件流、HTTP 响应等。
archive.finalize()：完成归档文件的创建。在所有数据添加到归档文件后，调用该方法以生成最终的归档文件。
除了上述常用 API 之外，archiver 还提供了一些其他的 API，如 archive.pointer()、archive.on() 等。可以参考官方文档获取更多信息。
```

## 常见问题解答

### 如何压缩文件时排除某些文件或目录？

可以使用 archive.glob() 方法的 ignore 选项或者 archive.file() 和 archive.directory() 方法的 exlude 选项来排除某些文件或目录。例如：

```js
// 排除所有以 .git 结尾的文件和目录
archive.glob('**', { ignore: '**/.git' });

// 排除指定的文件或目录
archive.file('file.txt', { name: 'file.txt', exclude: true });
archive.directory('dir', 'dir', { exclude: 'subdir' });
```

在上述示例中，我们分别使用了 ignore 和 exclude 选项来排除某些文件或目录。

### 如何设置归档文件的压缩级别？

可以在创建 archiver 实例时通过 zlib.level 或 gzip.level 属性设置压缩级别。例如：

```js
// 使用 zlib 压缩算法
const archive1 = archiver('zip', { zlib: { level: 9 } });

// 使用 gzip 压缩算法
const archive2 = archiver('tar', { gzip: { level: 9 } });
```

在上述示例中，我们分别使用了 zlib.level 和 gzip.level 属性来设置压缩级别。

### 如何设置归档文件的日期时间？

可以在创建 archiver 实例时通过 date 属性设置归档文件的日期时间。例如：

```js
const date = new Date('2021-01-01T00:00:00.000Z');
const archive = archiver('zip', { date });
```

在上述示例中，我们通过 new Date() 方法创建一个日期时间对象，并将其传给了 archiver 实例的 date 属性，来指定归档文件的日期时间。

###

在使用 archiver 压缩代码包时，可以通过设置 directoryPrefix 属性来指定要添加到压缩包内部的目录前缀。例如，以下示例代码将 ./src 目录下的所有文件和子目录添加到 ZIP 压缩包，并在每个文件和目录路径前添加 prefix 前缀：

```js
const archiver = require('archiver');
const fs = require('fs');

const output = fs.createWriteStream('output.zip');
const archive = archiver('zip');

archive.directory('./src', false, { prefix: 'prefix' });
zipArchiver.file('dir', {
  prefix: 'prefix',
  name: 'index.js',
});
archive.pipe(output);
archive.finalize();
```

在上面的示例中，directoryPrefix 设置为 'prefix'，因此压缩包内文件和目录的路径将以 'prefix' 开头。例如，位于 ./src/foo/bar.txt 的文件在压缩包内的路径将变为 prefix/foo/bar.txt。

如果您要添加多个目录到同一个压缩包内，并且需要为每个目录指定不同的前缀，可以通过多次调用 archive.directory() 方法并分别设置不同的 directoryPrefix 来实现。

## 总结

archiver 是一个方便易用的 Node.js 归档文件创建模块，提供了多种压缩算法和 API，可以满足各种归档文件创建需求。在使用 archiver 时，需要注意指定要创建的归档文件类型、压缩级别等选项，以及如何添加文件、目录和其他数据到归档文件中。

