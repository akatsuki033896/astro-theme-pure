---
title: 'Qt part.4: 多线程编程'
publishDate: 2026-06-06
updatedDate: 2026-06-06
description: '天天被拷打线程池烦都烦死了'
tags:
  - Qt
  - C++
language: 'Chinese'
---

## 实现多线程的方法

1. `QThread` 类 ：继承 `QThread` 并重写其 `run()` 方法，创建和管理线程。
2. `QObject::moveToThread()` ：将一个对象移到另一个线程中运行，通常配合事件循环使用，适用于 GUI 与工作线程之间的分离。
3. `QtConcurrent` ：这是 Qt 提供的并行编程框架，通过较为简单的接口（如 `QtConcurrent::run()` ）来并行执行函数。
4. `QThreadPool` ：管理一组线程，允许将任务提交给线程池，而不需要手动创建和管理线程。适用于任务较为简单且数量较多的情况。

### `QThread`

`QThread` 是 Qt 用来创建和管理线程的类，通常的使用步骤如下：

1. 继承 `QThread` 并重写 `run()`：在 `run()` 中编写需要在线程中执行的代码
2. 在主线程或别的地方创建线程类的实例
3. 启动线程：通过调用 `start()` 启动线程，`start()` 会自动调用 `run()` 
4. 如果需要线程交互、获得线程执行结果，使用信号槽机制实现线程间通信
5. 执行完毕时调用 `quit()`

```cpp
class MyThread : public QThread {
    Q_OBJECT

public:
    void run() override {
        // 执行耗时操作
    }
};

MyThread *thread = new MyThread();
thread->start();  // 启动线程
```

1. 等待线程完成：可以使用 `wait()` 方法来阻塞主线程，直到子线程完成执行。
2. 停止线程：可以使用 `terminate()` 强制停止线程（不推荐），或者通过设置标志位安全地结束线程。

继承 `QThread` 的方法适合在新线程中执行一些独立的工作逻辑，当线程与 `QObject` 结合时（如 GUI 线程中的工作对象），不建议直接继承 `QThread`。因为 `QObject` 不允许在多个线程中同时访问。

### `moveToThread`

将一个 `QObject` 派生的对象移到一个新的线程中，这样该对象的槽函数就可以在该线程中执行。适用于需要将某个对象放入工作线程，但不需要继承 `QThread`。可以避免 `QObject` 在多个线程中共享的问题，线程管理更灵活，但需要手动管理线程的启动、停止和对象的销毁。

## 线程间通信

1. 信号与槽机制：最常用的线程间通信方式。Qt 的信号与槽机制支持线程间的通信，线程 A 发射信号，线程 B 连接槽，并自动在线程之间传递信号。
2. 事件队列：通过 `QCoreApplication::postEvent()` 将事件发送到目标线程的事件队列，目标线程可以在 `event()` 中处理事件。
3. `QMutex` 和 `QSemaphore`：用于多线程间的同步，确保线程安全地访问共享资源。
4. `QWaitCondition` 和 `QMutex`：用于线程间的同步，允许一个线程等待某个条件满足再继续执行。

### 信号槽在多线程的工作方式

Qt 的信号与槽机制可以跨线程工作。Qt 的信号与槽机制支持线程间的通信，线程 A 发射信号，线程 B 连接槽，并自动在线程之间传递信号。当信号发射方和槽接收方在不同线程中时，Qt 会根据连接类型自动处理线程间的通信。

1. `Qt::AutoConnection`（默认）：自动决定使用哪种方式。当信号和槽在同一线程时，直接调用槽；如果在不同线程，则使用队列连接。
2. `Qt::DirectConnection`：信号与槽在同一线程时直接调用槽函数；如果在不同线程，信号会被丢弃。
3. `Qt::QueuedConnection`：即使在同一线程中也会将信号放入接收线程的事件队列中进行处理，适用于跨线程的信号槽连接。
4. `Qt::BlockingQueuedConnection`：与 `QueuedConnection` 类似，但发射信号的线程会被阻塞，直到槽函数执行完毕。

`Qt::DirectConnection` 适合同线程之间的通信， `Qt::QueuedConnection` 适合跨线程的通信。

## 同步与互斥

1. `QMutex`：互斥锁，确保在同一时刻只有一个线程可以访问共享资源。适用于线程需要独占资源的场景。
2. `QReadWriteLock`：读写锁，允许多个线程同时读取共享资源，但当有线程写入时，其他线程（包括读线程）不能访问。适用于读多写少的场景，如缓存读取。
3. `QSemaphore`：信号量，用于控制资源的访问数量，允许有限多个数量的线程访问资源。适用于有限资源的场景，如连接池、线程池等。

```cpp
QMutex m_Mutex;
m_Mutex.lock();
m_Mutex.unlock();
```

类似 `lock_guard` 和 `unique_lock`，也有RAII的锁 `QMutexLockermutexLocker`，从声明处开始（在构造函数中加锁），出了作用域自动解锁（在析构函数中解锁）

类似条件变量的 `QWaitCondition`

```cpp
QWaitCondtion m_WaitCondition;
m_WaitConditon.wait(&m_muxtex, time);
m_WaitCondition.wakeAll();
```

### 死锁避免

1. 避免循环锁定：确保多个线程获取资源时遵循统一的顺序。比如线程 A 获取资源 1 后再获取资源 2，线程 B 获取资源 2 后再获取资源 1，避免交叉等待。
2. 使用超时机制：可以在锁定时设置超时（如 `QMutex::tryLock()`），如果获取锁失败则放弃等待，避免长时间等待。
3. 死锁检测：定期检查锁定情况，通过监控机制检测并及时处理潜在死锁。
4. 避免嵌套锁：尽量避免多个线程同时请求多个资源，减少死锁发生的可能。

### 线程安全

1. `QMutex`：使用互斥锁来保护共享资源，确保每次只有一个线程可以访问该资源。
2. `QReadWriteLock`：在读多写少的场景下，允许多个线程同时读取资源，但写线程会独占资源。
3. `QAtomic` 类：提供对原子操作的支持，确保对数据的访问是线程安全的（如 `QAtomicInt`）。
4. 信号槽机制：Qt 的信号槽机制本身是线程安全的，可以用于线程间安全地传递消息。

## `QtConcurrent`：并行计算框架

QtConcurrent是 Qt 提供的并行计算框架，提供了高层次的并行操作接口，简化了多线程编程。

- **并行处理任务** ：如图像处理、文件处理等，能够将大任务拆分成多个小任务并行执行。
- **数据处理和计算密集型任务** ：通过简单的接口（如 `QtConcurrent::run()` ）来并行执行 CPU 密集型操作，显著提高效率。
- **高效的线程池管理** ： `QtConcurrent` 使用线程池来管理线程，避免直接创建和销毁线程的开销。

```cpp
QtConcurrent::run(someFunction);
```

## `QThreadPool`：线程池

Qt的线程池，使用 `start()` 开始指定的任务函数，线程池全局实例会接管函数的生命周期

```cpp
class HelloWorldTask : public QRunnable {
    void run() override {
        qDebug() << "Hello world from thread" << QThread::currentThread();
    }
};

int main() {
    //...
    HelloWorldTask *hello = new HelloWorldTask();
    // QThreadPool takes ownership and deletes 'hello' automatically
    QThreadPool::globalInstance()->start(hello);
    //...
}
```

`start()`有两个函数原型:

```cpp
void start(QRunnable *runnable, int priority = 0);
void start(Callable &&callableToRun, int priority = 0);
```

预留一个线程并用它来运行可运行对象，除非该线程会导致当前线程数超过 `maxThreadCount()`。在这种情况下，可运行对象会被添加到运行队列中。可以使用 `priority` 参数来控制运行队列的执行顺序。

[官方文档的示例代码](https://doc.qt.io/qt-6/qrunnable.html#details)定义一个任务类继承 `QRunnable` ，实际上 `QRunnable` 类可以是普通函数，也可以是lambda函数， `QRunnable` 提供了一个静态工厂函数 `create` 在内部直接从lambda表达式构建 `QRunnable` 对象

```cpp
QThreadPool* tp = QThreadPool::globalInstance();
void timer_task() {
    tp->start([&]() {
	        //...
	    }
	);
}
```

线程池全局实例指针 `tp` 启动一个lambda函数作为线程任务。