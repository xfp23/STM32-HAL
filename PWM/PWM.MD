### STM32 HAL 库开发指南：PWM 配置与使用

---

#### **版权信息**  
© 2024 . 未经许可不得复制、修改或分发。  
此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

---

### **STM32 PWM 编程指南**

#### 1. 配置 CubeMX

1. 打开 CubeMX 工具，选择定时器，启用对应的通道，并设置通道模式为 `PWM Mode`。
2. 在通道配置中选择需要的输出极性（正极性或负极性）和 PWM 模式（如 PWM1 或 PWM2）。
3. 根据应用需求，调整 `Prescaler`（PSC）和 `Auto Reload Register`（ARR）的值，确保频率和分辨率符合要求。

---

#### 2. PWM 输出频率计算

PWM 输出频率的公式为：

\[
f_{\text{PWM}} = \frac{f_{\text{TimerClock}}}{(\text{PSC} + 1) \times (\text{ARR} + 1)}
\]

**参数说明**：
- \( f_{\text{TimerClock}} \)：定时器的输入时钟频率。
- \( \text{PSC} \)：预分频器值（范围：0 ~ 65535）。
- \( \text{ARR} \)：自动重装载寄存器值（范围：0 ~ 65535）。

**计算步骤**：
1. **确定时钟源**：检查定时器的时钟来源（一般为 APB 时钟）。  
   - 若 APB 预分频因子为 1，则 \( f_{\text{TimerClock}} = f_{\text{PCLK}} \)。  
   - 若 APB 预分频因子不为 1，则 \( f_{\text{TimerClock}} = f_{\text{PCLK}} \times 2 \)。

2. **设置目标频率**：根据应用需求确定 PWM 的目标频率。

3. **计算 PSC 和 ARR**：
   - 选择一个合适的 \( \text{PSC} \)，并计算对应的 \( \text{ARR} \)。  
   示例：  
   - 定时器时钟频率：\( f_{\text{TimerClock}} = 170 \, \text{MHz} \)  
   - 目标频率：\( f_{\text{PWM}} = 1 \, \text{kHz} \)  

   使用公式计算：  
   \( \text{PSC} = 1699 \) 和 \( \text{ARR} = 99 \)

---

#### 3. 配置占空比

占空比通过捕获比较寄存器（CCR）的值配置，其公式为：

\[
\text{占空比} (\%) = \frac{\text{CCR}}{\text{ARR} + 1} \times 100
\]

**示例**：
- 若 \( \text{ARR} = 99 \) 且占空比为 50%，则  
  \( \text{CCR} = (\text{ARR} + 1) \times 50\% = 50 \)。

---

#### 4. 使用 HAL 库输出 PWM

以下是典型的代码示例：

```c
// 启动 PWM 输出
HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);

// 设置占空比为 50%
uint32_t ARR = __HAL_TIM_GET_AUTORELOAD(&htim2);
uint32_t CCR = (ARR + 1) * 0.5;  // 占空比 50%
__HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, CCR);
```

---

#### 5. 常见问题排查

1. **PWM 无输出**：
   - 检查 CubeMX 配置是否正确。
   - 确保定时器已启动，通道已使能。

2. **频率不正确**：
   - 检查时钟配置和 PSC、ARR 的计算是否正确。

3. **占空比异常**：
   - 确认 CCR 的值是否合理。

---

#### 6. 示例应用场景

- **LED 调光**：使用 PWM 调节 LED 的亮度。
- **电机控制**：通过互补 PWM 配合死区时间控制 H 桥电路。

---

### 附录：相关配置选项

| **选项**                     | **说明**                                                                                 |
|------------------------------|------------------------------------------------------------------------------------------|
| **PWM Mode (PWM 模式)**       | 用于输出 PWM 信号，调节占空比。                                                         |
| **Edge-Aligned Mode**         | 边沿对齐模式，PWM 输出在计数器到达指定值时变化。                                         |
| **Center-Aligned Mode**       | 中心对齐模式，PWM 输出在计数器正反计数过程中对称变化。                                   |
| **Capture/Compare Polarity**  | 捕获/比较极性，确定输出信号的高低电平切换边沿。                                         |
| **Output Compare Preload**    | 设置为 `Enable` 确保 PWM 输出更新的平滑性和稳定性。                                      |

---

