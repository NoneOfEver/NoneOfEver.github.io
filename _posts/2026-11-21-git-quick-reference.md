---
title: "Git 速查笔记"
description: "A quick reference for everyday Git commands and workflows."
lang: "zh-CN"
permalink: "/blog/git-quick-reference/"
---
## 1. 核心概念

Git 的三个主要区域

工作区（Working Tree） -> git add -> 暂存区（Index）-> git commit -> 本地仓库(Repository)

- Commit：一个版本快照。
    
- Branch：指向提交的可移动引用。
    
- HEAD：当前检出的提交位置。
    
- Remote：远程仓库，例如 GitHub。
    
- Merge：合并两条开发历史。
    
- Rebase：将提交重新应用到另一个基点。
    
- PR：请求将某分支的修改合入目标分支。
    

## 2. 基础命令

|命令|功能|
|---|---|
|`git status`|查看工作区状态|
|`git init`|初始化仓库|
|`git clone URL`|克隆仓库|
|`git add file`|暂存文件|
|`git add .`|暂存当前目录下的修改|
|`git commit -m "msg"`|创建提交|
|`git log --oneline --graph --all`|查看提交历史|
|`git diff`|查看未暂存修改|
|`git diff --staged`|查看已暂存修改|
|`git show`|查看提交详情|

## 3. 分支管理

|命令|功能|
|---|---|
|`git branch`|查看本地分支|
|`git switch branch`|切换分支|
|`git switch -c feature`|创建并切换分支|
|`git merge feature`|合并指定分支|
|`git fetch`|获取远程更新|
|`git pull`|获取并整合远程更新|
|`git push`|推送提交|
|`git push -u origin feature`|推送并设置上游分支|

## 4. 撤销与恢复

|命令|功能|
|---|---|
|`git restore file`|丢弃未暂存修改|
|`git restore --staged file`|取消暂存|
|`git commit --amend`|修改最近一次提交|
|`git revert HASH`|创建一个反向提交|
|`git stash`|临时保存工作区修改|
|`git stash pop`|恢复暂存的修改|

注意：`git restore file` 会丢弃对应文件的未暂存修改；`git reset --hard` 风险更高，使用前应确认数据不再需要。

## 5. GitHub 协作

典型工作流程：

```
git switch -c feature/navigation
# 修改代码
git add .
git commit -m "Improve navigation logic"
git push -u origin feature/navigation
```

之后在 GitHub 创建 PR，经过 Review 后合并。

关于 Contributors：提交最终进入仓库默认分支通常是重要条件；通过 `dev` 间接合入 `main` 也可能计入，但还受作者邮箱识别、合并方式和 GitHub 统计规则影响。

## 6. lazygit

在终端输入：

```
lazygit
```

它是 Git 的终端图形界面，适合查看差异、暂存修改块、提交、切换分支及推送。它与 Neovim 的 gitsigns 互补：gitsigns 关注编辑时的局部修改，lazygit 负责仓库级操作。