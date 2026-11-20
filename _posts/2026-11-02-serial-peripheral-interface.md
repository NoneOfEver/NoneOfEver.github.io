---
title: "SPI 学习笔记：单线制、双线制、四线制 SPI 的工作原理、流程与主要参数"
description: "SPI signaling, timing, and the differences between single, dual, and quad data lines."
lang: "zh-CN"
permalink: "/blog/serial-peripheral-interface/"
---
SPI（Serial Peripheral Interface，串行外设接口）是一种同步串行通信总线，常用于 MCU 与 Flash、PSRAM、ADC/DAC、传感器、显示屏等外设之间的数据交换。

这篇笔记里的“单线制、双线制、四线制”按**数据线数量**来理解：

| 类型 | 常见英文 | 数据线 | 每个时钟传输位数 |
| --- | --- | --- | --- |
| 单线制 SPI | Standard SPI / Single SPI / 1-bit SPI | IO0 或 MOSI/MISO 中的一根数据通道 | 1 bit |
| 双线制 SPI | Dual SPI / 2-bit SPI | IO0、IO1 | 2 bit |
| 四线制 SPI | Quad SPI / QSPI / 4-bit SPI | IO0、IO1、IO2、IO3 | 4 bit |

注意：很多资料也会把 `SCLK + CS + MOSI + MISO` 叫作“四线 SPI”，这是按**总信号线数量**来数；而在 Flash、PSRAM、QSPI 控制器语境下，“四线制 SPI”通常指 QSPI，也就是 4 根数据线并行传输。本文采用后者。

## 1. SPI 的基础信号

无论是单线、双线还是四线 SPI，一般都离不开两个控制信号：

| 信号 | 常见名称 | 方向 | 作用 |
| --- | --- | --- | --- |
| SCLK | SCK / CLK | 主机输出 | 串行时钟，由主机产生 |
| CS | SS / NSS / CS# | 主机输出 | 片选信号，通常低电平有效 |

传统 SPI 还会使用：

| 信号 | 常见名称 | 方向 | 作用 |
| --- | --- | --- | --- |
| MOSI | SDI / DI / IO0 | 主机输出，从机输入 | 主机向从机发送数据 |
| MISO | SDO / DO / IO1 | 从机输出，主机输入 | 从机向主机返回数据 |

在 Dual SPI 和 QSPI 中，传统的 `MOSI/MISO` 往往会复用为 `IO0/IO1`，再加上 `IO2/IO3` 组成多数据线传输。

## 2. SPI 时钟模式

SPI 的采样时序由 `CPOL` 和 `CPHA` 决定：

| 模式 | CPOL | CPHA | SCLK 空闲电平 | 常见理解 |
| --- | --- | --- | --- | --- |
| Mode 0 | 0 | 0 | 低电平 | 第一个边沿采样，第二个边沿改变数据 |
| Mode 1 | 0 | 1 | 低电平 | 第一个边沿改变数据，第二个边沿采样 |
| Mode 2 | 1 | 0 | 高电平 | 第一个边沿采样，第二个边沿改变数据 |
| Mode 3 | 1 | 1 | 高电平 | 第一个边沿改变数据，第二个边沿采样 |

主机的 SPI Mode 必须和从机数据手册一致，否则会出现读写错位、命令无法识别或数据偶发错误。

## 3. 单线制 SPI

单线制 SPI 指数据阶段只使用 1 根数据线传输。它可以是传统 SPI 的 `MOSI/MISO` 单 bit 通道，也可以是 Flash 中的 1-1-1 模式。

### 3.1 信号连接

常见连接如下：

```text
Master SCLK  ---->  Device SCLK
Master CS    ---->  Device CS
Master IO0   ---->  Device IO0    写入或命令阶段
Master IO1   <----  Device IO1    读取阶段，传统 SPI 常用
```

如果是半双工单线，也可能只用一根 `SDIO`：

```text
Master SCLK  ---->  Device SCLK
Master CS    ---->  Device CS
Master SDIO  <----> Device SDIO
```

### 3.2 工作原理

单线制每个 `SCLK` 周期传输 1 bit。

在传统 SPI 中：

- 主机通过 `MOSI/IO0` 输出命令、地址或写入数据。
- 从机通过 `MISO/IO1` 返回读取数据。
- `MOSI` 和 `MISO` 分开时可以全双工，但很多存储器读写协议本身仍按命令、地址、数据阶段顺序执行。

在半双工单线中：

- 写阶段由主机驱动 `SDIO`。
- 读阶段主机释放 `SDIO`，从机驱动 `SDIO`。
- 方向切换处可能需要 turnaround cycle，避免双方同时驱动总线。

### 3.3 典型写流程

1. 主机拉低 `CS`。
2. 主机按指定 SPI Mode 输出写命令。
3. 主机继续输出地址。
4. 主机继续输出写入数据。
5. 主机拉高 `CS`，结束传输。

```text
CS      ____-------------------------____
SCLK        _-_-_-_-_-_-_-_-_-_-_-_
IO0         CMD   ADDRESS   WRITE_DATA
方向        M->D  M->D      M->D
```

### 3.4 典型读流程

1. 主机拉低 `CS`。
2. 主机发送读命令。
3. 主机发送地址。
4. 主机输出 dummy cycles，给器件准备数据。
5. 从机通过单根数据通道返回数据。
6. 主机拉高 `CS`。

```text
CS      ____--------------------------------____
SCLK        _-_-_-_-_-_-_-_-_-_-_-_-_-_-_
IO0         CMD   ADDRESS   DUMMY
IO1/SDIO    X     X         READ_DATA
```

### 3.5 特点

- 优点：兼容性最好、时序最简单、硬件支持最广。
- 缺点：吞吐量最低，每个时钟只传 1 bit。
- 适用场景：低速传感器、寄存器配置、普通 SPI Flash 基础读写、简单外设控制。

## 4. 双线制 SPI：Dual SPI

双线制 SPI 通常指 Dual SPI，即使用 `IO0` 和 `IO1` 两根数据线并行传输数据。它常见于 SPI Flash、PSRAM 和部分显示接口。

### 4.1 信号连接

```text
Master SCLK  ---->  Device SCLK
Master CS    ---->  Device CS
Master IO0   <----> Device IO0
Master IO1   <----> Device IO1
```

在很多 Flash 中：

- 普通 SPI 模式下，`IO0` 是 `MOSI`，`IO1` 是 `MISO`。
- Dual SPI 模式下，`IO0` 和 `IO1` 都作为数据线使用。

### 4.2 工作原理

Dual SPI 每个 `SCLK` 周期传输 2 bit，理论吞吐量约为单线制的 2 倍。

常见阶段为：

1. 命令阶段：通常仍用 1 根线发送命令。
2. 地址阶段：可能用 1 根线，也可能用 2 根线。
3. dummy 阶段：主机输出空时钟，等待器件准备数据。
4. 数据阶段：`IO0` 和 `IO1` 同时传输数据。

不同指令会规定不同的线宽组合，例如 `1-1-2` 表示命令 1 线、地址 1 线、数据 2 线；`1-2-2` 表示命令 1 线、地址 2 线、数据 2 线。

### 4.3 典型读流程

1. 主机拉低 `CS`。
2. 主机发送 Dual Read 命令。
3. 主机发送地址。
4. 主机输出 dummy cycles。
5. 从机通过 `IO0/IO1` 输出数据。
6. 主机每个时钟采样 2 bit。
7. 主机拉高 `CS`。

```text
CS      ____---------------------------------------____
SCLK        _-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_
IO0         CMD/ADDR...  DUMMY   D0 D2 D4 D6 ...
IO1         X/ADDR...    DUMMY   D1 D3 D5 D7 ...
```

### 4.4 典型写流程

Dual SPI 写入是否支持 2 线数据阶段，取决于具体芯片。有的 Flash 只支持 Dual Read，不支持 Dual Program。

如果支持双线写入，流程通常是：

1. 主机拉低 `CS`。
2. 主机发送写使能或写命令。
3. 主机发送地址。
4. 主机通过 `IO0/IO1` 并行输出写入数据。
5. 主机拉高 `CS`。
6. 等待器件内部编程完成。

### 4.5 特点

- 优点：比单线制吞吐量高，通常只增加较少引脚成本。
- 缺点：协议配置更复杂，需要关注命令线宽、地址线宽、数据线宽和 dummy cycles。
- 适用场景：需要比普通 SPI 更快读数据，但还不需要 QSPI 的场景。

## 5. 四线制 SPI：Quad SPI / QSPI

四线制 SPI 指 QSPI，使用 `IO0~IO3` 四根数据线并行传输。它常用于高速 SPI Flash、PSRAM、显示刷新、外部存储器映射和 XIP（Execute In Place）。

### 5.1 信号连接

```text
Master SCLK  ---->  Device SCLK
Master CS    ---->  Device CS
Master IO0   <----> Device IO0
Master IO1   <----> Device IO1
Master IO2   <----> Device IO2
Master IO3   <----> Device IO3
```

在很多 SPI Flash 中：

- `IO0` 兼容传统 SPI 的 `MOSI`。
- `IO1` 兼容传统 SPI 的 `MISO`。
- `IO2` 可能复用 `WP#`。
- `IO3` 可能复用 `HOLD#` 或 `RESET#`。

因此启用 QSPI 前，常常需要先用普通 SPI 写状态寄存器或配置寄存器，打开 Quad Enable 位。

### 5.2 工作原理

QSPI 每个 `SCLK` 周期最多传输 4 bit，理论吞吐量约为单线制 SPI 的 4 倍。

典型阶段为：

1. 命令阶段：可能是 1 线，也可能是 4 线。
2. 地址阶段：可能是 1 线、2 线或 4 线。
3. 模式位阶段：某些 Flash 需要 mode bits 或 continuous read 配置。
4. dummy 阶段：输出若干空时钟，等待器件准备数据。
5. 数据阶段：通过 `IO0~IO3` 并行传输数据。

常见线宽写法：

| 写法 | 含义 |
| --- | --- |
| 1-1-4 | 命令 1 线，地址 1 线，数据 4 线 |
| 1-4-4 | 命令 1 线，地址 4 线，数据 4 线 |
| 4-4-4 | 命令 4 线，地址 4 线，数据 4 线 |

### 5.3 典型读流程

1. 主机拉低 `CS`。
2. 主机发送 Quad Read 命令。
3. 主机发送地址。
4. 必要时发送 mode bits。
5. 主机输出 dummy cycles。
6. 从机通过 `IO0~IO3` 输出数据。
7. 主机每个时钟采样 4 bit。
8. 主机拉高 `CS`。

```text
CS      ____------------------------------------------------____
SCLK        _-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_
IO0         CMD/ADDR...  DUMMY   D0  D4  D8  D12 ...
IO1         X/ADDR...    DUMMY   D1  D5  D9  D13 ...
IO2         X/ADDR...    DUMMY   D2  D6  D10 D14 ...
IO3         X/ADDR...    DUMMY   D3  D7  D11 D15 ...
```

### 5.4 典型写流程

QSPI 写入流程通常包括：

1. 主机发送 Write Enable 命令。
2. 主机拉低 `CS`，发送 Quad Page Program 命令。
3. 主机发送地址。
4. 主机通过 `IO0~IO3` 输出待写入数据。
5. 主机拉高 `CS`。
6. 轮询状态寄存器，等待 busy 位清零。

并不是所有器件都支持 4 线写入；有些器件只支持 4 线读取，写入仍使用 1 线。

### 5.5 特点

- 优点：吞吐量高，适合外部 Flash、PSRAM、图像数据和 XIP。
- 缺点：配置复杂，需要关注 Quad Enable、dummy cycles、采样延迟、驱动强度和 PCB 走线质量。
- 适用场景：大容量代码存储、高速资源读取、显示屏数据刷新、外部 RAM 扩展。

## 6. 主要参数

设计或调试 SPI 时，重点确认下面这些参数。

### 6.1 时钟频率

`SCLK` 频率决定传输速度，但不能只看控制器最大频率，还要看从机规格、PCB 走线、线缆长度、电平转换器和负载电容。

频率过高时，常见现象包括读数不稳定、偶发 bit 错误、边沿过慢、振铃或毛刺。

### 6.2 SPI Mode

确认 `CPOL` 和 `CPHA`。模式错误时，数据可能整体偏移 1 bit，或者命令完全无法识别。

### 6.3 bit 顺序

大多数 SPI 器件使用 MSB first，也就是最高位先传。少数器件支持或要求 LSB first。

### 6.4 数据线宽

需要区分命令、地址、数据三个阶段的线宽：

- `1-1-1`：命令、地址、数据都使用 1 根线。
- `1-1-2`：命令 1 线，地址 1 线，数据 2 线。
- `1-1-4`：命令 1 线，地址 1 线，数据 4 线。
- `1-4-4`：命令 1 线，地址 4 线，数据 4 线。
- `4-4-4`：命令、地址、数据都使用 4 根线。

### 6.5 数据宽度

常见数据宽度是 8 bit，也可能是 16 bit、24 bit、32 bit 或不规则 bit 长度。

例如 ADC 可能输出 12/16/24 bit，Flash 命令通常以 8 bit 为单位，部分显示屏可能使用 9 bit SPI 区分命令和数据。

### 6.6 CS 时序

`CS` 通常低电平有效。需要检查：

- `CS` 拉低后到第一个时钟边沿的建立时间。
- 最后一个时钟边沿到 `CS` 拉高的保持时间。
- 两次传输之间 `CS` 拉高的最小间隔。
- 一个完整命令期间 `CS` 是否必须保持低电平。

### 6.7 dummy cycles

高速读取时，命令和地址之后通常需要 dummy cycles。dummy cycles 越多，器件越有时间准备数据，但有效吞吐率会下降。

### 6.8 turnaround cycle

单线半双工、Dual SPI、QSPI 可能存在方向切换。turnaround cycle 用于避免主机和从机同时驱动同一根 IO。

### 6.9 IO 电平

确认主机和从机 IO 电平是否一致，例如 1.8 V、2.5 V、3.3 V 或 5 V。高速 SPI 下还要注意电平转换器的传播延迟和带宽。

### 6.10 Quad Enable 与复用引脚

QSPI 器件常需要打开 Quad Enable 位。由于 `IO2/IO3` 可能复用 `WP#`、`HOLD#` 或 `RESET#`，硬件连接和上拉配置也要满足数据手册要求。

### 6.11 采样延迟和驱动强度

高速 Dual SPI 和 QSPI 中，可能需要配置：

- sample delay：调整主机采样数据的时刻。
- drive strength：调整芯片 IO 输出驱动能力。
- dummy cycles：随频率升高而增加。

频率越高，越要关注真实波形质量和采样裕量。

## 7. 三种线制对比

| 类型 | 数据线 | 常见模式 | 每时钟数据量 | 主要优点 | 主要缺点 |
| --- | --- | --- | --- | --- | --- |
| 单线制 SPI | IO0 或 MOSI/MISO 单 bit 通道 | 1-1-1 | 1 bit | 简单、兼容性好 | 吞吐量最低 |
| 双线制 SPI | IO0、IO1 | 1-1-2 / 1-2-2 | 2 bit | 速度比单线高 | 指令和线宽配置更复杂 |
| 四线制 SPI / QSPI | IO0、IO1、IO2、IO3 | 1-1-4 / 1-4-4 / 4-4-4 | 4 bit | 高吞吐，适合外部存储 | 引脚更多，调试更复杂 |

## 8. 调试建议

1. 先用单线制普通 SPI 确认器件 ID、状态寄存器和基础读写正常。
2. 再打开 Dual SPI 或 QSPI 模式，不要一开始就上最高频率。
3. 从低 SCLK 频率开始，例如 100 kHz、1 MHz，再逐步提高。
4. 用逻辑分析仪确认 `CS`、`SCLK`、`IO0~IO3` 的命令、地址、dummy 和数据阶段。
5. 读取失败时，重点检查 SPI Mode、dummy cycles、地址线宽、数据线宽。
6. QSPI 不工作时，重点检查 Quad Enable 位、`IO2/IO3` 复用功能、上拉和硬件连线。
7. 高速下出现偶发错误时，降低频率、增加 dummy cycles、调整采样延迟或检查 PCB 波形。

## 9. 总结

按数据线数量来分，SPI 可以理解为单线制、双线制和四线制。单线制每个时钟传 1 bit，最简单也最兼容；双线制 Dual SPI 每个时钟传 2 bit，适合中等速度提升；四线制 QSPI 每个时钟传 4 bit，适合高速 Flash、PSRAM、XIP 和大数据量读取。

实际工程中，关键不只是“几根线”，还要确认每个阶段的线宽组合、SPI Mode、时钟频率、CS 时序、dummy cycles、turnaround cycle、IO 电平、Quad Enable 和采样延迟。多数 SPI 问题都来自这些参数和器件手册没有完全对齐。
