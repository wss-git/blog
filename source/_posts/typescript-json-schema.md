---
title: 将 ts 声明转成 schema
categories:
  - 轮子应用
excerpt: '将 ts 声明转成 schema: typescript-json-schema'
description: ''
date: 2023-12-12 10:25:49
---


## 安装依赖

```bash
npm install typescript-json-schema@0.58.1
```

声明文件 index.ts

```typescript
enum Runtime {
  'nodejs10' = 'nodejs10',
  'nodejs12' = 'nodejs12',
  'nodejs14' = 'nodejs14',
  'nodejs16' = 'nodejs16',
}

export interface IFunction {
  functionName: string;
  runtime: `${Runtime}`;
  handler?: string;
  codeUri?:
    | string
    | {
        src?: string;
        ossBucketName?: string;
        ossObjectName?: string;
      };
  description?: string;
  diskSize?: 512 | 10240;
}
```

## 编程写法

```typescript
import { generateSchema, getProgramFromFiles } from 'typescript-json-schema';
import path from 'path';

const program = getProgramFromFiles([path.join(__dirname, 'index.ts')]);
const schema = generateSchema(program, 'IFunction');
console.log(schema);
```

## 使用命令

```shell
npx typescript-json-schema ./index.ts IFunction --required -o ./dist/schema.json
```
