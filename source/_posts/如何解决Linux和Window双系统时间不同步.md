---
title: 如何解决Linux和Window双系统时间不同步
date: 2026-01-17 13:43:03
tags: [Windows, Linux, Ubuntu, time]
categories: Linux
---

当你在电脑上同时装上 Windows 和 Linux（Ubuntu）双系统时，可能会出现两个系统时间不同步问题。因此，本文将指导如何解决这个问题。

### 原因

首先我们要了解一个概念 BIOS 时间，它由主板上的实时时钟（RTC）维护。

时间不同步的原因是 Windows 系统与 Linux 系统对 BIOS 时间的理解不同。

在 Windows 上，BIOS 时间就是当地时间，因此 Windows 会直接显示该时间；而 Linux 则将 BIOS 时间当成 UTC 时间（格林尼治标准时间），因此 Linux 会将在 BIOS 时间基础上加上你本地的时差。

当你启动一个系统时，该系统会对 BIOS 时间进行修改，从而导致另一个系统的时间出现错误。

### 解决方法

#### 修改 Linux 配置（适用于以 Windows 为主系统）

这种方式是让 Linux 系统来适配 Windows 的本地时间规则。

- 在 Linux 中打开终端，执行以下命令
  ```bash
  timedatectl set-local-rtc 1 --adjust-system-clock
  ```
- 验证配置是否生效
  ```bash
  timedatectl
  ```
  输出中如果出现 RTC in local TZ: yes，就说明配置成功。

### 修改 Windows 配置（适用于以 Linux 为主系统）

这种方式让 Windows 适配 Linux 的 UTC 规则，需要修改 Windows 注册表。

- 按下 Win + R，输入 regedit 打开注册表编辑器。
- 定位到路径
  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`
- 在右侧空白处右键 → 新建 → DWORD (32 位) 值，命名为 RealTimeIsUniversal。
- 双击该值，将数值数据改为 1，基数选择 十六进制，点击确定。
- 重启 Windows 系统，时间即可和 Linux 同步。

PS：如果你是新手，可以优先选择方法一。
