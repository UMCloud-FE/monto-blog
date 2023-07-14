---
layout: post
title: CLI管理模版项目
date: 2023-06-30 18:54:31
tags: CLI
categories:
  - CLI
---

# 五、CLI管理模版项目

随着前端开发业务的增多，重复造轮子的情况屡次出现，现提供“云组件”功能：用户上传写好的组件到远程仓库，在本地开发的时候，通过 cli 按照名称拉取组件模板到本地，

## 1、配置下载命令

我们的模板应该能支持这几种功能：

1. 前端框架选择
2. 不同UI库选择
3. 不同css预编译器选择
4. 支持用户自定义模板仓库的功能

下面展示了命令配置的全部可选参数：

```js
// 默认的配置项
const positionals = [
  {
    key: 'type',
    payload: {
      describe: 'Frame type',
      type: 'string',
      choices: ['react', 'vue', 'angular'],
    }
  },
  {
    key: 'component',
    payload: {
      describe: 'Component name',
      type: 'string',
      choices: ['mui-less-v5/list'，'antd-less-v5/list'],
    }
  }
];

const options = {
  type: {
    alias: "t",
    type: "string",
    require: true,
    describe: '输入想要生成的前端框架类型',
    describeEN: 'Frame type you want to generate',
  },
  conponent: {
    alias: 'c',
    type: 'array',
    require: true,
    describe: '输入想要生成的组件名称',
    describeEN: 'Component name you want to generate',
  },
};

// config.js
{
  command: "generate [type] [component]",
  alias: 'g',
  descriptions: "生成模板组件",
  positionals: positionals,
  options: options,
  callback: async (argv) => {
    templateGenerate(argv);
  }
}
```
参数说明：

- t: 框架类型，比如 react
- c: 具体的功能性组件名称，比如 antd-less-v5/list 表示 antd5使用less的list组件

因为存在必选参数，index.js 需要改造：

```js
const getPositionalChaining = (chainStart) => {
  let next = chainStart.next();
  let result;

  while (!next.done) {
    result = (result || yargs).positional(next.value.key, next.value.payload);
  }

  return result;
};

yargs.command(
  command,
  description,
  (yargs) => getPositionalChaining(positionals[Symbol.iterator]()).options(options), // 这里需要做成链式，不能是数组
  (argv = {}) => {
    if (!argv) process.exit(0);
    callback({ ...argv });
  },
);
```

在回调函数中接受参数：

```js
// templateGenerate
const { type, component } = argv.argv;
```

获取到参数后就可以着手拉取模板了。

## 2、准备模版文件列表

我们针对不同的功能组件，设立了不同的模板仓库分支：

- react/mui-less-v5/list
- react/mui-less-v5/date-picker
...

我们准备了系统自带的模板仓库:

```
git@gitee.com:monto_1/cli-tepmlate.git
```

如果用户想要自定义，我们提供两种自定义配置项的方式。

1. 外挂配置

用户在启动命令行的目录下放置配置文件。

2. 用户通过 cli 指令设置自定义配置文件 url（待开发）
```
monto-dev-cli config git@gitee.com:monto_1/cli-tepmlate.git
```

配置文件格式如下：

```js
export const config = {
  templete: {
    remote: 'git@gitee.com:monto_1/cli-tepmlate.git',
    list: {
      react: ['antd-less-v5/list'],
      vue: []
    },
  },
  mock: {
    proxyApiUrl: ''
  }
}
```
请统一命名为 `monto.config.js`, 我们会在项目里检测自定义配置：

```js
// index.js
import { config } from './monto.config.js';

yargs.command(
  command,
  description,
  (yargs) => {
    // 1. 本地目录
    if (config && config.templete) {
      const customPositionals = [
        {
          key: 'type',
          payload: {
            describe: 'Frame type',
            type: 'string',
            choices: Object.keys(config.templete.list).sort()
          },
        }
      ];
      // {
      //   key: 'component',
      //   payload: {
      //     describe: 'Component name',
      //     type: 'string',
      //     // TODO: 需要根据上一次指令选择的结构动态指定下一次选项
      //     choices: ['mui-less-v5/list'，'antd-less-v5/list'],
      //   }
      // }
      return getPositionalChaining(customPositionals[Symbol.iterator]());
    }

    // 默认的本地配置 
    return getPositionalChaining(positionals[Symbol.iterator]()).options(options)

    // 2. 远程 TODO
    // return fetch('配置的文件地址')
  },
  (argv) => {
    // TODO: 嵌套式读取参数
  },
  (argv = {}) => {
    if (!argv) process.exit(0);
    callback({ ...argv });
  },
);
```

## 3、下载逻辑

1. 通过参数获取分支

> 参数举例：react -> antd-less-v5/list

```js
const branch = `${argv.type}/${argv.component}`
```

2. 拉取模板

生成目录：

```js
fs.mkdirSync(`${rootPath}/generate-components`);
```

拉取仓库到目录：
```js
const template = await execa(
  `git`,
  ['clone', `git@gitee.com:monto_1/cli-tepmlate.git#${branch}`],
  { cwd: `${rootPath}/generate-components` },
);
```

现在就可以在 rootPath 下看到生成的组件目录了。
