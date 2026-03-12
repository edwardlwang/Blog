---
title: 解决类原生安卓（如LineageOS）显示WIFI网络受限问题
date: 2026-01-22 19:19:51
tags: [Anddoid, LineageOS, WIFI, ADB]
categories: Android
---

如题，为了解决该问题，有以下几个步骤：

## 1、下载安卓调试桥工具（ADB）

下载地址 *https://developer.android.google.cn/tools/releases/platform-tools?hl=zh-cn*

## 2、adb调试

- 先删除默认的验证地址：
  ```powershell
  adb shell settings delete global captive_portal_http_url
  adb shell settings delete global captive_portal_https_url
  ```
- 再修改新的验证服务器地址：
  ```powershell
  adb shell settings put global captive_portal_https_url https://connect.rom.miui.com/generate_204
  adb shell settings put global captive_portal_http_url http://connect.rom.miui.com/generate_204
  ```
- 更新时钟同步服务器地址：

  ```powershell
  adb shell settings put global ntp_server ntp1.aliyun.com
  ```

- 当这些命令执行完毕之后，打开飞行模式，在关闭飞行模式，或重启手机后叉号或叹号即可去除。
