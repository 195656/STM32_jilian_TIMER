# STM32 定时器级联与硬件外部计数项目

本项目基于 **STM32F103C8T6** 开发板，通过 **STM32CubeMX (HAL 库)** 实现。核心目标是利用定时器的硬件级联机制（Timer Cascading）处理外部按键脉冲，构建一个低 CPU 负载、高确定性的硬件计数系统。

## 🛠 核心技术逻辑

### 1. 硬件级消抖 (Hardware Debouncing)
通过配置 TIM2 ETR 的数字滤波器（Clock Filter），在硬件层面过滤机械按键产生的纹波（Ripple），确保计数的**确切性（Determinism）**。

### 2. 定时器级联架构 (Master/Slave Mode)
系统采用双定时器链路：
* **主定时器 (TIM2)**：
    * **时钟源**：外部时钟模式 2 (ETR)，引脚 PA0。
    * **触发输出 (TRGO)**：配置为更新事件（Update Event）。
* **从定时器 (TIM4)**：
    * **从模式**：外部时钟模式 1 (External Clock Mode 1)。
    * **触发源 (Trigger Source)**：ITR1 (内部连接至 TIM2)。

### 3. 计数学理模型
总按键触发次数 $N_{Total}$ 计算公式：
$$N_{Total} = (ARR_{TIM2} + 1) \times (ARR_{TIM4} + 1)$$
当前配置：$ARR_{TIM2}=2, ARR_{TIM4}=1$，即每按下 **6 次**按键，触发一次 TIM4 中断并翻转 LED 状态。

## 📂 项目结构说明
* `Core/Src/tim.c`: 定时器级联与 ETR 参数的具体配置实现。
* `Core/Src/main.c`: 硬件启动逻辑与 OLED 实时状态监控。
* `Core/Src/stm32f1xx_it.c`: 中断服务程序，处理 TIM4 的溢出回调。

## 🚀 启动与运行
1.  **硬件连接**：按键一端接 `PA0`，另一端接 `GND`；LED 接 `PA1`。
2.  **现象**：松开按键时触发计数。通过 OLED 可以观察到 `TIM2->CNT` 在 $[0, 2]$ 之间循环。每当 `TIM2` 归零，`TIM4->CNT` 步进。
3.  **确切性验证**：排除机械抖动干扰，严格执行 6 次动作后 LED 翻转。

---
**项目维护者**：[你的名字]  
**专业背景**：北京交通大学 - 测控技术与仪器
