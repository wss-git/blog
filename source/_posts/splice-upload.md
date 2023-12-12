---
title: 切片上传和下载
categories:
  - 默认
excerpt: ''
description: ''
date: 2023-12-12 10:06:38
---


## 需求背景

将本地的文件或者文件夹上传到服务器。文件小则只有一个单文件，大则能达到上百万或者千万个文件，压缩之后能到达几百 M。

服务器是部署在函数计算上面，有请求体最大`4M`限制，还有一些费用问题。

## 解决方案

想到了两种方案。

方案一：遍历目录，将一个个文件和文件夹发送到服务器；服务器创建文件或者文件夹。

> 一些 case cover 不住，这里就不细说了。

方案二：如果是单文件，则直接上传；如果是文件夹，则压缩成 zip 上传，远端解压。下面再细说整体实现。

## 实现思路

![实现思路](/blog/splice-upload.jpg)

## 代码实现

### Client

```ts
import fse from 'fs-extra';
import axios from 'axios';
import path from 'path';
import md5File from 'md5-file';
import zip from '@serverless-devs/zip';
import async from 'async';

interface IFlat {
  start: number;
  size: number;
}

class Upload {
  private isDirectory: boolean;

  constructor(private localPath: string, private actualRemotePath: string) {
    const localFileStat = this.getFileStat(localPath);
    if (!localFileStat) {
      throw new Error('File not found');
    }

    this.isDirectory = localFileStat.isDirectory();
  }

  async upload() {
    // TODO: check remote path

    if (this.isDirectory) {
      await this.uploadDir();
    } else {
      await this.uploadFile(this.localPath, this.actualRemotePath);
    }
  }

  private async uploadDir() {
    const { outputFile } = await zip({
      codeUri: this.localPath,
    });

    // TODO: check remote path

    const tmpPath = path.join(this.actualRemotePath, '.demo.zip');

    await this.uploadFile(outputFile, tmpPath);

    await this.request('/unzip', {
      actualRemotePath: this.actualRemotePath,
      tmpPath,
    });
  }

  private async uploadFile(localResolvedSrc: string, actualRemotePath: string) {
    const { size, mode } = fse.lstatSync(localResolvedSrc);

    // 文件切片
    const fileOffSetCutByChunkSize = this.splitRangeBySize(0, size);
    // 创建文件
    await this.request('/createFile', {
      size,
      actualRemotePath,
    });
    // 上传文件
    await this.uploadFileByChunk(
      actualRemotePath,
      localResolvedSrc,
      fileOffSetCutByChunkSize,
    );
    // 修改文件权限并验证文件
    const fileHash = await md5File(localResolvedSrc);
    await this.request('/file/check', {
      filePath: actualRemotePath,
      fileHash,
      mode,
    });
  }

  /**
   * 按块上传文件
   */
  private uploadFileByChunk(
    remoteFile: string,
    localFile: string,
    fileOffSet: IFlat[],
  ) {
    return new Promise((resolve) => {
      const uploadQueue = async.queue(async (offSet, callback) => {
        try {
          const { start, size } = offSet;

          // 读取文件内容
          const fd = fse.openSync(localFile, 'r');
          const body = Buffer.alloc(size);
          const { bytesRead } = await fse.read(fd, body, 0, size, start);
          if (bytesRead !== size) {
            throw new Error(
              'ReadChunkFile function bytesRead not equal read size',
            );
          }
          await fse.close(fd);

          const res = await this.request(
            `/chunk/upload?filePath=${remoteFile}&fileStart=${start}`,
            body,
          );
          if (res.data.error) {
            throw new Error(res.data.error);
          }
        } catch (error) {
          // TODO：RETRY
          return;
        }
        callback();
      }, 5);

      uploadQueue.drain(() => resolve(''));
      uploadQueue.push(fileOffSet);
    });
  }

  /**
   * 获取文件属性
   * @param dirPath
   * @returns
   */
  private getFileStat(dirPath: string) {
    try {
      return fse.statSync(dirPath);
    } catch (ex) {
      console.debug(`getFileStat ${dirPath}: ${ex.code}, ${ex.message}`);
      return false;
    }
  }

  /**
   * 文件切片
   * @param start
   * @param end
   * @returns
   */
  private splitRangeBySize(start: number, end: number): IFlat[] {
    // 默认是 4M
    const chunkSize =
      parseInt(process.env.NAS_CHUNK_SIZE || '4', 10) * 1024 * 1024;

    const result: IFlat[] = [];
    while (start < end) {
      const size = Math.min(chunkSize, end - start);
      result.push({ start, size });
      start += size;
    }
    return result;
  }

  /**
   * 请求接口
   */
  private async request(p: string, body?: Record<string, any>) {
    const baseUrl = '';
    return await axios.post(`${baseUrl}${p}`, body);
  }
}
```

### Server

```ts
import express from 'express';
import fse from 'fs-extra';
import bodyParser from 'body-parser';
import md5File from 'md5-file';
import { exec } from 'child_process';

const app = express();
app.use(bodyParser.raw({ limit: '6mb' }));
app.use(bodyParser.json());

app.post('/createFile', async (req, res) => {
  console.log(
    `received commands request, query is: ${JSON.stringify(req.body)}`,
  );

  const { actualRemotePath, size } = req.body;
  if (!actualRemotePath) {
    throw new Error('missing actualRemotePath parameter');
  }
  if (!size) {
    throw new Error('missing size parameter');
  }

  try {
    fse.removeSync(actualRemotePath);
  } catch (_ex) {}

  const createFileCmd = `dd if=/dev/zero of=${actualRemotePath} count=0 bs=1 seek=${size}`;

  const execRs = await execute(createFileCmd);

  res.send(execRs);
});

// check if local file and NAS file are the same via MD5
app.post('/file/check', async (req, res) => {
  console.log(
    `received file/check request, body is: ${JSON.stringify(req.body)}`,
  );

  const { filePath, fileHash, mode } = req.body;

  if (isFile(filePath)) {
    throw new Error(
      `get file hash error, target is not a file, target path is: ${filePath}`,
    );
  }

  await execute(`chmod 0${(mode & parseInt('777', 8)).toString(8)} filePath`);

  const remoteFileHash = await md5File(filePath);

  if (remoteFileHash === fileHash) {
    res.send({
      stat: 1,
      desc: 'File saved',
    });
  } else {
    throw new Error('file hash changes, you need to re-upload');
  }
});

app.post('/chunk/upload', async (req, res) => {
  console.log(
    `received file/chunkUpload reqeust, query is: ${JSON.stringify(req.query)}`,
  );

  const { filePath } = req.query;
  const fileStart = parseInt(req.query.fileStart, 10);

  const chunkFileBuf = req.body;

  await writeBufToFile(filePath, chunkFileBuf, fileStart);

  res.send({
    desc: 'chunk file write done',
  });
});

app.post('/unzip', async (req, res) => {
  console.log(
    `received commands request, query is: ${JSON.stringify(req.body)}`,
  );

  const { actualRemotePath, tmpPath } = req.body;
  if (!actualRemotePath) {
    throw new Error('missing actualRemotePath parameter');
  }
  if (!tmpPath) {
    throw new Error('missing tmpPath parameter');
  }

  await execute(`unzip ${tmpPath} -d ${actualRemotePath}`);
  res.send('');
});

app
  .listen(9000, () => {
    console.log('start success.');
  })
  .on('error', (e) => {
    console.error(e.code, e.message);
  });

// exec commands
function execute(command: string) {
  return new Promise((resolve, reject) => {
    exec(
      command,
      {
        encoding: 'utf8',
        timeout: 0,
        maxBuffer: 1024 * 1024 * 1024,
        killSignal: 'SIGTERM',
      },
      (error, stdout, stderr) => {
        if (error) {
          reject(error);
          return;
        }
        resolve({
          stdout,
          stderr,
        });
      },
    );
  });
}

function writeBufToFile(dstPath: string, buf, start: number) {
  return new Promise((resolve, reject) => {
    const ws = fse.createWriteStream(dstPath, { start, flags: 'r+' });
    ws.write(buf);
    ws.end();
    ws.on('finish', () => {
      console.log(`${dstPath} wirte done`);
      resolve('');
    });
    ws.on('error', (error) => {
      console.log(`${dstPath} write error : ${error}`);
      reject(error);
    });
  });
}

function isFile(filePath: string): boolean {
  const stats = fse.lstatSync(filePath);
  return stats.isFile();
}
```

