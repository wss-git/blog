---
title: 自建流水线
categories:
  - 默认
excerpt: '从使用 dumi 创建一个博客延伸到建立简易版流水线'
description: ''
date: 2023-12-12 09:16:14
---

## 流水线开源版本

[开源版本](https://github.com/Serverless-Devs/serverless-cd)、[示例地址](https://app.serverless-cd.cn)

## 背景

一直想要搞一个自己的博客网站，但是懒癌晚期一直拖拉到今天。这个项目前前后后启动了三四次，也尝试过很多其他框架搭建项目，例如使用过 `mkdocs`、`docusaurus`，但是感觉都不如 <a target="_blank" href="https://d.umijs.org/guide">dumi</a> 对前端友好。（最重要的是在稳定的前提下最便宜）

最开始代码是托管在 Github 上面，后来就迁移到了 Gitee 上面（原因懂得都懂）。要实现推送代码自动发布文章，发现他们的流水线只能 200 分钟，这时长还不够测试呢（可能是对 Gitee 了解不深，不知道有没有其他方式跑一些自动化的脚本）。由于自己一直是在 Serverless 领域工作，可能对这块了解比较多，所以就下定决心搞一个简易定制版的流水线。反正不跑 case 又不要钱，放着呗。

言归正传，此篇文章又浅到深大概分以下几部分，可以挑感兴趣的部分阅读。

- [启动一个本地博客](#搭建本地博客)
- [博客上云](#将博客托管到函数计算)
- [自建流水线](#自建流水线)

> 需要的知识储备：Nodejs、函数计算产品

<br />

## 安装 Node 环境 <Badge>必须</Badge>

建议安装 <a target="_blank" href="https://nodejs.org/download/release/latest-v16.x/">nodejs16</a> 版本，稳定大部分包兼容比较好。

<br />

## 搭建本地博客

- 初始化 dumi (使用的是 v2.0 版本)

```shell
npx create-dumi

npm run start
```

- 根据提示访问链接，一般默认是 `http://localhost:8000`，此刻一个最简单的文档在本地就搭建好了。

> 默认主题配置查看链接 <https://d.umijs.org/theme/default>

- 执行 `npm run build` 生成静态文件

<br />

## 将博客托管到函数计算

其实将静态文件部署到云端的方式非常多，也有非常多更优的方式，比如 `OSS + CDN` 的方式。但是没办法，就对这块熟悉。

:::warning{title=温馨提示}
本文使用的是通过 S 工具上传文件
:::

1. 安装 Serverless Dev 工具

```shell
npm install @serverless-devs/s -g
```

2. 配置密钥，参考<a href="https://docs.serverless-devs.com/serverless-devs/quick_start#%E5%AF%86%E9%92%A5%E9%85%8D%E7%BD%AE" target="_blank">文档</a>

3. 编写配置文件。在项目根目录创建 `s.yaml` 文件

```yaml
edition: 1.0.0
name: website
access: default

services:
  website:
    component: fc
    actions:
      pre-deploy:
        - plugin: website-fc # 使用插件，文档地址 http://www.devsapp.cn/details.html?name=website-fc
    props:
      region: cn-hangzhou
      service:
        name: website
        description: 'hello world by serverless devs'
      function:
        name: blog-site
        description: 'hello world by serverless devs'
        runtime: nodejs16
        codeUri: ./dist
        memorySize: 128
        cpu: 0.05
        timeout: 20
        instanceConcurrency: 128
        diskSize: 512
      triggers:
        - name: httpTrigger
          type: http
          config:
            authType: anonymous
            methods:
              - GET
      customDomains:
        - domainName: auto # 会生成一个测试域名
          protocol: HTTP
          routeConfigs:
            - path: /*
```

4. 部署远端资源, 运行 `s deploy` 命令

5. 访问显示以 `.fc.devsapp.net` 结尾的域名就可以看到自己写的内容啦

<br />

## 自建流水线

代码地址: [Github](https://github.com/wss-git/custom-dumi-cd) [Gitee](https://gitee.com/wss-git/custom-dumi-cd)

在此之前先说一下整体的架构图 V1 版本，PS: 干掉了 nas 和投递函数部分，nas 部分放到后面 Strapi 在搞，投递函数就是一些钩子处理异常业务，项目太小用不到。更详细可以看[开源版本](https://github.com/Serverless-Devs/serverless-cd)

![cd 架构图](/image/fc-cd/cd-architecture.png)

### 制作层

因为是定制化，所以这里选择了层的方式加速加载依赖。将输出的 arn 填写到流水线函数的配置中，加上环境变量 `/opt/nodejs/node_modules/.bin:/opt/nodejs/node_modules`。

```json
{
  "dependencies": {
    "@serverless-devs/s": "latest",
    "@commitlint/cli": "^17.1.2",
    "@commitlint/config-conventional": "^17.1.0",
    "dumi": "^2.0.2",
    "husky": "^8.0.1",
    "lint-staged": "^13.0.3",
    "prettier": "^2.7.1"
  }
}
```

#### 通过控制台(推荐)

控制台在创建层或者创建版本时，`层上传方式`选择`在线构建`，填入依赖确定就好了。

![控制台构建层](/image/fc-cd/fc-cd-layer.jpg)

#### 在本地

```
zip -rqy my-layer-code.zip node
s cli fc layer publish --code ./my-layer-code.zip --compatible-runtime nodejs16,nodejs14,custom  --region cn-shenzhen --layer-name dumiAndS
```

### 创建网关函数

这里做了几件事情:

1. 验证 gitee 请求
2. 验证密钥：这里是用的简单的密码请求
3. 验证是否触发到了分支
4. 调用流水线函数

所以需要在部署的时候配置 gitee webhook 的密码和要触发的分支

#### yaml

```yaml
edition: 1.0.0
name: hello-world-app
access: 'default_serverless_devs_access'
services:
  gateway:
    component: fc
    props:
      region: 'cn-shenzhen'
      service:
        name: 'assembly-line'
        description: 'hello world by serverless devs'
        logConfig: auto
      function:
        name: 'gateway'
        description: 'hello world by serverless devs'
        runtime: nodejs16
        codeUri: ./gateway
        handler: index.handler
        memorySize: 128
        timeout: 20
        instanceConcurrency: 128
        environmentVariables:
          WEBHOOK_PASSWORD: ${env.password}
          REF: main
      triggers:
        - name: httpTrigger
          type: http
          config:
            authType: anonymous
            methods:
              - GET
              - POST
              - OPTIONS
```

运行 s gateway deploy 会输出一个 system_url, 保存创建 Webhook 时使用

### 创建流水线函数

主要就是加载代码，然后编译部署。 **注意此处需要修改 layer 的配置**

#### yaml

```yaml
edition: 1.0.0
name: hello-world-app
access: 'default_serverless_devs_access'

vars:
  region: 'cn-shenzhen'
  service:
    name: 'assembly-line'
    description: 'hello world by serverless devs'
    logConfig: auto

services:
  engine:
    component: fc
    actions:
      pre-deploy:
        - run: npm install --registry=https://registry.npmmirror.com
          path: './engine'
      post-deploy:
        - component: fc api UpdateFunction --region ${vars.region} --header
            '{"x-fc-disable-container-reuse":"True"}' --path
            '{"serviceName":"${vars.service.name}","functionName":"engine"}'
    props:
      region: ${vars.region}
      service: ${vars.service}
      function:
        name: 'engine'
        description: 'hello world by serverless devs'
        runtime: nodejs16
        codeUri: ./engine
        handler: index.handler
        memorySize: 640
        timeout: 600
        layers:
          - acs:fc:cn-shenzhen:1740298130743624:layers/s-dumi/versions/1
        environmentVariables:
          PATH: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/var/fc/lang/nodejs16_alinode/bin:/opt/bin:/opt/nodejs/node_modules/.bin:/opt/nodejs/node_modules
          TOKEN: ${env.token}
        asyncConfiguration:
          maxAsyncRetryAttempts: 1
          statefulInvocation: true
```

运行 s engine deploy。

### 创建 Webhook

> 需要简单了解一下 gitee 的 [WebHook 推送数据格式说明](https://gitee.com/help/articles/4186#article-header0)、[WebHook 密钥验证和验证算法](https://gitee.com/help/articles/4290)

在 gitee 仓库添加 Webhook 时，`URL` 填写`gateway`返回的 `system_url`，`WebHook 密码/签名密钥`配置`gateway` 的密码。

