---
title: Windows 小狼毫 Rime 雾凇拼音配置与个性化指南
date: 2026-10-13 16:36:19
tags:
  - Rime
  - 小狼毫
  - 雾凇拼音
  - Windows
  - 输入法
categories: 工具（Tools）
---

本文整理 Windows 下小狼毫（Weasel）+ 雾凇拼音的安装、部署、个性化配置，以及如何按应用分配中英文初始模式。

<!-- more -->

## 一、安装小狼毫并部署雾凇拼音

### 1. 安装小狼毫

前往 Rime 官网下载 Windows 版小狼毫安装包，按提示完成安装。

### 2. 下载雾凇拼音

从 GitHub 官方仓库或大陆镜像下载最新的 `full.zip`：

- GitHub：`iDvel/rime-ice`
- 镜像：南京大学镜像等

### 3. 手动部署（推荐新手）

1. 右键任务栏小狼毫图标 → **用户文件夹**。
2. 清空该文件夹内所有内容。
3. 将 `full.zip` 解压后的所有文件复制进去。
4. 右键小狼毫图标 → **重新部署**。

### 4. 命令行部署（可选）

在 **Git Bash** 中执行：

```bash
cd ~
git clone https://github.com/rime/plum.git plum
cd ~/plum
bash rime-install iDvel/rime-ice

# 双拼用户，将 double_pinyin_flypy 替换为你的方案
bash rime-install iDvel/rime-ice:others/recipes/config:schema=double_pinyin_flypy

# 可选：安装万象语法模型
bash rime-install iDvel/rime-ice:others/recipes/grammar:schema=rime_ice
```

执行完后同样需要 **重新部署**。

---

## 二、基础个性化配置

建议在用户文件夹中创建对应的 `.custom.yaml` 文件，避免更新词库时丢失个人设置。

### 1. 候选词数量

编辑 `default.custom.yaml`：

```yaml
patch:
  schema_list:
    - { schema: rime_ice }
  menu:
    page_size: 9
```

### 2. 皮肤与字体

编辑 `weasel.custom.yaml`：

```yaml
patch:
  "style/color_scheme": luna
  "style/font_face": "Microsoft YaHei"
  "style/font_point": 16
```

可以在 [Rime 皮肤在线预览站](https://pdog18.github.io/rime-soak/#/theme) 查看主题效果。

### 3. 深浅色皮肤自动切换

在 `weasel.custom.yaml` 中分别指定亮色和暗色主题：

```yaml
patch:
  "style/color_scheme": luna
  "style/color_scheme_dark": dark_temple
```

常用内置主题：

| 主题名        | 适用模式 | 说明                   |
| ------------- | -------- | ---------------------- |
| `luna`        | 亮色     | 默认亮色主题，蓝白色调 |
| `aqua`        | 亮色     | 清爽蓝色主题           |
| `dark_temple` | 暗色     | 内置深色主题           |
| `lost_temple` | 暗色     | 另一款深色主题         |

Windows 10 1809+ 支持系统深色模式检测。修改后需 **重新部署**。

### 4. 双拼支持

雾凇拼音默认支持全拼和多种双拼。部署后按 `F4`，在方案菜单中选择双拼方案即可。

### 5. 语法模型

语法模型可提升长句输入准确度。可从 [RIME-LMDG](https://github.com/amzxyz/RIME-LMDG) 下载 `.gram` 文件，放入用户文件夹，并在 `rime_ice.schema.custom.yaml` 中配置。

---

## 三、取消 Shift 切换中英文，只保留 Ctrl + Space

编辑 `default.custom.yaml`：

```yaml
patch:
  ascii_composer/switch_key:
    Shift_L: noop
    Shift_R: noop
```

保存后 **重新部署**。这样 Shift 不再切换中英文，`Ctrl + Space` 仍然可以切换 ASCII 模式。

如果还配置了 `Caps_Lock`、`Control_L` 等切换键，也可以一并设为 `noop`。

---

## 四、设置默认英文模式

### 方法一：强制每次启动重置为英文

编辑当前方案文件，例如 `rime_ice.schema.yaml`：

```yaml
switches:
  - name: ascii_mode
    reset: 1
    states: ["中文", "英文"]
```

`reset: 1` 表示默认英文，`0` 表示默认中文。

### 方法二：记忆最后一次输入状态

编辑 `default.custom.yaml`：

```yaml
patch:
  save_options:
    - ascii_mode
```

保存并重新部署后，手动切到英文，Rime 会记住这个状态。

> 注意：`save_options` 是 **全局记忆**，不是按应用独立。所有窗口共享同一个状态。

### 方法三：按应用独立设置初始模式

在 `weasel.custom.yaml` 中使用 `app_options`：

```yaml
patch:
  app_options:
    Code.exe:
      ascii_mode: true
    WindowsTerminal.exe:
      ascii_mode: true
    chrome.exe:
      ascii_mode: false
```

`app_options` 会在切换到该应用时自动应用初始状态，这才是 **按应用独立** 的配置方式。

### 方法对比

| 配置项         | 作用范围 | 行为                       |
| -------------- | -------- | -------------------------- |
| `reset`        | 方案级   | 每次启动强制重置           |
| `save_options` | 全局     | 记住上次状态，所有应用共享 |
| `app_options`  | 按应用   | 为每个应用设定独立初始状态 |

---

## 五、按应用分配初始输入模式

根据常见使用场景，可以将聊天、办公、浏览器设为默认中文，将 IDE、终端、开发工具设为默认英文。

### 分类建议

| 应用类型           | 代表应用                          | 初始模式 |
| ------------------ | --------------------------------- | -------- |
| 聊天 / 社交 / 会议 | 微信、QQ、钉钉、腾讯会议          | 中文     |
| 浏览器             | Edge、Firefox                     | 中文     |
| 邮件               | Thunderbird                       | 中文     |
| 办公套件           | Word、Excel、PowerPoint、Visio    | 中文     |
| 代码编辑器 / IDE   | VS Code、PyCharm、CLion           | 英文     |
| 终端 / 命令行      | Windows Terminal、PowerShell、CMD | 英文     |
| 记事本             | Windows 记事本                    | 中文     |
| 网盘 / 云存储      | 百度网盘、阿里云盘、夸克网盘      | 中文     |
| 远程控制           | ToDesk                            | 中文     |
| 游戏平台           | Steam、Epic、WeGame               | 中文     |
| 路径 / 开发工具    | WinSCP、Blender、GIMP             | 英文     |

### 完整 `weasel.custom.yaml` 示例

```yaml
patch:
  app_options:
    # ---------- 聊天 / 社交 / 会议：默认中文 ----------
    WeChat.exe:
      ascii_mode: false
    Weixin.exe:
      ascii_mode: false
    QQ.exe:
      ascii_mode: false
    QQNT.exe:
      ascii_mode: false
    DingTalk.exe:
      ascii_mode: false
    wemeetapp.exe:
      ascii_mode: false
    TencentMeeting.exe:
      ascii_mode: false

    # ---------- 浏览器：默认中文 ----------
    msedge.exe:
      ascii_mode: false
    firefox.exe:
      ascii_mode: false

    # ---------- 邮件：默认中文 ----------
    thunderbird.exe:
      ascii_mode: false

    # ---------- 办公套件：默认中文 ----------
    WINWORD.EXE:
      ascii_mode: false
    EXCEL.EXE:
      ascii_mode: false
    POWERPNT.EXE:
      ascii_mode: false
    VISIO.EXE:
      ascii_mode: false

    # ---------- 代码编辑器 / IDE：默认英文 ----------
    Code.exe:
      ascii_mode: true
    pycharm64.exe:
      ascii_mode: true
    clion64.exe:
      ascii_mode: true

    # ---------- 终端 / 命令行：默认英文 ----------
    WindowsTerminal.exe:
      ascii_mode: true
    pwsh.exe:
      ascii_mode: true
    powershell.exe:
      ascii_mode: true
    cmd.exe:
      ascii_mode: true

    # ---------- 文本 / 笔记：默认中文 ----------
    notepad.exe:
      ascii_mode: false

    # ---------- 网盘 / 云存储：默认中文 ----------
    BaiduNetdisk.exe:
      ascii_mode: false
    aDrive.exe:
      ascii_mode: false
    QuarkCloudDrive.exe:
      ascii_mode: false

    # ---------- 远程控制：默认中文 ----------
    ToDesk.exe:
      ascii_mode: false

    # ---------- 游戏平台：默认中文 ----------
    steam.exe:
      ascii_mode: false
    EpicGamesLauncher.exe:
      ascii_mode: false
    WeGame.exe:
      ascii_mode: false

    # ---------- 其他工具 ----------
    WinSCP.exe:
      ascii_mode: true
    blender.exe:
      ascii_mode: true
    gimp-2.10.exe:
      ascii_mode: true
```

### 注意事项

1. 若应用未生效，通常是 **进程名不匹配**。可在任务管理器 → 详细信息中查看实际进程名。
2. UWP / MSIX 应用可能无法直接通过 exe 名匹配，需使用窗口类名等更高级配置。
3. 修改任何配置文件后，都要右键小狼毫图标 → **重新部署**。
4. `app_options` 下不要重复定义，合并到同一个 `patch:` 中即可。

---

## 六、常见问题

### 1. 输入第一个字母不出现候选词

可尝试更新小狼毫目录下的 `rime.dll` 至 **1.8.5** 或更高版本。

### 2. 候选框显示异常

如果在小狼毫较新版本中遇到候选框不显示或颜色异常，建议使用 **手动复制法** 重新部署雾凇拼音。

### 3. 修改后不生效

优先检查：

- 是否执行了 **重新部署**
- YAML 缩进是否正确
- 是否重复定义了同一个节点
- 进程名是否与实际一致

---

## 七、总结

Windows 小狼毫 + 雾凇拼音的配置流程可以概括为：

1. 下载雾凇拼音并复制到用户文件夹。
2. 重新部署。
3. 通过 `.custom.yaml` 个性化候选词、皮肤、快捷键。
4. 用 `app_options` 为不同应用分配中英文初始模式。
5. 每次修改后重新部署。

这样就能实现：聊天办公默认中文，写代码和终端默认英文，皮肤跟随系统深浅色自动切换，并且摆脱 Shift 误切换的困扰。
