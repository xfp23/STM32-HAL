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

# 代码部分

```c

/**
*@brief 在主循环开始前必须使能定时器
*/
	HAL_TIM_Base_Start_IT(&htim1);
```

-  回调函数实现

```c

// 在此函数完成中断逻辑
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if (htim->Instance == TIM1)
    {
        print_flag = true;
    }
}


// 此处调用
void TIM1_UP_TIM16_IRQHandler(void)
{
  /* USER CODE BEGIN TIM1_UP_TIM16_IRQn 0 */

  /* USER CODE END TIM1_UP_TIM16_IRQn 0 */
  HAL_TIM_PeriodElapsedCallback(&htim1);
  HAL_TIM_IRQHandler(&htim1);
  /* USER CODE BEGIN TIM1_UP_TIM16_IRQn 1 */

  /* USER CODE END TIM1_UP_TIM16_IRQn 1 */
}
```