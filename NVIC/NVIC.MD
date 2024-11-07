# STM32 HAL库开发指南
**本频道只提供STM32 HAL库编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。


# STM32 NVIC 外部中断编程指南

- [STM32中断知识点扩展](./NVIC_EX.md)

## CubeMX 配置

1. 选择System Core ，打开NVIC选项卡

2. 根据自己的需要选择要使能的中断，选择在`Enable`选项栏使能 


# 初始化API

- 暂无，后续补充

# 串口中断处理数据 :

- 使用中断接收数据
```c
/**
 * @brief 使用中断的方式接收一定量的数据
 * @param 串口句柄
 * @param 接收数据的缓冲区地址
 * @param 数据大小，单位：字节
 * @return HAL_OK HAL_ERROR 
 */
HAL_StatusTypeDef HAL_UART_Receive_IT(UART_HandleTypedef *huart,uint8_t *data,uint16_t size);

```

- 回调函数,当上面的函数接收数据完成之后单片机会调用下方的回调函数

```c
void HAL_UART_RxCpltCallback(UART_HandleTypedef * huart);
```

- 执行流程说明：

1. 首先将`HAL_UART_Receive_IT`函数放进main函数的主循环之外，相当于`Arduino`的`setup()`函数里

2. 在main函数之外编写`HAL_UART_RxCpltCallback()`函数的定义，用于函数实现

- 回调函数示例: 接收不定长度的数据
```c
HAL_UART_Receive_IT(&huart,buffer,1); // 想要不定长度每次接收一个字节然后在回调函数中写存储信息逻辑即可

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    // 判断哪个串口触发的中断
    if (huart->Instance == USART1)
    {
        // 将接收到的数据存入缓冲区
        data_buffer[data_index++] = huart->pRxBuffPtr[0]; 

        // 如果缓冲区未满或数据未结束，则继续接收
        if (data_index < sizeof(data_buffer))
        {
            HAL_UART_Receive_IT(huart, huart->pRxBuffPtr, 1);  // 继续接收一个字节
        }
        else
        {
            // 数据缓冲区已满，进行处理（例如数据解析等）
            process_received_data();
        }
    }
}
```

# 外部中断的使用

- 在CubeMX配置完之后，需要手动写一个对应的中断函数实现

- 每个中断线都有一个对应的`EXTIx_IRQHandler`
```c
void EXTI0_IRQHandler(void)
{
    // 检查是否是 PA0 引脚触发了中断
    if (__HAL_GPIO_EXTI_GET_IT(GPIO_PIN_0) != RESET) {
        // 清除中断标志
        HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_0);

        // 执行中断处理（比如点亮 LED）
        HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);  // 切换 LED 状态
    }

    // 你可以添加更多引脚的判断，处理其他外部中断
    if (__HAL_GPIO_EXTI_GET_IT(GPIO_PIN_1) != RESET) {
        // 清除中断标志
        HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_1);
        
        // 执行其它引脚的处理
    }
}
```