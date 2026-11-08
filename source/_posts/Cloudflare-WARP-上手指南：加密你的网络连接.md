---
title: Cloudflare WARP 上手指南：加密你的网络连接
date: 2026-11-08 12:58:14
tags:
  - VPN
  - cloudflare
categories: 工具（Tools）
---

Cloudflare WARP 经常被误读成“免费 VPN”，但如果带着这个预期去用，大概率会困惑于它“为什么不能选国家”“为什么 IP 没变”。这篇指南先把 WARP 到底做什么、不做什么讲清楚，再分别给出 Windows、Linux、macOS 的安装步骤，最后附上连接验证方法和几条实用的排错思路。

---

### WARP 是什么，以及它明确不做什么

WARP 是 Cloudflare 在 1.1.1.1 应用里提供的一项**加密传输服务**。打开之后，设备发出的流量会经过加密，直接送到离你最近的 Cloudflare 边缘节点，再由 Cloudflare 的全球网络转发出去；DNS 查询则强制走 1.1.1.1 解析器。

它做的事很具体：加密你和 Cloudflare 节点之间的流量，让本地网络或 ISP 看不到你访问了什么；加密 DNS 查询，规避常见的 DNS 污染与劫持；在某些线路上改善路由，这也是 WARP 最初的卖点——让移动网络访问更快。

但它**不提供匿名性**。Cloudflare 官方文档的原话是 “WARP does not provide anonymity”。对你访问的网站来说，出口 IP 仍然是 Cloudflare 的地址，你并不是匿名的；WARP 也没有“选择服务器国家”的功能，不能用来解锁地区限制的流媒体；它更不是抗审查工具，Cloudflare 从未把它设计成翻墙软件。

理解这一点，后面的安装和使用就会顺畅很多：**WARP 解决的是“加密传输 + DNS 保护”的问题，而不是“换 IP”或“绕过封锁”的问题。**

#### WARP 的几种模式

客户端里不只一个“开/关”开关，不同模式路由的流量范围差别很大：

| 模式                                        | 行为                                                |
| ------------------------------------------- | --------------------------------------------------- |
| **DNS only**（原 1.1.1.1）                  | 只把 DNS 查询加密送到 1.1.1.1，**不隧道设备流量**   |
| **Traffic and DNS**（原 1.1.1.1 with WARP） | 全部设备流量加密走 WARP，DNS 也在隧道内处理         |
| **Traffic only**                            | 全部流量走 WARP，但 DNS 仍由操作系统管理            |
| **Local proxy**（桌面端）                   | 只有手动配置了代理的应用走 WARP，其余流量走普通连接 |

日常使用选默认的 **Traffic and DNS** 即可。如果只是想解决 DNS 被污染导致的“网站打不开”，**DNS only 模式往往更轻、更稳**——它只加密 DNS，不建立完整隧道，开销小得多。

WARP 底层目前优先使用 MASQUE 协议（基于 HTTP/3，走 UDP 443），WireGuard 作为备用选项。MASQUE 的端口特征和普通网页流量相近，这也是 Cloudflare 逐步用它替代 WireGuard 的原因之一。

---

### Windows 安装

**下载入口**：官方下载地址是 `https://downloads.cloudflareclient.com/v1/download/windows/ga`，也可以从 `one.one.one.one` 进入。

1. 下载 Windows 安装程序（`.exe`），运行。
2. 按照安装向导完成安装。安装完成后 WARP 会自动启动，在系统托盘（菜单栏）出现 Cloudflare 图标。
3. 点击 **Next**，接受隐私政策。
4. 打开开关，WARP 即进入连接状态。

安装完成后，点击托盘图标 → **齿轮图标 → Preferences**，可以在 **Connection** 选项卡里调整模式或 DNS 协议。如果你只想加密 DNS，把模式从 WARP 切换为 **1.1.1.1** 即可。

> **提醒**：首次连接时 Windows 可能会弹出防火墙授权窗口，允许即可。如果连接后没有任何效果，优先检查托盘图标是否真的显示“Connected”，而不是只看界面文字。

---

### Linux 安装（APT / YUM）

Cloudflare 为 Linux 提供了官方软件源，支持 Ubuntu、Debian、RHEL/CentOS、Fedora 等发行版。下面步骤中的公钥更新提醒来自官方说明：**如果你在 2025-09-12 之前安装过 Cloudflare 公钥，需要重新执行“添加 GPG 密钥”步骤**，否则仓库会在 2025-12-04 之后失效。

#### Ubuntu / Debian

```bash
# 添加 Cloudflare GPG 密钥
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

# 添加 APT 仓库
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

# 安装
sudo apt-get update && sudo apt-get install cloudflare-warp
```

Ubuntu 当前支持的版本包括 Resolute（26.04）、Noble（24.04）、Jammy（22.04）；Debian 包括 Trixie（13）和 Bookworm（12）。（以上内容来自你提供的官方安装说明。）

#### RHEL / CentOS / Fedora

RHEL 9+ 和 Fedora 需要先确保 **EPEL 仓库**可用，因为托盘的托盘图标和门户窗口依赖 EPEL 提供的组件。

```bash
# 如果之前安装过旧公钥，先移除并重新导入
sudo rpm -e 'gpg-pubkey(4fa1c3ba-61abda35)' && sudo rpm --import https://pkg.cloudflareclient.com/pubkey.gpg

# 添加仓库
curl -fsSl https://pkg.cloudflareclient.com/cloudflare-warp-ascii.repo | sudo tee /etc/yum.repos.d/cloudflare-warp.repo

# RHEL/CentOS 使用 yum；Fedora 使用 dnf
sudo yum update && sudo yum install cloudflare-warp
# 或
sudo dnf update && sudo dnf install cloudflare-warp
```

#### Linux 安装后的连接步骤

Linux 客户端是命令行工具 `warp-cli`。安装完成后，先注册，再连接：

```bash
warp-cli registration new   # 首次使用需要注册
warp-cli connect            # 连接
warp-cli status             # 查看状态
```

如果要切换模式，可以用 `warp-cli mode --help` 查看可用选项。

---

### macOS 安装

**下载入口**：`https://downloads.cloudflareclient.com/v1/download/macos/ga`。

1. 下载 `.pkg` 安装包，双击运行。
2. 按照向导完成安装。WARP 会自动启动，并在菜单栏出现 Cloudflare 图标。
3. 点击 **Next**，接受隐私政策。
4. 打开开关启用 WARP。

首次连接时 macOS 会要求允许系统扩展和 VPN 配置，按照系统提示授权即可。安装后同样可以通过菜单栏图标 → **齿轮图标 → Preferences → Connection** 来切换模式或 DNS 协议。

---

### 怎么确认 WARP 真的生效了

客户端显示“Connected”不代表隧道一定通了。最直接的方法是访问 Cloudflare 的追踪页面：

```
https://www.cloudflare.com/cdn-cgi/trace
```

在返回的文本中查找 **`warp=on`**。如果显示 `warp=on`，说明流量已经经过 WARP；如果仍是 `warp=off` 或页面无法访问，则隧道没有真正建立。

也可以对比开启前后的 IP 归属地：开启 WARP 后，IP 通常会变成 Cloudflare 的地址（常见归属地为境外节点），但**这不等于你“换了一个国家”**——出口仍然是离你最近的 Cloudflare 节点，只是路径被 Cloudflare 接管了。

---

### 几条实用的排错思路

**先换协议，再看其他。** Windows 和 macOS 的 Preferences → Advanced → Connection options 中可以在 Auto、MASQUE、WireGuard 之间切换。每次切换后重新连接，并用上面的 `warp=on` 验证，而不是只相信客户端的状态显示。

**如果只是某些网站打不开，先试 DNS only。** 很多“打不开”问题的根因是 DNS 污染，而不是流量本身被拦截。DNS only 模式只加密 DNS、不隧道全部流量，开销小，在只想解决解析问题的场景下通常更稳定。

**别把 WARP 当成唯一的跨境方案。** 在中国的可用性因省份、运营商、时段而异，公开反馈长期分裂。较新的 MASQUE 协议（UDP 443）在 2024—2025 年间出现过短暂“可用窗口”，但并没有稳定持续。如果某个时段连不上，换协议、换网络（比如从宽带切到蜂窝）是成本最低的尝试，但如果目标是稳定访问境外服务，WARP 不适合作为唯一依赖。

**一句话收束**：WARP 是一个轻量、免费的加密传输层——它把你的流量和 DNS 保护起来，交给 Cloudflare 的网络转发，但它不负责“让你看起来在另一个国家”。带着这个预期去用，它其实很好用；带着 VPN 的预期去用，就很容易失望。
