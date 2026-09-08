---
title: "PyQGIS二次开发 part.1: 基本QGIS操作"
publishDate: 2026-09-08
updatedDate: 2026-09-08
description: 介绍PyQGIS和如何使用PyQGIS操作QGIS特性
tags:
  - QGIS
language: Chinese
---

<p>
    <img src="https://img.shields.io/badge/QGIS-3%20%26%204-green?logo=qgis)" style="display:inline-block;">
    <img src="https://img.shields.io/badge/Qt-2CDE85?logo=Qt&logoColor=fff" style="display:inline-block;">
    <img src="https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=fff" style="display:inline-block;">
</p>

本文假设读者拥有 C++ 的 Qt 经验或 PyQt 经验，聚焦于介绍 QGIS 特性。

# What is PyQGIS?

~~不是在玩 What is Paledusk 的梗~~

**PyQGIS 是 QGIS 桌面程序的 Python 绑定，不是独立的 GIS 库。**`qgis.core` 里的 `QgsVectorLayer` 等类就是 C++ 类的包装（SIP 绑定），文档、枚举值、内存管理，规则都来自 C++ 侧。

## PyQGIS 的主要包

| 包           | 处理内容                                 | 用途          |
| ----------- | ------------------------------------ | ----------- |
| `qgis.core` | 数据、图层、要素、工程、处理算法                     | 任何 GIS 数据操作 |
| `qgis.gui`  | 界面控件（地图画布、图层下拉框、文件选择框）               | UI          |
| `qgis.PyQt` | 内嵌的 PyQt5（QtCore / QtWidgets / uic…） | 界面基础设施      |

## PyQt 特性

PyQt 是在Python上用 Qt 框架 来创建GUI的跨平台工具包。下列 PyQt 示例代码类似 C++ 的 Qt，描绘了一个确认弹窗。

```python
box = QtWidgets.QMessageBox(self)
box.setWindowTitle(u'删除业务区域')
box.setText(u'该业务区域有{}个图幅，是否删除？'.format(count))
confirm = box.addButton(u'确认', QtWidgets.QMessageBox.YesRole)   # 返回按钮对象
box.addButton(u'取消', QtWidgets.QMessageBox.NoRole)
box.exec_()                                   # 模态阻塞，等用户选择
if box.clickedButton() is confirm:            # 用按钮对象判断点了哪个
    ...
```

PyQt 和 Qt 该有的特性一致，仅介绍最重要的特性。

### 对象树

- PyQt 有类似 Qt 具有父子对象机制，像 C++ 的 Qt 一样 widget / action 挂到 parent 后由 parent 负责释放。
- PyQGIS 的图层调用 `addMapLayer` 进工程后由 `QgsProject` 持有。Python 侧只是包装对象，C++ 对象被销毁后 Python 引用还在，但调用就会崩溃。

### UI操作

所有 UI 操作必须在主线程。PyQGIS 的界面调用应该在 Qt 主线程里。

## PyQGIS API 和 C++ API

PyQGIS 实际上是通过 SIP 绑定了 C++ API 的封装实现的，它们的类名、方法名和核心功能几乎完全相同。SIP 是一个把 C++ 库绑定到 Python 的工具，就可以直接在 Python 中调用 C++ 的类。这样就可以具有 Python 胶水语言的便利性的情况下调用 C++ 对地图处理的功能。

# 插件结构

这里的插件结构指的是QGIS 的 Plugin Builder 3 生成的插件。QGIS 插件实际为一个含 `classFactory()` 的包，默认放进 `~\AppData\QGIS\QGIS3\profiles\default\python\plugins` 路径，也可以修改。

- 插件存放路径可以通过修改 `QGIS_PLUGINPATH` 环境变量来修改。在 QGIS 的**设置** > **选项** > **系统**中，于**环境**区域勾选“使用自定义变量”，添加变量名为 `QGIS_PLUGINPATH`，值设为你的自定义绝对路径，重启软件即可。

QGIS 所识别的 `__init__.py` 里的插件入口：

```python
def classFactory(iface):
    from .map_protect import mapProtect
    return mapProtect(iface)
```

插件的生命周期固定为四个方法：

```python
class mapProtect:
    def __init__(self, iface):
        self.iface = iface                       # 保存全局入口
        self.actions = []                        # 记录创建的 QAction，unload 时逐个清理
        self.first_start = None                  # 惰性创建对话框的标志

    def initGui(self):
        """QGIS 加载插件时调用：创建菜单项/工具栏按钮。"""
        icon_path = ':/plugins/map_protect/icon.png'   # :/ 前缀 = resources.py 里编译的资源
        self.add_action(icon_path, text='Map Protect',
                        callback=self.run, parent=self.iface.mainWindow())
        self.first_start = True

    def unload(self):
        """禁用插件时调用：把菜单项/按钮全部移除。"""
        for action in self.actions:
            self.iface.removePluginMenu(self.tr(u'&mapProtect'), action)
            self.iface.removeToolBarIcon(action)

    def run(self):
        """点击按钮时调用。"""
        if self.first_start:                     # 对话框只创建一次
            self.first_start = False
            self.dlg = mapProtectDialog()
        self.dlg.show()
        result = self.dlg.exec_()                # exec_() 进入 Qt 事件循环，模态阻塞
        if result:                               # OK 返回 1，取消返回 0
            pass
```

## 模块导入

```python
# GIS 类：从 qgis.core / qgis.gui 按名导入
from qgis.core import QgsFeature, QgsProject, QgsVectorLayer
from qgis.gui import QgsFileWidget

# Qt 类：永远从 qgis.PyQt 走，不要直接 import PyQt5
# （QGIS 自带的 PyQt5 与系统 pip 装的很可能不是同一套）
from qgis.PyQt.QtCore import QVariant, QSettings
from qgis.PyQt.QtWidgets import QAction, QDialog, QFileDialog
from qgis.PyQt import uic
```

## `iface`：程序入口

`iface` 是进入整个 QGIS 应用的入口。插件构造时收到的 `iface`（`QgsInterface` 类型）和 Python 控制台里的全局 `iface` 是同一个对象，通过它拿到主窗口、地图画布、图层列表等一切。

- QGIS 内（插件/控制台）`iface` 已就绪，`QgsProject.instance()` 已初始化
- 独立脚本 / pytest 需自己创建 `QgsApplication` 用于批处理或单元测试

### 常用方法

| 调用                                                              | 作用                  |
| --------------------------------------------------------------- | ------------------- |
| `iface.mainWindow()`                                            | 主窗口（做对话框 parent）    |
| `iface.mapCanvas()`                                             | 地图画布 `QgsMapCanvas` |
| `iface.addVectorLayer(path, name, 'ogr')`                       | 加载矢量图层进工程，返回图层      |
| `iface.setActiveLayer(layer)` / `iface.activeLayer()`           | 当前活动图层              |
| `iface.zoomToActiveLayer()`                                     | 缩放到活动图层             |
| `iface.addPluginToMenu(menu, action)` / `removePluginMenu(...)` | 插件菜单                |
| `iface.addToolBarIcon(action)` / `removeToolBarIcon(action)`    | 工具栏按钮               |
| `iface.layerTreeRoot()`                                         | 图层树                 |

# `QgsMapLayer`：图层操作

`QgsMapLayer` 子类表示 QGIS 的图层类，子类如 `QgsVectorLayer` 表示矢量图层，`QgsRasterLayer` 表示栅格图层。

## `dataProvider()`：修改图层数据

`QgsMapLayer`的一个核心函数。**获取图层的数据源驱动程序**，让开发者可以直接操作底层数据（如 Shapefile、GeoPackage、PostgreSQL 数据库或 Raster 栅格文件），而无需关心数据的具体存储格式。

不进入 QGIS 的编辑模式，不使用 QGIS 的撤销栈，直接修改数据

```python
provider = layer.dataProvider()

# 增（region_editor.py · _append_to_layer）
provider.addFeatures(features)               # 传 QgsFeature 列表，返回 (成功?, 变更列表)

# 删（region_editor.py · _delete_from_layer）
provider.deleteFeatures([fid1, fid2, ...])   # 传 feature.id() 列表

# 改属性：{要素id: {字段索引: 新值}}
provider.changeAttributeValues({fid: {name_idx: '新区域名'}})

# 加字段
provider.addAttributes([QgsField('REMARK', QVariant.String)])
layer.updateFields()                          # 加字段后必须刷新字段缓存

# 任何直写后，让画布重绘（region_editor.py 两处直写后都调用）
layer.triggerRepaint()
```

## `startEditing()`：实现图层交互式编辑

操作进 QGIS 撤销栈，用户可 Ctrl+Z、可整体回滚，适合地图工具交互编辑

```python
layer.startEditing()
layer.addFeature(feature)          # 此期间用 layer 的方法而不是 provider 的
layer.deleteFeature(fid)
layer.changeAttributeValue(fid, field_idx, value)
if layer.commitChanges():          # 提交落盘
    ...
else:
    layer.rollBack()               # 回滚全部未提交修改
```

## `QgsVectorLayer`：矢量图层操作

https://qgis.org/pyqgis/3.44/core/QgsVectorLayer.html
### 创建图层

```python
# 打开图层文件
# 数据源路径（文件路径或连接字符串）、图层名称、数据提供者名称
layer = QgsVectorLayer(r'D:\\data\\layer.shp', '基准图幅', 'ogr')
if not layer.isValid():
    print('打开失败：路径错/编码坏/被占用')

# 内存图层（临时数据，URI 语法定义字段）
mem = QgsVectorLayer('Polygon?crs=EPSG:4326&field=NAME:string(50)&field=CODE:string(20)',
                     '临时区域', 'memory')

# 数据库 URI（PostGIS 示例）
pg = QgsVectorLayer('dbname=mydb host=localhost table=sheets (geom) sql=', 'sheets', 'postgres')
```

### 常用属性与方法

| 表达式                                           | 返回                             |
| --------------------------------------------- | ------------------------------ |
| `layer.name()` / `layer.setName(s)`           | str                            |
| `layer.id()`                                  | str                            |
| `layer.source()`                              | str                            |
| `layer.fields()`                              | `QgsFields`                    |
| `layer.featureCount()`                        | int                            |
| `layer.wkbType()`                             | 枚举                             |
| `layer.crs()`                                 | `QgsCoordinateReferenceSystem` |
| `layer.extent()`                              | `QgsRectangle`                 |
| `layer.geometryType()`                        | 枚举                             |
| `layer.isValid()`                             | bool                           |
| `layer.selectByExpression('"NAME"=\'East\'')` | None                           |
| `layer.removeSelection()`                     | None                           |
| `layer.triggerRepaint()`                      | None                           |
| `layer.dataProvider()`                        | provider                       |

### `getFeatures()`：返回迭代器

`layer.getFeatures()` 返回**迭代器**（只能消费一次，中途修改图层可能失效；
需要反复用就 `list(layer.getFeatures())`）

```python
# 全量遍历
for feature in layer.getFeatures():
    print(feature['NAME'], feature.id())

# 表达式过滤（服务端/ provider 侧下推，大图层远快于 Python 里 if）
request = QgsFeatureRequest().setFilterExpression('"NAME" = \'East\'')
for feature in layer.getFeatures(request):
    ...
```

#### 性能优化

QGIS 中几何（Geometry）代表空间要素的形状与位置，而属性（Attributes）代表该要素的文字与数字描述信息，共同构成了矢量图层shapefile要素。

对于较大图层，只统计属性不需要几何（例如不需要坐标运算）时使用 `NoGeometry` 去除几何，`setNoAttributes()` 去除属性

```py
# 只要部分字段（省内存）——传入字段名列表
request = QgsFeatureRequest().setSubsetOfAttributes(['NAME', 'CODE'], layer.fields())

# 不要几何，只统计属性时
request = QgsFeatureRequest().setFlags(QgsFeatureRequest.NoGeometry)  # 不要几何
request = QgsFeatureRequest().setNoAttributes() # 连属性都不要
```

### 读取属性

```python
value = feature['NAME']        # 按字段名（推荐，自动处理字段索引）
value = feature[0]             # 按字段索引
attrs = feature.attributes()   # 整行属性 list（本项目 _delete_from_layer 用它按索引取值）
```

`.shp` 里的空字段读出来是 `NULL`，这个是PyQGIS的特殊对象，如果只写 `is None` 判断会失效。

```python
from qgis.core import NULL
if feature['NAME'] == NULL or not feature['NAME']:   # 两种都覆盖
    ...
```

# `QgsField`：字段编辑

https://qgis.org/pyqgis/3.44/core/QgsField.html

`QgsField` 是QGIS的核心 API 类之一，用于封装和管理矢量图层属性表(Attribute Table)中的字段(Field)数据。在 QGIS 中打开一个矢量图层，它的属性表由要素和字段组成。 `QgsField` 就是用来定义和描述具体的列的。

```python
fields = layer.fields()

# 遍历字段信息（本项目多处使用这个推导式）
names = [f.name() for f in layer.fields()]        # region_editor.py · _layer_field_indexes

# 按名查索引（不存在返回 -1，务必判断）
idx = fields.indexFromName('NAME')                # region_importer.py · _output_fields

# 单个字段的元信息
f = fields.field('NAME')      # 或 fields.at(idx)
f.name(), f.type(), f.length(), f.precision()

# 只挑某类型的字段（自动识别编号字段时用，region_importer.py · _detect_code_field）
string_fields = [f.name() for f in layer.fields() if f.type() == QVariant.String]
```

构建新字段

```python
out_fields = QgsFields()
for field in base_layer.fields():
    out_fields.append(field)   # QgsFields.append 会拷贝
region_field = QgsField('NAME', QVariant.String)
region_field.setLength(50)   # shp 必须给长度，否则默认 255 会被截断告警
out_fields.append(region_field)
```

# `QgsFeature / QgsGeometry`：要素与几何

## `QgsFeature`：构造要素

`QgsFeature` 是 PyQGIS 中核心的要素类，包含几何形状和属性数据。
创建要素后使用 `setGeometry()` 和 `setAtrributes()` 设定几何和属性 

```python
feature = QgsFeature(fields)                 # 传入 QgsFields 决定属性结构
feature.setGeometry(base_feature.geometry()) # 复制另一要素的几何
feature.setAttributes([                      # 顺序、数量必须与 fields 完全一致
    base_feature[f] if f in base_field_names
    else (region_name if f == 'NAME' else None)
    for f in field_names])
```

`feature.id()` 是数据源内的要素 ID（fid），删除/重建后会变，只能临时使用。

## `QgsGeometry`：几何操作

https://qgis.org/pyqgis/3.44/core/QgsGeometry.html

C++和PyQGIS中的几何图形类，用于表示、创建、修改和分析空间地理要素（如点、线、面等）的几何形状、支持和常见空间数据格式转换（WKT, WKB, GeoJSON）

```python
geom = QgsGeometry.fromWkt('POLYGON((0 0, 1 0, 1 1, 0 0))')  # WKT 构造
geom.asWkt()                # 导出 WKT
geom.area()                 # 面积（图层 CRS 单位，不是自动投影后的）
geom.boundingBox()          # QgsRectangle 外接矩形
geom.intersects(other)      # 相交判断
geom.contains(point)        # 包含
geom.isGeosValid()          # 有效性检查
centroid = geom.centroid()  # 质心，返回 QgsGeometry（点）
```

从要素中取几何，再从几何用 `asPoint()` 取坐标：

```python
point = feature.geometry().asPoint()   # 单点图层
point.x(), point.y()
```

# `QgsVectorFileWriter`

`QgsVectorFileWriter` 是 QGIS 的核心 API，用于将矢量图层写入并保存成磁盘文件，用于导出、转换和保存矢量地理数据，支持格式转换、创建新的矢量文件并添加地理要素、控制格式输出。

创建 `writer` → 逐要素写入 → 删除 `writer`，删除了之后才会写文件，如果不删除可能导致接下来 `QgsVectorLayer(out_path...)` 打不开或不完整。

```python
from qgis.core import (QgsFeature, QgsFeatureSink, QgsFields,
                       QgsProject, QgsVectorFileWriter, QgsVectorLayer)

options = QgsVectorFileWriter.SaveVectorOptions()
options.driverName = 'ESRI Shapefile'      # 'GPKG' 写 GeoPackage
options.fileEncoding = 'utf-8'             # shp 属性编码，写 .cpg
options.actionOnExistingFile = QgsVectorFileWriter.CreateOrOverwriteFile

writer = QgsVectorFileWriter.create(
    out_path,                 # 输出文件全路径
    target_fields,            # QgsFields 输出字段结构
    base_layer.wkbType(),     # 几何类型
    base_layer.crs(),         # 坐标系
    QgsProject.instance().transformContext(),   # 坐标变换上下文
    options)

if writer.hasError() != QgsVectorFileWriter.NoError:   # 检查创建错误
    print(writer.errorMessage())

for feature in features:
    writer.addFeature(feature, QgsFeatureSink.FastInsert)

del writer   # 删除

# 重新加载 + 加入工程
layer = QgsVectorLayer(out_path, layer_name, 'ogr')
assert layer.isValid() and layer.featureCount() == len(features)
QgsProject.instance().addMapLayer(layer)
```

# `QgsProject`：工程管理

`QgsProject` 是一个Python核心类，负责管理和维护当前打开的 QGIS 工程文件（`.qgz` 或 `.qgs`）的所有状态、图层和配置信息。

- `QgsProject.instance()` 是全局单例，代表当前 `.qgz` 工程
- 支持图层添加、移除操作

```python
project = QgsProject.instance()

project.addMapLayer(layer)                       # 图层进工程（含图层树）
project.addMapLayer(layer, addToLegend=False)    # 只注册不进图层树
project.removeMapLayer(layer.id())               # 移除（按 id，不是 name！）
project.mapLayers()                              # {图层id: QgsVectorLayer} dict
project.mapLayersByName('layer-name')               # 按名字找，返回列表
project.layerTreeRoot()                          # 图层树（控制顺序/分组）
project.fileName()                               # 工程文件路径，未保存为 ''
project.transformContext()                       # 写文件时要传
project.crs()                                    # 工程坐标系
```

按数据源路径 `path` 找图层：

```python
def find_layer_on_path(path):
    normalized = os.path.abspath(path)
    for layer in list(QgsProject.instance().mapLayers().values()):
        try:
            source = layer.source().split('|')[0]   # 去掉 '|subset=...' 等后缀
            if source and os.path.abspath(source) == normalized:
                return layer
        except Exception:
            continue
    return None
```

`addMapLayer` 后图层归QGIS的工程所有，如果想继续使用的话 Python 需要先取出。 `removeMapLayer` 之后，原来的 Python 包装对象随 C++ 对象销毁而失效。例如需要图层名数据时，移除前先取 `layer.name()` 存下来。

## 工程信号与图层信号

PyQt 的信号槽机制与 Qt 类似。

当有新的图层被添加到当前的 QGIS 项目中时，自动触发并执行指定的 `callback`（回调函数）：

```python
QgsProject.instance().layersAdded.connect(callback)     # 参数：[QgsMapLayer, ...]
QgsProject.instance().layersRemoved.connect(callback)   # 参数：[图层id(str), ...]
```

选择工具选中要素变化时触发并执行指定的回调函数：

```py
layer.selectionChanged.connect(callback) # 回调参数：(选中的 fid 集合, 取消选中的 fid 集合, 是否整表重选)
```

# `QgsCategorizedSymbolRenderer`：图层渲染

`QgsCategorizedSymbolRenderer` 是 QGIS C++ 和 Python (PyQGIS) API 中用于实现矢量图层分类渲染的核心类，为不同的属性值分配独有的符号或颜色。

下列代码示例按要素的 `NAME` 字段给区域着色：

```python
from qgis.core import (QgsCategorizedSymbolRenderer, QgsFillSymbol, QgsRendererCategory)

categories = []
for index, region in enumerate(regions):
    color = REGION_COLORS[index % len(REGION_COLORS)]
    symbol = QgsFillSymbol.createSimple({
        'color': '0,0,0,0',          # 填充 RGBA，全 0 = 透明
        'outline_color': color,      # 边框色 '#FF0000'
        'outline_width': '0.4',      # 线宽（毫米）
    })
    # QgsRendererCategory(分类值, 符号, 图例文字)
    categories.append(QgsRendererCategory(region['NAME'], symbol, region['NAME']))

layer.setRenderer(QgsCategorizedSymbolRenderer('NAME', categories))  # 参数：分类字段名
layer.triggerRepaint()
```

## `createSimple`

`createSimple` 是一个频繁用于快速创建地图符号的静态工厂方法，属于表格中点线面的符号类

| 符号类                  | 常用键                                                                          |
| -------------------- | ---------------------------------------------------------------------------- |
| `QgsFillSymbol`（面）   | `color`、`outline_color`、`outline_width`、`outline_style`（`solid`/`dash`/`no`） |
| `QgsLineSymbol`（线）   | `line_color`、`line_width`、`line_style`                                       |
| `QgsMarkerSymbol`（点） | `name`（`circle`/`square`/`star`）、`color`、`outline_color`、`size`              |

## 保存符号样式为 `qml`

`qml` 是 QGIS 原生支持的样式文件格式，和图层的 `shp` 文件放在一起，用于保存图层的符号体系、标签、透明度及其他渲染设置。

```python
result = layer.saveDefaultStyle()    # 写 <数据源同名>.qml，重开文件自动带样式
# 返回 (状态消息: str, 成功标志: bool) 
ok = result[1] if isinstance(result, tuple) else not result
msg = result[0] if isinstance(result, tuple) else result

layer.loadDefaultStyle()             # 从 .qml 恢复
qml, ok = layer.exportNamedStyle()   # 取样式 QML 字符串（也可 loadNamedStyle(path)）
```

# Reference

- [PyQGIS Cookbook](https://docs.qgis.org/3.16/en/docs/pyqgis_developer_cookbook/)
- [PyQGIS Cookbook 中译](https://luolingchun.github.io/PyQGIS-Developer-Cookbook-cn/)
- [QGIS 3.16 C++ API 参考](https://api.qgis.org/api/3.16/)
- [PyQGIS API 文档](https://qgis.org/pyqgis/3.16/)
