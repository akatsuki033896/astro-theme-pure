---
title: 'XML与 pugi::xml 使用'
publishDate: 2026-09-06
updatedDate: 2026-09-16
description: 'Breif tutorial about what is XML and how to parse XML using C++'
tags:
    - C++
language: 'Chinese'
---

## What is XML？

标记语言因为可以在定义文本结构、格式的同时包含内容，经常用于计算机数据的传输。XML（可扩展标记语言）是一种用于结构化、存储和传输数据的标记语言，可以通过自定义标签构建**树状的层级结构**来表示数据。

XML 经常用于各种软件的配置文件（可能扩展名并不一定是 `.xml` ），例如 VS 的项目配置文件，压制软件 ShanaEncoder 的配置等，还可以用于传输地理数据也就是 `.gml` 格式。

这是一个 XML 样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>          <!-- 1. Root 根元素：整个文档的最外层 -->
    <book>           <!-- 2. Child 子元素：属于 bookstore 的子级 -->
        <title>XML指南</title> <!-- 3. Sub-child 子元素的子元素 -->
        <price>39.9</price>
    </book>
</bookstore>
```

## XML 的反序列化/序列化

- 反序列化：一种特殊的解析过程，大体上可以等同解析。把数据提取成程序可以使用的数据，例如对于 C++ 来说就是被提取的内容以 C++ 的类的形式存储在内存中，用于满足业务逻辑。
- 序列化：将内存中的对象结构（如类、属性和字段）转换成标签格式的 XML 字符串或文件的过程。

通过序列化和反序列化，可以方便地保存数据、在不同系统之间传输信息。XML 的序列化产物是 XML，反序列化的产物是内存中的对象。序列化不需要验证，反序列化必须验证。

## XML 语法结构

### 声明

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

第一行的声明包含了文档的 XML 版本和编码格式。

### 树形结构

```xml
<root>
    <child>
          <subchild>.....</subchild>
    </child>
    <child1>
          <subchild>.....</subchild>
    </child1>
</root>
```

- 标签：结构为 `<key> [content] </key>`，可以嵌套，对大小写敏感
- 元素：从开始标签直到结束标签的部分，必须有根元素
- 属性：必须加引号

元素和属性的示例：

```xml
<person sex="female">
  <firstname>Anna</firstname>
  <lastname>Smith</lastname>
</person>
​
<person>
  <sex>female</sex>
  <firstname>Anna</firstname>
  <lastname>Smith</lastname>
</person>
```

属性难以阅读和维护，请尽量使用元素来描述数据。

有时候会向元素分配 ID 引用，这些 ID 索引可用于标识 XML 元素，并不组成数据。**元数据（有关数据的数据）应当存储为属性，而数据本身应当存储为元素**。

```xml
<messages>
  <note id="501">
    <to>Tove</to>
    <from>Jani</from>
    <heading>Reminder</heading>
    <body>Don't forget me this weekend!</body>
  </note>
  <note id="502">
    <to>Jani</to>
    <from>Tove</from>
    <heading>Re: Reminder</heading>
    <body>I will not</body>
  </note>
</messages>
```

### 实体引用

XML 的元素中如果包含了 `<` 会被视为一个新的标签的开始，从而引发错误

```xml
<message>if salary < 1000 then</message> <!-- 错误 -->
<message>if salary &lt; 1000 then</message>
```

为了解决这个问题，要使用实体引用。一共有5种预定义实体引用：

| 字符  | 实体名称 (Entity Name) | 十进制实体编号 (Decimal) | 十六进制实体编号 (Hexadecimal) | 描述        |
| --- | ------------------ | ----------------- | ---------------------- | --------- |
| `<` | `&lt;`             | `&#60;`           | `&#x3C;`               | 小于号（标签开始） |
| `>` | `&gt;`             | `&#62;`           | `&#x3E;`               | 大于号（标签结束） |
| `&` | `&amp;`            | `&#38;`           | `&#x26;`               | 与号 / 和号   |
| `"` | `&quot;`           | `&#34;`           | `&#x22;`               | 双引号       |
| `'` | `&apos;`           | `&#39;`           | `&#x27;`               | 单引号（撇号）   |

### 命名空间

XML 命名空间（XML Namespace）用于在一个 XML 文档中避免元素和属性名称冲突。它通过URI为标签和属性提供唯一的名称，通常使用 `xmlns` 属性在标签中进行定义。

## 使用 `pugi::xml` 在 C++ 实现反序列化和序列化

pugi::xml 是一个 C++ XML 处理库，它只由`pugixml.cpp`、`pugixml.hpp` 和 `pugiconfig.hpp` 三个文件组成，可以解析 XML、控制 DOM 树结构存储、修改和遍历 XPath 节点，支持多种字符编码转换。

### CMake 集成

```cmake
add_executable(test_pugi_xml
    main.cpp
    pugixml/pugixml.cpp
)
target_include_directories(test_pugi_xml PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/pugixml
)
```

### `load_file()`：解析xml文档

通过路径直接读取并解析

```cpp
#include "pugixml/pugixml.hpp"
#include <iostream>
int main() {
    pugi::xml_document doc;
    pugi::xml_parse_result result = doc.load_file("test.xml");
    if (!result) {
        std::cout << "XML parsed with errors, error description: " << result.description() << "\n";
        std::cout << "Error offset: " << result.offset << '\n';
        return 1;
    }
    std::cout << "XML Parse success. Root node: " << doc.document_element().name() << "\n";
    return 0;
}
```

`pugi::xml_document` ：整个 XML 文档的 DOM 树根节点

`pugi::xml_parse_result` ：表示 XML 解析操作结果的结构体。调用`load_file`或`load_string`返回该对象，含有几个成员

- **`status`**：解析状态枚举（`xml_parse_status`）。用来标明解析是成功完成，还是遇到了文件未找到、标签不匹配、内存不足等错误。
- **`offset`**：错误偏移量。如果解析失败，它会指示错误发生在输入数据的第几个字符位置，方便定位问题。
- **`encoding`**：源文档的编码格式（`xml_encoding`）。
- **`operator bool()`**：重载了布尔转型操作符。若解析成功，返回 `true`；若失败，返回 `false`。
- **`description()`**：返回一个可读的错误描述字符串。

### `load_string()` 解析字符串

读取内存中的XML字符串后解析，输出根节点为 `root`

```cpp
const char* xml =
        "<root>"
        "    <name>hello</name>"
        "</root>";
pugi::xml_document doc;
pugi::xml_parse_result result = doc.load_string(xml);
```

### `find_node()` 查找节点

传入一个函数、函数对象或 Lambda 表达式，在当前节点及其整个子树中查找第一个满足该条件的 XML 节点，返回 `xml_node` 类型对象，查找不到会返回空的`xml_node` 类型对象

```cpp
// 找出xml中第一个名称为”name”的节点
auto res = doc.find_node([] (pugi::xml_node& node) {
	 return strcmp(node.name(), "name") == 0;
});
```

### 常用接口

| API                  | 作用      | 例子                            |
| -------------------- | ------- | ----------------------------- |
| `document_element()` | 获取根元素   | `doc.document_element()`      |
| `child()`            | 获取子元素   | `node.child("name")`          |
| `child_value()`      | 获取子元素文本 | `node.child_value("name")`    |
| `text()`             | 获取文本节点  | `node.text().get()`           |
| `attribute()`        | 获取属性    | `node.attribute("id")`        |
| `first_child()`      | 第一个子节点  | `node.first_child()`          |
| `next_sibling()`     | 下一个兄弟   | `node.next_sibling()`         |
| `previous_sibling()` | 上一个兄弟   | `node.previous_sibling()`     |
| `children()`         | 遍历子节点   | `for(auto n:node.children())` |
| `find_child()`       | 按条件查找   | `node.find_child()`           |
| `find_node()`        | 递归查找    | `node.find_node()`            |

例如对于文本节点 `.text()` 后使用 `get()` 或 `set()` 返回对象和修改。

### `save()`： 输出 XML

```cpp
doc.save(std::cout, "    "); // 终端输出，指定缩进4个空格
doc.save_file("../output.xml", "    ");
```

- save的第一个参数是输出流，第二个参数是缩进，还可以指定第三个参数flag和第四个参数编码类型，适合调试用。
- save_file第一个参数是路径，格式可以是xml也可以是任何和xml一样的东西比如gml，第二个是缩进。

## Reference

- wiki: https://zh.wikipedia.org/zh-cn/XML
- Github: https://github.com/zeux/pugixml
- https://zhuanlan.zhihu.com/p/600144898
