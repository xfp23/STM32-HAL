# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# STM32 PWM编程指南


# 配置CubeMX
# STM32 HAL库开发指南
**本频道只提供STM32 HAL库编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# STM32 PWM编程指南

- 首先根据本文档的[定时器章节](/Timer/Timer.md)配置一个1ms的定时器，与之前不同，此定时器不使能中断

# CubeMX配置

1. 设置1ms的定时器成功后，选择`CubeMX`的`channel`选项卡，通道设置为带`PWM`前缀的模式

2. channel中的选项说明

| **选项**                      | **说明**                                                                                 |
|-------------------------------|------------------------------------------------------------------------------------------|
| **PWM Mode (PWM 模式)**       | 用于输出 PWM 信号。通过设置比较值来调节占空比。常用配置之一。                               |
| **Output Compare Mode (OC)**  | 用于输出比较模式。产生特定的输出电平变化，适用于定时触发等场景。                               |
| **Input Capture Mode (IC)**   | 输入捕获模式，用于测量外部信号的脉宽和频率等参数。                                           |
| **One Pulse Mode (OPM)**      | 单脉冲模式，产生一个脉冲信号后自动停止计数，适合单次触发操作。                                   |
| **Forced Active Mode**        | 强制输出为高电平。                                                                        |
| **Forced Inactive Mode**      | 强制输出为低电平。                                                                        |
| **Toggle Mode**               | 每次触发更新事件后翻转输出电平，用于输出交替的高低电平信号。                                   |
| **Complementary PWM Mode**    | 互补 PWM 模式，配合死区时间控制用于驱动半桥电路，适用于电机控制。                               |
| **Edge-Aligned Mode**         | 边沿对齐模式，PWM 输出在计数器到达指定值时变化，产生经典的边沿对齐 PWM 信号。                    |
| **Center-Aligned Mode**       | 中心对齐模式，PWM 输出在计数器从最低值到达最高值并回到最低值期间对称输出，适合电机控制。           |
| **Capture/Compare Polarity**  | 捕获/比较极性设置，确定 PWM 输出或输入捕获的触发边沿（上升沿、下降沿或双边沿）。                   |
| **Capture/Compare Pre-scaler**| 捕获/比较预分频，用于调整信号频率，使得定时器能够以适合的时间间隔进行捕获。                        |
| **Capture/Compare Filter**    | 捕获/比较滤波器，用于去除输入信号中的高频噪声。


3. pwm的输出模式中的后缀意义，选择前缀为PWM的通道

在 STM32 定时器的 PWM 输出模式中，通道后缀（如 `CH1`、`CH1N`、`CH1_CH1N`）用于区分输出引脚的配置及其用途。以下是这些后缀的含义：

| **后缀**      | **说明**                                                                                                                                 |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------|
| **CH1**       | **主通道 1**。定时器的通道 1 输出，标准的 PWM 输出模式，信号在指定的输出引脚上产生，例如 `TIMx_CH1`。用于普通的 PWM 输出。              |
| **CH1N**      | **主通道 1 的互补输出**。该通道通常用于与 `CH1` 配合，产生反相信号，且有死区时间保护，用于 H 桥、半桥电路控制，应用于电机驱动等场景。|
| **CH1_CH1N**  | **主通道 1 及其互补输出的组合**。该设置包括 `CH1` 和 `CH1N` 两个输出信号，适用于需要同时使用 PWM 主输出和互补输出的场合。         |

### 使用场景
- **CH1**：单一 PWM 输出场合，如简单的 LED 调光。
- **CH1N**：与 `CH1` 配合，用于电机驱动等场景，提供互补信号以控制不同功率晶体管的导通和关断。
- **CH1_CH1N**：需要互补输出并对信号进行死区时间控制的场景，如全桥、半桥驱动电路。

4. 设置完channel之后点击CubeMX下方弹出参数设置选项卡中的对应通道的设置

| **选项**                     | **说明**                                                                                  |
|------------------------------|-------------------------------------------------------------------------------------------|
| **Mode**                     | PWM 的模式选择，通常为 `PWM1` 或 `PWM2`。`PWM1` 模式在计数器小于比较值时输出高电平，反之低电平；`PWM2` 模式相反。       |
| **Pulse**                    | 初始占空比，通常设置为 `0`，可以动态调整以改变占空比。                                     |
| **Output Compare Preload**   | 输出比较寄存器的预加载，通常设置为 `Enable` 以确保 PWM 输出更新的平滑性和稳定性。           |
| **CH Polarity**              | 主通道的输出极性。设置为 `High` 表示正极性，输出为高电平。`Low` 则为负极性。                |
| **CHN Polarity**             | 互补输出的极性。设置为 `High` 表示互补输出为正极性，`Low` 表示为负极性，配合死区时间使用。



# 初始化一个PWM输出

```c
#include "stm32f4xx_hal.h"

// 假设使用 TIM3 和通道 1 (可以根据需要更改)
TIM_HandleTypeDef htim3;

void PWM_Init()
{
    // 1. 开启定时器和GPIO时钟
    __HAL_RCC_TIM3_CLK_ENABLE();      // 开启 TIM3 时钟
    __HAL_RCC_GPIOA_CLK_ENABLE();     // 开启 GPIOA 时钟
    
    // 2. 配置 PWM 输出引脚
    GPIO_InitTypeDef GPIO_InitStruct = {0};
    GPIO_InitStruct.Pin = GPIO_PIN_6;         // 假设 TIM3 CH1 连接到 PA6
    GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;   // 复用推挽模式
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    GPIO_InitStruct.Alternate = GPIO_AF2_TIM3; // TIM3 复用
    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
    
    // 3. 配置定时器
    htim3.Instance = TIM3;
    htim3.Init.Prescaler = 72 - 1;            // 假设 72MHz 主频，设置预分频器为 72，PWM 基频 1MHz
    htim3.Init.CounterMode = TIM_COUNTERMODE_UP;
    htim3.Init.Period = 1000 - 1;             // PWM 频率 = 1MHz / 1000 = 1kHz
    htim3.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    HAL_TIM_PWM_Init(&htim3);
    
    // 4. 配置 PWM 输出通道
    TIM_OC_InitTypeDef sConfigOC = {0};
    sConfigOC.OCMode = TIM_OCMODE_PWM1;       // 选择 PWM 模式 1
    sConfigOC.Pulse = 500;                    // 初始占空比为 50%（计数值 500）
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    HAL_TIM_PWM_ConfigChannel(&htim3, &sConfigOC, TIM_CHANNEL_1);
    
    // 5. 启动 PWM 信号输出
    HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_1); // 启动 TIM3 通道 1 的 PWM 输出
}


__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_1, 750); // 设置 TIM3 通道 1 的占空比为 75%


```