---
title: 'What I Use?'
publishDate: 2026-09-06
updatedDate: 2026-09-06
description: '整合 macos 和 windows11 所需配置的清单'
tags:
    - dotfiles
language: 'Chinese'
---

https://github.com/akatsuki033896/dotfiles

## Introduction

工作使用windows11，私人电脑是win11+macbook m1 air。macbook作为私人电脑主要用于娱乐和私人范围的轻度开发，选择的标准是**能开源就开源、尽量跨平台能够保持使用习惯一致。**

## Fonts

终端和所有编辑器、IDE均使用 [JetBrainsMono Nerd Font](https://www.jetbrains.com/lp/mono/)，目前在探索有没有新的好看的字体...

## General Tools for MAC

- [Keka](https://www.keka.io/en/)：压缩工具，支持 `rar` 和 `7z`，解决系统只支持解压 `zip` 和自带压缩存在潜在和windows系统互传解压错误的问题。
- [IINA](https://github.com/iina/iina)：开源媒体播放器，用于临时播放音频和媒体文件，不在 ffmpeg 耻辱柱上可放心使用。插件很多。
- [Raycast](https://www.raycast.com)：集成很多工具，实际上用得最多的是 `Command+Space` 启动应用程序，因为mac自带的启动台在你持有iPhone的时候会显示手机装的程序，很难用。免费版就很够用了，支持安装第三方扩展。
- [Ice](https://github.com/jordanbaird/Ice)：开源菜单栏管理器，只拿来默认隐藏菜单栏。
- [AeroSpace](https://github.com/nikitabobko/AeroSpace)：开源平铺式窗口管理器，屏幕小的其实很难有这个需求。
- [iStatMenus](https://bjango.com/mac/istatmenus/)：系统监控工具，支持在菜单栏显示。
- [Better Display](https://betterdisplaymac.com)：屏幕显示优化工具，实际上我只用来开启HiDPI，如果只有这个需求可以考虑开源脚本 [xzhih/one-key-hidpi](https://github.com/xzhih/one-key-hidpi) 。

---

### General Tools for Cross Platform

- [PicGo](https://github.com/Molunerfinn/PicGo)：开源图床，迁移到这边是因为以前用的 sm.ms 停止服务而且要钱了。
- [LocalSend](https://github.com/localsend/localsend)：开源局域网传输工具。
- [v2rayN](https://github.com/2dust/v2rayN)：开源代理工具。

## General Applications

- [Notion](https://www.notion.com/)：类 Markdown 的知识管理软件，主要拿来规划工作进度和快速记录工作相关内容设计知识。第三方模版生态很强。缺点是虽然说是云同步但实际上东西一多很占本地磁盘容量，不如 Obsidian 自己远程同步。
- [Obsidian](https://obsidian.md/)：完全使用 Markdown 的知识管理软件。用于编辑 Markdown 和构建知识系统，虽然只是大致把他丢进文件夹顶多用 tag 分类一下但是 Simple is the Best.

## Developer Tools & Applications

- [brew](https://brew.sh/)：mac最常见的包管理器。
- [Chocolatey](https://chocolatey.org/)：windows包管理器，通常还可以使用的是 [Scoop](https://scoop.sh/) 和windows11自带的winget，这三个基本能装的都装了。
- [xcode command line tools](https://developer.apple.com/documentation/xcode/installing-the-command-line-tools)：xcode是mac的开发环境，开发环境例如C++可能依赖此工具。
- [VSCode](https://code.visualstudio.com/)：代码编辑器，能忍着不用的是神人。
- [Qt](https://www.qt.io/development/download)：建议使用online installer安装，brew安装默认不装Qt Creator的，另一个理由是online installer安装带有GUI的Maintainance Tools，一般安装的时候只会选择对应环境，例如对于一个vs项目选择指定的Qt对应MSVC版本。
- [CLion](https://www.jetbrains.com/zh-cn/clion/)：C++跨平台开发IDE。
- [Ghostty](https://github.com/ghostty-org/ghostty)：基于zig的开源终端模拟器。还是喜欢自带标签页的，懒得折腾tmux，反正公司电脑是windows，tmux根本就不好用，统一都用标签页了。
- [QGIS](https://qgis.org)：开源的地理信息系统软件。
- [ZCode](https://zcode.z.ai/)：智谱集成GLM-5.3的ADE(Agentic Development Environment)。
- [Docker-Desktop](https://www.docker.com/products/docker-desktop/)：docker容器管理工具。
- [neovim](https://neovim.io/)：基于vim改进的开源代码编辑器，运行在终端，用来运行单个文件或就地改东西。避免过度配置陷阱，经常需要思考自己到底需不需要neovim。
- [MiniConda](https://docs.anaconda.net.cn/miniconda/install/)：Python虚拟环境，轻量的Anaconda。
- [PyCharm](https://www.jetbrains.com/zh-cn/pycharm/)：Python IDE，使用理由是支持批处理文件作为解释器用于开发QGIS插件，魔改批处理文件也可以解决这个问题。

## CLI Tools

- [starship](https://github.com/starship/starship)：命令行美化工具，支持多种shell，主要用于确认环境。
- [ffmpeg](https://www.ffmpeg.org/)：开源音频处理。有很多单纯套壳GUI就发布的软件，不建议使用。
- [git](https://git-scm.com/)：版本管理工具。

## Windows Specific Tools

- [Visual Studio 2022](https://visualstudio.microsoft.com/)：宇宙第一IDE，开发C++跨平台应用产品的windows版本。
- [Windows Terminal](https://github.com/microsoft/terminal)：Microsoft自己的终端模拟器。
- [Power Toys](https://learn.microsoft.com/zh-cn/windows/powertoys/)：Microsoft推出的工具，支持 `win+alt+Space` 唤出类似 Raycast 的面板运行应用，也支持快速修改环境变量（主要使用这两个功能而已...）
- [Tortoise SVN](https://tortoisesvn.net/index.zh.html)：开源的版本控制工具GUI客户端，公司指定。
- [MAS Script](https://massgrave.dev)：开源Office全家桶下载和破解脚本。
- [NanaZip](https://github.com/M2Team/Nanazip)：开源windows平台解压缩软件，原来使用BandiZip但我不想看广告。
- [Geek Uninstaller](https://geekuninstaller.com/)：windows平台卸载器，支持清除注册表。
- [Win11 Debloat](https://github.com/raphire/win11debloat)：windows11全面优化脚本，卸载不必要的内容

## Reference

- https://arthals.ink/blog/initialize-mac