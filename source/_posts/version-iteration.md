---
title: 记一次版本迭代引发的感想记一次版本迭代引发的感想
categories:
  - 默认
excerpt: ''
description: ''
date: 2023-12-12 10:33:10
---

> 所有仅代表个人观点

## 前言

先简单说一下项目组成：

yaml: 做配置管理和编排能力，已经使用的业务逻辑；完全是用户编写
组件/插件: 具体的业务处理能力；所有人都可以发布那种，类似 npm 包
cli: 解析 yaml，调用组件；项目核心，我们团队自己维护

此次做了 **yaml 规范的整体升级**，相对应 cli 也需要做升级

## 调研

维护的产品做大版本升级，很多包袱是否还需要背负呢？已经很明确这些交互或者能力很鸡肋，是否需要将这些冗余代码继续携带呢？

个人认为有些包袱是需要背负的，但是有些包袱没有不必要背负就放弃呗。最关键的是将变更内容怎样快速的让用户了解到，怎样能够快速的适应到。

所以简单看了一些产品别人怎么做迭代的：

### [Nodejs](https://nodejs.org/dist/latest-v18.x/docs/api/documentation.html)

文档 💯，在文档中将所有的能力进行分类标注。对多个版本同时维护，分别列出[迭代时间](https://github.com/nodejs/release#release-schedule)

```md
## Stability index

Throughout the documentation are indications of a section's stability. Some APIs are so proven and so relied upon that they are unlikely to ever change at all. Others are brand new and experimental, or known to be hazardous.

The stability indices are as follows:

> Stability: 0 - Deprecated. The feature may emit warnings. Backward compatibility is not guaranteed.
> Stability: 1 - Experimental. The feature is not subject to semantic versioning rules. Non-backward compatible changes or removal may occur in any future release. Use of the feature is not recommended in production environments.
> Stability: 2 - Stable. Compatibility with the npm ecosystem is a high priority.
> Stability: 3 - Legacy. Although this feature is unlikely to be removed and is still covered by semantic versioning guarantees, it is no longer actively maintained, and other alternatives are available.

Features are marked as legacy rather than being deprecated if their use does no harm, and they are widely relied upon within the npm ecosystem. Bugs found in legacy features are unlikely to be fixed.

Use caution when making use of Experimental features, particularly within modules. Users may not be aware that experimental features are being used. Bugs or behavior changes may surprise users when Experimental API modifications occur. To avoid surprises, use of an Experimental feature may need a command-line flag. Experimental features may also emit a warning.
```

### [Github API](https://docs.github.com/en/rest/overview/api-versions?apiVersion=2022-11-28#specifying-an-api-version)

```md
## 关于 API 版本控制

The GitHub REST API is versioned. The API version name is based on the date when the API version was released. For example, the API version 2022-11-28 was released on Mon, 28 Nov 2022.

Any breaking changes will be released in a new API version. Breaking changes are changes that can potentially break an integration. Breaking changes include:

- removing an entire operation
- removing or renaming a parameter
- removing or renaming a response field
- adding a new required parameter
- making a previously optional parameter required
- changing the type of a parameter or response field
- removing enum values
- adding a new validation rule to an existing parameter
- changing authentication or authorization requirements

Any additive (non-breaking) changes will be available in all supported API versions. Additive changes are changes that should not break an integration. Additive changes include:

- adding an operation
- adding an optional parameter
- adding an optional request header
- adding a response field
- adding a response header
- adding enum values

When a new REST API version is released, the previous API version will be supported for at least 24 more months following the release of the new API version.

## Specifying an API version

You should use the X-GitHub-Api-Version header to specify an API version. For example:

> curl --header "X-GitHub-Api-Version:2022-11-28" https://api.github.com/zen
> Requests without the X-GitHub-Api-Version header will default to use the 2022-11-28 version.

If you specify an API version that is no longer supported, you will receive a 400 error.
```

### [chalk](https://www.npmjs.com/package/chalk)

chalk 周下载量（2023-07-28 ～ 2023-08-03：256,191,254）8-9 位数的包；4.x.x 升级 [5.x.x](https://github.com/chalk/chalk/tree/v5.0.0) 也没考虑兼容问题。（想象也是芬芳一片）

## Python 版本的迭代

额... 这个就不用太多少说了，局外人都感觉不可思议。

....

## 总结

首先想到我们是做的 cli，所以先拿同类型对比：
Nodejs（npm）：保留一定的兼容性，然后多版本同时维护，并给出迭代时间。（文档做得好，就是香）  
Python（pip）：几乎不考虑兼容性，相信很多人本地存在 pip 和 pip3。

不同维度的对比：
chalk： 所作为 npm 包但也有参考意义，依赖包有一个特点是锁版本。如果是老用户，如果不主动升级那就不回有问题；如果新用户，那就按我新规范来就好了，不满足手动降级呗（我们项目目前还是 cjs，所以都是指定版本降级安装）。  
Github Api：作为 Api，他们的做法是通过请求头来锁定版本号，然后根据不同的头做相应处理。

## 思考

1. 如果不兼容的话那些用户最受影响。

安装在本地用户，他如果不手动升级 cli，那么就不回出现兼容问题，所以几乎不需要考虑。  
在 CI/CD 中，如果用户将 cli 的版本锁定，也不会出现兼容问题。  
重灾区其实在流水线中，直接安装最新版本的，几乎必死。

2. 怎么在兼容和迭代中取舍呢？

I. 再思考 1 的基础上再细化一下，流水线中用户会用到什么能力？  
做一些配置，运行组件/插件（可以理解为定制包）逻辑。那么是不是仅需要兼容这部分的逻辑好了呢？
II. 引导用户在流水线中锁定版本。

3. 我们的产品需要做那么多兼容吗？

结合思考 1、2，在解析 yaml 做一个分流，2.x 加载之前的业务，3.x 加载新业务。其他的边缘问题不考虑兼容，理论上 90% 甚至都会兼容到。

事实上，听的最多的一句话，不想影响老用户，兼容一下。按作者原话：case 太多了，cover 不住了。

