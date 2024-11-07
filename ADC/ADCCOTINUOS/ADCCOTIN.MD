# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# STM32 ADC编程指南

# CubeMX配置

1. 打开Analog选项卡，选择自己想使能的ADC模块

2. 观察ADC模块的配置选项`IN0~IN9`每个都对应一个引脚，选择自己想使能的ADC引脚

3. 查看时钟配置界面的ADC出时钟频率的大小，一般为4MHZ

4. 打开ADC配置的选项卡找到`ADC_Regular_ConversionMode`选项框


| **ADC_Regular_ConversionMode (常规序列设置)**    |                                                                  |
|--------------------------------------------------|------------------------------------------------------------------|
| **Enable Regular**                               | Enable (使能常规序列，选这个) | Disable (禁用)                        |
| **Number of Conversion (设置ADC转换的数量)**     | 范围(1~16) 用了几个ADC采样引脚就填几个                               |
| **External Trigger Conversion Source (常规序列的外部触发信号)** | Regular Conversion launched by software (使用编程手段启动常规序列，选这个不依靠中断) |




5. 设置了`Number of Conversion`选项卡之后，会弹出`Rank`选项卡


| **Rand选项卡**                                    |                                                                  |
|--------------------------------------------------|------------------------------------------------------------------|
| **channel**                                      | 选择通道                                                          |
| **Sampling Time**                                | 采样周期，通过计算得出合适的采样周期，[此处是计算方法](./ADC/ADC_EX.md) |
|                                                  | 根据计算结果设置合适的采样周期，选择最接近的那个，要求快速选小的，要求精度选大的 |


# 初始化

1. ADC句柄类型:
```c
ADC_HandleTypeDef;
```

2. 初始化示例:
```c
#include "stm32f1xx_hal.h"  // 根据实际芯片型号替换 HAL 库头文件

ADC_HandleTypeDef hadc1;

void MX_ADC1_Init(void) {
    ADC_ChannelConfTypeDef sConfig = {0};

    hadc1.Instance = ADC1;  // 使用 ADC1 实例

    // 扫描模式设置
    // ADC_SCAN_ENABLE：启用扫描模式（多通道采集）
    // ADC_SCAN_DISABLE：禁用扫描模式（单通道采集）
    hadc1.Init.ScanConvMode = ADC_SCAN_DISABLE;

    // 连续转换模式
    // ENABLE：启用连续转换模式
    // DISABLE：禁用连续转换模式
    hadc1.Init.ContinuousConvMode = DISABLE;

    // 间断模式设置
    // ENABLE：启用间断模式
    // DISABLE：禁用间断模式
    hadc1.Init.DiscontinuousConvMode = DISABLE;

    // 触发源选择
    // ADC_SOFTWARE_START：软件触发
    // 其他硬件触发信号，如 ADC_EXTERNALTRIGCONV_T1_CC1 等
    hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;

    // 数据对齐方式
    // ADC_DATAALIGN_RIGHT：右对齐
    // ADC_DATAALIGN_LEFT：左对齐
    hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;

    // 转换的通道数
    // 在扫描模式下，指定需要转换的 ADC 通道数量
    hadc1.Init.NbrOfConversion = 1;

    // 初始化 ADC 并检查是否成功
    if (HAL_ADC_Init(&hadc1) != HAL_OK) {
        // 初始化失败时的错误处理
        Error_Handler();
    }

    // 配置 ADC 通道（假设使用通道 0）
    // Channel：ADC 通道选择（例如 ADC_CHANNEL_0）
    sConfig.Channel = ADC_CHANNEL_0;

    // 转换顺序
    // Rank = 1 表示第一个转换通道，值越小优先转换
    sConfig.Rank = 1;

    // 采样时间设置
    // ADC_SAMPLETIME_1CYCLE_5：1.5 个周期
    // ADC_SAMPLETIME_7CYCLES_5：7.5 个周期
    // ADC_SAMPLETIME_13CYCLES_5：13.5 个周期
    // ADC_SAMPLETIME_28CYCLES_5：28.5 个周期
    // ADC_SAMPLETIME_41CYCLES_5：41.5 个周期
    // ADC_SAMPLETIME_55CYCLES_5：55.5 个周期
    // ADC_SAMPLETIME_71CYCLES_5：71.5 个周期
    // ADC_SAMPLETIME_239CYCLES_5：239.5 个周期
    sConfig.SamplingTime = ADC_SAMPLETIME_1CYCLE_5;

    // 初始化 ADC 通道配置
    if (HAL_ADC_ConfigChannel(&hadc1, &sConfig) != HAL_OK) {
        Error_Handler();
    }
}

```


# ADC操作API



1. **启动常规序列**  
```c
HAL_ADC_Start(ADC_HandleTypeDef *hadc);
```

2. **等待转换完成**  
```c
HAL_StatusTypeDef HAL_ADC_PollForConversion(ADC_HandleTypeDef *hadc, uint32_t Timeout);
```

3. **获取结果**  
```c
uint32_t HAL_ADC_GetValue(ADC_HandleTypeDef *hadc);
```

4. 转换电压 ： 
```c
//最终的电压结果是:
float vol = 原始值 * 3.3/4095; 

```

# demo示例

```c
#include "stm32f4xx_hal.h"

ADC_HandleTypeDef hadc1;

void ADC_Init(void)
{
    // 配置 ADC 的初始化结构体（例如通道、分辨率、采样时间等）
    __HAL_RCC_ADC1_CLK_ENABLE();  // 使能 ADC1 时钟

    hadc1.Instance = ADC1;
    hadc1.Init.Resolution = ADC_RESOLUTION_12B;      // 12位分辨率
    hadc1.Init.ScanConvMode = ADC_SCAN_DISABLE;       // 禁用扫描模式
    hadc1.Init.ContinuousConvMode = DISABLE;          // 禁用连续转换模式
    hadc1.Init.DiscontinuousConvMode = DISABLE;       // 禁用不连续转换模式
    hadc1.Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;  // 软件触发
    hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;       // 数据右对齐
    hadc1.Init.NbrOfConversion = 1;                    // 只进行一次转换
    hadc1.Init.DMAContinuousRequests = DISABLE;        // 禁用DMA
    hadc1.Init.EOCSelection = ADC_EOC_SINGLE_CONV;    // 单次转换完成时生成结束标志

    // 初始化 ADC
    if (HAL_ADC_Init(&hadc1) != HAL_OK)
    {
        // 错误处理
        Error_Handler();
    }
}

uint32_t ADC_SingleConversion(void)
{
    // 启动 ADC 转换
    if (HAL_ADC_Start(&hadc1) != HAL_OK)
    {
        // 启动失败，错误处理
        Error_Handler();
    }

    // 等待转换完成
    if (HAL_ADC_PollForConversion(&hadc1, 100) != HAL_OK)  // 100 毫秒超时
    {
        // 转换超时，错误处理
        Error_Handler();
    }

    // 获取转换结果
    uint32_t adc_value = HAL_ADC_GetValue(&hadc1);

    // 关闭 ADC
    HAL_ADC_Stop(&hadc1);

    return adc_value;
}

void Error_Handler(void)
{
    // 错误处理代码
    while (1)
    {
        // 可以在此处设置 LED 灯闪烁或其他调试机制
    }
}

int main(void)
{
    HAL_Init();           // 初始化 HAL 库
    ADC_Init();           // 初始化 ADC
    uint32_t adc_result = ADC_SingleConversion();  // 获取 ADC 单次转换结果

    // 可以将 adc_result 用于后续处理
    // 例如通过串口输出，或者控制外设
    while (1)
    {
        // 主循环代码
    }
}


```