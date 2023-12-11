## 项目简介

本项目使用 hexo 的 [baozi](https://zzyxka.github.io/) 主题。对其进行了简单修改，比如引入了 [beaudar](https://beaudar.lipk.org/) 评论系统。

可以使用 npm run new [pageName] 快速创建文章，新建的文件目录在 source/_posts 下。强烈建议 categories和excerpt 字段不要留空。

## 项目启动

```bash
npm install

npm run server
```

## 项目打包

```bash
npm run build
```

## 项目发布

部署使用了阿里云的函数计算，[工具使用文档](https://docs.serverless-devs.com/fc3/readme)。

需要做一些前置动作，比如

1. 阿里云密钥配置: .env.example 文件重命名为 .env，然后将阿里云密钥配置进去即可
2. 函数别名和别名触发器初始化流程需要关注一下，需要 创建函数->发布版本->创建别名->创建别名触发器。这个步骤并没有配置在 yaml 中，手动做好的。
3. 域名的名称需要修改成自己的。PS：测试时 domainName 可以使用 auto

```bash
npm run deploy
npm run pub
```