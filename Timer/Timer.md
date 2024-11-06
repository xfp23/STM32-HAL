# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。


# STM32 Timer编程指南

- [定时器知识扩展](./Timer_ex.md)

### **时钟源总线频率配置**

| **总线类型**   | **最大频率**    |
|----------------|-----------------|
| **AHB总线**    | 72 MHz          |
| **APB1总线**   | 36 MHz          |
| **APB2总线**   | 72 MHz          |


# CubeMX配置

1. 打开`Timer`选项卡，选择想使用的定时器编号

2. 配置模式,我们不关注其它选项，全部为disable
- 只关心`Clock Source`选项 ，我们选择 `Internal Clock` 此选项代表选择内部时钟源RCC

- 然后配置下面弹出的`configuration`选项卡

这是一个表格形式来展示定时器配置中的几个关键选项，按您的要求列出：

| **参数**                      | **说明**                                               |
|-------------------------------|--------------------------------------------------------|
| **PreScaler (PSC)**            | 设置定时器的预分频器值（16位）。通过此参数可调整定时器的频率。 |
| **Counter Mode**               | 计数模式，选择定时器是向上计数（Up）还是向下计数（Down）。  |
| **Counter Period (ARR)**       | 计数周期，或称自动重载值，用于控制计数器重装值的大小。     |
| **Internal Clock Division (TDTS)** | 内部时钟分频系数，用于调整输入滤波和死区时间，通常不常用。    |
| **Auto-reload Preload (ARR)**  | 自动重载预加载，控制是否启用自动重载寄存器（ARR）的预加载。一般需要使能。 |


3. 打开定时器配置页面下的`NVIC`选项卡，使能定时器中断 `update interrupt`

# 初始化API


- 初始化例程:

```c
#include "stm32f4xx_hal.h"

// 定时器句柄
TIM_HandleTypeDef htim2;

// 定时器初始化函数
void TIM2_Init(void)
{
    // 1. 开启定时器2和RCC的时钟
    __HAL_RCC_TIM2_CLK_ENABLE();

    // 2. 配置定时器基础结构
    htim2.Instance = TIM2;  // 选择定时器2
    htim2.Init.Prescaler = 71;  // 预分频器 PSC: (72 MHz / (71+1)) = 1 MHz -> 每个计数周期1微秒
    htim2.Init.CounterMode = TIM_COUNTERMODE_UP;  // 向上计数模式
    htim2.Init.Period = 999;  // 自动重载值 ARR: 1ms -> (1,000,000 Hz / 1000) - 1 = 999
    htim2.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;  // 时钟分频
    htim2.Init.RepetitionCounter = 0;  // 重复计数器，通常设置为 0

    // 3. 初始化定时器
    if (HAL_TIM_Base_Init(&htim2) != HAL_OK)
    {
        // 错误处理
        Error_Handler();
    }

    // 4. 启动定时器中断（如果需要中断）
    if (HAL_TIM_Base_Start_IT(&htim2) != HAL_OK)
    {
        // 错误处理
        Error_Handler();
    }
}

// 定时器中断回调函数
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if (htim->Instance == TIM2)
    {
        // 定时器2触发的回调函数
        // 每隔1毫秒触发一次
    }
}

```