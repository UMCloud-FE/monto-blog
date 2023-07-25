---
layout: post
title: Yargs介绍
date: 2023-07-14 18:54:31
tags: CLI
categories:
  - CLI
---

## 安装

```shell
$ npm install --save yargs
```

## 使用

我们从官网首页提供的例子下手，讲解一下他的使用流程：

```js
// example.js
#!/usr/bin/env node

require('yargs')
  .scriptName("pirate-parser")
  .usage('$0 <cmd> [args]')
  .command('hello [name]', 'welcome ter yargs!', (yargs) => {
    yargs.positional('name', {
      type: 'string',
      default: 'Cambi',
      describe: 'the name to say hello to'
    })
  }, function (argv) {
    console.log('hello', argv.name, 'welcome to yargs!')
  })
  .help()
  .argv
```

执行：

```shell
$ node example.js hello --name Parrot
hello Parrot welcome to yargs!
```

我们看看官网文档中关于这些指令的解释：

- scriptName：设置执行脚本名称，默认是 process.argv[1] 或者 process.argv[0]
- usage：

```js
.usage('$0 <port>', 'start the application server', (yargs) => {})
```
usage接受三个参数，


- command
- positional
- help
- options