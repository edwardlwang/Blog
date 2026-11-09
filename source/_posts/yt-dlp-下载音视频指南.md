---
title: yt-dlp 下载音视频指南
date: 2026-11-09 13:50:09
tags:
  - yt-dlp
  - FFmpeg
  - Youtube
categories: 工具（Tools）
---

`yt-dlp` 是目前最好用的命令行音视频下载工具之一。它继承自 `youtube-dl`，但更新更快、格式选择更灵活，配合 `FFmpeg` 可以完成音频提取、封面嵌入、字幕合并等操作。

本文整理一套我常用的命令：**下载视频最佳画质、只下载音频最佳音质、嵌入封面、指定保存位置**，以及播放列表、字幕、代理和配置文件等常见用法。

> 仅用于下载你有权保存的内容，请遵守平台条款和当地法律。

## 安装 yt-dlp 与 FFmpeg

`yt-dlp` 负责下载，`FFmpeg` 负责合并音视频、提取音频、转换封面等后处理。两者都建议安装。

### Windows

```powershell
winget install yt-dlp.yt-dlp
winget install Gyan.FFmpeg
```

也可以使用 `scoop` 或 `choco`：

```powershell
scoop install yt-dlp ffmpeg
```

### macOS

```bash
brew install yt-dlp ffmpeg
```

### Linux

推荐用 `pipx` 安装，避免污染系统 Python：

```bash
pipx install yt-dlp
sudo apt install ffmpeg
```

安装后检查：

```bash
yt-dlp --version
ffmpeg -version
```

如果提示找不到 `ffmpeg`，需要把 FFmpeg 的 `bin` 目录加入系统 `PATH`。

## 先看看有哪些格式

下载前可以先查看目标页面提供的格式：

```bash
yt-dlp -F "视频URL"
```

输出里常见几类：

- `video only`：只有视频轨，通常分辨率高
- `audio only`：只有音频轨，适合提取音频
- `video+audio`：音视频合在一起的低分辨率流

`yt-dlp` 的格式选择里：

- `bv`：最佳纯视频流
- `ba`：最佳纯音频流
- `bv*+ba/b`：最佳视频 + 最佳音频，若没有分离流则回退到最佳合并流
- `-f "ba"`：只选最佳音频

## 下载视频：保留最佳画质

最通用的最佳画质命令：

```bash
yt-dlp \
  -f "bv*+ba/b" \
  --merge-output-format mkv \
  --embed-metadata \
  -P "$HOME/Videos" \
  -o "%(title)s [%(id)s].%(ext)s" \
  "视频URL"
```

参数说明：

- `-f "bv*+ba/b"`：最佳视频 + 最佳音频，没有分离流时回退到最佳合并流
- `--merge-output-format mkv`：把音视频合并进 MKV 容器。MKV 对编码兼容性好，通常只是封装，不重新编码
- `--embed-metadata`：写入标题、作者等元数据
- `-P "$HOME/Videos"`：指定保存目录
- `-o "%(title)s [%(id)s].%(ext)s"`：文件名模板，附带视频 ID，避免重名

Windows 下可以把保存目录换成：

```bash
-P "D:\Videos"
```

如果只想要 1080p 以内：

```bash
yt-dlp \
  -f "bv*[height<=1080]+ba/b[height<=1080]" \
  --merge-output-format mkv \
  -P "$HOME/Videos" \
  -o "%(title)s [%(id)s].%(ext)s" \
  "视频URL"
```

如果你明确需要 MP4，可以把 `--merge-output-format mkv` 改成 `mp4`。但要注意：MP4 对某些编码组合兼容性不如 MKV，`yt-dlp` 默认只做封装，不会为了容器兼容而重新编码。如果封装失败，优先保留 MKV，而不是强行转码损失画质。

## 只下载音频：最佳音质 + 嵌入封面 + 指定位置

这是本文最核心的一条命令：

```bash
yt-dlp \
  -x \
  -f "ba" \
  --audio-format best \
  --embed-thumbnail \
  --convert-thumbnails jpg \
  --embed-metadata \
  -P "$HOME/Music" \
  -o "%(title)s.%(ext)s" \
  "视频URL"
```

Windows 示例：

```bash
yt-dlp -x -f "ba" --audio-format best --embed-thumbnail --convert-thumbnails jpg --embed-metadata -P "D:\Music" -o "%(title)s.%(ext)s" "视频URL"
```

关键参数解释：

- `-x`：只提取音频
- `-f "ba"`：选择最佳纯音频流
- `--audio-format best`：**不转码**，保留源站提供的原始音频编码，通常是 Opus 或 M4A/AAC
- `--embed-thumbnail`：把封面嵌入音频文件
- `--convert-thumbnails jpg`：先把封面转成 JPEG，兼容性更好
- `--embed-metadata`：写入标题、作者等元数据
- `-P`：指定保存目录
- `-o`：文件名模板

如果 URL 带有播放列表参数，但你只想下载当前单个视频，可以加上：

```bash
--no-playlist
```

如果你希望文件名更安全，避免特殊字符，可以加：

```bash
--windows-filenames
```

或者更激进地限制为 ASCII 文件名：

```bash
--restrict-filenames
```

## 关于“最佳音质”和转码

想保住最佳音质，核心原则是：**能保留原始音频流，就不要转码**。

`--audio-format best` 的意思就是保留源格式，不二次编码。很多网站提供的是 Opus 或 AAC，这些已经是有损格式，但它们是源站直接提供的音频流。保留它们，至少不会在本地再损失一次。

如果你强制转成 MP3：

```bash
--audio-format mp3 --audio-quality 0
```

即使 `--audio-quality 0` 表示“尽量高质量”，它仍然是**有损转有损**的二次编码，音质不会超过源文件。

那 FLAC 呢？FLAC 本身是**无损压缩格式**，但“把 Opus/AAC 转成 FLAC”并不会把丢失的细节变回来。FLAC 只是忠实地保存了那个已经有损的源。文件会变大，音质不会提升。所以：

- 想保留最佳音质：用 `--audio-format best`
- 目标设备只认 MP3：用 `--audio-format mp3 --audio-quality 0`，接受一次有损转码
- 目标设备只认 FLAC：可以用 `--audio-format flac`，但要知道它不会让有损源变成真无损

## 下载播放列表或频道

下载播放列表前 20 个音频，并按上传者和播放列表分目录：

```bash
yt-dlp \
  -x \
  -f "ba" \
  --audio-format best \
  --embed-thumbnail \
  --convert-thumbnails jpg \
  --playlist-items 1-20 \
  --download-archive archive.txt \
  -P "$HOME/Music" \
  -o "%(uploader)s/%(playlist_title)s/%(playlist_index)02d - %(title)s.%(ext)s" \
  "播放列表URL"
```

说明：

- `--playlist-items 1-20`：只下载第 1 到第 20 个
- `--download-archive archive.txt`：记录已下载过的视频 ID，下次跳过，避免重复下载
- `%(playlist_index)02d`：序号补零，例如 `01`、`02`
- `%(uploader)s`、`%(playlist_title)s`：按上传者和播放列表名自动建目录

如果只想下载单个视频而不是整个播放列表：

```bash
--no-playlist
```

## 下载字幕并嵌入视频

下载中英文字幕，转换成 SRT 并嵌入 MKV：

```bash
yt-dlp \
  -f "bv*+ba/b" \
  --merge-output-format mkv \
  --write-subs \
  --write-auto-subs \
  --sub-langs "zh-Hans,zh-CN,en" \
  --convert-subs srt \
  --embed-subs \
  -P "$HOME/Videos" \
  -o "%(title)s [%(id)s].%(ext)s" \
  "视频URL"
```

如果只想下载字幕文件、不嵌入视频，去掉 `--embed-subs` 即可。

## 代理、Cookies 和限速

需要代理时：

```bash
yt-dlp --proxy "http://127.0.0.1:7890" "视频URL"
```

SOCKS5 代理：

```bash
yt-dlp --proxy "socks5://127.0.0.1:1080" "视频URL"
```

需要登录才能访问的内容，可以从浏览器读取 Cookies：

```bash
yt-dlp --cookies-from-browser chrome "视频URL"
```

也可以使用导出的 cookies 文件：

```bash
yt-dlp --cookies cookies.txt "视频URL"
```

限速下载：

```bash
yt-dlp -r 5M "视频URL"
```

增加分片并发，改善部分站点的下载速度：

```bash
yt-dlp -N 4 "视频URL"
```

## 用配置文件保存常用参数

如果每次都要输入一长串音频参数，可以写进配置文件。

Linux/macOS 配置文件位置：

```text
~/.config/yt-dlp/config
```

Windows 配置文件位置：

```text
%APPDATA%\yt-dlp\config.txt
```

例如写入以下内容，每行一个完整参数：

```text
--extract-audio
--format=ba
--audio-format=best
--embed-thumbnail
--convert-thumbnails=jpg
--embed-metadata
--output=/home/你的用户名/Music/%(title)s.%(ext)s
```

Windows 可以把最后一行改成：

```text
--output=D:\Music\%(title)s.%(ext)s
```

之后直接运行：

```bash
yt-dlp "视频URL"
```

就会按配置只下载最佳音质音频，并嵌入封面。

## 常见问题

**提示 `ffmpeg not found`**

安装 FFmpeg，并确认 `ffmpeg -version` 能在终端中正常运行。Windows 需要把 FFmpeg 的 `bin` 目录加入 `PATH`。

**封面嵌入失败**

部分容器或格式对封面支持不完整。先尝试加上 `--convert-thumbnails jpg`。如果仍然失败，可以安装 `mutagen`：

```bash
pip install mutagen
```

或者安装 `AtomicParsley`，它对 M4A/MP4 封面嵌入很有帮助。

**下载 403 或要求登录**

先更新 `yt-dlp`，再尝试 `--cookies-from-browser` 或 `--cookies cookies.txt`。很多站点会频繁调整访问策略，保持工具更新很重要。

**文件名出现非法字符**

加上 `--windows-filenames`。如果希望文件名完全限制为 ASCII，可以使用 `--restrict-filenames`。

**如何更新 yt-dlp**

二进制或官方版本：

```bash
yt-dlp -U
```

如果是 `pipx` 安装：

```bash
pipx upgrade yt-dlp
```

## 小结

最常用的三套命令可以记成：

只下载音频最佳音质、嵌入封面、指定目录：

```bash
yt-dlp -x -f "ba" --audio-format best --embed-thumbnail --convert-thumbnails jpg --embed-metadata -P "保存目录" -o "%(title)s.%(ext)s" "视频URL"
```

下载视频最佳画质：

```bash
yt-dlp -f "bv*+ba/b" --merge-output-format mkv --embed-metadata -P "保存目录" -o "%(title)s [%(id)s].%(ext)s" "视频URL"
```

下载播放列表并去重：

```bash
yt-dlp -x -f "ba" --audio-format best --embed-thumbnail --convert-thumbnails jpg --download-archive archive.txt -P "保存目录" -o "%(uploader)s/%(playlist_title)s/%(playlist_index)02d - %(title)s.%(ext)s" "播放列表URL"
```

核心原则只有一句：**想保住最佳音质，就保留原始音频流，不要做不必要的转码。**
