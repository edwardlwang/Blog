---
title: Tmux 使用指南
date: 2026-09-07 10:45:09
tags:
  - Tmux
  - Linux
  - Shell
  - SSH
categories: Linux
---

> 会话保持、多窗格管理、远程协作的终端复用解决方案

## 一、Tmux 的价值

Tmux 是一个终端复用器（terminal multiplexer），主要解决以下三类问题：

- **会话持久化**：SSH 连接断开后，运行中的进程不受影响，重新连接可恢复原会话。
- **多窗格布局**：单个终端窗口内可分割为多个面板，同时查看日志、编辑文件、运行命令。
- **多人共享**：多个用户可同时附加（attach）到同一会话，实现实时协作调试。

终端模拟器（如 iTerm、GNOME Terminal）虽支持多标签页，但其作用域仅限于本地，无法在远程服务器上保持会话状态。Tmux 则运行于服务端，网络中断不影响任务执行。

## 二、核心概念

Tmux 采用三层架构（由大到小）：

```txt
Session（会话）          ← 一个独立的工作环境，可包含多个窗口
  └── Window（窗口）     ← 会话中的标签页，可包含多个面板
        └── Pane（面板） ← 窗口中被分割出的一个终端区域
```

| 概念    | 说明                     | 类比           |
| ------- | ------------------------ | -------------- |
| Session | 一个独立的 Tmux 工作空间 | 浏览器窗口     |
| Window  | 会话内的全屏终端         | 浏览器标签页   |
| Pane    | 窗口内划分的终端区域     | 标签页中的分屏 |

## 三、安装

各主流操作系统下的安装命令：

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install tmux

# CentOS / RHEL / Fedora
sudo yum install tmux

# macOS（Homebrew）
brew install tmux

# Arch Linux
sudo pacman -S tmux
```

安装完成后，验证版本：

```bash
tmux --version
```

建议使用 2.3 及以上版本，以获得更完善的鼠标支持及新特性。

## 四、前缀键（Prefix Key）

Tmux 的所有快捷键均以**前缀键**为起始，默认前缀键为 `Ctrl + b`。  
使用方法：按下前缀键并释放，再按下功能键。下文以 `Prefix` 代指 `Ctrl + b`。

若需查看当前所有快捷键绑定，可执行 `Prefix ?`。

## 五、会话管理

### 命令行操作

```bash
# 创建匿名会话
tmux

# 创建命名会话（推荐）
tmux new -s session_name

# 列出所有会话
tmux ls

# 附加到指定会话
tmux attach -t session_name
# 简写
tmux a -t session_name

# 终止指定会话
tmux kill-session -t session_name

# 终止所有会话（关闭 Tmux 服务）
tmux kill-server
```

### 会话快捷键

| 快捷键            | 功能                                       |
| ----------------- | ------------------------------------------ |
| `Prefix d`        | 脱离（detach）当前会话，会话及进程转入后台 |
| `Prefix s`        | 列出所有会话并切换                         |
| `Prefix $`        | 重命名当前会话                             |
| `Prefix (` 或 `)` | 切换至上一个/下一个会话                    |

## 六、窗口管理

| 快捷键       | 功能                   |
| ------------ | ---------------------- |
| `Prefix c`   | 创建新窗口             |
| `Prefix ,`   | 重命名当前窗口         |
| `Prefix n`   | 切换至下一个窗口       |
| `Prefix p`   | 切换至上一个窗口       |
| `Prefix l`   | 切换至最近选中的窗口   |
| `Prefix w`   | 列出所有窗口           |
| `Prefix &`   | 关闭当前窗口（需确认） |
| `Prefix 0-9` | 切换至指定编号的窗口   |

## 七、面板管理

面板（Pane）是窗口内分割出的独立终端区域。

| 快捷键               | 功能                          |
| -------------------- | ----------------------------- |
| `Prefix %`           | 垂直分割（左右分屏）          |
| `Prefix "`           | 水平分割（上下分屏）          |
| `Prefix 方向键`      | 切换至相邻面板                |
| `Prefix o`           | 切换至下一个面板              |
| `Prefix q`           | 显示面板编号                  |
| `Prefix x`           | 关闭当前面板（需确认）        |
| `Prefix z`           | 将当前面板最大化 / 恢复原尺寸 |
| `Prefix 空格`        | 循环切换面板布局              |
| `Prefix Ctrl+方向键` | 调整面板大小（步长 1）        |

也可直接在面板内执行 `exit` 或按 `Ctrl + d` 关闭面板。

## 八、滚屏与复制模式

默认情况下，Tmux 不响应鼠标滚轮，需进入**复制模式（Copy Mode）** 查看历史输出。

```bash
Prefix [          # 进入复制模式
方向键 或 j/k     # 逐行滚动
PageUp / PageDown # 翻页
q                 # 退出复制模式
```

若在配置文件中开启鼠标支持（见后文），则直接使用滚轮滚动，并可通过鼠标点击切换面板。

## 九、配置文件

Tmux 的配置文件通常位于 `~/.tmux.conf`（新版亦支持 `~/.config/tmux/tmux.conf`）。

### 1. 启用鼠标支持

```bash
# ~/.tmux.conf
set -g mouse on
```

启用后，鼠标可用于切换面板、调整面板大小以及滚动历史输出。

重新加载配置：

```bash
tmux source-file ~/.tmux.conf
```

或在 Tmux 内按 `Prefix :`，输入 `source-file ~/.tmux.conf` 并回车。

### 2. 修改前缀键（可选）

常用替代方案为 `Ctrl + a`，以避免与系统或编辑器快捷键冲突：

```bash
# ~/.tmux.conf
unbind C-b
set -g prefix C-a
bind C-a send-prefix
```

### 3. 状态栏自定义

```bash
set -g status-bg colour235
set -g status-fg colour136
set -g status-interval 5
```

### 4. 终端类型设置

```bash
set -g default-terminal "tmux-256color"
```

### 完整配置示例

```bash
# ~/.tmux.conf

# 鼠标
set -g mouse on

# 前缀键改为 Ctrl+a
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# 状态栏样式
set -g status-bg colour235
set -g status-fg colour136

# 终端类型
set -g default-terminal "screen-256color"

# 历史记录行数（默认 2000）
set -g history-limit 10000
```

## 十、插件管理器（TPM）

Tmux Plugin Manager（TPM）用于管理插件。

### 安装 TPM

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

### 配置 TPM

在 `~/.tmux.conf` 末尾添加：

```bash
# 插件列表
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'christoomey/vim-tmux-navigator'   # Vim 风格面板导航

# 初始化 TPM（必须置于末行）
run '~/.tmux/plugins/tpm/tpm'
```

### TPM 快捷键

| 快捷键               | 功能                 |
| -------------------- | -------------------- |
| `Prefix I`（大写 i） | 安装配置中列出的插件 |
| `Prefix U`           | 更新所有插件         |
| `Prefix Alt+u`       | 移除未列出的插件     |

### 常用插件推荐

- **tmux-plugins/tpm**：插件管理器本身
- **tmux-plugins/tmux-sensible**：一组基础优化配置
- **christoomey/vim-tmux-navigator**：支持 `Ctrl+h/j/k/l` 在面板间导航
- **tmux-plugins/tmux-resurrect**：保存并恢复会话状态

## 十一、进阶用法

### 脚本化创建工作区

可通过 Shell 脚本自动化创建多窗格布局：

```bash
#!/bin/bash
tmux new-session -d -s dev -x 200 -y 50
tmux send-keys -t dev "nvim ." Enter
tmux split-window -h -t dev
tmux send-keys -t dev "npm run dev" Enter
tmux split-window -v -t dev
tmux send-keys -t dev "git status" Enter
tmux attach -t dev
```

### 脚本幂等性处理

创建会话前检查是否已存在，避免重复创建：

```bash
tmux has-session -t mysession 2>/dev/null || tmux new-session -d -s mysession
```

### 向指定面板发送命令

```bash
tmux send-keys -t mysession:0.1 "ls -la" Enter
```

其中 `0.1` 表示第 0 个窗口的第 1 个面板（从 0 开始编号）。

## 十二、常见问题

### 1. SSH 断开后会话是否保留？

会话及其中运行的所有进程均会保留。重新 SSH 连接后，执行 `tmux attach` 即可恢复。

### 2. 脚本中提示 "no server running"

需先启动 Tmux 服务：`tmux start-server`，或在执行其他命令前先创建一个分离会话。

### 3. 创建分离会话时面板尺寸为 0×0

创建时显式指定尺寸：

```bash
tmux new-session -d -s mysession -x 200 -y 50
```

### 4. 终端颜色显示异常

检查 `TERM` 环境变量，设置为 `screen-256color`：

```bash
export TERM=screen-256color
```

或在配置文件中设置：

```bash
set -g default-terminal "screen-256color"
```

### 5. 快捷键冲突

使用 `show-options -g` 查看当前绑定，通过 `unbind` 解除冲突项。

## 十三、总结

Tmux 的核心功能可概括为：会话持久化、多窗格管理、远程恢复。  
推荐的工作流程：

1. 安装 Tmux。
2. 记忆默认前缀键 `Ctrl+b`（或自定义为 `Ctrl+a`）。
3. 创建命名会话：`tmux new -s name`。
4. 使用 `Prefix d` 脱离会话，`tmux attach` 恢复。
5. 配置 `~/.tmux.conf` 开启鼠标支持及其他优化。
6. 按需安装 TPM 插件。

将配置文件纳入版本控制，可在多台机器上快速复现一致的工作环境。
