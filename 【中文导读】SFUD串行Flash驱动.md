# SFUD（串行 Flash 通用驱动库）学习导读

> 目录：`02-文件系统与存储/SFUD-串行Flash通用驱动库`
> 来源：https://github.com/armink/SFUD （压缩包：SFUD-master.zip）

## 这是什么

SFUD（Serial Flash Universal Driver）是 armink 的另一个经典库：**一套代码驱动所有 SPI NOR Flash**（W25Q、GD25、MX25、EN25...）。原理是启动时读 Flash 的 JEDEC SFDP 标准参数表自动识别型号；老芯片不支持 SFDP 时回退到内置参数表。配合 littlefs/FlashDB/EasyFlash 使用，就是"存储方案全家桶"。代码量小、注释中文清晰。

## 目录导读

| 目录/文件 | 内容 |
| --- | --- |
| `sfud/src/` | 核心实现：`sfud.c`（主流程）、`sfud_sfdp.c`（SFDP 参数表解析，精华）、`sfud_flash.c`（设备管理） |
| `sfud/inc/` | 头文件 |
| `sfud/port/` | 移植接口示例（SPI 读写、延时） |
| `demo/` | 四个平台：`stm32f10x_non_os/`（裸机 Keil）、`stm32l475_non_os_qspi/`（QSPI）、`stm32f2xx_rtt/`（RT-Thread）、`esp32_ext_spi_flash/` |
| `docs/` | 中文文档（设计思路 + 移植指南） |

## 学习路径（建议）

1. 读 `docs/zh/` 设计文档，理解 SPI Flash 的命令集（读/写/擦除/使能写/读状态）。
2. 读 `sfud_sfdp.c`：**JEDEC SFDP 是行业标准，看它怎么解析厂商参数表**——读完你查任何 Flash 手册都会顺眼。
3. 跑 `demo/stm32f10x_non_os`：裸机最直观，没有 RTOS 干扰。
4. 组合拳实验：SFUD 识别 Flash → 挂 littlefs → 存文件 → 再叠 FlashDB 存 KV。

## 学习要点

1. **SPI Flash 三大命令体系**：标准 SPI、Dual/Quad Fast Read、QSPI——看 `sfud.c` 怎么抽象。
2. **写前必须擦除**：Flash 物理特性决定"只能 1→0"，写流程 = 擦扇区 + 页编程，理解 256 字节页限制。
3. **状态寄存器轮询**：`sfud_wait_for_write_completed` 在等什么（WIP 位）。
4. **SFDP 表结构**：基本头表 + 参数表，双字（DWORD）按位解析——练二进制解析能力的好材料。

## 建议购买/准备

- **推荐硬件**：
  - W25Q64 / W25Q128 SPI Flash 模块（¥3-8，必备）
  - STM32F103C8T6 蓝pill（¥15-30）或 ESP32（¥20-40）
  - 杜邦线若干
- 下载对应型号 datasheet 对照读（W25Q64 datasheet 全网公开）。

## 学习目标（自测）

- [ ] 能用逻辑分析仪（或看波形图）指出读 ID、擦除、页编程三种操作
- [ ] 能说清 SFDP 自动识别的完整流程
- [ ] 完成 SFUD + littlefs 组合挂载并读写文件
