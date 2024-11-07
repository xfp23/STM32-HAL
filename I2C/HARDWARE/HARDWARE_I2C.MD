# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

---

# STM32 HARDWARE I2C

# CubeMX配置

1. 打开`Connectivity`选项卡，选择要使用的I2C资源

2. 选择I2C 工作模式


### **I2C 配置模式**

| **Mode**                      | **Description**                   |
|-------------------------------|-----------------------------------|
| **I2C**                        | 标准 I2C 工作模式（常用）            |
| **SMBus-Alert-mode**           | SMBus 警报模式                      |
| **SMBus-two-wire-Interface**   | 系统管理总线（SMBus 两线接口）        |

---


3. I2C参数设置



### **I2C 主机设置**

| **Master Features** | **I2C 主机参数**         |
|---------------------|--------------------------|
| **I2C Speed Mode**   | **标准速度模式** (Standard Mode) <= 100kbps |
|                     | **快速模式** (Fast Mode) <= 400kbps |
| **I2C Clock Speed (Hz)** | 波特率 | 一般填最大，400k |
| **Fast Mode Duty Cycle** | 快速模式下时钟信号的占空比 |
|                     | **Duty cycle Tlow/Thigh = 2** | 低电平占两份，高电平占一份（常用） |
|                     | **Duty cycle Tlow/Thigh = 16/9** | 低电平占16份，高电平占9份 |

---

### **I2C 从机设置**

| **Slave Features**  | **I2C 从机参数**         |
|---------------------|--------------------------|
| **I2C Addressing Mode** | 地址模式 |
|                     | **7-bit Addressing Mode** | 7位地址模式 |
|                     | **10-bit Addressing Mode** | 10位地址模式 |
| **I2C Acknowledge**   | 响应设置 |
|                     | **Acknowledge Enable** | 启用应答 |
|                     | **Acknowledge Disable** | 禁用应答 |
| **I2C General Call**  | 广播地址模式 |
|                     | **Enable** | 启用 |
|                     | **Disable** | 禁用 |
| **I2C Duty Cycle**    | 快速模式下的时钟占空比 |
|                     | **Duty cycle Tlow/Thigh = 2** | 低电平占两份，高电平占一份（常用） |
|                     | **Duty cycle Tlow/Thigh = 16/9** | 低电平占16份，高电平占9份 |

---


# 初始化API

1. I2C句柄：
```c
I2C_HandleTypeDef;
```
2. I2C初始化代码:
```c
#include "stm32f1xx_hal.h"  // 请根据芯片型号替换 HAL 库头文件

I2C_HandleTypeDef hi2c1;

void MX_I2C1_Init(void) {
    hi2c1.Instance = I2C1;  // 选择 I2C1 作为 I2C 外设实例

    // 时钟速度设置
    // 常用值：100000 (标准模式，100kHz)，400000 (快速模式，400kHz)
    hi2c1.Init.ClockSpeed = 100000;

    // 地址模式设置
    // I2C_ADDRESSINGMODE_7BIT：7 位地址模式
    // I2C_ADDRESSINGMODE_10BIT：10 位地址模式
    hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;

    // 双地址模式
    // I2C_DUALADDRESS_DISABLE：禁用双地址模式
    // I2C_DUALADDRESS_ENABLE：启用双地址模式
    hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;

    // 第二地址
    // 如果启用双地址模式，可以设置另一个 7 位或 10 位地址
    hi2c1.Init.OwnAddress2 = 0;

    // 第二地址掩码
    // I2C_OA2_NOMASK：不屏蔽第二地址
    // I2C_OA2_MASK01 至 I2C_OA2_MASK07：用于屏蔽第二地址的位
    hi2c1.Init.OwnAddress2Masks = I2C_OA2_NOMASK;

    // 通用回呼模式
    // I2C_GENERALCALL_DISABLE：禁用通用回呼
    // I2C_GENERALCALL_ENABLE：启用通用回呼
    hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;

    // 无应答模式
    // I2C_NOSTRETCH_DISABLE：启用时钟拉伸
    // I2C_NOSTRETCH_ENABLE：禁用时钟拉伸（在主机模式下无效）
    hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;

    // 初始化 I2C 并检查是否成功
    if (HAL_I2C_Init(&hi2c1) != HAL_OK) {
        // 初始化失败时的错误处理
        Error_Handler();
    }
}


```
# I2C操作API
### **I2C 读写API**

| **API** | **描述** |
|--------|----------|
| `HAL_I2C_Master_Transmit` | 主机传输数据到从机 |
| `HAL_I2C_Master_Receive` | 主机从从机接收数据 |
| `HAL_I2C_Slave_Transmit` | 从机传输数据到主机 |
| `HAL_I2C_Slave_Receive` | 从机从主机接收数据 |

1. API原型:

- 主机传输数据到从机

```c
/**
 * @brief I2C发数据
 * @param i2c总线句柄
 * @param 从机地址
 * @param 要发送的数据的地址
 * @param 发送的数据的字节大小
 * @param 超时时间 ms 不使用传HAL_MAX_DELAY
 * @return HAL_OK HAL_ERROR HAL_BUSY HAL_TIMEOUT
 */
HAL_StatusTypedef HAL_I2C_Master_Transmit(I2C_HandleTypeDef *h2c,uint16_t devaddress,uint8_t *data,uint16_t size,uint32_t Timeout);
```

- 主机接收从机的信息
```c

/**
 * @brief I2C发数据
 * @param i2c总线句柄
 * @param 从机地址
 * @param 要接收的数据的缓冲区地址
 * @param 要接收的数据的字节大小
 * @param 超时时间 ms 不使用传HAL_MAX_DELAY
 * @return HAL_OK HAL_ERROR HAL_BUSY HAL_TIMEOUT
 */
HAL_StatusTypedef HAL_I2C_Master_Receive(I2C_HandleTypeDef *h2c,uint16_t devaddress,uint8_t *data,uint16_t size,uint32_t Timeout);

```

