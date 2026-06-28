---
title: '使用TOML配置Alacritty(>=0.13.2)'
publishDate: 2024-04-28
updatedDate: 2024-04-28
description: '配置终端模拟器Alacritty'
tags:
  - Linux
language: 'Chinese'
---

TOML是一种简单易读的配置文件语法，Alacritty在新版本已经把配置从yaml语言移到TOML了。

## 安装 Alacritty

Windows: 前往仓库的[Release页面](https://github.com/alacritty/alacritty/releases)下载.msi可执行文件

UNIX: 自行使用系统对应的包管理器命令安装，不要使用 cargo 给自己找麻烦

## 建立配置文件

Windows 路径: `%APPDATA%/alacritty/alacritty.toml`

UNIX 路径: `$HOME/.config/alacritty/alacritty.toml`

## 配置

使用vscode编辑时安装TOML Language Support获得语法高亮

```toml
#设置改变配置自动启动 [window]的需要重启
live_config_reload = true

#改变窗口设置
[window]
#窗口大小
dimensions = { columns = 135, lines = 30 }
padding = { x = 4, y = 2 }
dynamic_padding = true
#透明度
opacity = 0.9
#窗口名字
title = "Alacritty"

#字体
[font]
normal = { family = "FiraCode Nerd Font

", style = "Bold" }
bold = { family = "FiraCode Nerd Font", style = "Bold" }
italic = { family = "FiraCode Nerd Font", style = "Bold" }
size = 12.0

#颜色主题可以在github上找想要的主题，复制他的配色.
[colors.primary]
background = "#1a1b26"
foreground = "#a9b1d6"

[colors.normal]
black = "#32344a"
red = "#f7768e"
green = "#9ece6a"
yellow = "#e0af68"
blue = "#7aa2f7"
magenta = "#ad8ee6"
cyan =  "#449dab"
white = "#787c99"

[colors.bright]
black = "#444b6a"
red = "#ff7a93"
green = "#b9f27c"
yellow = "#ff9e64"
blue =  "#7da6ff"
magenta = "#bb9af7"
cyan =  "#0db9d7"
white = "#acb0d0"

#光标
[cursor]
style = { shape = "Beam", blinking = "On" }

[terminal]
#设定复制黏贴行为 默认只能复制
osc52 = "CopyPaste"
```

## Reference

1. [配置指南 Alacritty - TOML configuration file format](https://alacritty.org/config-alacritty.html)
2. [TOML语法wiki](https://toml.io/en/)
3. [Alacritty-Theme](https://github.com/alacritty/alacritty-theme)
4. [tokyo-night-alacritty-theme](https://github.com/zatchheems/tokyo-night-alacritty-theme)