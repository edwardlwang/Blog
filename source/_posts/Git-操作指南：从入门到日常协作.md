---
title: Git 操作指南：从入门到日常协作
date: 2026-09-09 16:11:15
tags: [Git, Linux]
---

> 无论是个人项目还是团队开发，Git 都是现代软件工程的基石。  
> 这份指南将带你从零开始，掌握 Git 的核心操作与常用工作流。

---

## 1. 初识 Git

Git 是一个分布式版本控制系统，由 Linus Torvalds 为 Linux 内核开发而设计。它能够高效地管理从小到大的项目，记录每一次变更，支持分支合并，让多人协作变得井然有序。

**核心概念**：

- **仓库（Repository）**：项目的完整版本数据库。
- **提交（Commit）**：一次快照，记录文件状态和提交信息。
- **分支（Branch）**：独立的工作线，用于并行开发。
- **远程（Remote）**：托管在服务器上的仓库副本（如 GitHub、GitLab）。

---

## 2. 安装与初始配置

### 安装 Git

- **Windows**：下载 [Git for Windows](https://git-scm.com/download/win) 并安装，包含 Git Bash。
- **macOS**：使用 Homebrew `brew install git` 或下载官方安装包。
- **Linux**：使用包管理器，如 `sudo apt install git`（Ubuntu/Debian）。

### 初次配置（全局）

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global core.editor "code --wait"   # 设置默认编辑器（VS Code示例）
git config --global init.defaultBranch main     # 默认主分支名改为 main
```

查看所有配置：`git config --list`

---

## 3. 基础操作：日常开发三板斧

### 3.1 初始化仓库或克隆

```bash
# 在现有目录初始化新仓库
git init

# 克隆远程仓库到本地
git clone https://github.com/用户名/仓库名.git
```

### 3.2 查看状态与变动

```bash
git status          # 查看当前工作区和暂存区状态
git diff            # 查看未暂存的修改
git diff --staged   # 查看已暂存但未提交的修改
```

### 3.3 添加与提交

```bash
git add <文件>      # 将文件添加到暂存区
git add .           # 添加当前目录所有变更
git commit -m "提交信息"   # 提交暂存区内容
git commit -am "信息"      # 跳过暂存，直接提交已跟踪文件的修改（慎用）
```

### 3.4 查看提交历史

```bash
git log             # 完整历史
git log --oneline   # 简洁一行显示
git log --graph --all --decorate  # 图形化展示分支结构
```

---

## 4. 分支管理：并行开发的基石

### 4.1 分支基本操作

```bash
git branch              # 列出本地分支
git branch <分支名>     # 创建新分支
git checkout <分支名>   # 切换分支
git switch <分支名>     # 更推荐的新命令（Git 2.23+）
git checkout -b <新分支> # 创建并切换到新分支
git branch -d <分支名>  # 删除分支（已合并）
git branch -D <分支名>  # 强制删除（未合并）
```

### 4.2 合并分支

```bash
# 先切换到目标分支（如 main）
git checkout main
git merge <要合并的分支>
```

**合并冲突**：当两个分支修改同一文件同一区域，Git 会暂停合并。需手动编辑冲突文件（`<<<<<<<` 标记区域），然后 `git add` 和 `git commit` 完成合并。

### 4.3 变基（Rebase）

```bash
git checkout feature
git rebase main   # 将 feature 的提交重新应用到 main 的最新提交之上
```

变基可使历史线性化，但**不要变基已公开的分支**。

---

## 5. 远程仓库协作

### 5.1 管理远程

```bash
git remote -v              # 查看远程仓库地址
git remote add origin <url> # 添加远程仓库（origin 为别名）
git remote set-url origin <新url>  # 修改远程地址
```

### 5.2 推送与拉取

```bash
git push origin <分支名>   # 推送本地分支到远程
git push -u origin <分支名> # 首次推送并建立跟踪关系
git pull origin <分支名>   # 拉取远程分支并合并（相当于 fetch + merge）
git fetch origin          # 仅拉取远程更新，不合并
```

### 5.3 跟踪远程分支

```bash
git branch --set-upstream-to=origin/main main   # 设置跟踪
git branch -vv            # 查看本地分支与远程分支的跟踪关系
```

---

## 6. 撤销与修正：反悔药大全

### 6.1 工作区撤销

```bash
git restore <文件>        # 丢弃工作区的修改（未暂存）
git restore --staged <文件> # 将已暂存的文件撤出暂存区（保留修改）
```

### 6.2 提交修正

```bash
git commit --amend -m "新信息"   # 修改最近一次提交信息或补漏（不增加新提交）
# 若忘记添加文件，先 git add，然后 git commit --amend --no-edit
```

### 6.3 回退版本

```bash
git reset --soft HEAD~1   # 撤销提交，保留修改在暂存区
git reset --mixed HEAD~1  # 撤销提交，保留修改在工作区（默认）
git reset --hard HEAD~1   # 彻底丢弃修改（危险！）
```

### 6.4 恢复丢失的提交

```bash
git reflog                # 查看所有 HEAD 变动记录
git reset --hard <commit-id>  # 恢复到特定提交
```

---

## 7. 暂存现场：Stash

当需要切换分支但不想提交当前未完成的工作时：

```bash
git stash               # 暂存当前修改
git stash list          # 查看暂存列表
git stash pop           # 恢复最新暂存并删除
git stash apply         # 恢复但不删除
git stash drop          # 删除指定暂存
```

---

## 8. 标签管理：标记版本

```bash
git tag                  # 列出标签
git tag v1.0.0           # 轻量标签
git tag -a v1.0.0 -m "版本说明"  # 附注标签
git push origin --tags   # 推送所有标签到远程
git tag -d v1.0.0        # 删除本地标签
git push origin :refs/tags/v1.0.0  # 删除远程标签
```

---

## 9. 效率神器：LazyGit

### 9.1 什么是 LazyGit？

LazyGit 是一个基于终端的 Git 图形界面工具（TUI），由 Jesse Duffield 于 2018 年用 Go 语言开发。它本质上是一个 Git 命令编排器与状态可视化器——并非重新实现 Git，而是调用系统 Git 命令并解析其输出，以直观的界面呈现。

**核心优势**：

- **极简交互**：无需记忆复杂 Git 命令，通过键盘快捷键即可完成 90% 的日常操作。
- **高效可视化**：实时展示分支结构、提交历史和工作区状态，告别纯文本信息过载。
- **跨平台兼容**：完美支持 Linux、macOS 和 Windows 系统。
- **强大的容错机制**：通过 Reflog 机制可以一键撤销误操作。
- **灵活的暂存功能**：支持逐行暂存，可精确选择文件中的某几行代码提交。

### 9.2 安装 LazyGit

#### macOS（Homebrew）

```bash
brew install lazygit
```

#### Windows

```bash
# 使用 Scoop
scoop bucket add extras
scoop install lazygit

# 使用 Winget
winget install --id JesseDuffield.lazygit -e

# 使用 Chocolatey
choco install lazygit
```

#### Linux

```bash
# Ubuntu/Debian
sudo apt install lazygit

# Arch Linux
sudo pacman -S lazygit

# Fedora/RHEL
sudo dnf copr enable atim/lazygit -y
sudo dnf install lazygit

# Alpine Linux
sudo apk add lazygit
```

#### 通用方式：从 GitHub Releases 下载

访问 [GitHub Releases](https://github.com/jesseduffield/lazygit/releases) 下载对应系统的二进制文件，解压后将 `lazygit` 放入 PATH 环境变量所包含的目录中。

验证安装：

```bash
lazygit --version
```

### 9.3 核心界面与导航

在任意 Git 仓库目录中打开终端，输入 `lazygit` 即可启动。

界面分为 5 个主要面板，可通过数字键 `1`-`5` 快速切换：

| 数字键 | 面板             | 用途                     |
| ------ | ---------------- | ------------------------ |
| `1`    | Status（状态）   | 查看仓库整体状态         |
| `2`    | Files（文件）    | **最常用**，处理本地改动 |
| `3`    | Branches（分支） | 管理分支                 |
| `4`    | Commits（提交）  | 查看/操作提交历史        |
| `5`    | Remotes（远程）  | 管理远程仓库             |

**通用导航快捷键**：

- `j`/`k` 或 `↑`/`↓`：上下移动光标
- `Enter`：选择项目/执行操作
- `q`：退出当前视图/程序
- `?` 或 `x`：查看当前面板所有快捷键帮助（**救命稻草！**）
- `←`/`→` 或 `[`/`]`：在不同面板间切换

### 9.4 常用操作快捷键

#### 文件操作（Files 面板，按 `2` 进入）

| 快捷键         | 操作               | 说明                     |
| -------------- | ------------------ | ------------------------ |
| `Space`        | 暂存/取消暂存文件  | 文件变绿表示已暂存       |
| `a`            | 全选/全取消暂存    | 对应 add                 |
| `c`            | 提交（Commit）     | 弹出输入框填写提交信息   |
| `A`（Shift+a） | 追加提交（Amend）  | 将新改动合并到上一次提交 |
| `d`            | 放弃单个文件的修改 | **不可逆，慎用**         |
| `D`（Shift+d） | 放弃所有本地修改   | **不可逆，慎用**         |
| `Enter`        | 进入文件详情       | 可逐行暂存代码           |

#### 分支操作（Branches 面板，按 `3` 进入）

| 快捷键         | 操作                 | 说明                     |
| -------------- | -------------------- | ------------------------ |
| `n`            | 创建新分支           | 基于当前分支创建         |
| `Space`        | 切换分支（Checkout） | 选中分支后按空格         |
| `M`（Shift+m） | 合并分支             | 将选中分支合并到当前分支 |
| `d`            | 删除分支             |                          |

#### 远程同步（全局快捷键）

| 快捷键         | 操作         | 说明               |
| -------------- | ------------ | ------------------ |
| `p`（小写）    | 拉取（Pull） | 从远程获取改动     |
| `P`（Shift+p） | 推送（Push） | 推送当前分支到远程 |

#### 高级操作

| 快捷键   | 操作                 | 说明                            |
| -------- | -------------------- | ------------------------------- |
| `r`      | 交互式变基（Rebase） | 可视化调整提交历史              |
| `z`      | 撤销（Undo）         | 通过 Reflog 撤销上一条 Git 命令 |
| `Ctrl+o` | 复制提交哈希         | 将当前提交的 SHA 复制到剪贴板   |
| `:`      | 执行 Shell 命令      | 调出命令行提示符                |

> **提示**：在任何界面按下 `?` 或 `x` 即可查看当前面板所有可用快捷键，完全不需要死记硬背。

### 9.5 典型工作流示例

**日常提交流程**：

1. 在项目目录运行 `lazygit`
2. 按 `2` 进入 Files 面板，查看修改的文件
3. 按 `Space` 暂存需要提交的文件
4. 按 `c` 输入提交信息，回车完成提交
5. 按 `P`（Shift+p）推送到远程

**拉取最新代码**：

- 按 `p`（小写）一键拉取

**创建并切换新分支**：

1. 按 `3` 进入 Branches 面板
2. 按 `n` 输入新分支名，自动创建并切换

**修改上一次提交**：

1. 暂存遗漏的文件
2. 按 `A`（Shift+a）追加到上一次提交

### 9.6 配置与个性化

LazyGit 的配置文件位于 `~/.config/lazygit/config.yml`，可自定义：

- 快捷键映射
- 默认分支名称
- 编辑器和分页器
- 界面颜色主题

示例配置：

```yaml
git:
  paging:
    colorArg: always
    pager: delta --dark
ui:
  theme:
    activeBorderColor:
      - green
      - bold
  commitLength: 80
keybinding:
  universal:
    quit: "q"
    confirm: "<enter>"
```

---

## 10. 常用工作流模型

### 10.1 Git Flow（经典）

- `main`：生产环境代码
- `develop`：开发主线
- `feature/*`：功能分支
- `release/*`：发布准备
- `hotfix/*`：紧急修复

### 10.2 GitHub Flow（简化）

- 从 `main` 拉出 `feature` 分支
- 开发完成后创建 Pull Request
- 经代码审查后合并回 `main`

### 10.3 GitLab Flow（环境分支）

- 增加 `pre-production`、`production` 等环境分支，配合 CI/CD。

---

## 11. 实用别名与技巧

配置常用命令缩写（`git config --global alias.<别名> '<命令>'`）：

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --graph --oneline --all"
```

**忽略文件**：在项目根目录创建 `.gitignore`，添加不需要版本控制的文件模式（如 `node_modules/`, `*.log`）。

---

## 12. 常见问题与解决

| 问题                                    | 解决方法                                                                        |
| --------------------------------------- | ------------------------------------------------------------------------------- |
| 提交时提示 "Please tell me who you are" | 重新设置 `user.name` 和 `user.email`                                            |
| 合并冲突后忘记解决                      | `git status` 查看冲突文件，手动修复后 `git add` 并 `git commit`                 |
| 不小心 `reset --hard` 丢了代码          | 尽快使用 `git reflog` 找回                                                      |
| 推送到远程被拒绝（非 fast-forward）     | 先 `git pull --rebase`，解决冲突后再次推送                                      |
| 想撤销上一次 `push`                     | `git revert HEAD` 后 `git push`（安全）或 `git reset` 后 `push --force`（危险） |
| LazyGit 中误操作                        | 按 `z` 撤销，或按 `?` 查看帮助                                                  |

---

## 13. 最佳实践总结

- **提交信息**：采用 `<类型>: <简短描述>` 格式（如 `feat: 添加登录功能`），内容清晰。
- **小而频繁的提交**：每次提交只做一件事，便于回滚和审查。
- **经常拉取**：保持本地分支与远程同步，减少冲突。
- **保护主干**：不要直接在 `main` 上开发，始终使用分支 + PR/MR 流程。
- **善用 .gitignore**：避免提交临时文件、依赖包、密钥等。
- **定期清理**：删除已合并的本地分支，保持仓库整洁。
- **尝试 LazyGit**：对于习惯终端操作的开发者，LazyGit 能在不离开命令行的前提下大幅提升 Git 操作效率。遇到不会的操作，记得按 `?`。

---

## 结语

Git 的学习曲线虽陡，但掌握常用命令后，你会感受到它带来的高效与安全感。  
**记住：任何你想撤销的操作，Git 几乎都有办法恢复**，前提是你了解 `reflog` 和 `reset`/`revert` 的用法。

建议在个人项目中多练手，遇到困惑时使用 `git help <命令>` 查看官方文档。  
如果觉得命令行操作繁琐，不妨试试 LazyGit——让 Git 操作变得更"懒"、更高效。  
愿你的代码永无冲突，每次 `push` 都顺顺利利！🎉
