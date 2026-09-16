---
title: 如何解决Linux和Windows双系统时间不同步
date: 2026-01-17 13:43:03
tags:
  - Windows
  - Linux
  - Ubuntu
  - time
categories: Linux
---

当你在同一台电脑上安装 Windows 和 Linux（如 Ubuntu）双系统后，很可能会遇到一个恼人的问题：**两个系统显示的时间不一致**，通常相差 8 小时（如果你在中国时区）。这篇文章将详细解释原因，并提供两种可靠的解决方法，你可以根据自己的使用习惯选择最适合的方案。

---

## 问题现象

- 在 Windows 下看到的时间是正确的，切换到 Linux 后发现时间快了 8 小时（或慢 8 小时）。
- 或者反过来，Linux 时间正确，Windows 时间错误。
- 每次校正后，一旦切换到另一个系统，时间又会被打乱。

---

## 为什么会这样？

这源于两个操作系统对 **主板硬件时钟（RTC，Real-Time Clock，即 BIOS 时间）** 的解读方式不同：

- **Windows 默认**：将硬件时钟视为 **本地时间（Local Time）**，即直接显示硬件时钟的值，不加任何时区偏移。
- **Linux 默认**：将硬件时钟视为 **UTC 时间（协调世界时）**，然后根据系统设置的时区（如 Asia/Shanghai，即 UTC+8）加上偏移量来显示本地时间。

举个例子（假设你在中国，时区 UTC+8）：

- 硬件时钟存储为 `2026-01-17 12:00:00`
- **Windows** 直接显示为 `2026-01-17 12:00:00`
- **Linux** 将其视为 UTC，加上 8 小时后显示为 `2026-01-17 20:00:00`

当你启动 Windows 时，它可能会将硬件时钟调整为当前本地时间；而当你启动 Linux 时，它又会将硬件时钟调整为 UTC 时间。于是每次切换系统，硬件时钟都被改写，导致另一个系统的时间错乱。

---

## 解决方法

要解决这个问题，有两种思路：

1. **让 Linux 把硬件时钟当作本地时间**（推荐 Windows 为主系统的用户）
2. **让 Windows 把硬件时钟当作 UTC 时间**（推荐 Linux 为主系统的用户）

你可以根据自己的日常主力系统来选择。如果你是新手，更推荐方法一，因为操作更简单、风险更低。

---

### 方法一：修改 Linux 配置（让 Linux 适配 Windows 的本地时间规则）

此方法适用于 **你更常用 Windows**，希望 Linux 的时间与 Windows 保持一致。

#### 操作步骤

1. **在 Linux 中打开终端**（Ctrl+Alt+T）。

2. **执行以下命令**：
   ```bash
   sudo timedatectl set-local-rtc 1 --adjust-system-clock
   ```

- `set-local-rtc 1`：告诉 Linux 将硬件时钟视为本地时间。
- `--adjust-system-clock`：根据硬件时钟的值自动调整系统时间，避免时间跳跃。

3. **验证配置是否生效**：

   ```bash
   timedatectl
   ```

   查看输出中是否有 `RTC in local TZ: yes`。如果有，表示配置成功。

4. **重启 Linux**（可选），然后检查时间是否正确。之后切换到 Windows，时间应该也会保持一致。

#### 注意事项

- 如果你以后想恢复为 Linux 默认的 UTC 模式，只需执行：
  ```bash
  sudo timedatectl set-local-rtc 0 --adjust-system-clock
  ```
- 此方法仅修改 Linux 的配置，对 Windows 无任何影响，非常安全。

---

### 方法二：修改 Windows 注册表（让 Windows 适配 Linux 的 UTC 规则）

此方法适用于 **你更常用 Linux**，希望 Windows 的时间与 Linux 保持一致。

#### 操作步骤

1. **在 Windows 中按 `Win + R`**，输入 `regedit`，回车打开注册表编辑器。

2. **定位到以下路径**：

   ```
   HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
   ```

3. **在右侧空白处右键** → **新建** → **DWORD (32 位) 值**，命名为 `RealTimeIsUniversal`。

4. **双击新建的值**，将 **数值数据** 改为 `1`，**基数** 选择 **十六进制**（或十进制均可，1 就是 1），点击 **确定**。

5. **重启 Windows**，时间即可与 Linux 同步。

#### 验证方法

重启后，检查 Windows 时间是否正确。你也可以在 Linux 中执行 `timedatectl` 确认硬件时钟仍为 UTC 模式（`RTC in local TZ: no`）。

#### 注意事项

- **修改注册表有一定风险**，建议操作前备份注册表（文件 → 导出）。
- 如果你使用的是 Windows 10/11 家庭版，同样支持此方法。
- 如果需要撤销更改，只需删除 `RealTimeIsUniversal` 这个值或将其改为 `0`，然后重启即可。

---

## 如何选择？

| 你的主力系统                     | 推荐方案               | 理由                                                 |
| -------------------------------- | ---------------------- | ---------------------------------------------------- |
| **Windows 为主**（偶尔用 Linux） | 方法一（修改 Linux）   | 操作简单，无风险，不影响 Windows                     |
| **Linux 为主**（偶尔用 Windows） | 方法二（修改 Windows） | 让 Windows 跟随 Linux 的 UTC 规则，更符合 Linux 习惯 |
| **不常用双系统，偶尔切换**       | 方法一                 | 更安全，易于恢复                                     |

---

## 常见问题

**Q：为什么我改了之后时间还是不对？**
A：请检查两边的时区设置是否正确。Windows 中右键右下角时间 → 调整日期/时间，确保时区正确；Linux 中执行 `timedatectl list-timezones` 查看并设置 `sudo timedatectl set-timezone Asia/Shanghai`。

**Q：修改注册表后 Windows 无法激活或出现其他问题？**
A：`RealTimeIsUniversal` 是官方支持的注册表项，不会影响激活或系统稳定性。若担心，可事先创建系统还原点。

**Q：我两个系统都用了很久，有没有一键修复工具？**
A：Linux 端可以使用 `hwclock` 命令手动调整，但更推荐上述标准方法。没有通用的第三方工具，因为这是系统底层行为差异。

---

## 总结

时间不同步的根本原因是 Windows 和 Linux 对硬件时钟的“世界观”不同。通过选择上述任一方法，你可以让两个系统和谐共处。**我个人建议新手优先使用方法一**（修改 Linux），因为它操作门槛低、安全可靠；如果你是一个 Linux 重度用户，则方法二能让你保持 Linux 的 UTC 准则，更加纯粹。

完成配置后，你的双系统时间问题将永久解决，不再需要每次切换都手动校时。祝你使用愉快！🎉
