## 使用介绍：

https://github.com/cofess/hexo-theme-pure/blob/master/README.cn.md

## 开发

1. 安装依赖

yarn

2. 新建文章

 - npx hexo new page "目录名"    
 - npx hexo new post "文章名"    

3. 文章抬头示例

```md
---
title: 如何给Hexo博客配置自动部署到服务器
date: 2023-05-24 20:57:30
categories:
  - Hexo
tags:
  - Hexo
toc: true
---
```

## 部署

### 手动

> node > 16.14.0

- 登入服务器：
  ssh root@xxx
  输入密码
- 进入目录
  cd /data/monto-blog/
- 拉取最新的代码
- yarn
- npx hexo generate (可能会有缓存问题)
- 重启nginx：nginx -s reload

### 自动

目前已实现 Github Action 自动部署，只需要将代码推送到 master 分支即可。