---
layout: post
title: CLI启动API模拟接口服务
date: 2023-06-30 18:55:28
tags: CLI
categories:
  - CLI
---

# 六、通过CLI启动API模拟接口服务

> 前端在开发的过程中，总会遇到后端接口无法按时提供的情况，这种情况下有很多解决方案，常见的通过客户端生成模拟api，在项目中引入Mock。

这里我们通过CLI来实现一个更加简单好用的模拟方案

## 1、方案原理

通过CLI启动一个或多个服务器，通过Express做路由转发本地的JSON文件

## 2、命令配置

首先我们配置mock命令，设定参数`type`、`port`、`create`分别代表API类型、启动Http服务器端口号，默认9000，`create`是否自动生成接口数据JSON文件。

```js
{
    command: "mock",
    descriptions: "启动一个本地服务，模拟返回接口数据",
    options: {
      type: {
        alias: "t",
        type: "string",
        default: "action",
        describe: "选择API类型",
        choices: ['action', 'restful']
      },
      port: {
        alias: "P",
        type: "number",
        default: 9000,
        describe: "选择启动的端口号",
      },
      create: {
        alias: "c",
        type: "boolean",
        default: false,
        describe: "如果mock目录不存在是否自动创建，默认不自动创建"
      },
    },
    callback: async (argv) => {
      mock({
        ...argv
      })
    }
 ```

## 3、Http服务处理

### 1、启动express服务器，配置参数解析和跨域设置，并获取mock数据的目录，启动Http服务器
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

const filePath = path.join(process.cwd(), `./mock/`)

app.listen(port, () => console.log(`Mock api listening on port ${port}!`));
```

### 2、如果是Action类型的API

比如请求url为：
```js
http://localhost:9000/acl
```
请求体为：
```json
{
    Action: "GetUsers"
}
```
此时在CLI执行的目录，新建`mock/acl/GetUsers.json`

GetUsers.json内容为返回的json数据
```json
{
    "RetCode": 0,
    "Message": "this is error",
    "Data": []
}
```
然后通过`fs`模块读取json文件的数据，直接返回模拟的数据，因为是实时读取的，所以更新json数据不需要重启服务。主要逻辑如下
```js
app.post('*', async(req, res) => {
    const key = req.params[0].substring(1)
    const { Action } = req.body
    const file = `${filePath}${key}/${Action}.json`;
    fs.readFile(file, 'utf-8', function(err, data) {
        if (err) {
            res.send(NotFoundResponse);
        } else {
            res.send(data);
        }
    })
})
```
### 3、如果是Restful类型的API

比如请求url为：
```js
http://localhost:9000/acl/users
```
post请求体为：
```json
{
    limit: 10
}
```
此时在CLI执行的目录，新建`mock/acl/users/post.json`

post.json内容为返回的json数据
```json
{
    "RetCode": 0,
    "Message": "get users",
    "Data": []
}
```

如果是get请求，只需要添加`get.json`就可以了。

然后同样通过`fs`读取对应的json数据

```
app.all("*", async(req, res) => {
    const key = req.params[0]
    const method = req.method
    const file =`${filePath}${key}/${method}.json`
    console.log("file", file)
    fs.readFile(file, 'utf-8', function(err, data) {
        if (err) {
            res.send(NotFoundResponse);
        } else {
            res.send(data);
        }
    })
})
```

这样就完成了CLI启动一个模拟的http服务，用起来很方便，并且可以支持直接在项目中使用。

具体可以试试看[u-admin-cli](https://www.npmjs.com/package/u-admin-cli)

> 对于大部分mock来说，复杂的配置是不需要的，这里仅仅是用了固定的json数据，我觉得就够用了，当然还可以自行处理数据，引入Mock规则，自行定义生成数据类型等

以上就是搭建一个CLI的过程，我们从最简单的开始构建了一个基础CLI，在这个基础上进行了交互优化，然后实际开发了CLI的两个功能：
- 1、通过CLI管理部门内的模版项目
- 2、通过CLI启动Http服务器，模拟接口生成