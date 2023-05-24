---
title: 如何给Hexo博客配置自动部署到服务器
date: 2023-05-24 20:57:30
tags: Hexo
toc: true
---

## 前言

Hexo官方提供了很多自动化[部署的方式](https://hexo.io/docs/one-command-deployment#Git)，如下
![](images/hexo-deploy.png)

但我们的项目会直接放在github仓库公开，不可以把密钥和主机地址公开，然后需要接入github action，通过提交代码来自动发布，所以这里记录下部署方式

## 准备工作

- 一台远程服务器
- github一个仓库并搭建Hexo项目

### nginx部署Hexo项目

> 正常部署，服务器安装nginx后，将Hexo生成的public目录上传到Nginx提供的服务地址

![](/images/deploy1.png)

这里部署成功后，开始增加自动化构建的能力

### 获取服务器的私钥

1、先获取服务器是否已经生成了私钥，一般在`~/.ssh`目录下，一般能看到id_rsa、id_rsa.pub两个文件，如果存在可以直接使用

2、如果没有或者想要另外使用新的，可以重新生成

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
命名`github-actions`，回车后将生成文件`github-actions`(私钥) `github-actions.pub`（公钥）

### 将公钥放入`authorized_keys`

将上一步获取的公钥`id_rsa.pub`或生成的`github-actions.pub`写入当前目录下的authorized_keys文件，如果没有可以新建，也可以执行下面命令

```bash
cat github.pub >> authorized_keys
```

### 配置github环境变量

将不方便公开的变量放入github的环境变量

将私钥、主机IP、用户名、主机上服务的部署目录等

打开github中Hexo的仓库，点击Setting => Secrets and variables => Actions

![](/images/github-action.png)

点击New repository secrets输入对应的变量，变量名自动回转化成大写，这里我们输入三个变量

- BLOGIP：目标主机的IP
- BLOGUSER：登录目标主机的用户名
- SSH_PRIVATE_KEY：目标主机的私钥，`id_rsa`或生成的`github-actions`

这里要注意，私钥可以执行命令`cat id_rsa`获取，要完整的录入。

## 配置Github Action

`.github/workflows/deploy.yml`

环境变量读取方式`${{ secrets.SSH_PRIVATE_KEY }}`

```yml
name: deploy blog

on:
  push:
    branches:
      - master # default branch

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout
        uses: actions/checkout@master
      - name: Build and Deploy
        run: |
          yarn install 
          yarn build
      - name: Deploy
        uses: easingthemes/ssh-deploy@main
        env:
            SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
            ARGS: "-rlgoDzvc -i"
            REMOTE_USER: ${{ secrets.BLOGUSER }}
            REMOTE_HOST: ${{ secrets.BLOGIP }}
            SOURCE: ./public/
            TARGET: /data/monto-blog/public/
```

## 提交测试

切换到master分支，提交代码，推送代码，打开Github Actions目录发现有任务执行，刷新站点，发现内容也更新了

![](/images/github-action1.png)



