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

#### 安装

```shell
$ npm install --save yargs
```

#### 使用

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
上面 usage 用于配置帮助菜单中打印自定义提示，argv 表示可返回上面配置的参数，一般写在最后，要想有输出参数功能，在实例化时就要传入 node 参数：`hideBin(process.argv)`，使用 hideBin 包裹一下，表示这里不需要 process.argv 前两项路径参数（可执行文件路径和脚本文件路径），相当于执行了 `process.argv.slice(2)`，以便得到更简洁的参数对象，目前版本的 yargs 可以直接解析；

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
positional 方法，往往与尖括号和方括号参数同步出现，其通过获取括号里的对应参数进行匹配，并输出对应提示和参数类型。

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

```js
// 声明方式1 - 提示参数
.command('命令名称 <必选参数> [可选参数]', '描述', [参数1], [参数2])

// 声明方式2 - 别名
.command(['命令名称', '别名'], '描述', [参数1], [参数2])
```

```shell
# 使用
cli名称 命令名称 [参数]
```

如果配置了必选参数，比如： `.command('login <name>')`，可以这样请求：

```js
cli名称 login 呀哈哈
```

如果参数配置在后边的属性回调中 (比如使用 .option 方法配置)，就需要使用双横线来表示参数了，我们下面会讲到。

> P.S. 注意，这里的必选参数是一个提示规范，yargs只会校验存在与否，若需要更高级的校验，还是需要用户在回调中自定义校验规则。

一个模拟 HTTP请求的例子：

```js
yargs(hideBin(process.argv))
  .command('get', 'make a get HTTP request', {
    url: {
      alias: 'u',
      default: 'localhost:3000/',
      type: 'string'
    }
  })
  .argv
```
打印帮助试试：

```md
命令：
  yargs.js get  make a get HTTP request

选项：
  --help         显示帮助信息                                                [布尔]
  --version      显示版本号                                                  [布尔]
  --get-url, --ug                                                          [字符串]
```
yargs 自动识别为`命令`，并分组提示。我们试着打印这个命令的 help ：

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
可以看到，帮助文档可以精确到具体的命令。常用的配置参数：

- alias：配置命令参数别名
- default：参数的默认值，不填写参数时的默认取值
- type：参数的接受类型，如果传错了，系统会自动转换为 type 的类型接受

上面配置参数的对象，我们一般会写成一个回调，接受 yargs 实例，来实现更灵活的配置，还可以有第三个参数，也是一个回调，获取处理后的参数：

```js
.command('get', 'make a get HTTP request', yargs => {
    return yargs.option('url', {
      alias: 'u',
      default: 'localhost:3000/',
      type: 'string'
    })
  },
  argv => {
    console.log(argv)
  }
)
```
我们试着执行一下这个模拟请求 HTTP 的指令看看：`node yargs.js get -u 192.168.1.1`：

打印结果：
```md
{
  _: [ 'get' ],
  u: '192.168.1.1',
  url: '192.168.1.1',
  '$0': 'yargs.js'
}
```
可以看到输出结果，配置的get指令的参数 url简写 u 生效并接收了进来，使用 u 或者 url 都可以接受这个参数。

命令还可以链式调用配置（subcommands模式）：

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
可以同时执行：`node yargs.js get --gu 192.168.1.1 && node yargs.js post --pu localhost:3000`，结果如下：

```md
{ _: [ 'get' ], gurl: '192.168.1.1', gu: '192.168.1.1', '$0': 'yargs.js' }
{ _: [ 'post' ], purl: 'localhost:3000', pu: 'localhost:3000', '$0': 'yargs.js' }
```

> P.S. 就算是别名，不止一个字母时，也要使用 -- 哦


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

给帮助参数改名，改完名字后默认名字失效：

```js
.help('h')

// 命令行中使用：node yargs.js -h 
```

定义帮助提示的说明：

```js
.help('h', '我的提示帮助')
```

打印结果：

```md
选项：
      --version  显示版本号                                                      [布尔]
  -h             我的提示帮助                                                     [布尔]
```

如果想要禁用 `--help` 的传参写法，可以这样：

```js
.help(false)
```

当然了，你也可以在任何能够获取到 yargs 实例的地方通过调用 API 显示当前的帮助信息：

```js
.command('post', 'make a get HTTP request', yargs => {
  yargs.showHelp();
})
```

> P.S. showHelp() 的打印，会终止程序并推出，一定要在程序终结之前调用。

这里需注意, showHelp 必须作为 yargs 示例的调用才有意义。如果想要在指令回调里调用来打印指令的帮助信息，可建立全局对象缓存实例：

```js
let instance = null;

yargs(hideBin(process.argv))
  .command('get', 'make a get HTTP request', yargs => {
    instance = yargs;
    return yargs.option('url', {
      alias: 'u',
      type: 'string'
    })
  },
  argv => {
    instance.showHelp();
    console.log(argv)
  }
)
```

当然了，yargs 的功能还有很多这里列举几个常用的配置：

- version： 可配置自定义 --version 帮助信息的指令。譬如自定义缩写、版本号与打印输出内容：`.version('1.0.0', '-v, --version', 'My First CLI !!')`
- parse：提供自定义解析参数的方法，往往与 command 结合进行使用，比如下面开饭的例子：

```js
// 给 lunch-train 命令添加一个参数 time, 并指定参数 restaurant 为 rudy's
yargs(hideBin(process.argv))
  .command('lunch-train <restaurant>', 'start lunch train', () => {}, function (argv) {
    console.log(argv.restaurant, argv.time)
  })
  .parse("lunch-train rudy's", {time: '12:15'})
```
当然也可以接受第三个回调函数参数，接受形参：`.parse("lunch-train rudy's", {time: '12:15'}, (err, argv, output) => {}`，这样就可以拿到解析的结果并自定义输出了。

parse 使用场景比较多，可以用来解析用户输入、配置信息或者生成帮助信息等。

- alias：配置别名。比如：`.alias('version', 'V') 或者 .alias({ version: 'V'})`
- array：告诉解析器哪些参数用数组接收，比如：`.array('books')`

------
现在我们回过头来再看看开始给出的官方案例，抄写如下：

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
可以看到，先是配置脚本名称为 `pirate-parser`，配置了使用说明 usage，并设置一个叫做 hello 的命令，接受一个可选参数 name，使用 positional 配置占位参数的属性，最后在回调中接受输入的 name 参数并打印。

是不是一目了然了！到这里 yargs 的使用我们就有一个基本的了解了。

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
