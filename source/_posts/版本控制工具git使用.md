---
title: 版本控制工具git使用
date: 2021/11/4
tags: ["git", "版本控制工具"]
---

## git 基础

- **git init** 初始化本地 git 仓库
- **git status** 查看文件状态
- **git ls-files** 查看本地仓库中存储的文件

- git 文件状态

  - untracked 未追踪的文件
  - modified 已修改
  - staged 已暂存
  - committed 已提交

- **git log 查看提交信息**

  - git log --oneline 查看提交简要信息

- **git restore**

  - git restore \<filename>: 将本地仓库的某文件从全部区域删除
  - git restore --staged \<filename>:取消暂存，将文件回退到工作区

- **git branch**

  - git branch: 查看分支
  - git branch \<branch name\>: 创建分支
  - git branch -d \<branch name\>: 删除分支，无法删除未合并的分支，通过 git diff 比较差异
  - git branch -D \<branch name\>: 强制删除分支

- **git checkout**

  - git checkout \<branch name>: 切换分支
  - git checkout -b \<branch name>: 切换到新分支

- **git diff**
- git diff \<branch name\>： 查看当前分支本地仓库和目标分支差异

- **git merge**
- git merge \<branch name\>： 合并指定分支到当前分支

- **git remote**
- git remote add \<repo name> \<repo url> 关联远程仓库
- git remote -v :查看远程仓库信息

- **git push**
  - git push --set-upstream origin dev:将本地分支推到远程 dev 分支(远程仓库没有 dev，则新建 dev 分支)
  - git push origin dev:将本地分支推到远程 dev 分支

## git 四个区

- 工作区：当前工作目录

  - git add \<filename> : 将文件推送暂存区
  - git restore \<filename>: 丢弃工作区的改动

- 暂存区：

  - git commit -m '注释' : 将文件推送到本地仓库
  - git rm --cache \<filename>:删除暂存区的文件（文件重置为未追踪状态）
  - git restore --staged \<filename>:取消暂存，将文件回退到工作区

- 本地仓库

  - git reset --soft \<commitid> :将本地仓库版本回到到 commitid 版本，将改动文件回退暂存区
  - git reset --hard \<commitid> :将全部区域（不包含远程仓库）回退至目标版本
  - git reset --mixed \<commitid> :将仓库温江回退至目标版本，将改动文件存在于工作区

  - git push ：将文件推送到远程仓库

- 远程仓库

  - git pull 拉取远程仓库文件(git fetch + git merge)
  - git fetch origin master
