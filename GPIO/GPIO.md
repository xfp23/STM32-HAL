# STM32 HAL库开发指南
**本频道只提供idf编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

---

 - STM32D的GPIO状态切换有三种模式，低速、中速、高速，分别对应 2MHZ、10MHZ、50MHZ

---

# CUBE_MX配置生成HAL库

## Pinout&Cofniguration 菜单

1. 选择SYS 

- Debug 选择Serial Wire (如果不选择Seria Wire ,下载一次程序后就不能再下载)

2. GPIO 选项卡功能介绍: 

- `GPIO output level` : 选择默认电平

- `GPIO mode` ： 选择引脚模式 ：

| **参数**                   | **说明**                      |
|----------------------------|-------------------------------|
| **output push-pull**        | 输出推挽（常见模式）           |
| **output open-drain**       | 输出开漏                       |
| **GPIO Pull-up/Pull-down**  | 选择上下拉                     |
| **Maximum output speed**   | 设置最大输出速度，中速、高速、低速 |
| **user_label**              | 设置用户自定义引脚标签         |


3. 生成MDK工程


- 所有初始化代码已经由CUBEMX完成

# 开始编程

## 初始化GPIO

### 初始化宏

- 输入配置宏
```c
GPIO_MODE_INPUT       // 输入模式
GPIO_PULLUP           // 上拉
GPIO_PULLDOWN         // 下拉
GPIO_NOPULL           // 无上下拉

```

- 输出配置宏 
```c
GPIO_MODE_OUTPUT_PP   // 推挽输出
GPIO_MODE_OUTPUT_OD   // 开漏输出
GPIO_NOPULL           // 无上下拉
GPIO_PULLUP           // 上拉
GPIO_PULLDOWN         // 下拉
```

- 复用配置宏

```c
GPIO_MODE_AF_PP       // 复用推挽输出
GPIO_MODE_AF_OD       // 复用开漏输出
GPIO_NOPULL           // 无上下拉
GPIO_PULLUP           // 上拉
GPIO_PULLDOWN         // 下拉

```

- 模拟模式配置宏
```c
GPIO_MODE_ANALOG      // 模拟模式
GPIO_NOPULL           // 无上下拉

```

- 速度
```c
#define GPIO_SPEED_FREQ_LOW      0x00000000U   // 低速（2MHz）
#define GPIO_SPEED_FREQ_MEDIUM   0x00000001U   // 中速（10MHz）
#define GPIO_SPEED_FREQ_HIGH     0x00000003U   // 高速（50MHz）

```
### 示例代码
- 输入模式配置（带上下拉电阻）：
```c
GPIO_InitTypeDef GPIO_InitStruct = {0};
GPIO_InitStruct.Pin = GPIO_PIN_0;        // 选择引脚
GPIO_InitStruct.Mode = GPIO_MODE_INPUT;  // 设置为输入模式
GPIO_InitStruct.Pull = GPIO_PULLUP;      // 设置为上拉
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;  // 设置为低速
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);  // 初始化 GPIOA 的 Pin 0

```

- 输出模式配置（推挽输出）：
```c
GPIO_InitTypeDef GPIO_InitStruct = {0};
GPIO_InitStruct.Pin = GPIO_PIN_0;        // 选择引脚
GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;  // 设置为推挽输出模式
GPIO_InitStruct.Pull = GPIO_NOPULL;      // 设置无上下拉
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_MEDIUM;  // 设置为中速
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);  // 初始化 GPIOA 的 Pin 0

```

- 复用模式配置（推挽输出，复用功能）：
```c
GPIO_InitTypeDef GPIO_InitStruct = {0};
GPIO_InitStruct.Pin = GPIO_PIN_0;        // 选择引脚
GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;  // 设置为复用推挽输出模式
GPIO_InitStruct.Pull = GPIO_NOPULL;      // 设置无上下拉
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;  // 设置为高速
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);  // 初始化 GPIOA 的 Pin 0
 
```

- 模拟引脚配置
```c
GPIO_InitTypeDef GPIO_InitStruct = {0};
GPIO_InitStruct.Pin = GPIO_PIN_0;        // 选择引脚
GPIO_InitStruct.Mode = GPIO_MODE_ANALOG; // 设置为模拟模式
GPIO_InitStruct.Pull = GPIO_NOPULL;      // 设置无上下拉
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);  // 初始化 GPIOA 的 Pin 0

```

- 外部中断配置
抱歉给您带来困扰！我明白了，下面是您要求的宏和示例代码块：

### GPIO 相关的宏定义

```c
#define GPIO_MODE_IT_RISING          0x10100000U    // 上升沿触发中断
#define GPIO_MODE_IT_FALLING         0x10200000U    // 下降沿触发中断
#define GPIO_MODE_IT_RISING_FALLING  0x10300000U    // 上升沿和下降沿触发中断

#define GPIO_PULLUP                  0x00000001U    // 上拉
#define GPIO_PULLDOWN                0x00000002U    // 下拉
#define GPIO_NOPULL                  0x00000000U    // 无上下拉
```

### 示例代码：配置 GPIO 为边沿触发

```c
GPIO_InitTypeDef GPIO_InitStruct = {0};

// 配置 GPIO 为输入模式，上升沿和下降沿触发中断
GPIO_InitStruct.Pin = GPIO_PIN_0;                          // 选择引脚（GPIOA Pin 0）
GPIO_InitStruct.Mode = GPIO_MODE_IT_RISING_FALLING;        // 设置为上升沿和下降沿触发
GPIO_InitStruct.Pull = GPIO_NOPULL;                         // 设置无上下拉电阻
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;               // 设置为低速
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);                    // 初始化 GPIOA 的 Pin 0

// 配置 NVIC 中断优先级
HAL_NVIC_SetPriority(EXTI0_IRQn, 0, 0);                   // 设置中断优先级
HAL_NVIC_EnableIRQ(EXTI0_IRQn);                            // 使能外部中断
```

## 操作GPIO
```c
/**
* @brief 给高低电平
* @param IO端口 GPIOA GPIOB GPIOC
* @param 具体的GPIO引脚号
* @param 引脚状态 GPIO_PIN_RESER 低电平 GPIO_PIN_SET
*/ 
void HAL_GPIO_WritePin(GPIOX,GPIO_Pin,PinState);

/**
* @brief 读取引脚
* 
*/
void HAL_GPIO_ReadPin(GPIOX,GPIO_Pin);

/**
* @brief 延时函数
* @param ms 时间
*/

HAL_Daly(uint32_t ms);
```

# NVIC外部中断

` 由于外部中断又要去MX配置，所以此篇章到此结束`

[此处查看GPIO外部中断](./GPIONVIC.MD)
