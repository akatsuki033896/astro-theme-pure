---
title: 'PyQGIS二次开发 part.0: QGIS插件开发环境配置'
publishDate: 2026-08-03
updatedDate: 2026-09-03
description: '使用Plugin Builder 3和PyCharm开发QGIS插件'
tags:
  - Qt
  - Python
  - QGIS
language: 'Chinese'
---

## Dependencies

<Aside type='caution'>使用最新版本的QGIS可以无视本节内容，两个插件安装最新版即可。</Aside>

新版的Plugin Builder 3需要强制输入github仓库链接，使用旧版，同时因为使用旧版QGIS3.16.1，内置的Python是3.7，不支持新版Plugin Reloader的 `:=` 语法，使用旧版。

QGIS插件安装有三种方式，分别是直接像vscode一样软件内搜索插件安装，通过zip安装，直接复制源代码到插件路径安装。QGIS内只能安装最新版的插件，旧版插件的安装包可以在插件市场下载。

- QGIS(3.16.1)：默认安装路径为 `C:\Program Files\QGIS 3.16`
- Plugin Builder 3 插件(3.2.1)：创建插件工程
- Plugin Reloader 插件(0.7.9)：动态加载插件，而不需要重启QGIS

## 使用Plugin Builder 3创建工程

1. 填写信息，插件模板窗口选择Tool button with dialog
2. Select Output Directory: 插件工程源码生成的位置，点击Generate后弹出Readme，注意区分**源码位置**和**指定的存放插件的地址**，**在QGIS运行插件之前需要把源码文件夹整个复制到指定的存放插件的地址**

```
（源码）Your plugin PluginDemo was created in:
    E:/PluginDemo\plugin_demo

（指定的存放插件的地址）Your QGIS plugin directory is located at:
    C:/Users/Administrator/AppData/Roaming/QGIS/QGIS3/profiles/default/python/plugins
```

生成的插件工程如下

![](https://picgocloud.com/m/b62a64f2-fac3-49db-b4cb-6f48039d758c.png)

## PyCharm配置开发环境

QGIS3.16默认的安装路径为 `C:\Program Files\QGIS 3.16`，这里以 `$QGIS` 指代QGIS安装路径

### 解释器

添加 `$QGIS\bin\python-qgis.bat`，也可能根据版本不同，例如长期发行版本的批处理文件名为 `python-qgis-ltr.bat`，vscode不支持设置 `.bat` 为解释器，建议使用PyCharm

### 外部工具

#### Pyrcc5 / Pyuic5

pyuic5 和 pyrcc5 是 PyQt5 开发中常用的两个命令行工具，用于将设计好的界面和资源文件转换成 Python 代码

1. 将 `$QGIS\apps\Python37\Scripts` 添加进环境变量。
2. Pycharm设置→外部工具中新建外部工具，名字为Pyrcc5
    - 程序：选择QIGS安装路径中的 `$QGIS\apps\Python37\Scripts\pyrcc5.bat` ，部分版本可能是 `.exe` 程序
    - 实参：执行的命令，填写 `$FileName$ -o $FileNameWithoutExtension$.py`
    - 工作目录：$FileDir$ ，否则会提示找不到文件
3. 项目会生成 `resource.qrc` ,右键→外部工具选择Pyrcc5可以执行把`resource.qrc`转为`resource.py`的操作，否则QGIS会提示无法载入插件，缺乏`resource.py`。根据官方文档，每次更改`resource.qrc` 之后都要重新生成`resource.py`

Pyuic5类似Pyrcc5，在QGIS安装目录中的存储位置也是Scripts文件夹。

#### QtDesigner

QGIS 自带的 Qt Designer 和普通的 Qt Designer 本质上是同一个工具，但 QGIS 自带的版本（通常通过 OSGeo4W 或独立安装包随附）在环境变量配置和内置专属 GIS 组件上有关键区别：

- 集成 QGIS 专属控件（Custom Widgets）：工具箱中直接包含面向 GIS 开发的自定义控件，例如图层选择框（`QgsMapLayerComboBox`）、字段选择框（`QgsFieldComboBox`）、文件选择组件（`QgsFileWidget`）和坐标选择等，可以直接拖拽出带有 QGIS 特色交互功能的控件，大幅减少编写额外绑定代码的工作量。
- 开箱即用匹配运行环境：自带的启动脚本（如 Windows 下的 `qgis-designer.bat` 快捷方式）已经自动配置好了 Qt 插件路径和环境变量，能直接加载 QGIS 的动态库，避免找不到自定义插件或版本冲突。

检测designer是否支持pyqgis相关属性，可以看帮助→关于插件，查看有没有qgis相关插件，如果不使用qgis的designer会导致ui保存的时候丢失qgis特有属性，例如`QgsFileWidget` 等

![](https://picgocloud.com/m/66add003-2497-4816-86b1-8814e09d1627.png)

1. 添加外部工具
2. 配置参数：
    - 程序：`$QGIS\bin\qgis-designer.bat`
    - 实参：执行的命令，填写 $FileName$
    - 工作目录：`$FileDir$
3. 右键 `.ui` 文件打开外部工具QtDesigner

## 插件发布

将插件打包成压缩包即可，例如插件源码位于Plugin文件夹，将该文件夹打包成Plugins.zip即可。

## Reference

- https://blog.csdn.net/TJLCY/article/details/123680217
- 插件市场：https://plugins.qgis.org/plugins/


