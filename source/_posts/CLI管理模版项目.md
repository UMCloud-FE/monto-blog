---
layout: post
title: 六、使用CLI实现前端云组件模板管理功能（实践篇）
date: 2023-06-30 18:54:31
tags: CLI
categories:
  - CLI
---

# 六、使用CLI实现前端云组件模板管理功能（实践篇）

随着前端开发业务的增多，重复造轮子的情况屡次出现，我们可以将封装好的组件用单独的代码仓库来维护，并提供 cli 来触达组件：用户上传写好的组件到远程仓库，在本地开发的时候，通过 cli 按照名称拉取组件模板到本地.

通过命令行生成模板到本地，一般要经历这几步：

1. 输入模板参数与指令：确定是哪个模板
2. 输入本地路径：确定拉取到本地的位置
3. 执行 git clone 指令拉取仓库

## 1、配置模板参数

我们的模板应该能支持这几种功能：

1. 前端框架选择
2. 不同UI库选择
3. 不同css预编译器选择
4. 支持用户自定义模板仓库的功能

为了提高配置的用户灵活度，我们不但内置了一些模板库和配置项，还支持用户自定义自己的模板仓库和扩展配置项。我们内置的模板库，每一个分支就是一个独立的组件，命名如下：`react/mui-less-v5/list`，用户也可以通过配置文件自定义名称。

关于参数，这里总结了一下，一共这么几个：

1. 配置文件名
2. 模板框架列表
3. 各个框架对应的模板分支路径

下面展示了默认内置的配置模板：

```js
// 默认的配置项 (monto.config.default.js)
template: {
  generateDirectory: `${process.cwd()}/generate-component`,
  remoteRegistry: 'git@gitee.com:monto_1/cli-template.git',
  types: ['react', 'vue'],
  components: {
    react: ['mui-less-v5/list/prod'],
    vue: ['antd-css-v5/list'],
  },
},
```

用户可在命令执行的目录下新建一个文件 monto.config.js，结构目录与默认配置相似，参考如下：

```js
template: {
  remoteRegistry: '你的远程仓库',
  generateDirectory: '你的放置目录',
  types: ['你的框架类型1', '你的框架类型2'],  // 你的可选框架列表
  components: {  // 你的可选组件列表
    "你的框架类型1": ['你的组件名称1', '你的组件名称2'],
  },
}
```

上面参数都为可选，没有配置则默认使用系统默认。

接下来写入合并的逻辑：

```js
// lib/config.js

// 参数介绍
// defaultConfigName: 'monto.config.default.js',
// configName: 'monto.config.js',
const defaultConfig = require(path.join(__dirname, '../', defaultConfigName));
const userConfig = require(path.resolve(configName));

// 用户配置覆盖默认配置
const template = Object.assign({}, defaultConfig.template, userConfig.template);

export { template }
```

其中defaultConfig便是上面monto.config.default.js的默认参数，而userConfig便是用户自定义的配置。这里需要注意，获取默认配置的额地方，需要使用 __dirname 来获取源码目录lib的位置，而用户配置userConfig则使用path.resolve来解析路径，获取用户命令执行所在目录。

至此，我们已经讲了如何配置以及获取自定义配置的参数了。接下来讲讲配置指令。

## 2. 配置指令

我们在根目录下的config.js中添加命令配置：

```js
const templateGenerate = require('./command/template-generate');

const commandConfigs = [
  {
    command: ['generate', 'g'],
    showInHelp: true,
    description: '生成模板组件',
    descriptionEN: 'generate component',
    options: {
      type: {
        alias: 't',
        type: 'string',
        describe: '输入想要生成的前端框架类型',
        describeEN: 'Frame type you want to generate',
      },
      component: {
        alias: 'c',
        type: 'string',
        describe: '输入想要生成的组件名称',
        describeEN: 'Component name you want to generate',
      },
    },
    callback: async (argv) => {
      templateGenerate(argv);
    },
  },
  // ...
]
```

指令配置为 generate，别名叫 g。有两个可选参数:

- type (-t): 框架类型，比如 react
- component (-c): 具体的功能性组件名称，比如 antd-less-v5/list 表示 antd5使用less的list组件

目前关于参数我们还有一个问题：仓库地址和目录，我们可以拿用户的或者使用我们系统内置的，但是生成组件的信息如何获取呢？配置文件里只配置了列表，并没有具体指定是哪个框架（type）和哪个组件（component），用户命令行里如果没有带这两份参数怎么处理呢？

> 在配置文件里具体指定组件名称，作者认为没有太大的应用场景，与其想每次都要改一下配置文件 monto.config.js来切换生成组件，还不如直接带指令参数或者交互界面选择来得好。

## 3. 处理模板参数

我们关注一下 templateGenerate 函数。其接收的参数 argv，如果命令行里加入了参数，就会在argv里，命令行里没有的参数，argv里也不会有。因此在这个生成函数中，我们需要判断，命令行里带了完整的参数的话就直接使用，否则就根据用户的配置项（types和components）让用户选择并提供交互界面。

```js
// templateGenerate
const { template } = config(); // lib/config.js 拿到的处理后的配置参数
const { types, components, generateDirectory, remoteRegistry, } = { ...template };

// 如果参数完整，则直接进入生成函数。
if (argv.type && argv.component) {
  await generate({
    generateDirectory,
    remoteRegistry,
    ...argv,
  });
  return;
}
```

上面的分支适合如下指令： `monto-dev-cli g -t react -c antd-less-v5/list`.

另一种情况，我直接输入 `monto-dev-cli g`，系统自然不知道你想要生成那个组件，但是你有配置文件啊，通过读出 types, components 这两个可选项，可以你提供一个选择界面。交互界面我们使用 inquirer 库来实现，由于我们的选择有级联关系（先选择type，在选择component），我们使用一个数组来配置：

```js
const getComponentOptions = () => {
  return types.map((type) => {
    return {
      name: 'component',
      type: 'autocomplete',
      message: 'Please select the component: ',
      source: searchOptions(components[type]),
      when: (answers) => answers.type === type,
    };
  });
};

const questions = [
  {
    name: 'type',
    type: 'autocomplete',
    message: 'Please select the framework: ',
    source: searchOptions(types),
  },
  ...getComponentOptions(),
];
```
上面的代码，我们使用 when 关键字来判断，若选择了 type，并且选择的 type 是对应 component 的 type，就显示选择 component 的选项。你可能会好奇，为啥我的配置项的 type 不是list呢？因为这里如果列表太多的话，移动键盘上下键选择会比较麻烦，我这里使用 inquirer-autocomplete-prompt库来支持模糊搜索，并配置了一个自定义的属性 autocomplete：


```js
const autocomplete = require('inquirer-autocomplete-prompt');

inquirer.registerPrompt('autocomplete', autocomplete);

const searchOptions = (options) => (answers, input) => {
  return new Promise((resolve) => {
    const filteredOptions = options.filter((option) =>
      // 忽略大小写
      option.toLowerCase().includes(input.toLowerCase()),
    );
    resolve(filteredOptions);
  });
};
```

如此便清楚了，选择选项后拿到的数据 result 结构为 ```{ type: 'react', component: 'mui-less-v5/list/prod' }```，根据这个结构，我们直接调用生成函数即可：

```js
await generate({
  generateDirectory,
  remoteRegistry,
  ...answers,
});
```
上面的功能，判断选择类型是 type 时，收集参数到argv，并推送一个选择 component的prompt；如果已经选择了component，则调用generate组件进行下一步生成。

至此，模板参数的收集和处理工作已经完成了，我们最终拿到了类似如下结构的数据：

```
{
  type: "", // 命令行参数或者用户选择得来
  component: "", // 命令行参数或者用户选择得来
  generateDirectory: "", // 本地路径，配置文件得来
  remoteRegistry: "" // 远程仓库，配置文件得来
}
```

接下来将重点放在 generate 函数里，实现拉取模板仓库！

## 4. 根据配置拉取模板仓库到本地

generate 函数接受四个参数：

```js
const { type, component, generateDirectory, remoteRegistry } = argv;
```

针对现type, component和remoteRegistry可以拼接出git拉取的仓库路径。

我们针对不同的功能组件，设立了不同的模板仓库分支：

- react/mui-less-v5/list
- react/antd-css-v5/date-picker
...

目前为了教学，我们分支的命名逻辑是 ```[框架]/[组件名]```：

```js
const branchName = `${argv.type}/${argv.component}`
```
上面的代码，兼容了反斜杠路径模式，并且自动处理了不规范的路径写法，最后的 path.resolve 用户获取真实路径，如果用户传的目录是一个相对目录，resolve可以拿到当前 cwd 的目录，如果传了绝对路径则使用用户的路径。这对以规范和格式化输出比较有帮助。

接下来拉取仓库到目录：

拉取模板前还需要注意处理一下放置目录的规范化。需要兼容反斜杠路径模式，并且处理不规范的路径写法，如果用户传的目录是一个相对目录，需要使用path.resolve拿到当前 cwd 的目录，如果传了绝对路径则使用用户的路径：

```js
const normalizedDirectoryPath = path.resolve(
  path.normalize(generateDirectory).replace(/\\/g, '/'),
);
```

接下来拉取仓库到目录：

```js
await execa(`git`, ['clone', '-b', branchName, remoteRegistry, normalizedDirectoryPath]);
```

上面执行了一个 git clone 指令，其相当于执行了：

```shell
git clone -b [branchName] [remoteRegistry] [normalizedDirectoryPath]
```

---
至此，自定义组件模板的指令配置、参数收集以及组件获取已经讲完了。