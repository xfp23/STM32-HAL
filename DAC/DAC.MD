# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。


# DAC编程指南

1. STM32的DAC总线在`APB1`上，DAC的工作时钟信号是`PCLK1`

2. DAC输出的数值有寄存器`DORx`的数值和参考电压VREF决定的，公式为：

DACoutput = DORx的数值/4096(2的12次方) * VREF+


## 配置DAC模块

- 不同的芯片配置方式可能略有差异

## F4系列
1. 选择DAC选项卡，使能要配置的DAC模块

2. **config**选项卡
- 下面以`DAC1`举例


---

| **选项**                        | **含义**                                                                                                                                     |
|---------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| **Output Buffer**               | 必须使能，提供增强的驱动能力。                                                                                                                 |
| **Trigger**                     | 触发方式选择。<br>**Software Trigger**：由软件触发更新；<br>**Timerx Trigger**：由定时器触发更新。                                                  |
| **Wave Generation Mode**        | 生成波形的方式。<br>**Triangle Wave Generation**：生成三角波；<br>**Noise Wave Generation**：生成随机噪声波形。                                     |
| **Maximum Trigger Amplitude**   | 设置三角波的最大幅值，范围为 2 的 n 次方减 1（如 4095）。                                                                                   |
| **Noise Amplitude**             | 设置噪声波形的幅值。                                                                                                                                  |
| **Unmask DAC Channel LFSR Bits**| 配置 LFSR 位数，影响噪声的随机性。<br>高位数（如 [11:0]）提供更高的随机性。                                                                     |

---

## G4系列

- 配置G4系列的DAC两个DAC资源是独立配置的

打开DAC1选项卡会看到有`OUT1 Mode` 和 `OUT2 Mode` 字样

以下是关于 **`OUT1 Mode` 选项卡** 的补充说明：

---

**`OUT1 Mode` 选项卡**

| **选项**                               | **含义**                                                                                   |
|----------------------------------------|--------------------------------------------------------------------------------------------|
| **Connected to external pin only**     | 该输出仅连接到外部引脚。输出信号只通过外部引脚传递，不能与内部外设连接。适用于需要将 DAC 输出到外部设备的情况。 |
| **OUT Connected to on-chip peripherals only** | 该输出仅连接到芯片内的外设。输出信号只在芯片内与其他外设交互，不输出到外部引脚。适用于需要内部信号处理或用于内部应用的情况。 |
| **Connected to external and chip-peripherals** | 该输出同时连接到外部引脚和芯片内部外设。输出信号既可以与外部设备交互，也可以用于芯片内部的信号处理，适用于需要同时处理外部和内部信号的场景。 |

---

OUT2 Mode 选项卡跟OUT1 Mode 是一样的，只不过是为了开启更多的DAC输出引脚而已



---

**`DAC OUTx Settings` 配置选项**

#### **Mode Selected**
| **选项**              | **含义**                                                                                   |
|-----------------------|--------------------------------------------------------------------------------------------|
| **Normal Mode**        | 正常模式：DAC 输出按要求的输入信号进行转换，输出信号会立即反映输入信号的变化。              |
| **Sample and Hold Mode** | 采样保持模式：DAC 会定期采样输入信号，并将其保持在一个固定值，直到下一次采样，适用于要求较低功耗或需要固定输出的应用。 |

#### **选择了 `Normal Mode` 时的配置：**

| **选项**         | **含义**                                                                                   |
|------------------|--------------------------------------------------------------------------------------------|
| **Signed Format** | 输出的格式是有符号数（Signed）还是无符号数（Unsigned）。有符号格式支持负值输出。              |
| **Trigger**       | 触发方式选择。用于控制 DAC 更新输出信号的时机。可以选择软件触发或外部事件触发。              |
| **Trigger2**      | 第二触发源，用于增加更复杂的触发机制，允许多种触发方式组合。                                 |

#### **选择了 `Sample and Hold Mode` 时新增的配置项：**

| **选项**         | **含义**                                                                                   |
|------------------|--------------------------------------------------------------------------------------------|
| **Sample Time**   | 采样时间：定义 DAC 采样输入信号的时间长度，即输入信号持续被采样的时间。                         |
| **Hold Time**     | 保持时间：DAC 在采样后保持输出信号的时间长度，直到下一次采样发生。                             |
| **Refresh Time**  | 刷新时间：定义 DAC 每次刷新输出信号的时间间隔，通常用于控制输出的更新速率。                    |

---


## 高级配置

1. 如果你配置了定时器和DMA触发，对于G4那种你必须选择`Connected to external and chip-peripherals`.

2. DMA必须选择`Circular`模式，循环模式

### 配置定时器

1. 首先需要配置对应的定时器以配合想要的DAC频率,一般选择一个通用定时器

2. 选择内部时钟源

频率计算公式:

### 配置定时器频率计算公式

$\[f_{\text{DAC Trigger}} = \frac{f_{\text{Timer Clock}}}{(\text{PSC} + 1) \times (\text{ARR} + 1)}\]$


**参数说明：**
- **$\( f_{\text{Timer Clock}} \)$**：定时器时钟频率（与 APB 总线频率有关）。  
- **PSC**：预分频器值（分频倍数为  $\( \text{PSC} + 1 \)$ S）。  
- **ARR**：自动重装载值（定时器周期）。


3. 设置完`PSC`和`ARR`之后，找到`Trigger Output parameters`选项卡
将`Trigger Event Selection TRGO`选项改为`Update Event`
触发事件选择 TRGO

# 编程

- 现在你已经完成了基本配置，现在开始写程序

- 由于可写入寄存器的最大值为2^12即4096，那么DAC输出的可调范围也就在0~4095之间，所以写波形计算公式的时候需要注意数值会不会溢出。

这是DMA的API
```c
/**
 * @brief  停止指定 DAC 通道的 DMA 模式。
 * @param  hdac: 指向 DAC 句柄的指针。
 * @param  Channel: 要停止 DMA 的 DAC 通道。
 *         可能的值:
 *           - DAC_CHANNEL_1: DAC 通道 1
 *           - DAC_CHANNEL_2: DAC 通道 2
 * @retval HAL status
 *         返回值:
 *           - HAL_OK: 操作成功
 *           - HAL_ERROR: 操作失败
 *           - HAL_BUSY: DAC 正在忙
 *           - HAL_TIMEOUT: 操作超时
 */
HAL_StatusTypeDef HAL_DAC_Stop_DMA(DAC_HandleTypeDef *hdac, uint32_t Channel);

/**
 * @brief  使用 DMA 启动指定 DAC 通道的输出。
 * @param  hdac: 指向 DAC 句柄的指针。
 * @param  Channel: 要启动 DMA 输出的 DAC 通道。
 *         可能的值:
 *           - DAC_CHANNEL_1: DAC 通道 1
 *           - DAC_CHANNEL_2: DAC 通道 2
 * @param  pData: 指向包含要输出数据的数组指针（数据类型为 uint32_t）。
 * @param  Length: 输出数据的长度（数组元素个数）。
 * @param  Alignment: 数据对齐方式。
 *         可能的值:
 *           - DAC_ALIGN_12B_R: 12 位右对齐
 *           - DAC_ALIGN_12B_L: 12 位左对齐
 *           - DAC_ALIGN_8B_R:  8 位右对齐
 * @retval HAL status
 *         返回值:
 *           - HAL_OK: 操作成功
 *           - HAL_ERROR: 操作失败
 *           - HAL_BUSY: DAC 正在忙
 *           - HAL_TIMEOUT: 操作超时
 */
HAL_StatusTypeDef HAL_DAC_Start_DMA(DAC_HandleTypeDef *hdac, uint32_t Channel, uint32_t *pData, uint32_t Length, uint32_t Alignment);

```

这是普通输出的API

```c
/**
 * @brief  启动指定 DAC 通道的普通输出模式。
 * @param  hdac: 指向 DAC 句柄的指针。
 * @param  Channel: 要启动输出的 DAC 通道。
 *         可能的值:
 *           - DAC_CHANNEL_1: DAC 通道 1
 *           - DAC_CHANNEL_2: DAC 通道 2
 * @retval HAL status
 *         返回值:
 *           - HAL_OK: 操作成功
 *           - HAL_ERROR: 操作失败
 *           - HAL_BUSY: DAC 正在忙
 *           - HAL_TIMEOUT: 操作超时
 */
HAL_StatusTypeDef HAL_DAC_Start(DAC_HandleTypeDef *hdac, uint32_t Channel);

/**
 * @brief  停止指定 DAC 通道的普通输出模式。
 * @param  hdac: 指向 DAC 句柄的指针。
 * @param  Channel: 要停止输出的 DAC 通道。
 *         可能的值:
 *           - DAC_CHANNEL_1: DAC 通道 1
 *           - DAC_CHANNEL_2: DAC 通道 2
 * @retval HAL status
 *         返回值:
 *           - HAL_OK: 操作成功
 *           - HAL_ERROR: 操作失败
 *           - HAL_BUSY: DAC 正在忙
 *           - HAL_TIMEOUT: 操作超时
 */
HAL_StatusTypeDef HAL_DAC_Stop(DAC_HandleTypeDef *hdac, uint32_t Channel);


/**
  * @brief  设置指定 DAC 通道的输出数据。
  * @param  hdac: 指向 DAC 句柄的指针。
  * @param  Channel: 选择 DAC 通道。
  *         可能的值:
  *           - DAC_CHANNEL_1: DAC 通道 1
  *           - DAC_CHANNEL_2: DAC 通道 2
  * @param  Alignment: 数据对齐方式。
  *         可能的值:
  *           - DAC_ALIGN_8B_R: 8-bit 右对齐
  *           - DAC_ALIGN_12B_R: 12-bit 右对齐
  *           - DAC_ALIGN_12B_L: 12-bit 左对齐
  * @param  Data: 要输出到 DAC 的数据值（范围：0 到 4095，具体范围取决于对齐方式）。
  * @retval HAL status
  *         返回值:
  *           - HAL_OK: 操作成功
  *           - HAL_ERROR: 操作失败
  *           - HAL_BUSY: DAC 正在忙
  *           - HAL_TIMEOUT: 操作超时
  */
HAL_StatusTypeDef HAL_DAC_SetValue(DAC_HandleTypeDef *hdac, uint32_t Channel, uint32_t Alignment, uint32_t Data);


```


# 程序是示例:


---

## 普通 DAC 输出示例

```c
#include "stm32f4xx_hal.h"

// DAC 句柄
DAC_HandleTypeDef hdac;

// 输出的数值（0~4095）
// 假设我们需要输出一个正弦波
uint16_t sine_wave[100];

// 初始化 DAC 和正弦波数据
void DAC_Init(void) {
    // 初始化 DAC（假设已通过 CubeMX 配置）
    HAL_DAC_Init(&hdac);

    // 生成一个简单的正弦波（仅示例）
    for (int i = 0; i < 100; i++) {
        sine_wave[i] = (uint16_t)(2047 * (1 + sin(i * 2 * M_PI / 100)));
    }
}

// 主程序
int main(void) {
    HAL_Init();  // 初始化 HAL 库
    DAC_Init();  // 初始化 DAC 和数据

    // 启动 DAC 输出
    HAL_DAC_Start(&hdac, DAC_CHANNEL_1);

    // 在主循环中输出正弦波
    while (1) {
        for (int i = 0; i < 100; i++) {
            // 设置 DAC 输出的值
            HAL_DAC_SetValue(&hdac, DAC_CHANNEL_1, DAC_ALIGN_12B_R, sine_wave[i]);
            HAL_Delay(10);  // 延时控制输出速率
        }
    }
}
```

## DMA DAC 输出示例

```c
#include "stm32f4xx_hal.h"

// DAC 句柄
DAC_HandleTypeDef hdac;

// 输出的正弦波数据
uint16_t sine_wave[100];

// 初始化 DAC 和 DMA
void DAC_Init(void) {
    // 初始化 DAC（假设已通过 CubeMX 配置）
    HAL_DAC_Init(&hdac);

    // 生成一个简单的正弦波（仅示例）
    for (int i = 0; i < 100; i++) {
        sine_wave[i] = (uint16_t)(2047 * (1 + sin(i * 2 * M_PI / 100)));
    }

    // 启动 DMA 输出
    HAL_DAC_Start_DMA(&hdac, DAC_CHANNEL_1, (uint32_t *)sine_wave, 100, DAC_ALIGN_12B_R);
}

// 主程序
int main(void) {
    HAL_Init();  // 初始化 HAL 库
    DAC_Init();  // 初始化 DAC 和数据

    // 在 DMA 模式下持续输出正弦波
    while (1) {
        // 无需手动更新数据，DMA 会自动处理
        HAL_Delay(100);  // 控制主循环的速率
    }
}
```
---
                                                                                                                                                                本章完