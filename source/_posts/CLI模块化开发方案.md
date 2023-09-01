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

Node开发时，在服务器端由于文件都位于本地磁盘，加载速度快，通常使用`commonjs`加载方案。

但由于历史原因，很多`Nodejs`库用的是Commonjs，也有很多新版本目前仅支持ESM，这就导致了我们在写CLI的时候要考虑使用包的加载方式。

下面我们了解下`Commonjs`和`ESM`是如何使用的？

## 二、Commonjs和ESM两种加载模式的区别

在开发之前，我们先简单了解一下Commonjs和ESM的基础知识，如果比较熟悉，可以直接跳过

### 1、如何使用`Commonjs`语法？

> Commonjs语法，导出使用关键字module.exports、导入使用require

- 文件以`.cjs`命名
- `package.json`不声明type字段或type字段为`commonjs`时

```js
// lib.cjs
const name = "LibName"
function handleAction(){
  console.log("lib")
}
module.exports = {
    name,
    handleAction
}

// main.cjs
const action = require('./action');

action.handleAction(); // "lib"
action.name // "LibName"
```

### 2、如何使用ESM语法？

> ESM语法，导出使用关键字export和export default、导入使用import

- 文件以`.mjs`命名
- `package.json`声明type字段为`module`时

```js
// lib.mjs
export const name = "LibName"
export default handleAction(){
    console.log("lib")
}

// main.mjs
import handleAction, { name } from './lib.js'
handleAction(); // "lib"
console.log("name")
```

## 三、CLI中模块化选择

### 1、包加载方案选择

对于新开发一个CLI来说，ESM模块化是未来的趋势，很多库已经不再支持Commonjs，所以我们优先使用支持ESM的库，如果没有再考虑兼容。

那么开发过程中，必须要考虑的问题：

- 1、在ESM项目中，引用的包不支持ESM怎么办呢？
- 2、在Commonjs项目中，拉取的包不支持Commonjs怎么办？

### 2、ESM模式下，如何兼容仅支持`Commonjs`的包

> 我们以`mockjs`包为例，这个包仅支持`Commonjs`

#### 1）整体加载（推荐）

ESM可以使用import加载Commonjs模块，但只能整体加载，不能只加载单一的输出项。

```js
// 正确写法
import Mock from 'mockjs';
const { mock } = Mock;

mock({"age|0-100": 20}) // 20

// 错误写法
const { mock } from 'mockjs';
```

#### 2) 利用Node.js内置方法

可以使用`module.createRequire()`方法加载Commonjs模块，但这样处理就导致ESM和Commonjs混用了。

```js
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const { mock } = require('mockjs');

mock({"age|0-100": 20}) // 20
```

## 四、同时支持两种导出方法

ESM是为了打平浏览器和服务器写法的差异，可以支持打包成UMD全环境支持。

如果我们的包只需要运行在服务端，那么支持ESM即可。

### 1、判断当前环境，导出不同的格式（不推荐）
```js
// lib.js  
export function lib() {  
    console.log('Hello, world!');  
}  
  
if (typeof module === 'object' && typeof module.exports === 'object') {  
    // CommonJS 环境  
    module.exports = lib;  
}
```

在ESM模块中导入

```js
import { lib } from './lib.js'
lib(); // Hell, world!
```
在Commonjs模块中导入

```
const lib = require('./lib.js');
lib.lib(); // Hell, world!
```

### 2、通过配置webpack、rollup等第三方插件，自定义生成不同的包。(推荐)

#### 1）webpack配置

通过配置**webpack.config.js**，即可简单构建支持不同加载方式的包。

```js
const path = require('path')
module.exports = [{
    entry: './index.js',
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: 'umd.js',
        library: {
            name: 'umd-name',
            type: 'umd'
        }
    }
},{
    entry: './index.js',
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: 'common.js',
        library: {
            name: 'umd-name',
            type: 'commonjs2'
        }
    }
}]
```

#### 2）rollup配置（推荐）

安装`yarn add -D rollup @rollup/plugin-node-resolve、@rollup/plugin-commonjs `

**rollup.config.js**配置文件内容
```js
import resolve from "@rollup/plugin-node-resolve";
import commonjs from "@rollup/plugin-commonjs";

export default [
  {
    input: "index.js",
    output: {
      name: "rollup-lib",
      file: "dist/browser.js",
      format: "umd",
    },
    plugins: [
      resolve(), 
      commonjs(),
    ],
  },
  {
    input: "index.js",
    output: [
      { file: "dist/lib-cjs.js", format: "cjs" },
      { file: "dist/lib-es.js", format: "es" },
    ],
  },
];
```

## 五、总结

这一节主要讲关于CLI选择加载模式的选择，包括我们常常遇到的问题
- 到底选择Commonjs语法还是ESM语法？
- 使用ESM语法的过程中，如何引用第三方Commonjs的库？
- 如果想要将构建的库打包成多份如何配置？

有了这些基础支持，我们就可以继续愉快的开发了。
