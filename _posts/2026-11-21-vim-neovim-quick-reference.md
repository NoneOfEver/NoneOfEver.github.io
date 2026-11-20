---
title: "Vim NeoVim速查笔记"
description: "A quick reference for Vim and Neovim concepts and commands."
lang: "zh-CN"
permalink: "/blog/vim-neovim-quick-reference/"
---
## 1. 基础概念

- Vim：基于模式的文本编辑器。
    
- Neovim：Vim 的现代化分支，支持 Lua 配置、内置 LSP 等。
    
- Buffer：内存中的文件内容，不等于窗口。
    
- Window：显示 Buffer 的视图。
    
- Tab：Window 布局的容器，不等于传统 IDE 的文件标签。
    
- Register：寄存器，用于保存复制、删除、宏等内容。
    
- LSP：语言服务器协议，提供代码跳转、诊断、补全等功能。
    

## 2. 模式切换

|按键|功能|
|---|---|
|`i`|在光标前进入插入模式|
|`a`|在光标后进入插入模式|
|`o`|在下一行插入|
|`O`|在上一行插入|
|`Esc`|返回普通模式|
|`v`|字符可视模式|
|`V`|行可视模式|
|`Ctrl+v`|块可视模式|
|`:`|命令行模式|
|`Esc Esc`|退出终端模式（自定义）|

## 3. 移动与编辑

|命令|功能|
|---|---|
|`h j k l`|左下上右|
|`w` / `b`|下一个 / 上一个单词|
|`e`|移动到单词末尾|
|`0` / `^` / `$`|行首 / 首个非空字符 / 行尾|
|`gg` / `G`|文件开头 / 末尾|
|`Ctrl+d` / `Ctrl+u`|向下 / 向上半页|
|`f<char>`|向右寻找本行字符|
|`%`|跳转匹配括号|
|`dd`|删除整行|
|`yy`|复制整行|
|`p` / `P`|在后面 / 前面粘贴|
|`u` / `Ctrl+r`|撤销 / 重做|
|`ciw`|修改当前单词|
|`diw`|删除当前单词|
|`ci"`|修改引号内内容|
|`dap`|删除当前段落|
|`.`|重复上一次修改|

Vim 的核心编辑语法：

`操作符 + 数量 + 移动范围`

例如 `d3w` 删除接下来三个单词，`c$` 修改至行尾，`yap` 复制一个段落。

## 4. 文件、Buffer、Window、Tab

|命令|功能|
|---|---|
|`:e path`|打开或准备新建文件|
|`:w`|保存|
|`:wq`|保存并退出|
|`:q!`|放弃修改并退出|
|`:ls`|查看 Buffer 列表|
|`:bnext` / `:bprev`|切换 Buffer|
|`:bd`|关闭 Buffer|
|`:split` / `:vsplit`|水平 / 垂直分屏|
|`Ctrl+w h/j/k/l`|在窗口间移动|
|`Ctrl+w q`|关闭窗口|
|`:tabnew`|新建 Tab|
|`gt` / `gT`|下一个 / 上一个 Tab|

你的自定义 Tab 快捷键：

- `Space tn`：新建 Tab
    
- `Space tc`：关闭 Tab
    
- `Space tl`：下一个 Tab
    
- `Space th`：上一个 Tab
    

## 5. 搜索、替换与复制

|命令|功能|
|---|---|
|`/pattern`|向下搜索|
|`?pattern`|向上搜索|
|`n` / `N`|下一个 / 上一个匹配|
|`*` / `#`|搜索光标下单词|
|`:%s/old/new/g`|全文件替换|
|`:%s/old/new/gc`|全文件替换，逐项确认|
|`:%y+`|全文件复制到系统剪贴板|
|`"+yy`|复制当前行到系统剪贴板|

你的配置中，普通模式下 `Esc` 会清除搜索高亮。

## 6. 宏

|命令|功能|
|---|---|
|`qa`|开始录制到寄存器 a|
|`q`|结束录制|
|`@a`|执行宏 a|
|`@@`|重复上一个宏|
|`10@a`|执行宏 a 十次|

适用于批量修改格式相似的代码。

## 7. LSP 与 clangd

|快捷键|功能|
|---|---|
|`gd`|跳转定义|
|`grr`|查找引用|
|`K`|显示符号文档|
|`Space rn`|重命名符号|
|`Space ca`|Code Action|
|`Space e`|查看诊断|
|`[d` / `]d`|上一条 / 下一条诊断|
|`:LspInfo`|查看 LSP 状态|

clangd 的重要依赖：

- `compile_commands.json`：记录每个源文件的编译参数。
    
- CMake 工程可通过 `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` 生成。
    
- 缺少编译数据库可能导致头文件路径、宏定义等解析不正确。
    

## 8. 插件与 Git 集成

|快捷键|功能|
|---|---|
|`Space ff`|fzf-lua 搜索文件|
|`Space fb`|搜索 Buffer|
|`Space fg`|全文搜索|
|`Space fs`|搜索 LSP 工作区符号|
|`]h` / `[h`|下一个 / 上一个 Git 修改块|
|`Space hp`|预览修改块|
|`Space hs`|暂存修改块|
|`Space hr`|撤销修改块的工作区更改|
|`Space hb`|查看 Git blame|

插件管理使用 `:Lazy`。

配置入口为 `~/.config/nvim/init.lua`，各模块放在 `lua/config/` 和 `lua/plugins/` 中。