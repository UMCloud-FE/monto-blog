---
layout: post
title: 认识CLI
date: 2023-06-30 18:49:32
tags: CLI
---

# 一、CLI有啥用，认识CLI

前端开发过程中常见的CLI有：
- create-react-app 
- vue-cli 
- webpack-cli
- prettier-cli

基本复杂一点的工具都在集成CLI，为啥都要搞成CLI呢？

因为CLI可以提供更强大的功能:

- 通过命令搭配实现不同的功能
- 管理项目模版
- 启动本地服务
- 生成模版文件
- 对代码进行格式化

# 四、CLI优化

## 1、提取命令行配置到一个文件中

启动文件`index.js`

```js
const yargs = require('yargs/yargs')
const { hideBin } = require('yargs/helpers')
const config = require('./config')

const yargsCommand = yargs(hideBin(process.argv))

config.forEach(commandConfig => {
  const { command, descriptions, options, callback } = commandConfig
  yargsCommand.command(
    command,
    descriptions,
    yargs => yargs.options(options),
    (argv, ...rest) => {
      callback(argv, ...rest);
    }
  )
})
  
yargsCommand.help().argv
```
指令文件`config.js`

```js
const commandOptions = [
  {
    command: "create",
    descriptions: "拉取一个项目模版",
    options: {
      name: {
        alias: "n",
        type: "string",
        require: true,
        describe: "项目名称",
      },
    },
    callback: async (argv) => {
      create({name: argv.name})
    }
}]
```
提取配置，就很清晰了

## 2、配置ESLint和Prettier，之前专门写了自动化检查代码的文章。

[ 前端工程化：Prettier+ESLint+lint-staged+commitlint+Hooks+CI 自动化配置处理](https://juejin.cn/post/7074893218034384927)

## 3、提取脚本生成使用文档

上一步我们对配置进行了提取，接着根据配置生成使用文档，如下

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1159ef7974cd43ac86808c797145ea69~tplv-k3u1fbpfcp-watermark.image?)

**使用方法**

```
yarn add json2md@1.12.0 -D
```

```js
const json2md = require("json2md")

// 按json2md需要的数据格式组合
const data = json2md([
    { h1: "JSON To Markdown" }
  , { blockquote: "A JSON to Markdown converter." }
]))

// 写入Readme.md文档
fs.writeFile(path.join(__dirname, "../Readme.md"), json2md(data), (err) => {
  if (err) throw err;
  console.log("update docs success");
});
```