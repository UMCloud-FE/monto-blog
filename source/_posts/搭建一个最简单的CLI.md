---
layout: post
title: 搭建一个最简单的CLI
date: 2023-06-30 18:52:07
tags: CLI
---

# 二、搭建一个最简单的CLI

我们先搭建一个最简单的CLI来体验下，然后逐步实现复杂点的功能。

## 1、新建项目

首先新建项目作为CLI源代码地址

```bash
mkdir cli-demo
cd cli-demo
npm init
```
## 2、安装依赖

[yargs文档配置](https://yargs.js.org/docs/#api-reference)：该插件将用户通过终端输入的参数解析成对象，可以配置各种参数。

```
yarn add yargs
```
命令使用语法：
```
.command(cmd, desc, [builder], [handler])
```
## 3、CLI命令配置

下面的配置说明：
- 配置命令`get`
- 声明信息`make a get HTTP request`
- 并配置了参数`url`信息
- 最后callback函数获取用户执行的参数

具体使用：编辑`index.js`，第一行一定要加注释，表明运行在node环境下
```index.js
#!/usr/bin/env node

const yargs = require('yargs/yargs')
const { hideBin } = require('yargs/helpers')

yargs(hideBin(process.argv))
  .command(
    'get',
    'make a get HTTP request',
    function (yargs) {
      return yargs.options({
        url: {
            alias: 'u',
            describe: 'the URL to make an HTTP request to'
        }
      })
    },
    function (argv) {
      console.log("callback", argv.url)
    }
  )
  .help()
  .argv
```

执行命令，可以获取输入的参数

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d59ebf089874208a7218d30aedb5a8a~tplv-k3u1fbpfcp-watermark.image?)

这里get就是定义的指令，url是指令下的key值，用--url输入，alias就是别名，用-u表示，后面跟要输入的参数。

## 3、npm发布

接着我们发布下npm，然后一个CLI就完成了。

登陆npm仓库，没有的话去注册一个,[地址](https://www.npmjs.com/)
```
npm login
```
选择一个中意的cli名字，查一下是否存在，这里我们起个名字`cli-demo3`

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a822f1db2b74eab930ec3ff10584dac~tplv-k3u1fbpfcp-watermark.image?)

执行`npm version patch && npm publish --registry=https://registry.npmjs.org`，不出意外的话，就发布成功了。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95b8a392dccb475b9285279d0f14fef5~tplv-k3u1fbpfcp-watermark.image?)

## 4、下载使用

然后我们本地使用下，首先下载到本地，全局下载`npm install -g cli-demo3`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93366377d58f42c7af9c21dae58a6cf1~tplv-k3u1fbpfcp-watermark.image?)

**cli-demo3是我们前期测试使用写的CLI，后面继续使用我正常部署的CLI：u-amin-cli，原理是一样的**