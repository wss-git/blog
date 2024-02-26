---
title: 本地端口的服务映射到公网
categories:
  - 默认
excerpt: '本地端口的服务映射到公网: localtunnel'
description: '本地端口的服务映射到公网: localtunnel'
date: 2024-02-26 14:26:20
---

## 开始使用

1. 安装及访问

- 项目内部安装

```bash
npm install -D localtunnel

npx lt --port 3000
```

- 全局安装及访问

```bash
npm install -g localtunnel

lt --port 3000
```

2. 访问生成的地址。此时提示 `To access the website, please enter the tunnel password below.` 这是需要填写你本地的公网IP，可以通过 `https://ip.900cha.com/#google_vignette` 或者  `https://loca.lt/mytunnelpassword` 查询


## 参考

- [lt](https://www.npmjs.com/package/localtunnel)
