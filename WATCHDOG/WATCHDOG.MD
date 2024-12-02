
**本频道只提供STM32-HAL库编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# STM32 看门狗

- Stm32内部带有独立看门狗、窗口看门狗。独立看门狗使用时，只要喂狗时间不大于设定的时间就可以，是最常用的看门狗用法；而窗口看门狗，需要喂狗时间在一个上下限范围内，才能重置看门狗计时器。
# STM32独立看门狗(IWDG)

---

## 1. 独立看门狗（IWDG）概述

STM32 提供了两种看门狗：
- **独立看门狗（IWDG）**：与系统时钟无关，基于内部低速振荡器（LSIRC）工作，适用于系统出问题后进行复位操作。
- **窗口看门狗（WWDG）**：需要在一个特定的窗口时间内进行喂狗操作，否则会复位系统。与系统时钟相关，通常用于高精度的超时检测。

本文主要介绍 **独立看门狗（IWDG）** 的配置和使用方法。[此处跳转窗口看门狗](WINDOG.MD)

## 2. 配置 STM32 独立看门狗

### 2.1 使用 CubeMX 配置 IWDG

1. **打开 CubeMX**，选择目标 STM32 微控制器。

2. 在 **"System Core"** 中启用 **IWDG**：
   - 进入 **"IWDG"** 配置页面。
   - 启用 **IWDG** 功能，勾选 **"Activated"**。

3. 配置 IWDG 时钟源：
   - STM32 的 IWDG 时钟源来自内部低速振荡器（LSIRC）。在 CubeMX 中，IWDG 时钟源频率为 40kHz，不同 STM32 系列芯片可能会有所不同。
   - 你可以在 **Clock Configuration** 页面查看 **LSIRC** 时钟源，并确认其频率。

4. 配置 **IWDG** 定时参数：
   - **分频系数（Prescaler）**：用于调整 IWDG 时钟频率，常用的选项有 `4`, `8`, `16`, `32`, `64`, `128`, `256` 等。分频系数决定了定时器的最大计数周期。
   - **重载值（Reload）**：决定了 IWDG 的超时时间。最常见的重载值为 0xFFF，代表最大值。

   例如：  
   对于 `Prescaler = 64`，`Reload = 0xFFF`（最大值），则计算公式为：
   \[
   IWDG\_Time = \frac{{Prescaler \times Reload}}{{LSIRC}} = \frac{{64 \times 0xFFF}}{{40,000}} = 1.6 \text{ seconds}
   \]

### 2.2 生成代码并查看 IWDG 初始化函数

1. 生成代码并使用 **Keil** 打开生成的工程。

2. 在 **main.c** 中，可以看到自动生成的 IWDG 初始化代码。以下是 `MX_IWDG_Init()` 函数的一个示例：

```c
/* Initialize IWDG */
void MX_IWDG_Init(void)
{
  /* Set IWDG Prescaler and Reload values */
  hiwdg.Instance = IWDG;
  hiwdg.Init.Prescaler = IWDG_PRESCALER_64;   // Set the prescaler
  hiwdg.Init.Reload = 0xFFF;  // Set the reload value for 1.6s timeout
  hiwdg.Init.Window = 0xFFF;  // Set the window value for WWDG (not used in IWDG)
  
  HAL_IWDG_Init(&hiwdg);  // Initialize IWDG
}
```

这段代码将 IWDG 配置为 1.6 秒的超时周期。

## 3. 使用 IWDG 喂狗

### 3.1 主循环中定期喂狗

在 STM32 中使用 IWDG 时，必须在主程序的主循环中定期喂狗。每次喂狗时都会重新加载计数器，防止其超时。

通常，你需要在每个主循环的开始处调用 `HAL_IWDG_Refresh()` 函数。

#### 示例代码：

```c
#include "main.h"
#include "stm32f1xx_hal.h"

IWDG_HandleTypeDef hiwdg;

int main(void)
{
  HAL_Init();  // Initialize the HAL Library

  /* Initialize the IWDG */
  MX_IWDG_Init();

  while (1)
  {
    // 主程序代码
    HAL_IWDG_Refresh(&hiwdg);  // 喂狗，防止看门狗复位系统

    // 其他任务
  }
}
```

### 3.2 喂狗操作的注意事项
1. **定期喂狗**：你必须定期调用 `HAL_IWDG_Refresh()`，否则看门狗会超时并复位 MCU。在主循环中每 1.6 秒左右进行一次喂狗操作。
2. **IWDG 启动后不可重新配置**：IWDG 启动后无法修改其配置，确保在初始化时设置好所有参数。
3. **看门狗复位**：如果 IWDG 超时且没有喂狗，系统将复位，恢复到上电后的初始状态。

## 4. IWDG 定时计算

IWDG 的定时计算公式为：

\[
\text{Timeout Time} = \frac{{\text{Prescaler} \times \text{Reload Value}}}{{\text{LSIRC Frequency}}}
\]

- `Prescaler`：IWDG 时钟的分频系数。
- `Reload`：计数器的重载值。
- `LSIRC`：低速内部振荡器的频率，STM32 大多数芯片的频率为 40 kHz。

例如：
- `Prescaler = 64` 和 `Reload = 0xFFF` 时，超时时间为 1.6 秒。

## 5. 结论

STM32 的独立看门狗（IWDG）是一种常用的硬件看门狗，可以有效地防止系统进入死循环。使用时，需要：

1. 在 **CubeMX** 中启用 IWDG，并配置时钟源、分频系数和重载值。
2. 在主循环中定期喂狗，防止看门狗超时复位系统。

IWDG 是一个非常简便和有效的方案，适用于许多嵌入式应用中，以确保系统稳定性和可靠性。

---
