# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# STM32 SPI编程指南

- [SPI知识扩展](/SPI/SPI_Expand.md)

### 四种时钟模式

| 极性/相位   | 第一边沿 0            | 第二边沿 1            |
|-------------|-----------------------|-----------------------|
| **低 (0)**  | 00 模式 0（CPOL=0, CPHA=0） | 01 模式 1（CPOL=0, CPHA=1） |
| **高 (1)**  | 10 模式 2（CPOL=1, CPHA=0） | 11 模式 3（CPOL=1, CPHA=1） |


# CubeMX配置

1. 打开`Connectivity`选项卡，选择使用的SPI硬件资源

2. 配置spi

你所提到的不同工作模式是SPI协议的常见工作模式，通常用于定义数据传输的方向、双工模式以及主从机之间的通信关系。以下是你列出的这些工作模式的表格形式：

### SPI 工作模式选择

| 工作模式              | 描述                             |
|-----------------------|----------------------------------|
| **Full-Duplex Master** | 主设备全双工模式，既能发送也能接收数据。（常用）|
| **Full-Duplex Slave**  | 从设备全双工模式，既能发送也能接收数据。|
| **Half-Duplex Master** | 主设备半双工模式，只能发送或接收数据。|
| **Half-Duplex Slave**  | 从设备半双工模式，只能发送或接收数据。|
| **Receive Only Master**| 主设备只接收数据。|
| **Receive Only Slave** | 从设备只接收数据。|
| **Transmit Only Master**| 主设备只发送数据。|
| **Transmit Only Slave**| 从设备只发送数据。|

 

### 硬件 NSS 片选信号模式，一般用软件cs

| **模式**                         | **描述**                             |
|----------------------------------|--------------------------------------|
| **HardWare NSS Input Signal**    | 使用硬件NSS作为输入信号。适用于多主机通信。 |
| **HardWare NSS Output Signal**   | 使用硬件NSS作为输出信号。适用于多主机通信。 |


3. 观察下面底层选项卡，
以下是整理后的表格：

| **类别**                | **参数**                        | **说明**                          |
|-------------------------|---------------------------------|-----------------------------------|
| **基本参数**            | **Data size（数据位的长度）**   | 8 或 16位                         |
|                         | **First Bit（数据位的传输顺序）**| MSB（先传最高有效位）<br>LSB（先传最低有效位）|
|                         | **Prescaler（分频系数）**       | 改变分频系数以生成所需的波特率    |
| **时钟参数**            | **Clock polarity（时钟极性）**   | Low（低极性）<br>High（高极性）   |
|                         | **Clock phase（时钟相位）**     | 1Edge（第一边沿采集）<br>2Edge（第二边沿采集）|
| **高级参数**            | **CRC Calculation（CRC校验）**  | 是否启用CRC校验                  |
|                         | **NSS Signal Type（NSS类型）**  | 使用硬件NSS（片选信号）           |

这个表格覆盖了SPI接口的基本参数、时钟参数和高级参数设置，可以根据你的需求选择不同的配置。


# 初始化API

1. spi句柄类型: 
```
SPI_HandleTypedef
```

2. 初始化示例
```c
#include "stm32f1xx_hal.h" // 请根据芯片型号替换 HAL 库头文件

SPI_HandleTypeDef hspi1;

void MX_SPI1_Init(void) {
    hspi1.Instance = SPI1;  // 使用 SPI1 作为外设实例

    // SPI 模式设置
    // SPI_MODE_MASTER：主模式
    // SPI_MODE_SLAVE：从模式
    hspi1.Init.Mode = SPI_MODE_MASTER;

    // SPI 数据传输方向
    // SPI_DIRECTION_2LINES：全双工（两线）
    // SPI_DIRECTION_2LINES_RXONLY：全双工，仅接收
    // SPI_DIRECTION_1LINE：单线半双工
    hspi1.Init.Direction = SPI_DIRECTION_2LINES;

    // SPI 数据宽度
    // SPI_DATASIZE_8BIT：8 位数据帧
    // SPI_DATASIZE_16BIT：16 位数据帧
    hspi1.Init.DataSize = SPI_DATASIZE_8BIT;

    // SPI 时钟极性
    // SPI_POLARITY_LOW：空闲状态下时钟线为低电平（CPOL = 0）
    // SPI_POLARITY_HIGH：空闲状态下时钟线为高电平（CPOL = 1）
    hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;

    // SPI 时钟相位
    // SPI_PHASE_1EDGE：第一个时钟边沿采样数据（CPHA = 0）
    // SPI_PHASE_2EDGE：第二个时钟边沿采样数据（CPHA = 1）
    hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;

    // NSS（片选）引脚管理
    // SPI_NSS_SOFT：软件管理 NSS 信号
    // SPI_NSS_HARD_INPUT：硬件管理，作为从设备时使用
    // SPI_NSS_HARD_OUTPUT：硬件管理，作为主设备时使用
    hspi1.Init.NSS = SPI_NSS_SOFT;

    // 波特率分频
    // 可用值：SPI_BAUDRATEPRESCALER_2, SPI_BAUDRATEPRESCALER_4, 
    // SPI_BAUDRATEPRESCALER_8, SPI_BAUDRATEPRESCALER_16, 
    // SPI_BAUDRATEPRESCALER_32, SPI_BAUDRATEPRESCALER_64, 
    // SPI_BAUDRATEPRESCALER_128, SPI_BAUDRATEPRESCALER_256
    // 分频值越小，传输速率越高
    hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_8;

    // 数据位序
    // SPI_FIRSTBIT_MSB：高位在前（MSB first）
    // SPI_FIRSTBIT_LSB：低位在前（LSB first）
    hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;

    // TI 模式使能
    // SPI_TIMODE_DISABLE：禁用 TI 模式
    // SPI_TIMODE_ENABLE：启用 TI 模式
    hspi1.Init.TIMode = SPI_TIMODE_DISABLE;

    // CRC 校验使能
    // SPI_CRCCALCULATION_DISABLE：禁用 CRC 校验
    // SPI_CRCCALCULATION_ENABLE：启用 CRC 校验
    hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;

    // CRC 多项式值，仅在启用 CRC 校验时使用
    hspi1.Init.CRCPolynomial = 10;

    if (HAL_SPI_Init(&hspi1) != HAL_OK) {
        // 初始化错误处理
        Error_Handler();
    }
}

```
# SPI操作API

1. API

- 发送数据

```c
/**
 * @brief 使用spi发送数据
 * @param spi句柄地址
 * @param 要发送的数据
 * @param 发送数据的字节大小
 * @param 超时时间 
 * @return HAL_OK HAL_ERROR HAL_BUSY HAL_TIMEOUT
*/
HAL_Status_TypeDef HAL_SPI_Transmit(SPI_HandleTypedef *hspi,uint8_t *data,uint16_t size,uint32_t timeout);
```
- 接收数据

```c
/**
 * @brief 使用spi发送数据
 * @param spi句柄地址
 * @param 接收数据的缓冲区大小
 * @param 要接收的数据字节大小
 * @param 超时时间 
 * @return HAL_OK HAL_ERROR HAL_BUSY HAL_TIMEOUT
*/
HAL_Status_TypeDef HAL_SPI_Receive(SPI_HandleTypedef *hspi,uint8_t *data,uint16_t size,uint32_t timeout);
```

- 发送数据的同时接收数据
```c
/**
 * @brief 发送数据的同时接收数据
 * @param spi句柄指针
 * @param 发送数据的地址
 * @param 接收数据的缓冲区的指针
 * @param 数据大小，字节为单位 发送的大小 = 接收的大小
 * @param 超时时间
 * @return HAL_OK HAL_ERROR HAL_BUSY HAL_TIMEOUT
 * 
 *
 */
HAL_Status_TypeDef HAL_SPI_TransmitReceive(SPI_HandleTypedef *hspi,uint8_t *tx_data,uint8_t *rx_data,uint16_t size,uint32_t Timeout);

```