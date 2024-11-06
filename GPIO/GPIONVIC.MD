```markdown
# STM32 NVIC 外部中断配置指南

## CubeMX 配置

1. **打开 NVIC 选项卡**

   - 在 CubeMX 中，选择 **Configuration** -> **NVIC** 选项卡。
   - 在此界面可以看到所有中断控制器的配置项。

2. **选择外部中断**

   - 找到需要配置的外部中断（例如 `EXTI0`, `EXTI1`, 等）。
   - 勾选相应的 **Enabled** 复选框，启用中断。

3. **设置优先级**

   - 在 `Priority` 列下选择所需的优先级。
   - STM32 中的中断优先级由两部分组成：**抢占优先级** 和 **子优先级**。可以根据需要设置。

4. **配置触发条件**

   - 配置中断触发的方式，如 **上升沿**、**下降沿** 或 **双边沿**。
   - 在 **GPIO** 配置中选择相应的触发模式，例如：上升沿触发、下降沿触发或双边沿触发。

5. **生成代码**

   - 配置完成后，点击 **Project** 生成代码。
   - CubeMX 将自动生成所有的 NVIC 配置代码及初始化代码。

## 中断服务函数

在中断回调函数中编写处理逻辑。示例代码如下：

```c
// 外部中断处理函数
void EXTI0_IRQHandler(void)
{
    // 检查是否为外部中断线0触发
    if(__HAL_GPIO_EXTI_GET_IT(GPIO_PIN_0) != RESET)
    {
        // 清除中断标志位
        __HAL_GPIO_EXTI_CLEAR_IT(GPIO_PIN_0);
        
        // 用户自定义的中断处理代码
        // 例如：切换LED状态
        HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);
    }
}
```

### 其他常用操作

1. **使能外部中断**
   使用 `HAL_NVIC_EnableIRQ()` 函数使能中断。

   ```c
   HAL_NVIC_EnableIRQ(EXTI0_IRQn);
   ```

2. **禁用外部中断**
   使用 `HAL_NVIC_DisableIRQ()` 函数禁用中断。

   ```c
   HAL_NVIC_DisableIRQ(EXTI0_IRQn);
   ```

3. **设置中断优先级**
   使用 `HAL_NVIC_SetPriority()` 函数设置中断的优先级。

   ```c
   HAL_NVIC_SetPriority(EXTI0_IRQn, 1, 0);  // 抢占优先级1，子优先级0
   ```

## 其他配置项

- **中断优先级**：根据需求调整不同外设的优先级，确保优先级合理。
- **外部中断线配置**：可以通过配置 `GPIO` 引脚来设置触发中断的条件（上升沿、下降沿或双边沿）。

```c
// 使能外部中断触发条件
HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_0);  // 处理外部中断
```