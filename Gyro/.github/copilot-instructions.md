# GitHub Copilot 代码注释与格式规范

## 适用范围
本规范适用于 C/C++ 嵌入式项目（STM32），使用 C++14 标准。

---

## 核心原则
- **所有注释使用中文**
- **删除所有 Doxygen 格式注释**（`@brief`、`@param`、`@retval` 等）
- **保持代码简洁清晰**
- **大括号必须对称对齐**

---

## 1. 流水代码注释规则

### 规则
- **位置**：注释写在代码行尾
- **内容**：使用中文简洁描述代码功能
- **对齐**：尽可能在某一列对齐注释（推荐第 40-70 列）
- **例外**：如果某行代码特别长，注释可以不强制对齐

### 示例
```c
HAL_Init();                       // 初始化HAL库
SCB->VTOR = 0x08005000U;          // 重定位中断向量表到新的应用起始地址
SystemClock_Config();             // 配置系统时钟
```

### 生成代码时
- 为关键操作添加行尾注释
- 注释尽量对齐
- 使用中文，简洁明了

---

## 2. 块注释规则

### 规则
- **位置**：块代码的 `{` 后添加注释
- **内容**：描述该块代码的意图，简洁明了
- **格式**：注释与 `{` 同行，`{` 后留一个空格

### 示例
```c
if (g_IsLegal == 0)
{ // 检查合法性
    blink_time = 260;             // 程序不合法提示
}
else
{ // 合法状态处理
    blink_time = 1000;            // 正常闪烁间隔
}

for (int i = 0; i < count; i++)
{ // 遍历所有设备
    ProcessDevice(i);             // 处理设备
}

while (isRunning)
{ // 主循环
    Update();                     // 更新状态
}
```

### 生成代码时
- 每个控制块（if/else/for/while/switch）的开括号后必须添加注释
- 描述该块的目的

---

## 3. 复杂逻辑注释规则

### 规则
- **位置**：复杂逻辑前添加多行注释
- **内容**：详细描述逻辑规则或背景信息
- **格式**：使用多行 `//` 注释，段落清晰

### 示例
```c
// 闪灯规则：
// 未注册（程序不合法）：快速闪烁50ms
// 已注册（程序合法）：正常闪烁1s
// CAN设备号有效：200ms闪烁一次
if (!IsRegistered())
{ // 未注册状态
    blink_interval = 50;
}
else if (HasValidCANID())
{ // CAN设备号有效
    blink_interval = 200;
}
```

### 生成代码时
- 对于复杂的状态机、算法或业务逻辑
- 在代码块前添加多行注释说明
- 解释设计思路和注意事项

---

## 4. 函数注释规则

### 规则
- **禁止使用** Doxygen 格式（`@brief`、`@param`、`@retval` 等）
- **位置**：只在函数名后的 `{` 处添加中文注释
- **内容**：简洁描述函数功能
- **格式**：注释与 `{` 同行，`{` 后留一个空格

### 错误示例（不要生成）
```c
/**
 * @brief LED线程函数
 * @param argument 线程参数
 * @retval None
 */
static void LED_Thread1(void const *argument)
{
    // ...
}
```

### 正确示例
```c
static void LED_Thread1(void const *argument)
{ // LED1闪烁任务
    for (;;)
    { // 主循环
        HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);   // 翻转LED状态
        osDelay(1000);                                // 延时1秒
    }
}

void InitSystem(void)
{ // 初始化系统
    HAL_Init();                   // 初始化HAL库
    SystemClock_Config();         // 配置系统时钟
}

int CalculateSum(int *data, int len)
{ // 计算数组和
    int sum = 0;                  // 累加器
    for (int i = 0; i < len; i++)
    { // 遍历数组
        sum += data[i];           // 累加
    }
    return sum;                   // 返回结果
}
```

### 生成代码时
- 删除任何 Doxygen 格式注释
- 只在函数开括号后添加简短的中文功能描述
- 函数体内的重要操作添加行尾注释

---

## 5. 大括号使用规则

### 规则
- **必须对称在同一列上**
- **左括号通常不单独占一行**（函数定义除外）
- **右括号单独占一行**

### 示例
```c
void MyFunction(void)
{ // 函数功能说明
    if (condition)
    { // 条件成立
        DoSomething();            // 执行操作
    }
    else
    { // 条件不成立
        DoOtherThing();           // 执行其他操作
    }
}

int main(void)
{ // 主函数
    while (1)
    { // 主循环
        if (flag)
        { // 标志有效
            Process();            // 处理
        }
    }
    return 0;                     // 返回
}
```

### 生成代码时
- 确保所有大括号对齐
- 保持一致的缩进（Tab 或 4 空格）

---

## 6. 特殊说明

### 注释密度
- 不是每一行都需要注释
- 关键操作、状态变化、重要逻辑必须注释
- 自解释的代码可以不加注释

### 注释语言
- 统一使用中文
- 专业术语可保留英文（如 HAL、GPIO、CAN、RTOS）
- 避免中英文混杂

### 变量声明
```c
osThreadId LEDThread1Handle;      // 线程1句柄
uint32_t blink_time = 1000;       // 闪烁时间间隔
GPIO_InitTypeDef GPIO_Init;       // GPIO初始化结构体
```

---

## 7. 完整示例

```c
#include <stm32f1xx_hal.h>
#include <cmsis_os.h>

osThreadId LEDThreadHandle;                                       // LED线程句柄

static void LED_Thread(void const *argument);

int main(void)
{ // 主函数
    HAL_Init();                                                   // 初始化HAL库
    SystemClock_Config();                                         // 配置系统时钟
    
    __GPIOB_CLK_ENABLE();                                         // 使能GPIOB时钟
    GPIO_InitTypeDef GPIO_Init;                                   // GPIO初始化结构体
    
    GPIO_Init.Pin = GPIO_PIN_9;                                   // 配置引脚9
    GPIO_Init.Mode = GPIO_MODE_OUTPUT_PP;                         // 推挽输出模式
    GPIO_Init.Speed = GPIO_SPEED_FREQ_HIGH;                       // 高速
    GPIO_Init.Pull = GPIO_NOPULL;                                 // 无上下拉
    HAL_GPIO_Init(GPIOB, &GPIO_Init);                             // 初始化GPIOB
    
    osThreadDef(LED, LED_Thread, osPriorityNormal, 0, 128);       // 定义线程
    LEDThreadHandle = osThreadCreate(osThread(LED), NULL);        // 创建线程
    
    osKernelStart();                                              // 启动调度器
    
    for (;;)
    { // 主循环
        ;
    }
}

void SysTick_Handler(void)
{ // 系统滴答中断处理
    HAL_IncTick();                                                // 递增系统时钟
    osSystickHandler();                                           // RTOS滴答处理
}

static void LED_Thread(void const *argument)
{ // LED闪烁任务
    (void) argument;                                              // 抑制未使用参数警告
    
    for (;;)
    { // 主循环
        HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_9);                    // 翻转LED状态
        osDelay(1000);                                            // 延时1秒
    }
}
```

---

## 重要提示

在生成或修改代码时，请：
1. ? **始终使用中文注释**
2. ? **在函数开括号后添加功能注释**
3. ? **在控制块开括号后添加目的注释**
4. ? **为关键代码行添加行尾注释并对齐**
5. ? **确保大括号对称对齐**
6. ? **不要使用 Doxygen 格式注释**
7. ? **不要生成英文注释**
8. ? **不要省略块注释**

遵循这些规则可以提高代码的可读性和维护性。
