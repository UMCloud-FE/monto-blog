---
layout: post
title: 二、CLI 常用的工具库介绍
date: 2023-07-14 18:54:31
tags: CLI
categories:
  - CLI
---

# 二、CLI 常用的工具库介绍

在介绍项目之前，有一个前置工作，就是讲述一下我们会用到的第三方工具库，这些工具库对于处理参数、美化交互等有很大的帮助。接下来我们开始。

## yargs

[yargs](https://yargs.js.org/)，按照官网的定义，它是一个帮助你在命令行工具中解析参数并生成优雅界面的工具。我们也知道，通过上一讲我们知道，不用任何框架，使用 `process.argv` 也可以获取请求参数，所以可以这样理解，`yargs` 是一个基于命令行工具的一个框架（或者库），其规范了参数获取、命令配置等，使得命令行工具的开发变得有法可循。

## 安装

```shell
$ npm install --save yargs
```

## 使用

我们从官网首页提供的例子下手，讲解一下他的使用流程：

```js
// example.js
#!/usr/bin/env node
import yargs from 'yargs'

yargs()
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

上面的例子就是一个比较完整的使用案例。我们不用着急看着一大坨是什么，我们一行一行的来分析。

- yargs 实例

我们先看一下 yargs示例是什么：
```js
console.log(yargs())
```
结果：

```js
YargsInstance {
  customScriptName: false,
  parsed: false,
  '$0': 'yargs.js',
  argv: [Getter]
}
```
可以看到，返回了当前配置的属性。上面四个参数中，customScriptName 表示是否自定义了脚本执行的名称，如果没有设置，默认是 `process.argv[1]`。


我们使用下面的例子：

```js
// yargs.js
import yargs from 'yargs'
import { hideBin } from 'yargs/helpers';

console.log(process.argv);

yargs(hideBin(process.argv))
  .usage('$0: My usage: cli-test [command] <option>')
  // .scriptName('cli-test')
  .argv
```
上面 usage 用于配置帮助菜单中打印自定义提示，argv 表示可返回上面配置的参数，一般写在最后，要想有输出参数功能，在实例化时就要传入 node 参数：`hideBin(process.argv)`，使用 hideBin 包裹一下，表示这里不需要 process.argv 前两项路径参数（可执行文件路径和脚本文件路径），以便得到更简洁的参数对象；

我们直接使用node运行：

```shell
node yargs.js --help
```

运行结果如下：

```md
[
  '/Users/user/.nvm/versions/node/v16.14.2/bin/node',
  '/Users/user/Documents/js学习心得/cli-test/yargs.js',
  '--help'
]
yargs.js: My usage: cli-test [command] <option>

选项：
  --version  显示版本号                                                          [布尔]
  --help     显示帮助信息                                                         [布尔]
```

输出 usage 的地方，$0 默认是 process.argv[1] 的默认路径的文件名。我们解开注释再运行一遍：

```js
yargs(hideBin(process.argv))
  .usage('$0: My usage: cli-test [command] <option>')
  .scriptName('cli-test')
  .argv
```
运行结果：

```md
cli-test: My usage: cli-test [command] <option>

选项：
  --help     显示帮助信息                                                         [布尔]
  --version  显示版本号                                                          [布尔]
```
可以看到，yargs.js 的文件名变为了自定义的 cli-test。

- usage('$0 <cmd> [args]')

用于设置脚本使用信息，可以配置多行提示，甚至可以通过回调函数来设置提示文案的分组信息：

```js
yargs(hideBin(process.argv))
  .usage('$0 <port>', 'start the application server', yargs => {
    return yargs.positional('port', {
      describe: 'This is your server port：',
      type: 'number'
    })
  })
  .scriptName('cli-test')
  .argv
```
执行命令：

```shell
node yargs.js --help
```

输出：

```md
cli-test <port>

start the application server

位置：
  port  This is your server port：                                           [数字]

选项：
  --help     显示帮助信息                                                         [布尔]
  --version  显示版本号                                                          [布尔]
```
positional 方法，通过获取尖括号里的对应参数进行匹配，并输出对应提示和参数类型。

上面的例子可以看到，usage 可以接收参数，所以可以用于默认命令的配置。但是一般不建议使用 usage 来配置命令，他的功能应该局限于配置提示。如果你并列写了两个 usage，则会被认定为配置了指令，这会造成错乱：

```js
// ×
.usage('$0 <port> [name]', 'start the application server', yargs => {
  yargs.positional('port', {
    describe: 'This is your server port：',
    type: 'number'
  })
  yargs.positional('name', {
    describe: 'This is your server name：',
    type: 'string'
  })
})
.usage('$0 <type>', 'the type of your application', yargs => {
  return yargs.positional('type', {
    describe: 'This is your app type',
    type: 'string'
  })
})
```
输出不符合预期：
```md
cli-test <type>

the type of your application

命令：
  cli-test <port> [name]     start the application server
  cli-test <type>            the type of your application                  [默认值]

位置：
  type  This is your app type                                              [字符串]

选项：
  --help     显示帮助信息                                                         [布尔]
  --version  显示版本号                                                          [布尔]
```
上面命令和位置同时打出，显然是错乱了。

- command

command 是用于配置命令的方法，使用格式如下：

```shell
# 声明
.command('命令名称', '描述', [参数1], [参数2])

# 使用
cli名称 命令名称 [参数]
```

一个模拟 HTTP请求的例子：

```js
yargs(hideBin(process.argv))
  .command('get', 'make a get HTTP request', {
    url: {
      alias: 'u',
      default: 'localhost:3000/'
    }
  })
  .argv
```
打印帮助试试：

```md
命令：
  yargs.js get  make a get HTTP request

选项：
  --help     显示帮助信息                                                         [布尔]
  --version  显示版本号                                                          [布尔]
```
yargs 自动识别为命令，并分组提示。我们试着打印这个命令的 help ：

```shell
node yargs.js get --help
```
输出：

```md
yargs.js get

make a get HTTP request

选项：
      --help     显示帮助信息                                                     [布尔]
      --version  显示版本号                                                      [布尔]
  -u, --url                                             [默认值: "localhost:3000/"]
```
可以看到，帮助文档可以精确到具体的命令。上面第二个参数我们一般会写成一个回调，接受 yargs实例，来做一个更灵活的配置，还可以有第三个参数，也是一个回调，获取处理后的参数：

```js
.command('get', 'make a get HTTP request', yargs => {
    return yargs.option('url', {
      alias: 'u',
      default: 'localhost:3000/'
    })
  },
  argv => {
    console.log(argv)
  }
)
```
上面，yargs.option 用于配置指令里各个配置项的属性。我们试着执行以下这个模拟请求 HTTP 的指令看看：`node yargs.js get -u 192.168.1.1`：

```md
{
  _: [ 'get' ],
  u: '192.168.1.1',
  url: '192.168.1.1',
  '$0': 'yargs.js'
}
```
可以看到输出结果，配置的get指令的参数 url简写 u 生效并接收了进来，使用 u 或者 url 都可以接受这个参数。

命令还可以链式配置：

```js
.command('get', 'make a get HTTP request', yargs => {
    return yargs.option('get-url', {
      alias: 'gu',
    })
  },
  argv => {
    console.log(argv)
  }
)
.command('post', 'make a get HTTP request', yargs => {
    return yargs.option('post-url', {
      alias: 'pu',
    })
  },
  argv => {
    console.log(argv)
  }
)
```
可以同时执行：`node yargs.js get -gu 192.168.1.1 post -pu localhost:3000`，结果如下：

```md
{
  _: [ 'get', 'post' ],
  g: true,
  u: [ '192.168.1.1', 'localhost:3000' ],
  p: true,
  '$0': 'yargs.js'
}
```
参数缩写默认是一个字母，这里多个命令同时执行时，若缩写不止一个字母，且有共同后缀（这里是 u），则会用一个数组来接收，前缀会用布尔值表示是否存在。

我们将配置文件后缀改为前缀试试：gu -> ug，pu -> up:

再执行指令：`node yargs.js get -ug 192.168.1.1 post -up localhost:3000`：

```md
{
  _: [ 'get', 'post' ],
  u: [ true, true ],
  g: '192.168.1.1',
  p: 'localhost:3000',
  '$0': 'yargs.js'
}
```

这里可以总结一下：多个命令一起执行时，若各个缩写有公共后缀，则参数会被收集在一个后缀字母做key的数组里；若各个缩写有公共前缀，则配置参数分别以各个后缀为key存放，公共前缀是一个布尔数组，表示是否有值。即公共部分放在数组里。

我们这里改一下配置文件缩写为一个字母 gu -> g，pu -> p，再试试指令 `node yargs.js get -g 192.168.1.1 post -p localhost:3000`，结果如下：

```md
{
  _: [ 'get', 'post' ],
  g: '192.168.1.1',
  'get-url': '192.168.1.1',
  getUrl: '192.168.1.1',
  p: 'localhost:3000',
  '$0': 'yargs.js'
}
```
这里的参数就会分开接收了。具体怎么配置，在项目里按照使用场景配置即可。


同类型的命令方法还有一些，我这里汇总一下：

|命令方法|描述|使用|
|-|-|-|
|recommendCommands|用于配置是否在用户输入未知指令时显示建议命令|recommendCommands() 或 recommendCommands(true) 可开启，指令输入不正确时，自动打印 help 信息，并给出推荐：`是指 get?`|
|demandCommand|用于设置是否要求至少输入一个子命令|使用方式： demandCommand(1, 3, '最少一个命令', '最多三个命令') 开启，开启后用户不输入命令，报错：`缺少 non-option 参数：传入了 0 个, 至少需要 1 个`|
|strictCommands|与recommendCommands类似，严格匹配输入指令不对时显示帮助信息|strictCommands() 或 strictCommands(true) 可开启，命令不对时，弹出 help 提示，并显示错误：`Unknown command: ge`|

- help

顾名思义，用于配置和获取帮助信息。使用方式：

```js
yargs(hideBin(process.argv)).help();
```

但 yargs 自带有默认的帮助信息，你如果只需要默认提示即可，则无需配置。这里讲一下自定义配置 help。

配置缩写：

```shell
.help('h')

# 使用：node yargs.js -h 
```



## ora

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

### chalk

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

## inquirer

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

## execa

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
