# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

---

# STM32 HARDWARE UART

# CubeMX配置
1. 选择`Connectivity`选项卡

2. 选择串口
3. 点击`mode` 选择模式: 选择异步模式

| 类型                          | 描述         |
|-------------------------------|--------------|
| **Asynchronous**               | 异步模式 (常用) |
| **Synchronous**                | 同步模式     |
| **Single Wire**                | 半双工       |
| **Multiprocessor Communication**| 多处理器通信  |
| **IrDA**                        | IRDA         |
| **LIN**                         | LIN          |
| **SmartCard**                  | 智能卡       |

4. 打开GPIO选项卡，选择UART选项查看GPIO的配置，一般不做更改,有关GPIO信息，请参阅[GPIO编程指南](/GPIO/GPIO.md)
5. 返回UART选项卡，查看串口参数

| **Basic parameters**           | **基本参数**      |
|---------------------------------|-------------------|
| **Baud rate**                  | 波特率            |
| **Word Length**                | 数据位长度（一般8bit）|
| **Parity**                     | 校验位（none: 不检验，Even: 偶校验，Odd: 奇校验） |
| **Stop Bits**                  | 停止位            |

| **Advanced Parameters**        | **高级参数**      |
|---------------------------------|-------------------|
| **Data Direction**             | 数据方向          |
|                                 | Transmit-only: 只发送 |
|                                 | Receive-only: 只接收 |
|                                 | Receive and Transmit: 收发 |


5.  保存配置，CubeMX将自动生成UART的驱动

# 初始化串口API

1. 串口句柄类型: 

```c
UART_HandleTypedef; // 串口句柄
```


2. 初始化示例代码:
```c
#include "stm32f1xx_hal.h"  // 请根据芯片型号替换 HAL 库头文件

UART_HandleTypeDef huart1;

void MX_USART1_UART_Init(void) {
    huart1.Instance = USART1;  // 选择 USART1 作为串口外设实例

    // 波特率设置
    // 常见波特率：9600、115200 等
    huart1.Init.BaudRate = 115200;

    // 字长设置
    // UART_WORDLENGTH_8B：8 位数据帧
    // UART_WORDLENGTH_9B：9 位数据帧
    huart1.Init.WordLength = UART_WORDLENGTH_8B;

    // 停止位设置
    // UART_STOPBITS_1：1 个停止位
    // UART_STOPBITS_2：2 个停止位
    huart1.Init.StopBits = UART_STOPBITS_1;

    // 校验设置
    // UART_PARITY_NONE：无校验
    // UART_PARITY_EVEN：偶校验
    // UART_PARITY_ODD：奇校验
    huart1.Init.Parity = UART_PARITY_NONE;

    // 模式设置
    // UART_MODE_TX_RX：发送和接收模式
    // UART_MODE_TX：仅发送模式
    // UART_MODE_RX：仅接收模式
    huart1.Init.Mode = UART_MODE_TX_RX;

    // 硬件流控制设置
    // UART_HWCONTROL_NONE：无硬件流控制
    // UART_HWCONTROL_RTS：RTS 流控制
    // UART_HWCONTROL_CTS：CTS 流控制
    // UART_HWCONTROL_RTS_CTS：RTS 和 CTS 流控制
    huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;

    // 过采样模式
    // UART_OVERSAMPLING_16：16 倍过采样
    // UART_OVERSAMPLING_8：8 倍过采样（某些型号不支持）
    huart1.Init.OverSampling = UART_OVERSAMPLING_16;

    // 初始化 UART 并检查是否成功
    if (HAL_UART_Init(&huart1) != HAL_OK) {
        // 初始化失败时的错误处理
        Error_Handler();
    }
}


```
# UART 操作API

- 发送数据 
```c
/**
*@brief 向串口发送数据
*@param 串口句柄地址
*@param 向串口发送的数据的地址
*@param 要发送的字节大小
*@param  超时时间 单位 : ms HAL_MAX_DELAY(表示无限等待，直到数据发送完)
*@return 返回数据发送的结果 HAL_OK 成功    其他：失败 HAL_ERROR 发送出错 HAL_BUSY 串口忙 HAL_TIMEOUT 发送超时
*/
HAL_StatusTypedef HAL_UART_Transmit(UART_HandleTypedef * ,uint8_t *data,uint16_t size,uint32_t Timeout);

```

- 接收数据

```c

/**
 * @brief 接收串口数据
 * @param 串口句柄地址
 * @param 接收串口数据的缓冲区地址
 * @param 要接收的字节数量
 * @param 超时时间 ms 不使用填 HAL_MAX_DELAY 无限大
 * @return 返回数据接收的结果 HAL_OK 成功 HAL_ERROR 接收出错 HAL_BUSY 串口忙 HAL_TIMEOUT 接收超时
*/
HAL_StatusTypedef HAL_UART_Receive(UART_HandleTypedef* ,uint8_t *data,uint16_t size,uint32_t Timeout);

```