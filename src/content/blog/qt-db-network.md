---
title: 'Qt part.3: 网络编程 / 数据库编程'
publishDate: 2026-06-06
updatedDate: 2026-06-06
description: '直接调API不好吗'
tags:
  - Qt
  - C++
language: 'Chinese'
---

## 网络编程

### 网络编程常用类

1. `QTcpSocket` / `QUdpSocket` ：用于 TCP / UDP 网络通信的客户端套接字
2. `QTcpServer` ：用于创建 TCP 服务器，监听客户端连接
3. `QNetworkAccessManager` ：用于发起和管理网络请求，支持 HTTP、HTTPS、FTP 等协议。
4. `QNetworkReply` ：封装来自 `QNetworkAccessManager` 的响应信息。
5. `QHostAddress` ：用于表示 IP 地址。
6. `QNetworkConfigurationManager` ：用于管理网络配置，如网络接口、连接状态等。
7. `QWebSocket` ：用于 WebSocket 通信的客户端和服务器。

这些类提供了方便的接口来实现各种类型的网络功能，包括 socket 通信、HTTP 请求、FTP 操作等。

### `QTcpSocket` / `QUdpSocket`：套接字

- `QTcpSocket`：TCP（Transmission Control Protocol） 是一种面向连接的协议，保证数据的可靠性和顺序性。使用场景：需要可靠、顺序传输数据的应用，如聊天应用、文件传输、数据库连接等。保证数据传输的顺序和完整性，适合需要持久连接的应用。

- `QUdpSocket`：UDP（User Datagram Protocol） 是一种无连接的协议，数据包可以乱序到达且可能丢失。使用场景：对实时性要求高但对数据丢失容忍度较高的应用，如视频流、实时游戏、VoIP 等。适合大规模、多点广播的应用，但不保证数据的顺序和可靠性。

### 使用 `QTcpServer` 实现服务器

1. 创建一个 `QTcpServer` 对象并调用 `listen()` 方法启动监听。
2. 连接 `newConnection()` 信号，当有客户端连接时接收到此信号。
3. 在 `newConnection()` 里创建 `QTcpSocket` 对象来与客户端通信。
4. 通过 `QTcpSocket` 的 `read()` 和 `write()` 方法进行数据收发。

```cpp
QTcpServer *server = new QTcpServer(this);
if (server->listen(QHostAddress::Any, 1234)) {
    connect(server, &QTcpServer::newConnection, this, &MyServer::onNewConnection);
}

void MyServer::onNewConnection() {
    QTcpSocket *clientSocket = server->nextPendingConnection();
    connect(clientSocket, &QTcpSocket::readyRead, this, &MyServer::onReadyRead);
}

void MyServer::onReadyRead() {
    QTcpSocket *clientSocket = qobject_cast<QTcpSocket*>(sender());
    QByteArray data = clientSocket->readAll();
    // 处理收到的数据
}
```

### `QNetworkAccessManager`：管理和发起网络请求的类

`QNetworkAccessManager` 是 Qt 网络编程中用于管理和发起网络请求的类。它支持多种协议（如 HTTP、HTTPS、FTP、FTPS）并可以进行 GET、POST 等请求。

作用：

1. 发送 HTTP/HTTPS 请求，并返回 `QNetworkReply` 对象
2. 支持异步网络请求，通过信号与槽机制接收请求结果
3. 提供对上传和下载的控制（如设置请求头、发送表单数据等）

```cpp
QNetworkAccessManager *manager = new QNetworkAccessManager(this);
QNetworkRequest request(QUrl("http://example.com"));
QNetworkReply *reply = manager->get(request);

connect(reply, &QNetworkReply::finished, this, &MyClass::onFinished);

void MyClass::onFinished() {
    QNetworkReply *reply = qobject_cast<QNetworkReply*>(sender());
    QByteArray data = reply->readAll();
    // 处理响应数据
}
```

`QNetworkAccessManager` 还允许设置请求头、处理 SSL/TLS 加密等功能

### Websocket

`QWebSocket` 类提供了对 WebSocket 协议的支持，用于进行双向通信。WebSocket 适用于实时消息推送应用，如在线聊天、实时数据流等。

1. 客户端：使用 `QWebSocket` 连接到 WebSocket 服务器。可以通过 `sendTextMessage()` 发送消息，`textMessageReceived()` 信号接收消息。
2. 服务器端：使用 `QWebSocketServer` 启动 WebSocket 服务器，监听客户端连接。当连接建立后，使用 `QWebSocket` 进行通信。

```cpp
QWebSocket *socket = new QWebSocket();
socket->open(QUrl("ws://example.com"));

connect(socket, &QWebSocket::textMessageReceived, this, &MyClass::onMessageReceived);
connect(socket, &QWebSocket::disconnected, this, &MyClass::onDisconnected);

void MyClass::onMessageReceived(const QString &message) {
    // 处理收到的消息
}
```

### 处理数据的粘包和拆包

**粘包**和**拆包**是网络编程中的常见问题，通常发生在基于流的协议（如 TCP）中，解决这个问题的常见方法是使用自定义的协议进行数据分隔和解析。

常见解决方案：

1. 固定长度数据包：每个数据包有固定的大小，接收方按固定长度读取。优点：实现简单，适合数据量固定的场景。缺点：不能处理长度变化的数据。
2. 使用分隔符：数据包之间使用特定的字符（如 `\n` 或 `\0`）作为分隔符，接收方根据分隔符划分数据包。优点：适用于变长数据包。缺点：需要确保分隔符不会出现在数据中。
3. 数据包头部包含长度信息：在每个数据包的头部添加数据包长度信息，接收方先读取长度字段，再根据长度读取完整数据。优点：适用于变长数据。缺点：需要管理长度字段。

## 数据库编程

### 连接数据库

1. **加载数据库驱动** ：Qt 支持多种数据库（如 SQLite、MySQL、PostgreSQL 等）。需要加载适当的数据库驱动。
2. **创建** **`QSqlDatabase`** **对象** ：通过 `QSqlDatabase::addDatabase()` 来创建一个数据库连接对象，并设置连接类型（例如 SQLite、MySQL）。
3. **设置连接参数** ：配置数据库连接的参数，如数据库文件路径、用户名、密码等。
4. **打开数据库** ：调用 `open()` 方法来打开数据库连接，并检查连接是否成功。

```cpp
void MainWindow::open_db() {
    db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("/Users/akatsuki/account.db");
    if (db.open()) {
        qDebug() << "open db success";
    } else {
        qDebug() << db.lastError().text();
    }
}
```

### 操作 SQLite 数据库

SQLite 是一种轻量级的嵌入式数据库，不需要安装额外的数据库服务，适合存储本地数据。

1. **配置数据库驱动** ：确保 SQLite 驱动已启用（Qt 默认支持 SQLite）。
2. **创建数据库连接** ：通过 `QSqlDatabase::addDatabase("QSQLITE")` 来连接 SQLite 数据库，并指定数据库文件路径。
3. **执行 SQL 语句** ：使用 `QSqlQuery` 执行 SQL 语句进行查询或操作。
4. **使用模型（如`QSqlTableModel`）** ：可以通过 `QSqlTableModel` 直接操作数据库中的表格，简化数据展示和交互。

### 事务处理

https://www.bilibili.com/video/BV17qKbzGEW7/?spm_id_from=333.337.search-card.all.click&vd_source=9d28f0e4734f1bde4c84c3169e3a429d

**数据库的事务处理**：作为单个逻辑工作单元执行的一系列数据库操作。这些操作要么全部成功要么全部失败回滚，绝不存在“部分成功”的中间状态，以此来保证数据的完整性和一致性。事务处理可以通过 `QSqlDatabase` 提供的接口实现。

1. 开始事务：调用 `QSqlDatabase::transaction()` 来启动一个事务。这会确保后续的所有数据库操作都被视为一个单独的事务。
2. 执行数据库操作：在事务中进行一系列数据库操作（如插入、更新、删除等）。这些操作不会立即提交到数据库中。
3. 提交事务：如果所有操作都成功执行，调用 `QSqlDatabase::commit()` 来提交事务，将所有变更保存到数据库。
4. 回滚事务：如果在操作过程中出现错误，可以调用 `QSqlDatabase::rollback()` 来回滚事务，撤销所有变更，恢复到事务开始时的状态。

```cpp
QSqlDatabase db = QSqlDatabase::database(); // 获取默认数据库连接

// 1. 启动事务
if (db.transaction()) {
    QSqlQuery query(db);
    
    // 2. 执行一系列 SQL 操作
    query.exec("INSERT INTO accounts (id, balance) VALUES (1, 100)");
    query.exec("UPDATE accounts SET balance = balance - 100 WHERE id = 2");

    // 3. 检查执行结果并提交或回滚
    if (query.lastError().isValid()) {
        db.rollback(); // 出现错误，撤销所有操作
    } else {
        db.commit();   // 无错误，提交事务生效
    }
}
```

**事务的好处** ：

- 确保数据一致性：即使在系统崩溃或错误发生时，数据也不会被破坏。
- 提供原子性：一组操作要么全部成功，要么全部失败。

**常见应用场景** ：

- 批量插入数据时使用事务可以提高性能。
- 多步操作必须全部成功的情况下使用事务，如转账操作等。

### 数据库的 Model

1. `QSqlQueryModel`：是一个只读模型，用于显示从 SQL 查询返回的结果集。它适合用于显示执行 `SELECT` 查询的结果，通常是一次性的查询结果。不支持直接修改数据，只用于显示。
2. `QSqlTableModel`：用于操作数据库中的某一张表，支持增、删、改操作。它会与数据库中的表进行同步，自动更新显示内容。提供了与视图（如`QTableView`）绑定的能力，可以方便地展示和操作数据库表数据。
3. `QSqlRelationalTableModel`：是 `QSqlTableModel` 的扩展，支持外键关系。适合在表格中展示有关系的多个表格数据。它允许自动处理外键字段的显示，使得操作更为灵活和直观。

