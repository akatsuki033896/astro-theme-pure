---
title: 'Qt part.2: GUI'
publishDate: 2026-06-06
updatedDate: 2026-06-06
description: '使用QWidget制作GUI'
tags:
  - Qt
  - C++
language: 'Chinese'
---

## GUI

- `QWidget` 是所有控件的基类，用于创建任何类型的控件
- `QMainWindow` 是一个窗口的基类，通常用来创建应用程序的主窗口，支持菜单栏、工具栏、状态栏等多种组件

Qt Designer是 Qt 提供的可视化界面设计工具，通过拖拽控件来快速设计界面，生成 `.ui` 文件，后续通过 `uic` 工具转换成代码，也可以手写代码创建界面，适合自定义复杂界面，但开发周期较长。

### 布局管理器

- `QHBoxLayout`：水平布局，将控件从左到右排列，不够了会自己调整
- `QVBoxLayout` ：垂直布局，将控件从上到下排列，不够了会自己调整
- `QGridLayout`：网格布局，控件按照行和列排列，适合创建表单和矩阵类型的布局
- `QFormLayout` ：表单布局，自动将标签和控件（如文本框）按表单格式排列
- `QStackedLayout` ：堆叠布局，可以堆叠多个控件，只有一个控件可见
- `QSplitter` ：分割布局，允许动态调整控件的大小，常用于分隔不同区域

### 自定义控件

1. 继承 `QWidget` 或其他合适的 Qt 控件（如 `QPushButton` ）
2. 重写必要的事件函数，如 `paintEvent()` 来实现自定义绘制， `mousePressEvent()` 来处理鼠标事件等
3. 重写 `resizeEvent()` 来调整控件在大小变化时的行为（可选）
4. 根据需要，重写 `sizeHint()` 和 `minimumSizeHint()` 来指定控件的默认大小（可选）

### Qt样式表(Qss)

类似于 CSS，用于定制控件的外观，可以设置控件的颜色、字体、边框、背景等。通过调用控件的 `setStyleSheet()` 方法，可以应用QSS样式表来改变控件的外观。

例如，QPushButton 可以通过 QSS 设置背景色、文字颜色和字体大小等属性。

全局加载 `style.qss`

```cpp
QFile file(":/style.qss");
if (file.open(QFile::ReadOnly)) {
    QString style = QLatin1String(file.readAll());
    app.setStyleSheet(style);
}
```

### 界面国际化

`QTranslator` 类来实现国际化：

1. **创建翻译文件** ：使用 Qt Linguist 工具创建 `.ts` 文件，进行语言翻译。
2. **加载翻译文件** ：在应用启动时，通过 `QTranslator` 加载并安装翻译文件，动态更改界面语言。

## `QPainter` 绘图

`QPainter` 是 Qt 中用于绘制图形和文本的核心类，基于 `QPaintDevice` 和 `QPaintEngine`，通常与 `QWidget` 的 `paintEvent()` 一起使用

1. **创建 `QPainter` 对象** ：通过传递目标设备（如 `QWidget` 、 `QPixmap` 、 `QImage` 等）来创建一个 `QPainter` 对象。
2. **设置绘制参数** ：使用 QPainter 提供的 API 设置绘图的属性，例如笔触颜色（ `setPen()` ）、画刷颜色（ `setBrush()` ）、字体（ `setFont()` ）等。
3. **执行绘制操作** ：调用 QPainter 的绘图函数，如 `drawRect()`, `drawText()` 等，来绘制具体内容。
4. **结束绘制** ：完成绘制后，调用 `end()` 来结束绘图操作，释放相关资源。

示例流程：

```cpp
void MyWidget::paintEvent(QPaintEvent *event) {
    QPainter painter(this);
    painter.setPen(QPen(Qt::blue, 2));  // 设置蓝色的粗笔
    painter.drawRect(10, 10, 100, 100);  // 绘制矩形
}
```

### `QPaintDevice`

所有可以进行绘制操作的对象的基类，如 `QWidget`, `QImage`, `QPixmap` 等。它提供了一个统一的接口，使得 `QPainter` 可以在不同的设备上进行绘制。

### `QPaintEngine`

`QPaintEngine` 是 `QPainter` 的一个底层类，负责将绘图操作实际输出到设备（如屏幕、打印机、图片等）。不同的绘图设备（如窗口、图像等）有不同的 `QPaintEngine` 实现。它为 `QPainter` 提供了实现细节，保证绘图操作被正确渲染。

## 图像数据处理

1. `QPixmap`：用于优化屏幕显示的图像，特别是在高效渲染时使用。适用场景：显示图像到屏幕、处理图像的高效渲染，通常在需要快速显示图像时使用（如游戏开发、界面绘制）
2. `QImage`：用于处理原始像素数据，能够支持多种格式（如 JPEG、PNG 等），且提供了更灵活的接口来访问像素数据。适用场景：需要对图像数据进行修改（如像素级别的操作、图像处理）时使用
3. `QBitmap`：特殊类型的图像，主要用于存储位图（黑白图像）。QBitmap 是 QImage 的一个子类，但仅支持黑白像素的存储。适用场景：通常用于图标、掩码或透明区域的处理

## 模型-视图架构 (Model/View)

一种将**数据、界面显示和用户交互分离**的设计模式

- **Model** 负责数据的存储和管理
- **View** 负责数据的展示
- **Delegate** 负责数据的绘制和编辑

模型和视图之间通过 **信号与槽机制**自动同步，当模型中的数据发生变化时，视图会自动更新显示。这种架构可以让同一份数据被多个视图同时显示，提高代码的复用性和可维护性。

### `QAbstractItemModel` 

`QAbstractItemModel` 是 Qt 所有模型类的抽象基类，用于定义模型与视图交互的统一接口

它规定了模型必须实现的基本函数，例如：

- `rowCount()`
- `columnCount()`
- `data()`
- `setData()`
- `index()`
- `parent()`

通过继承 `QAbstractItemModel` ，可以自定义任意结构的数据模型，供 `QListView` 、 `QTableView` 、 `QTreeView` 等视图使用

### 自定义Model

一般通过继承`QAbstractItemModel`或其子类来实现

1. 简单场景：继承 `QAbstractListModel` / `QAbstractTableModel`
2. 复杂场景：直接继承 `QAbstractItemModel`

实现时需要重写的关键函数包括：

- `rowCount()`
- `columnCount()`
- `data()`
- `setData()` （如果需要可编辑）
- `flags()`

如果是树形结构，还需要实现 `index()`和`parent()`

### 视图

- `QListView` 用于显示一维列表数据只有行，没有列适合显示简单列表，如文件名列表、日志列表
- `QTableView` 用于显示二维表格数据有行和列常用于表格数据，如配置表、数据库表
- `QTreeView` 用于显示层级结构数据支持父子节点关系适合目录结构、组织结构、树形数据

三者的主要区别在于数据结构不同，但都可以使用同一个Model

### 数据角色

数据角色（Role）用于**区分同一数据在不同用途下的表现形式**

在 `data()` 函数中，根据不同的 Role 返回不同的数据内容。

常见角色包括：

- `Qt::DisplayRole` ：用于显示的文本
- `Qt::EditRole` ：用于编辑的数据
- `Qt::DecorationRole` ：图标或图片
- `Qt::ToolTipRole` ：提示信息
- `Qt::TextAlignmentRole` ：对齐方式
- `Qt::ForegroundRole` ：文字颜色
- `Qt::BackgroundRole` ：背景颜色

通过 Role 机制，可以在不改变数据结构的情况下，实现丰富的显示效果。

### 代理(Delegate)

主要负责数据的绘制和编辑，它决定了数据在视图中的显示样式，以及用户如何编辑数据。Delegate 常用于实现自定义单元格样式、下拉框、复选框、进度条等编辑效果。

Qt默认使用 `QStyledItemDelegate` ，但在需要自定义显示或编辑控件时，可以继承它来自定义Delegate。

自定义 Delegate 通常需要重写以下函数：

- `paint()` ：自定义绘制方式
- `sizeHint()` ：设置单元项大小
- `createEditor()` ：创建编辑控件
- `setEditorData()` ：将模型数据设置到编辑器
- `setModelData()` ：将编辑结果写回模型


