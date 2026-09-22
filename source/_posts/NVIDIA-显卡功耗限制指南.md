---
title: NVIDIA 显卡功耗限制指南
date: 2026-09-22 08:58:47
tags:
  - GPU
  - power
categories: NVIDIA
---

## 前言

NVIDIA 显卡在满载运行时功耗往往远超日常需求。一张标称 250W 的显卡，在游戏或渲染场景中可能持续运行在功耗墙边缘，带来的直接后果是：

- **机箱内温度升高**，风扇噪音加剧
- **电源负载增大**，ITX 或小型工作站可能触发保护
- **电费开支累积**，尤其对 7×24 运行的设备

提到限制显卡功耗，多数人第一反应是 MSI Afterburner（微星小飞机）等第三方软件。但 NVIDIA 驱动本身已内置了 **nvidia-smi**（NVIDIA System Management Interface），一个随驱动分发的命令行工具，可以直接读取和设置 GPU 的功耗上限，无需安装任何额外软件。

nvidia-smi 随 NVIDIA 官方驱动安装，在 Windows 的 `C:\Windows\System32\` 和 Linux 的 `/usr/bin/` 下均可直接调用。本文介绍如何仅使用该工具完成功耗限制的全流程。

---

## 一、原理：功耗墙的三层来源

在设置功耗限制之前，有必要理解 NVIDIA GPU 的功耗限制由哪些层决定。根据 NVIDIA 官方文档，GPU 的功耗上限有三个来源：

| 来源           | 说明                                                 | 用户是否可改      |
| -------------- | ---------------------------------------------------- | ----------------- |
| **VBIOS**      | 显卡出厂固件中定义的最大 TGP（Total Graphics Power） | 否（除非刷 BIOS） |
| **nvidia-smi** | 通过主机由用户设置的功耗上限                         | 是                |
| **SMBPBI**     | 通过带外通道（BMC）设置的功耗上限                    | 服务器场景使用    |

我们通过 nvidia-smi 设置的，就是第二层“用户功耗上限”。它的有效范围被限制在 VBIOS 定义的 **\[Min Power Limit, Max Power Limit\]** 区间内。也就是说，你只能把功耗**往下调**，无法超过显卡出厂允许的最大值。

设置生效后，GPU 的功耗管理单元（PMU）会在多个来源中选择**最保守的策略**来限制功耗，从而将实际功耗控制在设定的上限之内。

---

## 二、第一步：查询当前功耗信息

无论 Windows 还是 Linux，第一步都是查看当前 GPU 的功耗状态和可设置范围。

```bash
nvidia-smi -q -d POWER
```

该命令输出每个 GPU 的详细电源信息，关键字段包括：

```
Power Readings
    Power Management                  : Supported
    Power Draw                        : 85.23 W        # 当前实时功耗
    Current Power Limit               : 200.00 W       # 当前生效的功耗上限
    Requested Power Limit             : 200.00 W       # 请求的功耗上限
    Default Power Limit               : 200.00 W       # 出厂默认值
    Min Power Limit                   : 100.00 W       # 可设置的最小值
    Max Power Limit                   : 250.00 W       # 可设置的最大值
```

**关键判断**：如果输出中**没有** `Min Power Limit` 和 `Max Power Limit` 字段，或者 `Power Management` 显示为 `Not Supported`，说明当前显卡不支持通过 nvidia-smi 调整功耗。

也可以使用更简洁的查询格式：

```bash
nvidia-smi --query-gpu=index,name,power.draw,power.limit,power.min_limit,power.max_limit --format=csv
```

输出示例：

```
index, name, power.draw [W], power.limit [W], power.min_limit [W], power.max_limit [W]
0, NVIDIA GeForce RTX 4070, 85.23 W, 200.00 W, 100.00 W, 250.00 W
```

如果系统中有多张 GPU，用 `-i` 参数指定目标 GPU 的索引（从 0 开始）：

```bash
nvidia-smi -i 0 -q -d POWER
```

---

## 三、设置功耗上限

确认可调范围后，使用 `-pl`（`--power-limit` 的缩写）参数设置目标功耗值，单位瓦特。

### Linux

需要 root 权限：

```bash
sudo nvidia-smi -i 0 -pl 150
```

将 0 号 GPU 的功耗上限设为 150W。如果只有一张 GPU，可以省略 `-i 0`：

```bash
sudo nvidia-smi -pl 150
```

### Windows

需要**以管理员身份**打开 PowerShell 或命令提示符，然后执行：

```powershell
nvidia-smi -i 0 -pl 150
```

如果普通权限执行，会收到 `Insufficient Permissions` 错误。

### 验证设置是否生效

设置完成后，再次查询：

```bash
nvidia-smi -i 0 -q -d POWER
```

查看 `Current Power Limit` 是否已变为目标值。也可以在 `nvidia-smi` 的默认输出中查看 `Pwr:Usage/Cap` 字段：

```
Pwr:Usage/Cap            : 45W / 150W
```

其中 `150W` 即为新设的上限。

---

## 四、使设置持久化

nvidia-smi 的功耗设置有一个重要特性：**在某些条件下，重启或驱动重载后会恢复默认值**。NVIDIA 官方文档明确指出：“power cap adjustment must be reestablished after each new driver load”。

因此，如果希望功耗限制在每次开机后自动生效，需要额外的持久化配置。

### Linux：使用 systemd 服务

Linux 下最可靠的方式是创建一个 systemd 服务，在 GPU 驱动加载完成后执行 nvidia-smi 命令。

**1. 创建脚本文件** `/usr/local/sbin/nv-power-limit.sh`：

```bash
#!/bin/bash
# 等待 GPU 驱动就绪
sleep 5

# 启用持久模式（防止功耗设置在无应用运行时被重置）
nvidia-smi -pm 1

# 设置每张 GPU 的功耗上限（根据实际情况修改）
nvidia-smi -i 0 -pl 150
nvidia-smi -i 1 -pl 150

echo "GPU power limits applied at $(date)" >> /var/log/nv-power-limit.log
```

赋予执行权限：

```bash
sudo chmod +x /usr/local/sbin/nv-power-limit.sh
```

**2. 创建 systemd 服务文件** `/etc/systemd/system/nv-power-limit.service`：

```ini
[Unit]
Description=Set NVIDIA GPU Power Limits
After=nvidia-persistenced.service
Wants=nvidia-persistenced.service

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/nv-power-limit.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

**3. 启用服务**：

```bash
sudo systemctl daemon-reload
sudo systemctl enable nv-power-limit.service
sudo systemctl start nv-power-limit.service
```

此后每次开机，systemd 会在 NVIDIA 持久化守护进程启动后自动执行功耗设置脚本。

> `nvidia-smi -pm 1` 的作用是开启持久模式（Persistence Mode）。在持久模式下，GPU 驱动保持加载状态，功耗设置不会因为无 CUDA 应用运行而被重置。该命令在 Linux 上有效，在 Windows 上不生效。

### Windows：使用任务计划程序

Windows 下 `nvidia-smi -pm 1` 无法生效，因此需要借助任务计划程序在开机时自动执行功耗设置命令。

**1. 创建批处理文件** `set_gpu_power.bat`，放在任意固定路径下（如 `C:\Scripts\`）：

```bat
@echo off
nvidia-smi -i 0 -pl 150
```

如果有多张 GPU，逐行添加即可。

**2. 打开“任务计划程序”**，创建基本任务：

- **名称**：`NVIDIA Power Limit`
- **触发器**：选择“计算机启动时”
- **操作**：选择“启动程序”，程序或脚本填写批处理文件的完整路径（如 `C:\Scripts\set_gpu_power.bat`）
- **勾选**“使用最高权限运行”

**3. 在“条件”选项卡中**，取消勾选“只有在计算机使用交流电源时才启动此任务”（笔记本场景），并确保“如果任务失败，按以下频率重新启动”设置为合理值。

保存后，每次开机功耗限制会自动应用。

---

## 五、常见问题与注意事项

### 5.1 并非所有显卡都支持

功耗限制功能对显卡型号和驱动版本均有要求：

| 系列                          | 支持情况 |
| ----------------------------- | -------- |
| GeForce RTX 20 / 30 / 40 系列 | 支持     |
| GeForce GTX 10 系列及以下     | 部分支持 |
| Tesla / Quadro 系列           | 支持     |

如果在 `nvidia-smi -q -d POWER` 中看不到 `Min Power Limit` 和 `Max Power Limit`，说明该卡不支持软件功耗限制。

### 5.2 设置值不能超出硬件允许范围

尝试设置低于 `Min Power Limit` 或高于 `Max Power Limit` 的值时，nvidia-smi 会报错。例如：

```
Setting power limit for GPU 00000000:01:00.0 to 50.00 W
ERROR: The requested power limit is outside the supported range [100.00 W, 250.00 W]
```

应使用 `nvidia-smi -q -d POWER` 查询到的实际范围来设定。

### 5.3 功耗限制不等于性能线性下降

将功耗上限降低 20%，通常不会导致性能下降 20%。GPU 的功耗-性能曲线在接近功耗墙之前相对平缓，多数情况下 70%~80% 的功耗上限仍能维持 90% 以上的性能。具体比例因架构、工作负载和散热条件而异，建议以 5~10W 为步进逐步下调，观察实际表现。

### 5.4 功耗限制与温度限制的区别

`-pl` 设置的是**功耗上限**，而非温度上限。GPU 的温度控制有独立的机制。如果需要同时限制温度，可以使用 `--gpu-target-temp` 参数（部分型号支持）：

```bash
sudo nvidia-smi -gtt 80
```

该命令将 GPU 目标温度设为 80°C。当温度达到该值时，GPU 会主动降频。

### 5.5 恢复默认功耗

如需恢复出厂默认功耗上限，使用 `-pl` 设置为 `Default Power Limit` 的值即可。也可以在 Linux 下执行 `sudo nvidia-smi -pl $(nvidia-smi -q -d POWER | grep "Default Power Limit" | awk '{print $4}')` 自动读取并恢复默认值。

---

## 六、总结

仅使用 NVIDIA 官方自带的 nvidia-smi 工具，即可完成显卡功耗限制的全流程，无需安装 MSI Afterburner 或其他第三方软件。核心操作可以概括为三条命令：

```bash
# 查询功耗范围
nvidia-smi -q -d POWER

# 设置功耗上限（Linux 需 sudo，Windows 需管理员权限）
nvidia-smi -i 0 -pl 150

# Linux 下启用持久模式
sudo nvidia-smi -pm 1
```

配合 systemd 服务（Linux）或任务计划程序（Windows），可以让功耗限制在每次开机后自动生效，适合长期部署在服务器、工作站或 ITX 小机箱中的场景。
