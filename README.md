基于武汉芯源半导体 **CW32L031** 微控制器的海洋物联网（Marine IoT）数据通信系统，实现岸站与海上信标之间的双向数据传输。

## 功能概述

- **以太网通信** — 通过 CH390H 以太网芯片建立 TCP/IP 客户端-服务器链路，与上位机服务器交换数据
- **铱星卫星通信** — 基于铱星 SBD（Short Burst Data）短报文服务，实现远程海上信标数据回传及指令下发
- **GPS 定位采集** — 获取实时经纬度、速度、航向等定位信息
- **海水检测** — 监测设备是否入水
- **自定义通信协议** — 支持 WZXX（位置信息）、FKXX（反馈信息）、TXSQ（通信申请）、YXSQ/YXGL（运行申请/管理）、ZJXX（自检信息）等多种帧格式
- **LED 状态指示** — 板载 LED 驱动

## 系统架构

```
┌──────────────┐      以太网/TCP      ┌──────────────┐
│  上位机服务器  │ ◄──────────────────► │  岸站 (receive) │
│  (Server)    │                      │  CW32L031     │
└──────────────┘                      └──────┬───────┘
                                             │ 铱星 SBD
                                             │ 卫星链路
                                      ┌──────┴───────┐
                                      │  信标 (send)   │
                                      │  CW32L031     │
                                      └──────────────┘
```

项目包含两个子工程：
| 工程 | 目录 | 角色 |
|------|------|------|
| **receive** | `receive/UART_Printf/UART_Printf/` | 岸站端：以太网服务器桥接 + 铱星 SBD 链路管理 |
| **send** | `send/UART_Printf/UART_Printf1/` | 信标端：传感器数据采集 + 铱星 SBD 上报 |

## 硬件平台

| 组件 | 型号/说明 |
|------|----------|
| 主控 MCU | CW32L031 (ARM Cortex-M0+) |
| 以太网芯片 | CH390H (SPI 接口) |
| 卫星通信模块 | 铱星 9602/9603 SBD 模块 (UART AT 指令) |
| GPS 模块 | UART 接口 GPS |
| 开发环境 | Keil MDK / IAR EWARM |

## 目录结构

```
CW32test2/
├── receive/UART_Printf/UART_Printf/    # 岸站端工程
│   ├── bsp/                            # 板级驱动
│   │   ├── ch390h.c/h                  # CH390H 以太网驱动
│   │   ├── net_tcp_client.c/h          # TCP 客户端协议栈
│   │   ├── yxsbd.c/h                   # 铱星 SBD 通信驱动
│   │   ├── agreement.c/h               # 自定义通信协议
│   │   ├── gps.c/h                     # GPS 数据采集
│   │   ├── seawater.c/h                # 海水检测
│   │   ├── leddrv.c/h                  # LED 驱动
│   │   ├── datatreating.c/h            # 数据处理与调试输出
│   │   └── time.c/h                    # 定时任务
│   ├── USER/
│   │   ├── inc/                        # 用户头文件
│   │   └── src/                        # 用户源码 (main.c, gpio.c, usart.c 等)
│   ├── MDK/                            # Keil MDK 工程文件
│   └── EWARM/                          # IAR EWARM 工程文件
│
└── send/UART_Printf/UART_Printf1/      # 信标端工程 (目录结构同上)
    ├── bsp/
    ├── USER/
    ├── MDK/
    └── EWARM/
```

## 通信协议

协议帧定义见 `bsp/agreement.h`，主要帧类型：

| 帧类型 | 说明 | 长度 |
|--------|------|------|
| WZXX | 位置信息上报 (经纬度/速度/航向) | 27 字节 |
| FKXX | 反馈确认信息 | 16 字节 |
| TXSQ | 通信申请 | 18 字节起 |
| YXSQ | 运行申请 | 16 字节起 |
| YXGL | 运行管理 | 9 字节 |
| YXZK | 运行状况 | 9 字节 |
| ZJXX | 自检信息 | 12 字节 |
| ZJZK | 自检状况 | 17 字节 |

数据采用大端序（Big-Endian），异或校验。

## 快速开始

### 1. 环境准备

- **Keil MDK** (推荐 5.x) 或 **IAR EWARM**
- 安装 CW32L031 器件包（Device Family Pack）

### 2. 编译

用 Keil MDK 打开对应工程文件：

- 岸站端：`receive/UART_Printf/UART_Printf/MDK/Project.uvprojx`
- 信标端：`send/UART_Printf/UART_Printf1/MDK/Project.uvprojx`

编译输出位于 `MDK/output/exe/` 目录下。

### 3. 烧录

使用 CW-Writer 或兼容的 SWD 调试器（如 J-Link、DAP-Link）烧录生成的 `.hex` 文件。

### 4. 配置

主要配置项（`bsp/net_tcp_client.h`）：

```c
#define NET_LOCAL_IP      192.168.1.50    // 本地 IP
#define NET_GATEWAY_IP    192.168.1.1     // 网关
#define NET_SERVER_IP     192.168.1.100   // 上位机服务器 IP
#define NET_SERVER_PORT   5000            // 服务器端口
```

铱星通信参数（`bsp/yxsbd.h`）：
```c
#define SBD_MAX_LEN       340             // SBD 单次最大传输字节
```

## 许可证

武汉芯源半导体有限公司示例代码许可。

---

**Keywords**: CW32L031, ARM Cortex-M0+, Iridium SBD, CH390H, Marine IoT, TCP/IP, 海洋物联网, 铱星通信, 嵌入式
