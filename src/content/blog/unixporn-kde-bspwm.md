---
title: 'KDE桌面环境更换窗口管理器为bspwm'
publishDate: 2026-01-06
updatedDate: 2024-08-14
description: 'KDE桌面环境更换窗口管理器为bspwm'
tags:
  - Linux
language: 'Chinese'
# heroImage: { src: './thumbnail.jpg', color: '#D58388' }
---

## 运行环境
- arch系发行版（以manjaro为例）
- 桌面环境 kde plasma 6.0.5
- qt 6.7.2

## 安装组件

```sh
sudo pacman -S bspwm sxhkd picom nitrogen rofi polybar dunst
```

- bspwm: 窗口管理器
- sxhkd: 改变快捷键
- picom: 窗口效果混成器, 改变不透明度等
- dunst: 通知（可选）
- polybar: 桌面顶部的状态栏
- rofi: 应用选择菜单
- nitrogen: 壁纸更改（可选）

## 替换kwin为bspwm

为当前用户禁用 plasma-kwin_x11.service 服务避免KWin启动

```sh
systemctl --user mask plasma-kwin_x11.service
```

创建文件 `~/.config/systemd/user/plasma-custom-wm.service` 作为新的用户单元，编辑内容

```toml
[Install]
WantedBy=plasma-workspace.target

[Unit]
Description=Plasma Custom Window Manager
Before=plasma-workspace.target

[Service]
ExecStart=/usr/bin/bspwm # 你的窗口管理器的安装位置
Slice=session.slice
Restart=on-failure
```

重新扫描用户单元，确保Kwin服务plasma-kwin_x11.service已经禁用，然后启用新的plasma-custom-wm.service窗口管理器服务

```sh
systemctl --user daemon-reload
systemctl --user enable plasma-custom-wm.service
```

复制默认配置文件

```sh
cp /usr/share/doc/bspwm/examples/bspwmrc bspwm/
cp /usr/share/doc/sxhkd/examples/sxhkdrc sxhkd/
cp /etc/xdg/picom.conf picom/
cp /etc/polybar/config.ini polybar/
cp /etc/dunst/dunstrc dunst/
```

在bspwmrc启动脚本中加入自动启动应用，注意配置文件有哪些组件记得启动

```sh
sxhkd &
picom --config $HOME/.config/picom/picom.conf &
nitrogen --restore &
dunst &
polybar &
```

注销账号重新进入即可

## Reference

1. [BSPWM Installation and basic Configuration Guide](https://www.youtube.com/watch?v=XTcf8g54RuU) 
2. [arch linux wiki - KDE](https://wiki.archlinux.org/title/KDE)






