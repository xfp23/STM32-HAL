
---

### 版权信息

© 2024 . 未经许可不得复制、修改或分发。此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

---

## STM32 窗口看门狗（WWDG）

STM32内置两种看门狗：独立看门狗（IWDG）和窗口看门狗（WWDG）。独立看门狗主要用于简单的看门狗功能，只要在定时器溢出之前进行喂狗即可，而窗口看门狗则具有更严格的条件，要求在一定的时间窗口内喂狗。

**本篇文章主要介绍STM32的窗口看门狗（WWDG）配置与使用方法。**

---

### 1. 配置

1. **启用窗口看门狗**  
   在 STM32CubeMX 中进行配置，选择 `WWDG` 模块并启用。在 `System Core` 中使能 `WWDG`，并配置 `Prescaler` 和 `Window value` 参数。

2. **时钟源配置**  
   窗口看门狗的主时钟源来自于 **APB1 时钟（PCLK1）**。其时钟分频可以通过配置 `Prescaler` 来控制。一般来说，`WWDG` 的时钟频率为 `PCLK1 / 8`。STM32 的默认 `APB1` 时钟频率为 `36 MHz`，因此分频后的时钟频率约为 `4.5 MHz`。

3. **窗口值（Window Value）**  
   窗口值的选择非常重要，它决定了喂狗的时间窗口。当窗口计数器的值从 `0x7F` 开始递减到窗口值时，只有在这个区间内喂狗才有效。`Window value` 必须介于 `0x3F` 和 `0x7F` 之间，通常建议设置为 `0x70`，表示只有当计数器值低于 `0x70` 时才允许喂狗。

4. **计数器配置**  
   窗口看门狗的计数器是一个递减计数器，计数范围从 `0x7F` 到 `0x3F`。如果计数器没有在规定时间内喂狗，系统会发生复位。可以通过设置 `WWDG free-running downcounter value` 来设定初始计数器的值。

---

### 2. 时钟和喂狗窗口时间计算

#### 时钟计算：
窗口看门狗的时钟源是 `PCLK1`，分频系数通常设置为 `8`，最终时钟频率为 `PCLK1 / 8`，假设 `PCLK1` 为 36 MHz，则窗口看门狗的时钟频率为：

```
WWDG_clk = PCLK1 / 8 = 36 MHz / 8 = 4.5 MHz
```

#### 喂狗窗口时间计算：
喂狗的窗口时间由窗口值（`Window value`）与窗口看门狗时钟频率决定。可以通过以下公式来计算喂狗时间：

```
T_feed = (Window_value - 0x3F) / WWDG_clk
```

例如，如果设置 `Window_value = 0x70`，则喂狗窗口时间为：

```
T_feed = (0x70 - 0x3F) / 4.5 MHz
       = (112 - 63) / 4.5 MHz
       = 49 / 4.5 MHz
       ≈ 10.89 ms
```

这表示喂狗的窗口时间大约为 **10.89 毫秒**。

---

### 3. 早期唤醒中断

窗口看门狗支持早期唤醒中断。当计数器值即将低于 `Window_value` 时，可以触发早期唤醒中断，提示程序准备喂狗。该中断用于提前警告，确保在窗口内及时喂狗。

#### 使能早期唤醒中断：
1. 在 STM32CubeMX 中，启用 `WWDG Early Wakeup Interrupt`。
2. 在代码中，在 `NVIC` 中启用该中断。

---

### 4. 程序示例

#### 4.1 代码配置

在 `wwdg.c` 中配置窗口看门狗，启用早期唤醒中断，并在主循环中定期喂狗。

```c
#include "main.h"

// 早期唤醒中断回调函数
void HAL_WWDG_EarlyWakeupCallback(void)
{
    // 在此设置一个标志位，表示即将到喂狗的时间
    watchdog_flag = 1;
}

// 配置WWDG
void WWDG_Init(void)
{
    // 配置WWDG分频器和窗口值
    WWDG_HandleTypeDef hwwdg;
    hwwdg.Instance = WWDG;
    hwwdg.Init.Prescaler = WWDG_PRESCALER_8;  // 分频器
    hwwdg.Init.Window = 0x70;                  // 窗口值
    hwwdg.Init.Counter = 0x7F;                 // 初始计数值
    hwwdg.Init.EWIMode = WWDG_EWI_DISABLE;     // 早期唤醒中断禁用
    HAL_WWDG_Init(&hwwdg);
    
    // 使能早期唤醒中断
    HAL_WWDG_EnableIT(&hwwdg);
}
```

#### 4.2 主循环中的喂狗

在主循环中，我们需要定期喂狗，确保计数器不会溢出，从而触发复位。喂狗的时间间隔应当在窗口值设置的时间窗口内。

```c
int main(void)
{
    // 初始化所有硬件
    HAL_Init();
    WWDG_Init();

    while (1)
    {
        // 检查是否到了喂狗的时间
        if (watchdog_flag == 1)
        {
            // 进行喂狗操作
            HAL_WWDG_Refresh(&hwwdg);
            watchdog_flag = 0;  // 清除标志位
        }
    }
}
```

---

### 5. 总结

1. **时钟配置**：窗口看门狗时钟源来自 `PCLK1`，通常通过 `Prescaler` 设置为 `PCLK1 / 8`。
2. **喂狗窗口时间**：根据 `Window value` 和时钟频率计算，确保在适当的时间窗口内喂狗。
3. **早期唤醒中断**：通过启用早期唤醒中断，提前警告主程序进行喂狗操作。
4. **程序设计**：在主循环中定期喂狗，避免系统复位。
