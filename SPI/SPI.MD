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

- spi句柄类型: 
```
SPI_HandleTypedef
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