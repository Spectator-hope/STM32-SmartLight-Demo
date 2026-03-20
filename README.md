# STM32 Multi-Mode PWM LED & OLED Controller
本项目是一个基于 **STM32F10x** 微控制器（Keil MDK 开发环境）的嵌入式交互系统 Demo。主要实现了一个**通过外部中断按键无缝切换多模式 PWM 控制 LED，并结合 7 针 SPI OLED 屏幕实时显示运行状态**的应用程序。
项目展示了典型的前后台系统架构、非阻塞式中断打断机制、屏幕局部刷新优化以及引脚重映射等核心嵌入式开发技巧。
## 🌟 核心特性 (Features)
*   **多模式 PWM 渲染**：支持 4 种状态无缝切换（低亮、高亮、呼吸灯动画、闪烁动画）。
*   **非阻塞式交互打断**：借助 EXTI 外部中断，按键响应不受底层 `Delay` 延时函数阻塞，实现呼吸/闪烁动画的瞬间打断与状态转换。
*   **OLED 防闪烁优化**：引入状态追踪机制 (`previous_mode`)，仅在模式发生改变时触发 SPI 屏幕通讯与局部刷新，大幅提升显示效率，避免全屏擦除带来的闪烁。
*   **规范的工程结构**：底层硬件驱动 (Hardware) 与上层逻辑 (User) 严格分离，提高了代码的可读性与复用性。
*   **IO 口复用技巧**：代码中通过 SWJ 重映射技术安全地禁用了 JTAG（保留 SWD），成功将默认占用引脚（PB4）释放为普通外部中断 IO。
## 📂 工程目录结构 (Directory Structure)
*   **`User/`**: 用户应用程序核心，包含主入口 `main.c` 与业务逻辑控制。
*   **`Hardware/`**: 底层硬件外设驱动库：
    *   `EXTI.c`：外部中断驱动（按键捕获）
    *   `Mode.c` / `PWM.c`：LED 工作模式与 PWM 发生器引擎
    *   `OLED_SPI.c`：SPI OLED 屏幕底层驱动
    *   `Key.c` / `LED.c`：基础外设控制
*   **`System/`**: 系统级基础设施（如软件 Delay 延时等）。
*   **`Start/` & `Library/`**: STM32 标准外设库与设备启动文件。
## 🔌 硬件引脚分配 (Pin Definitions)
### 1. LED 控制输出
| 外设 | 引脚 | 说明 |
| :--- | :--- | :--- |
| **LED 红** | `PA0` | **核心 PWM 控制输出** (TIM2_CH1)，用于呼吸灯模式渲染 |
| **LED 蓝** | `PA3` | 基础 GPIO 点亮控制 |
| **LED 黄** | `PA6` | 基础 GPIO 点亮控制 |
### 2. 按键与中断
| 外设 | 引脚 | 说明 |
| :--- | :--- | :--- |
| **主控按键** | `PB4` | **核心 EXTI 外部按键中断** (`EXTI_Line4`)，用于切换系统模式 |
| 预留按键 1 | `PB0` | 轮询式按键（未启用中断） |
| 预留按键 2 | `PB7` | 轮询式按键 |
> **注意**：`PB4` 默认为 JTAG 的 NJTRST 引脚。本项目在初始化阶段已配置 `GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable, ENABLE)` 以确保 PB4 可作为普通 GPIO 使用。
### 3. SPI OLED 屏幕 (七针)
| OLED 引脚 | STM32 引脚 | 说明 |
| :--- | :--- | :--- |
| **CS** | `PB11` | 片选信号 (Chip Select) |
| **RES** | `PB12` | 复位信号 (Reset) |
| **SCL / D0**| `PB13` | SPI 时钟 (Clock) |
| **DC** | `PB14` | 数据/命令选择 (Data/Command) |
| **SDA / D1**| `PB15` | SPI 数据输出 (MOSI) |
## 🚀 快速上手 (Getting Started)
1. 克隆或下载本仓库代码到本地。
2. 使用 **Keil uVision5 (MDK-ARM)** 打开工程文件。
3. 按照上述引脚分配清单连接好单片机、LED、按键以及 7 针 SPI OLED 屏幕。
4. 编译工程，确保无报错使用 ST-Link 或 DAP-Link 下载到 STM32F103 开发板中。
5. 按下接在 PB4 的按键，观察 LED 在 **低亮 -> 高亮 -> 呼吸 -> 闪烁** 四种模式中切换，并且 OLED 屏幕会同步显示正在运行的 Mode。
## 📝 开发日记与分析 
有关项目的深度技术总结（例如状态机设计、前后台架构解析等），请参考 `User/` 目录下的 [Project_Analysis.md] 及端口配置文件
