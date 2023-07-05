---
layout: post
title: CLI终端交互优化
date: 2023-06-30 18:53:52
tags: CLI
categories:
  - CLI
---

# 三、终端交互优化

## 1、log-symbols：输出结果用图标标识

```bash
# 这里使用4版本，升级到5版本不再支持require方式导入
yarn add log-symbols@4.1.0
````

**使用方法**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83e2277a2b6346dab453b720ea2d7ee9~tplv-k3u1fbpfcp-watermark.image?)

**使用效果**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/375335b4ccb14ecfa6fa727dfa875e77~tplv-k3u1fbpfcp-watermark.image?)


## 2、chalk：打印出各种颜色的文案提示，功能强大，几乎支持所有的颜色

```bash
# 这里使用4版本，升级到5版本不再支持require方式导入
yarn add chalk@4.1.2
````

**1、支持传入string，array和模版字符串**

**使用方法**

```js
console.log(chalk.blue("this is blue"))

console.log(chalk.blue('Hello', 'World!', 'Foo', 'bar', 'biz', 'baz'));

console.log(`
CPU: ${chalk.red('90%')}
RAM: ${chalk.green('40%')}
DISK: ${chalk.yellow('70%')}
`);
```
**使用效果**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75d28a1213c547b09891990ad362f477~tplv-k3u1fbpfcp-watermark.image?)

**2、支持修改背景色、各种颜色格式、指定不同文本的颜色**

**使用方法**

```js
console.log(chalk.blue.bgRed.bold('Hello world!'));

console.log(chalk.red('Hello', 
chalk.underline.yellowBright('world') + '!'));

console.log(chalk.hex('#DEADED').underline('Hello, world!'))

console.log(chalk.rgb(15, 100, 204).inverse('Hello!'))
```
**使用效果**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/956f5452cecb460e8297d8c43cb273fb~tplv-k3u1fbpfcp-watermark.image?)

**3、支持自行封装一些颜色**

**使用方法**

```js
const error = chalk.bold.red;
const warning = chalk.hex('#FFA500'); // Orange color

console.log(error('Error!'));
console.log(warning('Warning!'));
```
**使用效果**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffb5047ba6154d3aa164535553127e30~tplv-k3u1fbpfcp-watermark.image?)

## 3、ora：增加loading效果

```bash
# 这里使用5版本，升级到6版本不再支持require方式导入
yarn add ora@5.4.1
````
**使用方法**

```js
const ora = require('ora');
 
const spinner = ora('Loading unicorns').start();
 
setTimeout(() => {
    spinner.color = 'yellow';
    spinner.text = 'Loading rainbows';
}, 1000);
```
**使用效果**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0d282f16b1f48ac88220ca23e1c54d1~tplv-k3u1fbpfcp-watermark.image?)

## 4、inquirer：在终端选择命令

**使用方法**

```js
var inquirer = require('inquirer');
inquirer
  .prompt([
    {
      name: "templateType",
      type: "list",
      default: "vue",
      choices: [
        {
          name: "React",
          value: "react",
        },
      ],
      message: "Select the template type.",
    }
  ])
  .then((answers) => {
    // Use user feedback for... whatever!!
  })
  .catch((error) => {
    if (error.isTtyError) {
      // Prompt couldn't be rendered in the current environment
    } else {
      // Something else went wrong
    }
});
```
**使用效果**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c37c79b14d6d48dcb7973ffbbddb24cd~tplv-k3u1fbpfcp-watermark.image?)

这样就可以给客户提示选择来处理逻辑。

## 5、download-git-repo：download Git仓库

**使用方法**

```js
const download = require("download-git-repo");

download('direct:https://gitlab.com/flippidippi/download-git-repo-fixture.git#my-branch', 'test/tmp', { clone: true }, function (err) {
  console.log(err ? 'Error' : 'Success')
})
```

还有很多类似的插件，来丰富cli的功能，根据需要添加即可
- open 打开浏览器
- yargs/commander 执行cli命令
- get-port 获取当前端口号