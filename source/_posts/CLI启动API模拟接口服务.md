---
layout: post
title: 实战 - 通过CLI实现API模拟接口服务
date: 2023-06-30 18:55:28
tags: CLI
categories:
  - CLI
---

## 一、前言

> 这个章节我们实战学习，如何通过CLI来进行完成实现API模拟接口的服务

前端在开发的过程中，总会遇到后端接口无法按时提供的情况，这种情况下有很多解决方案，常见的通过客户端生成模拟api，在项目中引入Mock，配置反向代理模拟API服务。

在实际使用过程中，重复的工作比较繁琐，我们如何将这个过程简化出来呢？CLI就起到了它的作用。

## 二、方案原理

分析可以了解，我们不需要复杂的API服务，只需要可以将前端服务串联起来的API就可以满足，基于此，我们可以通过CLI启动一个或多个静态文件服务器，读取本地写好的API返回数据，然后转化成JSON通过接口返回。

TODO: 插件化这一步：在这个基础上，我们甚至可以将JSON文件通过后端的Swagger来生成，这样连模拟的JSON数据都不用写了

- 1、启动Http服务
- 2、通过请求参数匹配对应的模拟数据
- 3、模拟数据支持Mockjs规则
- 4、提供其他配置项

**说明**
- **源码项目如下：**
- **项目中使用到的插件不再展示写，具体可以Clone项目cli-utils来跑一下看看各个插件的效果或者到npm官方查看**
## 三、项目实战

### 1、配置命令参数

首先我们配置mock命令，设定参数
`type`: 分别代表API类型，这里写一下Restful风格的实现思路
`port`: 服务启动的端口
`timeout`: 这是接口延时返回
`customPath`: 设置Mock数据的存放地址
`withoutOpenBrowser`: 默认打开浏览器

具体实现如下：

```js
const mock = require("./command/mock")
{
    command: 'mock',
    description: 'Start a local server to mock returning API data.',
    options: {
      type: {
        type: 'string',
        default: '',
        describe: 'Choosing an API style',
        choices: ['', 'restful', 'action'],
      },
      port: {
        type: 'number',
        default: 9000,
        describe: 'Choosing a port number for startup',
      },
      timeout: {...},
      customPath: {...},
      headers: {...},
      withoutOpenBrowser: {...},
    },
    callback: async (argv) => {
      mock({
        ...argv,
      });
    }
}
```
然后通过`yargs`插件将这些配置传入，执行指令即可在mock函数中取得。

### 2、检查、生成Mock数据

用户首次使用一般不会自己新建JSON数据，所以要直接生成一份，供使用方快速上手。
在调用的时候也要校验下对应的JSON数据是否存在，如果不存在提示用户新建。

具体的实现过程：

**先写一份预置的JSON数据，用来在用户初次使用的时候生成对应的JSON文件, 命名：restfulRes.json**

```json
{
  "get": {
    "v1/user": {
      "RetCode": 0,
      "Message": "success",
      "Data": [
        {
          "Id": 1,
          "UserName": "name"
        }
      ]
    }
  },
  "post": {
    "v1/create": {
      "RetCode": 0,
      "Message": "success",
      "Data": ""
    }
  }
}
```

根据`restfulRes.json`为用户生成对应的json文件，作为API的数据源

这里生成的逻辑是，在`mock/restful/get`目录下，以请求路径命名的json文件（以短横线连接）

```js
function checkAPIPath(filePath) {
  // 显示loading效果
  const spinner = ora('Generating mock data...').start();
  try {
    // 读取预置的JSON文件，循环在用户的运行CLI指令的目录下生成Mock文件，并包含JSON数据
    const data = readJson(path.join(__dirname, 'restfulRes.json'));
    Object.keys(data).forEach((key) => {
      fs.mkdirSync(`${filePath}/${key}`, { recursive: true });
      Object.keys(data[key]).forEach((api) => {
        const apiName = api.split('/').join('-');
        let dataJson = JSON.stringify(data[key][api], '', '\t');
        fs.writeFileSync(`${filePath}/${key}/${apiName}.json`, dataJson);
      });
    });
    spinner.succeed('Generate mock data is success');
  } catch (err) {
    logger.error(err.message);
    spinner.fail(`There is an error: ${err.message}`);
  }
}
```
TODO: 截图指令运行的过程

TODO: 截图展示生成的Mock数据

### 3、启动Http服务器

启动express服务器，配置参数解析和跨域设置，并获取mock数据的目录，启动Http服务器

```js
const app = express()
app.use(bodyParser.json({limit: '50mb'}))
app.use(express.urlencoded({ extended: true }));

app.all('*', (req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*'); //访问控制允许来源：所有
    res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept'); //访问控制允许报头 X-Requested-With: xhr请求
    res.header('Access-Control-Allow-Metheds', 'PUT, POST, GET, DELETE, OPTIONS'); //访问控制允许方法
    res.header('X-Powered-By', 'nodejs'); //自定义头信息，表示服务端用nodejs
    res.header('Content-Type', 'application/json;charset=utf-8');
    next();
});

// 接收http请求，处理请求地址为mock文件地址，匹配mock目录下的文件
function generateApi(app, filePath) {
  app.all('*', async (req, res) => {
    const method = req.method.toLowerCase();
    const apiName = req.url.substr(1).split('/').join('-');
    const file = `${filePath}/${method}/${apiName}.json`;
    fs.readFile(file, 'utf-8', function (err, data) {
      if (err) {
        res.send(env.notFoundResponse);
      } else {
        res.send(JSON.parse(data));
      }
    });
  });
}
```

### 4、使用CLI

启动CLI

`node index.js`

调用接口

修改配置再次调用接口
### 5、CLI优化

- 引入mockjs

为了让我们返回的API接口更加接近于真实数据，我们引入mockjs来做数据的过滤

在CLI中引入`yarn add mockjs`，然后修改代码

```js
const mockjs = require('mockjs');

res.send(Mock.mock(JSON.parse(data)));
```
修改mock下的json数据

```json
{
  "Data": "@name"
}
```

再次启动CLI，我们发现接口数据每次都会变化

## 四、总结

> 对于大部分mock来说，复杂的配置是不需要的，这里仅仅是用了固定的json数据，我觉得就够用了，当然还可以自行处理数据，引入Mock规则，自行定义生成数据类型等

以上就是搭建一个CLI的过程，我们从最简单的开始构建了一个基础CLI，在这个基础上进行了交互优化，然后实际开发了CLI的两个功能：
- 1、通过CLI管理部门内的模版项目
- 2、通过CLI启动Http服务器，模拟接口生成