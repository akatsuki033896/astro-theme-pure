---
title: 'Ardupilot part2. Ardupilot基础'
publishDate: 2026-07-20
updatedDate: 2026-07-20
description: 'Ardupilot基础及代码架构'
tags:
  - Ardupilot
  - C++
language: 'Chinese'
---

## 代码架构

![](https://ardupilot.org/dev/_images/ArduPilot_HighLevelArchecture.png)

## `AP_HAL`

`AP_HAL` 层是硬件抽象层，使 ArduPilot 能够移植到多种不同的平台上。在 `libraries/AP_HAL` 目录下有一个顶层 `AP_HAL`，它定义了其余代码与特定板卡功能交互的接口，然后运行在ChibiOS的版本、SITL等其他版本继承该类实现多态，给代码提供相同的接口。

此外，每个板卡类型都有一个对应的 `AP_HAL_XXX` 子目录，例如，基于 STM32 的板卡对应 `AP_HAL_ChibiOS`。

## 代码样例和程序入口

每个代码样例都有 `setup()` 和 `loop()` 函数

- `setup()` 函数在开发板启动时调用，只调用一次，通常用于初始化库。实际的调用来自每个开发板的 HAL 层，因此 `main()` 函数隐藏在 HAL 层内部，并在开发板特定的启动完成后调用 `setup()` 函数。
- `setup()` 函数执行完毕后，`loop()` 函数会被持续调用（由 AP_HAL 层中的主代码调用）。程序的主要工作通常都在 `loop()` 函数中完成。
- 代码最后的 `AP_HAL_MAIN()` 是一个 HAL 宏，它生成声明 C++ 主函数所需的代码，以及 HAL 的任何板级初始化代码。

## 传感器驱动

ArduPilot 支持来自众多不同制造商的各种传感器，支持I2C / SPI / UART / CAN 协议，链接里给出了这些协议的后端样例

### 前后端分离

- 前端：驱动管理层，专注于通用逻辑，如数据解析、单位转换、队列管理、状态管理（例如：获取当前距离、检查传感器是否健康）
- 后端：实际操作层，特定硬件操作，如读取I2C寄存器、配置特定速率、处理中断。

![](https://ardupilot.org/dev/_images/code-overview-sensor-drivers-febesplit.png)

![](https://ardupilot.org/dev/_images/copter-code-overview-architecture2.png)

主线程定时运行，通过驱动程序前端的函数访问最新可用数据

来自传感器的原始数据被采集、转换为标准单位，然后存储在驱动程序内部的缓冲区中,对于使用 I2C 或 SPI 的驱动程序，必须在后台线程中运行，以避免与传感器的高速通信影响主循环的性能。但对于使用UART接口的驱动程序，可以在主线程中安全运行，因为底层串行驱动程序本身会在后台采集数据并包含缓冲区。

例如测距仪驱动程序的前端更新：驱动程序有机会在主线程中执行任何它可能需要的常规处理。每个后端的更新方法都会依次被调用。

```cpp
// AP_RangeFinder.cpp
/*
  update RangeFinder state for all instances. This should be called at
  around 10Hz by main loop
 */
void RangeFinder::update(void)
{
    for (uint8_t i=0; i<num_instances; i++) {
        if (drivers[i] != nullptr) {
            if ((Type)params[i].type.get() == Type::NONE) {
                // allow user to disable a rangefinder at runtime
                state[i].status = Status::NotConnected;
                state[i].range_valid_count = 0;
                continue;
            }
            drivers[i]->update();
        }
    }
#if HAL_LOGGING_ENABLED
    Log_RFND();
#endif
}
```

## UART

定义了8个UART，用 SITL 模拟时可以指定，格式为 `--serialX=uart:<device>:<baudrate>`

还可以指定 TCP 服务器/客户端等，具体查看文档。运行 SITL 脚本命令为

```sh
sim_vehicle.py -v ArduCopter -A "--serial1=uart:/dev/ttyUSB0:115200" --console --map
```

`libraries/AP_HAL/examples/UART_test` 打印消息的案例：

```cpp
/*
  simple test of UART interfaces
 */

#include <AP_HAL/AP_HAL.h>
#include <stdio.h>

void setup();
void loop();

const AP_HAL::HAL& hal = AP_HAL::get_HAL();

/*
  setup one UART at 57600
 */
static void setup_uart(AP_HAL::UARTDriver *uart, const char *name) {
    if (uart == nullptr) {
        // that UART doesn't exist on this platform
        return;
    }
    uart->begin(57600);
}

void setup(void) {
    /*
      start all UARTs at 57600 with default buffer sizes
    */

    hal.scheduler->delay(1000); //Ensure that hal.serial(n) can be initialized

    setup_uart(hal.serial(0), "SERIAL0");  // console
    setup_uart(hal.serial(1), "SERIAL1");  // telemetry 1
    setup_uart(hal.serial(2), "SERIAL2");  // telemetry 2
    setup_uart(hal.serial(3), "SERIAL3");  // 1st GPS
    setup_uart(hal.serial(4), "SERIAL4");  // 2nd GPS
}

static void test_uart(AP_HAL::UARTDriver *uart, const char *name) {
    if (uart == nullptr) {
        // that UART doesn't exist on this platform
        return;
    }
    uart->printf("Hello on UART %s at %.3f seconds\n",
                 name, (double)(AP_HAL::millis() * 0.001f));
}

void loop(void) {
    test_uart(hal.serial(0), "SERIAL0");
    test_uart(hal.serial(1), "SERIAL1");
    test_uart(hal.serial(2), "SERIAL2");
    test_uart(hal.serial(3), "SERIAL3");
    test_uart(hal.serial(4), "SERIAL4");

    // also do a raw printf() on some platforms, which prints to the
    // debug console
    ::printf("Hello on debug console at %.3f seconds\n", (double)(AP_HAL::millis() * 0.001f));

    hal.scheduler->delay(1000);
}

AP_HAL_MAIN();
```

### 硬件流控制

RS232支持硬件流控、软件流控和无流控三种模式，无流控模式仅需要2根串行数据线，硬件流控需要额外的2个信号(RTS和CTS)，Ardupilot使用硬件流控制

参数 `BRD_SER1_RTSCTS` 控制流控制，Values: **0** = disabled (SITL default), **1** = enabled, **2** = auto-detect

### Debug

在 `AP_HAL/AP_HAL_Boards.h` 查看开发板如果有 `HAL_OS_POSIX_IO`，则可以使用 `::printf()`

## 程序入口 `AP_HAL_MAIN()`

包括主函数的宏和主函数的回调函数宏

```cpp
#define AP_HAL_MAIN_CALLBACKS(CALLBACKS) extern "C" { \
    int AP_MAIN(int argc, char* const argv[]); \
    int AP_MAIN(int argc, char* const argv[]) { \
        hal.run(argc, argv, CALLBACKS); \
        return 0; \
    } \
}
```

## Reference

- Sensor Drivers: https://ardupilot.org/dev/docs/code-overview-sensor-drivers.html
- 硬件流控制的概念: https://zhuanlan.zhihu.com/p/89440015
- UART: https://ardupilot.org/dev/docs/learning-ardupilot-uarts-and-the-console.html
