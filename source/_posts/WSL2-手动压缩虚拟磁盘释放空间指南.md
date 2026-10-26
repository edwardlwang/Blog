---
title: WSL2 手动压缩虚拟磁盘释放空间指南
date: 2026-10-26 09:47:12
tags:
  - WSL2
  - Windows
  - Linux
  - disk compression
  - 磁盘压缩
categories: 运维（O&M）
---

WSL2 的虚拟磁盘文件 `ext4.vhdx` 只会动态扩展，**不会在删除 Linux 内部文件后自动收缩**。因此，如果你在 WSL 中删除了大量文件，Windows 侧的磁盘空间可能并不会立即释放，需要手动压缩虚拟磁盘。

<!-- more -->

## 为什么需要两步操作？

如果直接在 Linux 里删除文件，WSL2 只是将这些数据块标记为“可用”，但并不会通知底层的 Windows 虚拟磁盘。`fstrim` 的作用就是告诉虚拟磁盘哪些数据块已不再使用，为后续的物理压缩做好准备。

手动压缩的核心流程是：

1. 在 WSL 内部标记空闲空间；
2. 彻底关闭 WSL；
3. 从 Windows 侧压缩 `ext4.vhdx` 文件。

---

## 操作步骤

### 步骤一：在 WSL 内部标记空闲空间

打开 WSL 终端，例如 Ubuntu，执行：

```bash
sudo fstrim -av
```

如果看到类似下面的输出，说明标记成功：

```text
/: 12.3 GiB (13209260032 bytes) trimmed
```

如果提示不支持，可以尝试用 `dd` 命令写入零来填充空闲空间。这种方式效率较低，仅作备选。

### 步骤二：彻底关闭 WSL

这一步非常关键。DiskPart 无法操作正在运行的虚拟磁盘，因此需要先在 Windows 的 PowerShell 或 CMD 中执行：

```powershell
wsl --shutdown
```

为确保所有发行版都已停止，可以运行：

```powershell
wsl -l -v
```

确认状态列均为 `Stopped`。

### 步骤三：定位 `ext4.vhdx` 文件

WSL 发行版的虚拟磁盘通常位于：

```text
%LOCALAPPDATA%\Packages\<发行版包名>\LocalState\ext4.vhdx
```

也可以通过以下 PowerShell 命令快速找到它：

```powershell
(Get-ChildItem "HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss" | Where-Object { $_.GetValue("DistributionName") -eq '你的发行版名称' }).GetValue("BasePath") + "\ext4.vhdx"
```

请将 `'你的发行版名称'` 替换为实际名称，例如 `'Ubuntu'`。

### 步骤四：执行压缩

以下两种方法任选其一。

#### 方法 A：使用 DiskPart

这种方法通用，无需 Hyper-V。

以**管理员身份**打开 PowerShell 或 CMD，进入 `diskpart` 交互模式，依次执行：

```text
diskpart
DISKPART> select vdisk file="你的ext4.vhdx完整路径"
DISKPART> attach vdisk readonly
DISKPART> compact vdisk
DISKPART> detach vdisk
DISKPART> exit
```

注意将 `file=` 后的路径替换为你实际的 `ext4.vhdx` 路径。

`attach vdisk readonly` 会以只读模式挂载虚拟磁盘，可防止操作过程中意外写入数据。

#### 方法 B：使用 `Optimize-VHD`

如果你已启用 Hyper-V 功能，可以以**管理员身份**打开 PowerShell，运行：

```powershell
Optimize-VHD -Path "你的ext4.vhdx完整路径" -Mode Full
```

`-Mode Full` 会执行完整的压缩操作。

如果未启用 Hyper-V，使用方法 A 更方便。

---

## 关键细节：稀疏属性 Sparse

部分 WSL 安装会将 `ext4.vhdx` 设置为**稀疏文件**。这可能导致 `compact vdisk` 或 `Optimize-VHD` 无法有效压缩，甚至直接失败。

如果你在压缩后发现文件大小几乎没变，或者命令报错，可以按以下步骤处理。

### 1. 解除稀疏属性

在管理员 CMD 中执行：

```cmd
fsutil sparse setflag "你的ext4.vhdx完整路径" 0
```

### 2. 确认已清除

```cmd
fsutil sparse queryflag "你的ext4.vhdx完整路径"
```

应返回“该文件未设置稀疏”。

### 3. 重新执行压缩

回到步骤四，重新使用 DiskPart 或 `Optimize-VHD` 压缩虚拟磁盘。

### 4. 可选：压缩后恢复稀疏属性

```cmd
fsutil sparse setflag "你的ext4.vhdx完整路径" 1
```

不恢复也可以正常使用，只是文件会表现为普通文件，后续写入会直接占用物理空间。

---

## 常见问题与备选方案

### 压缩后空间没释放？

请检查 `ext4.vhdx` 是否启用了**稀疏属性**。这是压缩失效最常见的原因。

### 权限不足或文件只读？

如果提示“拒绝访问”，可能是文件权限或只读属性问题。可以尝试：

- 在文件属性中为当前用户添加“完全控制”权限；
- 检查并取消“只读”勾选。

若问题依旧，可考虑**导出再导入**发行版：

```powershell
wsl --export 发行版名称 备份文件.tar
wsl --import 新发行版名称 安装目录 备份文件.tar
```

这会创建一个全新的、紧凑的 VHDX。但操作前务必**备份数据**。

### 其他辅助工具

社区也有一些封装好的脚本或工具，例如 `wslcompact`，可以自动化上述流程，适合希望简化操作的用户。其本质原理与手动步骤一致。

---

## 压缩后验证

压缩完成后，重新启动 WSL，然后在 Windows 中检查 `ext4.vhdx` 的文件大小，应该已经显著减小。

同时，也可以在 WSL 内执行：

```bash
df -h /
```

确认 Linux 文件系统的可用空间未受影响。

---

## 总结

WSL2 虚拟磁盘压缩的关键是：

1. 用 `sudo fstrim -av` 标记空闲块；
2. 用 `wsl --shutdown` 彻底关闭 WSL；
3. 用 DiskPart 或 `Optimize-VHD` 压缩 `ext4.vhdx`；
4. 如果压缩无效，检查并解除稀疏属性后重试。

操作前建议备份重要数据，避免意外情况导致 WSL 环境受损。
