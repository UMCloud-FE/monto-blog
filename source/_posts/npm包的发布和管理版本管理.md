---
layout: post
title: npm包的发布和管理版本管理
date: 2023-07-19 17:34:06
tags: CLI
categories:
  - CLI
---
## 一、npm 包的发布

> 发布一个npm包是很简单的，但如果不熟悉，那么也是有很多坑要踩的。
### 1、npm版本管理

在发布前，我们要规划好CLI的版本，毕竟这不是一锤子的事。具体如何管理版本，这里简单介绍下，就不再班门弄斧了，可以参考[semver文档](https://semver.org/lang/zh-CN/)

版本定义：X.Y.Z-[标识符]
```
版本格式：主版本号.次版本号.修订号，版本号递增规则如下：

X主版本号：当你做了不兼容的 API 修改，
Y次版本号：当你做了向下兼容的功能性新增，
Z修订号：当你做了向下兼容的问题修正。

先行版本号及版本编译信息可以加到“主版本号.次版本号.修订号”的后面，作为延伸。
```
**标识符的用法**

| 标题 | 表示的意义 |
| --- | --- |
| alpha | 内部测试，bug较多 |
| beta | 稳定版本，可能有bug，公测中 |
| rc | 接近发布rc.1 rc.2 |

**具体的使用**

1、以`0.1.0`版本开始开发，0开头的版本，作为第一版本上线前使用，上线的第一版本为`1.0.0`
2、非考虑完全的、不兼容的重大功能，不建议大版本升级

为了快速生成版本，我们在`package.json`配置`script`脚本

```json
"scripts": {
	"start": "node ./index.js",
  "publish:patch": "npm version patch && npm publish --registry=https://registry.npmjs.org",
  "publish:prerelease": "npm version prerelease && npm publish --registry=https://registry.npmjs.org"
}
```

配置多个指令，下面表格说明

`npm version [xxx]`

- major  0.0.1  1.0.0
- minor  0.0.1  0.1.0
- patch  0.0.1  0.0.1

[参考、实践下](https://www.jianshu.com/p/5565536a1f82)

### 2、npm包的结构

```
── dist/index.js
── .npmignore
── CHANGELOG.md 
── README.md 
── package.json
```
npm发布可以发布源码，也可以发布压缩打包后的代码。

必须要上传的文件
在package.json中，有几个重要的字段

```json
{
  "description": "", // 搜索npm包时会展示描述信息
  "main": "",  // cli的入口文件
  "bin": "",  // 执行cli时的命令
  "files": "", // npm要上传的文件
  "dependencies": {}, // 当前插件必须依赖的插件
  "devDependencies": {} // 当前插件开发使用的插件
}
```

### 3、npm官方仓库

- 选择一个包的名称，可以在npm官方仓库查看
- npm注册账号
- npm login
- npm publish

### 4、npm私有仓库

- 注册私有仓库npm账号（很简单，不用被吓到）
- 选择一个npm仓库没有的包
- npm login --registry=https://[private]
- npm publish

## 二、npm包发布的常见问题

**1、分支上有为提交的代码，无法发布**

> 版本发布时，当前的git分支上必须时clean状态，否则无法发布

**2、npm发布半小时内删除，需要24小时后再次发布，之后不允许删除**

发布要慎重，如果发布半个小时内就删除了包，那么24小时内将不允许发布

**3、npm发布后拉取不到最新版本**

有时候明明npm包已经上传成功了，但是在使用`npm i`却获取不到最新的版本，此时有几种解决方式

- 等待几分钟，npm包同步需要几分钟
- 清理npm缓存`npm cache clean --force`
- 确定指定的npm源，像淘宝镜像可能要晚个5分钟的。比如`--registry=https://registry.npmjs.org`

> npm仓库密码设置位数小于8位

TODO: 确认下改了没？注册npm仓库有个很搞笑的地方，允许设置小于8位的密码，登陆的时候却不允许

获取包：
```
npm install monto-dev-cli --registry=https://registry.npmjs.org
```

## 三、添加一些非必须的效果

- 用户使用安装CLI后，展示欢迎界面

package.json
```json
"scripts": {
  "postinstall": "node printLogo.js"
}
```

```js
import chalk from 'chalk';

//           ____   __  __  ______  _____
//  /'\_/`\/\  __`\/\ \/\ \/\__  _\/\  __`\    
// /\      \ \ \/\ \ \ `\\ \/_/\ \/\ \ \/\ \   
// \ \ \__\ \ \ \ \ \ \ , ` \ \ \ \ \ \ \ \ \  
//  \ \ \_/\ \ \ \_\ \ \ \`\ \ \ \ \ \ \ \_\ \ 
//   \ \_\\ \_\ \_____\ \_\ \_\ \ \_\ \ \_____\
//    \/_/ \/_/\/_____/\/_/\/_/  \/_/  \/_____/
                                        
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

