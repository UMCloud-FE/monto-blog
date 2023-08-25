---
layout: post
title: 使用CLI实现前端项目自动化部署
date: 2023-08-11 11:40:34
tags:
---

使用github action部署CLI

> 开发完成CLI后，我们要执行eslint、prettier、单元测试、打压缩包、版本发布等一系列操作，那么如果每次人工执行成本就很高，基于此，我们使用自动化工具来帮助我们实现这些能力，这里我们选择`github action`

### 一、github action介绍和开通

`github action`是一种持续集成和持续交付(CI/CD) 平台，可用于自动执行生成、测试和部署管道。

他能做什么？

- 作为DevOps，测试、自动打包、自动发布代码
- 自定义工作流、部署服务器

如何开通？

`github actions`默认是不开通的，如果想要某个项目使用action，那么需要开启服务：

> github项目 > Settings > Actions > General

![github actions](images/cli/github-action.png)

选择第一项，允许操作，注意：第一个选项是最高权限，可以根据自己的需要选择

专有名词介绍

**Actions Runners**

首先我们不需要配置Runner，因为官方已经有可以直接使用的Runner。不过还是要了解一下Runner，为后续的工作打基础。

实现自动化构建，那么就要有一个执行这些操作的地方，而Runner就是做这个事情的。比如提交完成代码后，要打包源文件，以往都是在本地主机、服务器构建。

通过自动化构建，我们需要将本地主机或者服务器和Github Action关联起来，这样就可以在你的本地或者远程服务器执行操作，而将Github Actions和本地主机或服务器连接起来的工具就是Runner。

所以配置Runner要做的就是下载actions-runner，然后到执行操作的本地主机或服务器安装，启动。这样github action就可以读取可用的Runner了。

需要画图解读：本地打包构建和CI - runner的区别

**Actions仓库**

我们在Runner执行的操作很多都是重复的，比如检出分支，安装node环境等，如果重复配置就会很麻烦，所以官方就创建了一个仓库，叫github/actions，一些常见的操作只需要使用github/actions就可以了。

### 二、github action的配置文件的简单介绍

当配置结束后，运行action将会看到如下的工作流程

![github actions simple](images/cli/overview-actions-simple.webp)

使用github action只需要在项目根目录下新建文件`.github/workflows/[xx].yml`，yml文件名称可以自行命名，也可以有多个文件。

一些基本概念

- workflow
- job
- step
- action



### 三、github action的使用实例

通过github action

### 四、配置Github Runner




