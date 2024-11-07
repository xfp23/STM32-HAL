# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# STM32 DMA编程指南

- DMA为单片机的一个辅助资源，它是被其它模块所使用的，此处只介绍串口和SPI的DMA发送与接收

# DEMO示例

在 STM32 HAL 库的初始化过程中，**DMA 通道的绑定**通常由 **STM32CubeMX** 或手动代码配置完成。在 STM32CubeMX 中，配置好 DMA 后会自动生成相关代码。

如果你手动编写代码，需要明确设置 **DMA 通道和流** 以及 **优先级** 等参数。以下是手动配置 DMA 并绑定到 SPI 和 UART 外设的步骤。

### 1. 手动配置 DMA 传输通道

在手动配置 SPI 和 UART 的 DMA 时，需要定义并初始化 `DMA_HandleTypeDef` 结构体，并将其绑定到相应的外设。

### 代码示例

以下示例展示了 **SPI1 和 UART2** 使用 DMA 进行发送和接收的手动配置过程。

```c
#include "stm32f4xx_hal.h"

SPI_HandleTypeDef hspi1;
UART_HandleTypeDef huart2;
DMA_HandleTypeDef hdma_spi1_tx;
DMA_HandleTypeDef hdma_spi1_rx;
DMA_HandleTypeDef hdma_usart2_tx;
DMA_HandleTypeDef hdma_usart2_rx;

// DMA 配置函数
void DMA_Init(void)
{
    // 使能 DMA 时钟
    __HAL_RCC_DMA1_CLK_ENABLE();

    // 配置 SPI1 TX 的 DMA 通道
    hdma_spi1_tx.Instance = DMA1_Stream3; // 选择 DMA 流
    hdma_spi1_tx.Init.Channel = DMA_CHANNEL_3; // 选择通道
    hdma_spi1_tx.Init.Direction = DMA_MEMORY_TO_PERIPH; // 方向：内存到外设
    hdma_spi1_tx.Init.PeriphInc = DMA_PINC_DISABLE;
    hdma_spi1_tx.Init.MemInc = DMA_MINC_ENABLE;
    hdma_spi1_tx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
    hdma_spi1_tx.Init.MemDataAlignment = DMA_MDATAALIGN_BYTE;
    hdma_spi1_tx.Init.Mode = DMA_NORMAL;
    hdma_spi1_tx.Init.Priority = DMA_PRIORITY_LOW;

    if (HAL_DMA_Init(&hdma_spi1_tx) != HAL_OK)
    {
        Error_Handler();
    }
    __HAL_LINKDMA(&hspi1, hdmatx, hdma_spi1_tx); // 绑定 DMA 到 SPI1 TX

    // 配置 SPI1 RX 的 DMA 通道
    hdma_spi1_rx.Instance = DMA1_Stream2;
    hdma_spi1_rx.Init.Channel = DMA_CHANNEL_3;
    hdma_spi1_rx.Init.Direction = DMA_PERIPH_TO_MEMORY; // 方向：外设到内存
    hdma_spi1_rx.Init.PeriphInc = DMA_PINC_DISABLE;
    hdma_spi1_rx.Init.MemInc = DMA_MINC_ENABLE;
    hdma_spi1_rx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
    hdma_spi1_rx.Init.MemDataAlignment = DMA_MDATAALIGN_BYTE;
    hdma_spi1_rx.Init.Mode = DMA_NORMAL;
    hdma_spi1_rx.Init.Priority = DMA_PRIORITY_LOW;

    if (HAL_DMA_Init(&hdma_spi1_rx) != HAL_OK)
    {
        Error_Handler();
    }
    __HAL_LINKDMA(&hspi1, hdmarx, hdma_spi1_rx); // 绑定 DMA 到 SPI1 RX

    // 配置 UART2 TX 的 DMA 通道
    hdma_usart2_tx.Instance = DMA1_Stream6;
    hdma_usart2_tx.Init.Channel = DMA_CHANNEL_4;
    hdma_usart2_tx.Init.Direction = DMA_MEMORY_TO_PERIPH;
    hdma_usart2_tx.Init.PeriphInc = DMA_PINC_DISABLE;
    hdma_usart2_tx.Init.MemInc = DMA_MINC_ENABLE;
    hdma_usart2_tx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
    hdma_usart2_tx.Init.MemDataAlignment = DMA_MDATAALIGN_BYTE;
    hdma_usart2_tx.Init.Mode = DMA_NORMAL;
    hdma_usart2_tx.Init.Priority = DMA_PRIORITY_LOW;

    if (HAL_DMA_Init(&hdma_usart2_tx) != HAL_OK)
    {
        Error_Handler();
    }
    __HAL_LINKDMA(&huart2, hdmatx, hdma_usart2_tx); // 绑定 DMA 到 UART2 TX

    // 配置 UART2 RX 的 DMA 通道
    hdma_usart2_rx.Instance = DMA1_Stream5;
    hdma_usart2_rx.Init.Channel = DMA_CHANNEL_4;
    hdma_usart2_rx.Init.Direction = DMA_PERIPH_TO_MEMORY;
    hdma_usart2_rx.Init.PeriphInc = DMA_PINC_DISABLE;
    hdma_usart2_rx.Init.MemInc = DMA_MINC_ENABLE;
    hdma_usart2_rx.Init.PeriphDataAlignment = DMA_PDATAALIGN_BYTE;
    hdma_usart2_rx.Init.MemDataAlignment = DMA_MDATAALIGN_BYTE;
    hdma_usart2_rx.Init.Mode = DMA_NORMAL;
    hdma_usart2_rx.Init.Priority = DMA_PRIORITY_LOW;

    if (HAL_DMA_Init(&hdma_usart2_rx) != HAL_OK)
    {
        Error_Handler();
    }
    __HAL_LINKDMA(&huart2, hdmarx, hdma_usart2_rx); // 绑定 DMA 到 UART2 RX
}

// 主初始化函数
void System_Init(void)
{
    HAL_Init();
    DMA_Init(); // 初始化 DMA
    // SPI 和 UART 初始化按需添加
}
```

### 说明

1. **DMA_HandleTypeDef**：配置 DMA 相关属性，比如 `Direction`（传输方向）、`PeriphInc`（外设地址增量）、`MemInc`（内存地址增量）、`Priority`（优先级）等。
2. `__HAL_LINKDMA`：将 DMA 句柄与外设（SPI 或 UART）句柄中的 DMA 字段绑定，从而使 HAL 库函数可以自动处理 DMA 数据传输。
3. `DMA1_StreamX` 和 `DMA_CHANNEL_X`：需要根据 **STM32 数据手册** 中的 SPI 和 UART 外设对应的 DMA 流和通道配置。


抱歉让你不满了，我这次直接给出 HAL 库中 **SPI 和 UART 使用 DMA 发送和接收数据的 API 示例**。

### SPI DMA 发送和接收数据示例

```c
#include "stm32f4xx_hal.h"

extern SPI_HandleTypeDef hspi1;

void SPI_DMA_Send(uint8_t *pData, uint16_t size)
{
    if (HAL_SPI_Transmit_DMA(&hspi1, pData, size) != HAL_OK)
    {
        // 处理错误
        Error_Handler();
    }
}

void SPI_DMA_Receive(uint8_t *pData, uint16_t size)
{
    if (HAL_SPI_Receive_DMA(&hspi1, pData, size) != HAL_OK)
    {
        // 处理错误
        Error_Handler();
    }
}
```

### UART DMA 发送和接收数据示例

```c
#include "stm32f4xx_hal.h"

extern UART_HandleTypeDef huart2;

void UART_DMA_Send(uint8_t *pData, uint16_t size)
{
    if (HAL_UART_Transmit_DMA(&huart2, pData, size) != HAL_OK)
    {
        // 处理错误
        Error_Handler();
    }
}

void UART_DMA_Receive(uint8_t *pData, uint16_t size)
{
    if (HAL_UART_Receive_DMA(&huart2, pData, size) != HAL_OK)
    {
        // 处理错误
        Error_Handler();
    }
}
```

### 代码说明

- **HAL_SPI_Transmit_DMA** 和 **HAL_SPI_Receive_DMA**：用于 SPI 的 DMA 传输与接收。
- **HAL_UART_Transmit_DMA** 和 **HAL_UART_Receive_DMA**：用于 UART 的 DMA 传输与接收。
  
你只需在 DMA 初始化完成后调用这些函数，即可通过 DMA 方式完成 SPI 和 UART 的数据传输。