---
layout: post
title: CLI模块化开发方案
date: 2023-08-13 11:04:53
tags: CLI
categories:
  - CLI
---

## 一、前言

我们知道js的模块化是一步步发展的，js在浏览器端由于网络请求等原因往往采取异步加载方案

在服务器端由于js位于本地磁盘，加载速度更快，通常使用commonjs加载方案。

Node版本大于12后，js支持的模块化方案引入了ESM。

由于之前历史原因，很多低版本的nodejs库开发都是用的是Commonjs，很多新版本目前仅支持ESM，但也有遗留很多库没有升级，这就导致了我们在写CLI的时候要考虑Commonjs和ESM模块化的选择。

ESM模块化是未来的趋势，所以我们优先使用支持ESM的库，如果没有在考虑兼容。

下面我们了解下Commonjs和ESM是如何使用，以及CLI中如何处理模块化方案。

## 二、Commonjs

### 1、导出

**action.js**
```js
function handleAction(){
  console.log("action")
}
module.exports = handleAction
```

### 2、引入

```js
const handleAction = require('./action');

handleAction();
```

commonjs的导入使用`module.export`，导出使用`require`

文件可以使用`.cjs`

使用utils项目测试

## 三、ESM

**import、export**

```js
export default

export

import
```

ESM文件默认使用.mjs

## 三、CLI中模块化选择

在CLI中，我们使用ESM有两种配置方式

1、在package.json中 `type: 'module'`

2、文件名结尾使用.mjs

那么开发过程中，有些包不支持ESM怎么解决呢？

我们可以使用

```js
import { } from 'module'
```

在Commonjs模块加载ES6模块

```js
(async () => {
  await import('./index.js')
})
```

在ES6模块中加载CommonJS



