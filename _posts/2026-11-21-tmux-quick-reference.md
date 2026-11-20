---
title: "tmux 速查笔记"
description: "A quick reference for tmux sessions, windows, and panes."
lang: "zh-CN"
permalink: "/blog/tmux-quick-reference/"
---
## 1. 核心概念

tmux 是终端复用器，主要有三个层次：

Session（会话）

Window（窗口）

Pane 

一个 Session 可以有多个 Window，每个 Window 可以拆分成多个 Pane。

最重要的特点：SSH 断开后，远程 tmux Session 通常仍会继续运行。 但如果 NUC 关机或重启，会话不会自动保留。

## 2. 会话管理

|命令|功能|
|---|---|
|`tmux`|新建会话|
|`tmux new -s dev`|新建命名会话|
|`tmux ls`|查看会话|
|`tmux attach -t dev`|重新连接会话|
|`tmux kill-session -t dev`|结束指定会话|
|`tmux kill-server`|结束所有会话|

## 3. 快捷键

tmux 默认前缀键是 `Ctrl+b`。下面的 `Prefix` 均表示先按 `Ctrl+b`，松开后再按后续按键。

| 快捷键              | 功能               |
| ---------------- | ---------------- |
| `Prefix d`       | 分离会话，后台保留        |
| `Prefix c`       | 新建 Window        |
| `Prefix n` / `p` | 下一个 / 上一个 Window |
| `Prefix 0–9`     | 切换到编号窗口          |
| `Prefix %`       | 左右分屏             |
| `Prefix "`       | 上下分屏             |
| `Prefix o`       | 切换 Pane          |
| `Prefix 方向键`     | 切换相邻 Pane        |
| `Prefix z`       | 最大化 / 恢复 Pane    |
| `Prefix x`       | 关闭 Pane（确认）      |
| `Prefix [`       | 进入复制/滚动模式        |
| `Prefix ?`       | 查看所有快捷键          |

## 4. tmux 与 Neovim 的区别

|          | tmux        | Neovim              |
| -------- | ----------- | ------------------- |
| 管理对象     | 终端进程、会话     | 文件、Buffer、编辑视图      |
| 分屏       | Pane        | Window              |
| 多窗口      | tmux Window | Neovim Tab / Window |
| SSH 断线保持 | 支持          | 单独运行时通常不支持          |
| 主要用途     | 组织终端任务      | 编辑代码                |

例如你可以在 tmux 的一个 Pane 中运行 Neovim，在另一个 Pane 中运行 `colcon build`，第三个 Pane 中查看 ROS2 日志。

## 推荐的实际工作流程

```
# 1. SSH 连接 NUC
ssh wbr-nuc

# 2. 创建或恢复 tmux 会话
tmux new -As robot

# 3. 在 tmux 中打开项目
cd ~/dust-control
nvim .

# 4. 临时修改代码
# 使用 fzf-lua 搜索文件、clangd 跳转定义

# 5. 保存并退出 Neovim
# :wq

# 6. 查看 Git 修改
git diff
lazygit

# 7. 离开时分离 tmux
# Ctrl+b 然后 d
```

以后重新 SSH 连接后执行 `tmux attach -t robot`，就可以继续之前的终端工作。