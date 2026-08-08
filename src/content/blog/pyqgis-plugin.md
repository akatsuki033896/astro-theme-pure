---
title: 'QGIS二次开发: QGIS插件开发环境配置'
publishDate: 2026-08-03
updatedDate: 2026-08-03
description: '使用Plugin Builder 3创建你的第一个QGIS插件项目'
tags:
  - Qt
  - Python
language: 'Chinese'
---

## Dependencies

<Aside type='caution'>使用最新版本的QGIS可以无视本节内容，两个插件安装最新版即可。</Aside>

新版的Plugin Builder 3需要强制输入github仓库链接，使用旧版，同时因为使用旧版QGIS3.16.1，内置的Python是3.7，不支持新版Plugin Reloader的 `:=` 语法，使用旧版。

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

#### Pyrcc5

pycharm设置→外部工具中新建外部工具，名字为Pyrcc5

- 程序：选择QIGS安装路径中的 `$QGIS\apps\Python37\Scripts\pyrcc5.bat` ，部分版本可能是 `.exe` 程序
- 实参：执行的命令，填写 `$FileName$ -o $FileNameWithoutExtension$.py`

项目会生成 `resource.qrc` ,右键→外部工具选择Pyrcc5可以执行把`resource.qrc`转为`resource.py`的操作，否则QGIS会提示无法载入插件，缺乏`resource.py`

根据官方文档，每次更改`resource.qrc` 之后都要重新生成`resource.py`

#### QtDesigner

按照上面的方法新建外部工具为QtDesigner，添加 `$QGIS\apps\Qt5\bin\designer.exe`，之后对项目中的`.ui`文件右键外部工具，即可使用QtDesigner

## Reference

- https://blog.csdn.net/TJLCY/article/details/123680217
- 插件市场：https://plugins.qgis.org/plugins/


