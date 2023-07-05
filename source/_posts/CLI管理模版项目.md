---
layout: post
title: CLI管理模版项目
date: 2023-06-30 18:54:31
tags: CLI
categories:
  - CLI
---

# 五、CLI管理模版项目

## 1、配置下载命令

```js
{
    command: "create",
    descriptions: "拉取一个项目模版",
    options: {
      name: {
        alias: "n",
        type: "string",
        require: true,
        describe: "项目名称",
      },
    },
    callback: async (argv) => {
      create({name: argv.name})
    }
}
```
## 2、准备模版文件列表

```js
const map = {
   vue: "https://github.com/luchx/ECHI_VUE_CLI3.0.git",
   react: "git@github.com:richLpf/antd-template-demo.git#main"
}
```

## 3、下载逻辑

1、inquirer提示用户选择对应的模版，具体可以看前面用法

2、通过download下载模版文件

```js
module.exports = function downloadFromRemote(url, name) {
  return new Promise((resolve, reject) => {
    download(`direct:${url}`, name, { clone: true }, function (err) {
      if (err) {
        reject(err);
        return;
      }
      resolve();
    });
  });
};
```
3、通过ora在下载过程中展示loading

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fd3edaa44794da09893047f4c4d2bcb~tplv-k3u1fbpfcp-watermark.image?)

**这是主要的下载逻辑，关于日志，我们可以更好的展示，也可以配置更复杂的命令：替换下载的项目名称，删除.git目录，修改package.json配置**