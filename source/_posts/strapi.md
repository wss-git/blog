---
title: Strapi
categories:
  - 轮子应用
excerpt: 'strapi 是一款能够快速上手，让一个懂一点 Node.js 的前端开发就能够快速的开发出增删改查的接口。'
description: 'Strapi'
date: 2023-12-12 10:20:14
---


## 快速入门指南

strapi 官网：[docs.strapi.io/](https://docs.strapi.io/dev-docs/quick-start#_1-install-strapi-and-create-a-new-project)

### 安装 Node 环境 <Badge>必须</Badge>

建议安装 <a target="_blank" href="https://nodejs.org/download/release/latest-v16.x/">nodejs16</a> 版本，稳定大部分包兼容比较好。

### 运行安装脚本

在终端运行命令:

```shell
yarn create strapi-app my-project --quickstart
```

或者

```shell
npx create-strapi-app@latest my-project --quickstart
```

此处可能会安装依赖异常，可以进入到项目中运行以下命令安装依赖:

```shell
npm config set sharp_binary_host "https://npmmirror.com/mirrors/sharp"
npm config set sharp_libvips_binary_host "https://npmmirror.com/mirrors/sharp-libvips"
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install

npm run develop
```

### 注册第一个管理员用户

安装完成后，您的浏览器会自动打开一个页面。

通过填写表格，您可以创建自己的帐户。完成后，您将成为此 Strapi 应用程序的第一个管理员用户。

### 汉化【可选】

配置路径是: 点击左下角的名称 > Profile > Interface language, 可以看到 strapi 默认语言是英语和法语。

进入到项目目录，创建文件 `src/admin/app.js`，内容如下：

```js
const config = {
  locales: ['zh-Hans'],
};

const bootstrap = (app) => {
  console.log(app);
};

export default {
  config,
  bootstrap,
};
```

重启项目。`ctrl + c`，然后运行 `npm run develop` 或 `yarn develop`。然后再安装上面的配置路径选择中文就好了。

### 使用 Content-type Builder 创建集合类型

Content-type Builder 插件可帮助您创建数据结构。

1. 点击 Content-type Builder
2. 点击 `Create new collection type`
3. 键入名称 `Restaurant`，然后单击 `Continue`
4. 点击`Text`字段，键入`Name`
5. 切换到`Advanced Settings`选项卡，并检查`Required`字段和`Unique`字段设置。
6. 点击保存，Strapi 自动重启

Strapi 重新启动之后，可以在 `Content Manager` > `Collection types` 看到刚刚创建`Restaurant`！

![创建gif图](/image/strapi/strapi_create_collection_type.gif)

### 创建数据

创建`Restaurant`数据：

1. 转到 `Content Manager` > `Collection types`，然后点击 `Restaurant`。
2. 点击 `Create new entry`
3. 填入具体的内容
4. 点击 `Punlish` 发布这条数据

创建一个用户：
和上面步骤类似， `confirmed`选择 `TRUE`、角色选择`Authenticated`，用于调用创建修改和删除接口。

### 设置权限 【重点】

[users-roles-permissions 文档](https://docs.strapi.io/user-docs/users-roles-permissions)

设置公共读：

1. 单击 `Settings`。
2. 在 `Users & Permissions` 插件下，选择 `Roles`
3. 然后选择 `Public` 角色
4. 在选项卡`Permissions`中，找到`Restaurant`并点击
5. 点击选项卡 `find` 和 `findOne`
6. 点击 `Save`

设置管理员用户可以 CRUD：

1. 单击 `Settings`。
2. 在 `Users & Permissions` 插件下，选择 `Roles`
3. 然后选择 `Authenticated` 角色
4. 在选项卡`Permissions`中，找到`Restaurant`并点击选项卡 `Select all`
5. 点击 `Save`

### 访问 API

[REST API](https://docs.strapi.io/dev-docs/api/rest)

```js
const axios = require('axios');

const identifier = process.env.identifier || 'wss'; // 可以是注册时的邮箱或者用户名
const password = process.env.password || 'q`12345';

axios.defaults.baseURL = process.env.base_url || 'http://0.0.0.0:1337';
axios.defaults.headers = {
  accept: 'application/json',
  'Content-Type': 'application/json',
};

async function find() {
  const { data } = await axios.get('/api/restaurants');
  console.log('find: ', JSON.stringify(data, null, 2));
}

async function findOne(id = 1) {
  const { data } = await axios.get(`/api/restaurants/${id}`);
  console.log('findOne: ', JSON.stringify(data, null, 2));
}

async function login() {
  const { data } = await axios.post('/api/auth/local', {
    identifier,
    password,
  });
  console.log('login data: ', JSON.stringify(data, null, 2));

  axios.defaults.headers = {
    ...axios.defaults.headers,
    Authorization: `Bearer ${data.jwt}`,
  };
  return data.jwt;
}

async function CUD() {
  const { data } = await axios.post('/api/restaurants', {
    data: { name: 'xxxx' },
  }); // 这里有一个小坑，需要用 data 包裹一下，而不是直接传递
  console.log('create data: ', JSON.stringify(data, null, 2));
  const createId = data.data.id;
  await findOne(createId); // 能够查询到刚创建的信息
  await axios.put(`/api/restaurants/${createId}`, { data: { name: 'ssssss' } });
  console.log('update data: ');
  await findOne(createId); // 能够查询到刚创建的信息已经被修改
  await axios.delete(`/api/restaurants/${createId}`);
  await find(); // 能够查询到刚创建的信息列表中已经不存在
}

(async function () {
  // 此时还没有登陆，谁都可以调用的
  await find();
  await findOne();
  // 此时没有登陆，调用 cud 会失败
  try {
    await CUD();
  } catch (e) {
    console.log(
      'error status: ',
      e.response.status,
      e.config.url,
      e.config.method,
    );
  }
  // 登陆
  await login();
  try {
    await CUD();
  } catch (e) {
    console.log(
      'error status: ',
      e.response.status,
      e.config.url,
      e.config.method,
    );
  }
})();
```

## 上云：部署到函数计算

:::warning{title=温馨提示}
需要对函数计算以及 s 工具有一定的理解。
:::

[简单示例](https://github.com/wss-git/strapi)

目前存在两个分支：

### layer 分支

是用于制作层的分支，此项目存在不同系统包会出现兼容问题，而且每次部署压缩和上传上百 MB 的文件会特别影响效率，所以专门使用开源的[流水线](https://github.com/serverless-devs/serverless-cd)（优势：跑在函数计算中的流水线，环境完全一致，出现包兼容概率极低）将一些核心依赖包打进了层中实现复用。

目前存在深圳地区的层有一个是可以工作的，使用方式（可以参考主分钟中的 s.yaml 文件）：

新增 layers 配置 `acs:fc:cn-shenzhen:1740298130743624:layers/strapi/versions/1`
环境变量新增 `NODE_PATH: /opt/nodejs/node_modules`、`PATH: /opt/nodejs/node_modules/.bin:/opt/nodejs/node_modules`

### main 分支

是项目主分支，存放项目文件。并且可以直接通过 s deploy 直接部署到函数计算。
注意三个文件：s.yaml、.fcignore、server.js

> 未完成部分: 使用 Nas、通过 serverless-cd 部署
