---
layout: post
title: CLI工具介绍与封装
date: 2023-07-14 18:54:31
tags: CLI
categories:
  - CLI
---

# yargs

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

# ora

ora 是一个用于显示终端加载文案的开源库，基本使用如下：

```js
import ora from 'ora';

const spinner = ora('Loading...').start();
spinner.color = 'red';

setTimeout(() => {
	spinner.text = 'Cli loaded';
  spinner.succeed();
}, 1000);
```

上面的代码，使用 setTimeout 模拟一个耗时的方法，通过 start 启动并设置初始化文案，然后使用 text 属性可以修改文案，succeed / fail / stop 方法可以终止当前行的打印，如果想要打印新行，重新调用 start 方法即可。

> succeed / fail / stop 都是终止当前命令行打印，不同的是，stop 默认会清除最后打印的文字，succeed 和 fail 会保留该行文字，并加入状态前缀。

你还可以设置文案头的状态图标。比如加载的图标，默认是六个点的动画，官方提供了不少可选项，我们点进去 spinner，可以看到很多的选项：

image.png


选项中，你可以设置为一个月亮：

```js
const spinner = ora({
  text: 'Loading...',
  spinner: 'moon',
}).start();

setTimeout(() => {
	spinner.text = 'Cli loaded';
  spinner.succeed();

  // 加载第二个文案
  spinner.text = 'Another loading';
  spinner.start();
}, 1000);

setTimeout(() => {
  spinner.fail();
}, 3000)
```

如果你的自定义的图标不在可选项中，可以这样：

```js
const spinner = ora({
  text: 'Loading...',
  spinner: 	{
		interval: 80, // Optional
		frames: ['\u2665', '\u2665', '\u2665']
	}
}).start();
```
这样的loading图标就会在实心的心和空心的心之间闪烁了。


你还可以使用 suffixText 在loading图标前再加前缀：

```js
const spinner = ora('Loading...').start();

spinner.prefixText = '🤡';

setTimeout(() => {
  spinner.succeed();
}, 1000);
```

> start 和 stop 方法一定要有一定的时间间隔，不然界面渲染文字会有冲突

# chalk

chalk 是一个基于 node 的格式化终端输出的工具库，提供了不同的色彩和字体样式等功能。

我们可以这样打印彩色的文字：

```js
import chalk from 'chalk';
                                        
const art = `
  ${chalk.bold.red('          ____   __  __  ______  _____')}
  ${chalk.bold.redBright(` /'\\_/\`\\/\\  __\`\\/\\ \\/\\ \\/\\__  _\\/\\  __\`\\   `)}
  ${chalk.bold.yellow('/\\      \\ \\ \\/\\ \\ \\ \`\\\\ \\/_/\\ \\/\\ \\ \\/\\ \\  ')}
  ${chalk.bold.green('\\ \\ \\__\\ \\ \\ \\ \\ \\ \\ , \` \\ \\ \\ \\ \\ \\ \\ \\ \\  ')}
  ${chalk.bold.cyan(' \\ \\ \\_/\\ \\ \\ \\_\\ \\ \\ \\\`\\ \\ \\ \\ \\ \\ \\ \\_\\ \\ ')}
  ${chalk.bold.blueBright('  \\ \\_\\\\ \\_\\ \\_____\\ \\_\\ \\_\\ \\ \\_\\ \\ \\_____\\')}
  ${chalk.bold.blue('   \\/_/ \\/_/\\/_____/\\/_/\\/_/  \\/_/  \\/_____/')}
`;

console.log(art);
console.log('Welcome to monto-dev-cli !');

process.stdout.write('\n')
```

这样，我们就打印出了一个从上到下依次渐变的 LOGO 了。

在 cli 项目里，我们使用 chalk 对打印日志进行了封装：

```js
const colorMapper = {
  dim: (msg) => chalk.dim(msg),
  log: (msg) => msg + '',
  tip: (msg) => chalk.blue.italic(msg),
  success: (msg) => chalk.green(msg),
  warn: (msg) => chalk.yellowBright(msg),
  error: (msg) => chalk.red.bold(msg),
  step: (msg) => chalk.magentaBright(msg.step) + ' ' + msg.content,
};
```

这样，只需要调用 colorMapper 里的方法就可以获取到这些彩色字体。

chalk 还可以和上面的 ora 库结合来改变文案的颜色：

```js
spinner.text = chalk.red('Loading...');
```

# inquirer

inquirer 是开源的命令行提示工具库，提供用户交互的输入、选择等功能。下面是其使用的例子：

```js
const colors = [
  { name: `${chalk.red('Red')}`, value: 'red', short: chalk.red('R') },
  { name: `${chalk.green('Green')}`, value: 'green', short: chalk.green('G') },
  { name: `${chalk.blue('Blue')}`, value: 'blue', short: chalk.blue('B') },
  { name: `${chalk.yellow('Yellow')}`, value: 'yellow', short: chalk.yellow('Y') },
];

inquirer
  .prompt({
    type: 'list',
    name: 'color',
    message: 'Select your favorite color:',
    choices: colors,
    filter: function (selectedValue) {
      // 在这里进行对用户选择的值的处理
      return selectedValue.toUpperCase();
    },
  })
  .then((answers) => {
    console.log(`You selected: ${answers.color}`);
  })
  .catch((error) => {
    console.error('Error:', error);
  });
```

在上面的例子中，提供了用户交互式的选择项，采用异步获取用户选择参数的方式。其实 inquirer.prompt 的参数还可以是一个数组，这就可以实现多个有级联关系的选择配置：

```js
const questions = [
  {
    type: 'confirm',
    name: 'wantFruit',
    message: 'Do you want some fruit?',
    default: false,
  },
  {
    type: 'input',
    name: 'fruit',
    message: 'Which fruit do you want?',
    when: function (answers) {
      // 只有在用户回答 "Do you want some fruit?" 为 true 时才会显示此问题
      return answers.wantFruit;
    },
  },
];

inquirer.prompt(questions).then((answers) => {
  console.log('Answers:', answers);
});
```

我们还可以有更多的扩展，比如选择项太多该怎么提供模糊搜索功能等，这些牵扯到具体的业务封装，我们在之后的文章中进行讲解。

# execa

execa是一个用于在 Node.js 环境中执行外部进程的轻量级库。让开发者可以方便地执行命令行命令，并获取命令的输出结果、错误信息、退出码等信息。execa 的设计目标是为了取代 Node.js 内置的 child_process 模块，提供更加易用和可靠的进程管理功能。

你可以使用它执行shell命令：

```js
execa('ls').then((result) => {
  console.log(result.stdout);
}).catch((error) => {
  console.error(error);
});
```

也可以操作npm项目：

```js
execa('yarn', ['add', 'vue'], {
  cwd: process.cwd(), // 设置命令执行的当前工作目录
  env: { NODE_ENV: 'development' }, // 设置环境变量
}).then((result) => {
  console.log(result.stdout);
}).catch((error) => {
  console.error(error);
});
```

当然了，也可以进行监听进程、 git管理等操作，使用比较广泛。
