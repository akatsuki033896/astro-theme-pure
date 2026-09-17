---
title: 'Visual Studio Tutorial'
publishDate: 2026-09-17
updatedDate: 2026-09-17
description: '哪来的傻逼开了三十几节付费课就为了教人用VS啊？'
tags:
  - C++
language: 'Chinese'
heroImage: { src: './thumbnail.webp', color: '#5b6e78' }
---

## 解决方案和项目

解决方案：`.sln` ，一个解决方案可以包含多个项目，只能用一个程序入口（一个 `main()` ）2026版本引入 `.slnx` ，实质为 `.xml` 格式

项目：`.vcxproj` ，VS上的代码以项目为独立的单位运行，包含具体的源代码、资源文件和编译器设置，最终编译成可执行文件 `.exe`或动态库 `.dll`

## VS 的项目构建

### Msbuild

MSBuild 是微软提供的一个用于构建应用程序的平台，它以 XML 架构的项目文件 `.vcxproj` 来控制怎么构建。Visual Studio 会使用 MSBuild，但MSBuild 并不依赖 Visual Studio，可以在没有安装 VS 的系统中独立工作。

使用 CMake 配置项目构建和在 VS 里除了 CMake 通过写配置，VS 通过 GUI 配置构建，还有以下区别：

| CMake | Visual Studio |
| --- | --- |
| `CMakeLists.txt` | `.vcxproj` |
| `cmake -B build` | VS 根据 `.vcxproj` 准备/构建项目 |
| `cmake --build build` | `msbuild xxx.vcxproj` |
| CMake Generator | 决定生成什么构建系统 |
| `CMAKE_CXX_STANDARD` | `.vcxproj` 中的 C++ 标准设置 |
| `CMAKE_BUILD_TYPE`（Ninja等） | `Debug/Release` 配置 |
| `CMAKE_CXX_COMPILER` | MSVC 工具链相关设置 |
| `target_link_libraries()` | `.vcxproj` 中的链接器依赖设置 |

### `.vcxproj` 的路径编写

编写 `.vcxproj` 时，MSBuild 默认路径并不是解决方案所在的文件夹，是以**项目文件（`.vcxproj`）所在的目录**，同时也作为相对路径的基准点（即 `.` 路径）做文件处理的时候发现找不到文件的时候务必注意。

可以直接写 `..\lib`，但为了避免路径解析出错，可以结合 MSBuild 内置的**路径宏**来写相对路径。这样即使项目被移动，路径也不会错乱。常用的两个核心宏：

• **`$(ProjectDir)`**：项目文件 `.vcxproj` 所在的文件夹路径。
• **`$(SolutionDir)`**：解决方案 `.sln` 所在的文件夹路径。

## 平台工具集

https://learn.microsoft.com/zh-cn/windows-hardware/drivers/devtest/platform-toolset

待补充

## 使用 VS 进行 Debug

https://learn.microsoft.com/zh-cn/visualstudio/debugger/getting-started-with-the-debugger-cpp?view=visualstudio

f11（逐语句 Step Into）：启动“单步执行”命令，让应用一次执行一条语句。如果当前行包含函数或方法调用，会**直接进入**函数内部的第一行代码，逐行调试函数内部的逻辑。当你怀疑错误出在某个函数内部，或者想了解函数的具体执行过程时使用。进入包含的函数的语句后可以跳出(`shift+f11`).

f10（逐过程 Step Over）：执行当前行代码。如果当前行包含函数或方法调用，**不会**进入函数内部，而是直接运行完该函数，停在当前行的下一行。确定某个函数没有问题，不需要看它的内部执行细节时使用。

Debug 和 Release 本质上是一组不同编译选项的集合。Debug 用于开发时找错误，不优化代码并带有完整调试信息，Release 用于最终发布，会对代码进行全面优化，运行更快、体积更小。

调试过程中，变量值或变量名显示为**红色**，通常表示**该变量的值自上一次代码执行或上一次评估（Evaluation）以来已经发生了改变**

### `.pdb` 符号文件

Debug 模式下程序数据库 `.pdb` 文件（也称为符号文件、映射项目的源代码中的标识符和语句）用于存储调试程序的符号信息，将调试器链接到源代码，从而启用调试。使用标准调试生成配置从 Visual Studio IDE 生成项目时，编译器会创建相应的符号文件。

---
 
## VS Qt Tools

需要用Qt maintainence tool或online installer先安装msvc的构建工具

创建项目：可选Qt/MSBuild的vs项目和CMake的项目

在vs中：

1. 扩展→扩展管理器安装 qt visual studio tools
2. 扩展→Qt VS Tools→Qt Version中选择安装好的msvc构建工具中的qmake.exe或qtpath.exe添加Qt版本

### 通过右键资源文件打开Qt Designer / Qt Linguist闪退

Qt→General→Qt Designer Run in detached windows 改为 True

### 给项目指定qt版本

项目属性→Qt Project Settings→General→Qt Installation选择Qt版本

### vs外运行Qt程序冲突

若把别的Qt的bin添加到PATH里了可能导致此问题。

例如安装过 QGIS，系统 PATH 里 QGIS 自带的Qt排在安装的Qt之前，在 VS 里 F5 运行不受影响（Qt 工具会自动设置调试 PATH），但直接双击 exe 会加载到错误版本的 Qt DLL 导致启动失败。若有此需求，可把 Qt DLL 部署到 exe 目录，或直接更改PATH，移到别软件自带Qt的前面。

## 常见问题

### visual studio2022无法启动程序，\ALL_BUILD拒绝访问

主要是使用 CMake 生成 VS 项目常出现这个问题。在项目上右键，添加想要的程序运行入口的项目为启动项。

待补充

## 常见操作

### 改变源代码编码

文件→另存为→选择需要的编码保存

### 清理构建产物

当 Visual Studio 的**增量构建缓存失效**、**依赖关系错乱**或**旧的二进制文件残留**导致代码修改后不生效、出现莫名其妙的编译错误时，清理构建产物并重新构建（Clean & Rebuild）可以解决问题。更改构建配置但过去的构建依赖cache，需要重新构建的时候使用。

生成→清理解决方案

### 配置当前工作目录

项目→属性→配置属性→调试→工作目录

- `$(ProjectDir)` 是项目文件所在目录
- `$(OutDir)`是构建产物所在目录例如 `解决方案目录\x64\Debug\`

### 生成事件

分为生成前事件、链接前事件、生成后事件

例如对于实现构建生成后自动把某个文件拷贝到别的目录的操作，可以在生成后事件里填写这个操作的命令。

右键项目→ 属性→生成事件

### 添加外部库

先把对应库的文件和 `lib`, `dll` 复制黏贴到项目，能在 VS 里改就不要乱动项目构建文件，血的教训

#### 头文件项目

项目 → 属性→C/C++→常规→附加包含目录，添加头文件(include)

#### 链接的库

项目 → 属性→链接器→常规→附加库目录，添加库(lib)

如果使用的是动态链接库，必须将对应的 `.dll` 文件复制到你生成的 `.exe` 文件所在的同级目录下（通常是 `x64/Debug` 或 `x64/Release` 文件夹中），否则程序运行时会报错找不到DLL

### clangpowertools 把 msbuild 项目转换出 compile_commands.json

clangpowertools是vs的一个拓展。右键项目选择Export Compilation Database，在vscode里的settings.json中

```json
{
  // 必须禁用微软官方的 IntelliSense，防止抢占和冲突
  "C_Cpp.intelliSenseEngine": "disabled", 

  "clangd.arguments": [
    // 强制指定编译数据库的目录（项目根目录）
    "--compile-commands-dir=${workspaceFolder}",
    
    // 核心：允许 clangd 调用 MSVC 编译器探测 Windows 系统的标准库路径
    // 如果你使用的是 64 位系统且安装在 C 盘，可以直接用下面的通配符
    "--query-driver=C:\\Program Files*\\Microsoft Visual Studio\\*\\*\\VC\\Tools\\MSVC\\*\\bin\\Host*\\*\\cl.exe",
    
    // 开启后台并行索引，加快大型 MSBuild 项目的加载速度
    "-j=4",
    // 开启增强的补全
    "--completion-style=detailed"
  ]
}
```