---
title: hexo项目部署github
date: 2019/11/13
---

### github 建仓库

注意：仓库名称必须为 昵称.github.io

### 配置文件

1）安装 npm install --save hexo-deployer-git
2）修改配置文件/blog/\_config.yml

```json
deploy:
  type: git;
  repository: https://github.com/bellasource/bellasource.github.io.git
  https: branch: master;
```

### 部署

执行命令：hexo d
成功后，项目即被推送到了远程仓库，并且可以通过 **bellasource.github.io** 打开网页

### 修改配置更新

```json
git add .
git commit -m "修改"
hexo clean //清除缓存文件
hexo generate //生成静态文件
hexo deploy  //部署網站
hexo g
hexo d
hexo s
git push
```
