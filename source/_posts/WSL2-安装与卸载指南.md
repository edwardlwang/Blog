---
title: WSL2 安装与卸载指南
date: 2026-10-26 15:45:53
tags:
  - WSL2
categories: 工具（Tools）
---

本文整理了 Windows 10 / 11 上 WSL2（适用于 Linux 的 Windows 子系统）的安装与卸载流程，包含一键安装、手动安装、仅移除发行版以及彻底卸载 WSL 功能的完整步骤，并附上关键命令与注意事项。

<!-- more -->

## 🐧 安装 WSL2

### 先决条件

在开始之前，请确认你的电脑满足以下条件：

- **Windows 版本**：Windows 10 版本 2004 或更高（Build 19041+），或 Windows 11。
- **硬件虚拟化**：CPU 需支持并已在 BIOS 中启用虚拟化（Intel VT-x / AMD-V）。
- **系统架构**：64 位系统。

### 一键安装（推荐，适用于 Windows 10 2004+ / Windows 11）

以**管理员身份**打开 PowerShell，执行以下命令，然后重启电脑：

```powershell
wsl --install
```

该命令会自动启用运行 WSL 所需的 Windows 功能，并安装默认的 Linux 发行版（通常是 Ubuntu）。重启后，Ubuntu 会自动启动并完成初始化，按提示设置用户名和密码即可。

> **提示**：如果 `wsl --install` 因网络问题卡在 0.0%，可以尝试使用 `wsl --install --web-download` 命令，让系统先下载再安装。

### 手动安装（适用于旧版 Windows 或需要自定义时）

如果你的系统版本较低，或希望手动控制安装过程，可以按以下步骤操作。

#### 1. 启用必要的 Windows 功能

以管理员身份打开 PowerShell，执行：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

完成后**重启计算机**。WSL2 依赖“虚拟机平台”功能，因此必须启用。

#### 2. 将 WSL2 设为默认版本

重启后，在 PowerShell 中执行：

```powershell
wsl --set-default-version 2
```

Windows 11 通常已默认使用 WSL2，但 Windows 10 用户需要执行此命令以确保新安装的发行版使用 WSL2。

#### 3. 安装 Linux 发行版

- **查看可用发行版**：`wsl --list --online`
- **安装指定发行版**（例如 Ubuntu）：`wsl --install -d Ubuntu`

也可以直接从 Microsoft Store 搜索并安装你喜欢的 Linux 发行版。

### 验证安装

安装完成后，在 PowerShell 中运行以下命令，确认 WSL 版本和运行状态：

```powershell
wsl --status
wsl --list --verbose
```

如果 `VERSION` 列显示为 `2`，说明 WSL2 已正常启用。

## 🗑️ 卸载 WSL2

卸载 WSL2 通常有两种需求：**仅移除某个 Linux 发行版**，或**彻底卸载 WSL 功能本身**。请根据你的目标选择对应步骤。

### 仅卸载某个 Linux 发行版（保留 WSL 功能）

如果只是想删除某个发行版，而保留 WSL 以便继续使用其他发行版，执行以下步骤：

#### 1. 注销（删除）发行版及其数据

以管理员身份打开 PowerShell：

```powershell
wsl --list --verbose          # 查看已安装的发行版名称
wsl --unregister <发行版名称>   # 例如：wsl --unregister Ubuntu
```

`wsl --unregister` 会**永久删除**该发行版及其所有文件、设置和用户数据，请务必提前备份重要文件。

#### 2. （可选）通过设置卸载

你也可以在“设置 > 应用 > 已安装的应用”中找到对应的发行版（如 Ubuntu），选择“卸载”。但请注意，通过“设置”卸载有时仅移除了启动器，数据目录可能仍然残留，因此更推荐使用 `wsl --unregister` 来确保数据被清除。

### 彻底卸载 WSL 功能

如果你不再需要使用 WSL，希望从系统中完全移除它，请按以下顺序操作：

#### 1. 注销所有发行版

```powershell
wsl --list --all
```

对列表中每个发行版执行 `wsl --unregister <名称>`，确保所有 Linux 环境的数据被清除。

#### 2. 关闭 WSL 运行环境

```powershell
wsl --shutdown
```

此命令会停止所有正在运行的 WSL 实例和虚拟机。

#### 3. 卸载 WSL 应用（如果系统版本支持）

较新的 Windows 版本将 WSL 作为一个独立应用分发。可以尝试执行：

```powershell
wsl --uninstall
```

如果该命令不可用，可前往“设置 > 应用 > 已安装的应用”，查找“Windows Subsystem for Linux”或类似条目并卸载。

#### 4. 禁用 Windows 功能

打开“启用或关闭 Windows 功能”（可在开始菜单搜索 `optionalfeatures`），取消勾选以下两项，然后**重启电脑**：

- **适用于 Linux 的 Windows 子系统**
- **虚拟机平台**（Virtual Machine Platform）

> **注意**：卸载 WSL 本身**不会自动禁用“虚拟机平台”**，如果你确定不再需要运行任何依赖 Hyper-V 的服务（如 Docker Desktop 的 WSL2 后端、Windows 沙盒等），才需要手动取消勾选。否则，可以保留“虚拟机平台”不变。

#### 5. 清理残留文件（可选但推荐）

重启后，可以手动检查并删除以下可能残留的文件夹：

- `%LocalAppData%\Packages`（查找与发行版相关的文件夹，如 `CanonicalGroupLimited.Ubuntu*`）
- `C:\ProgramData\Microsoft\Windows\Start Menu\Programs` 中与 WSL/Linux 相关的快捷方式
- 如果曾安装过 “Ubuntu Pro for WSL” 等附加应用，也可一并清理其数据目录和注册表项。

#### 6. 验证卸载

重新打开 PowerShell，执行 `wsl --status`。如果提示“系统找不到指定路径”或类似信息，通常说明 WSL 已从系统中移除。

## ⚠️ 关键注意事项

- **数据备份**：`wsl --unregister` 会不可逆地删除发行版内的所有数据。在执行任何卸载操作前，请务必将需要保留的文件从 Linux 环境复制到 Windows 盘符（如通过 `/mnt/c/...` 路径）。
- **“虚拟机平台”的去留**：如果你只是重装 WSL 或暂时不用，建议**保留“虚拟机平台”功能**，以免影响其他依赖虚拟化的软件。仅在完全确定不再需要任何虚拟化功能时，才将其禁用。
- **Windows 10 与 Windows 11 的差异**：Windows 11 对 WSL2 的支持更为集成，一键安装和 `wsl --uninstall` 通常可直接使用；Windows 10（尤其是较旧版本）可能需要更多手动步骤。如果 `wsl --uninstall` 报错，优先通过“设置”和“Windows 功能”来完成卸载。
