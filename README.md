# STM32 电控组第一次培训 基础作业

本仓库为第一次作业，按任务拆分为三个独立的 Keil 工程。

## 完成内容

- 任务1：外接 LED 以 500ms 周期闪烁，使用 `HAL_Delay(500)`。
- 任务2：外接 LED 以 500ms 周期闪烁，不使用 `HAL_Delay`，使用 `HAL_GetTick()` 非阻塞计时。
- 任务3：按键 EXTI 控制 LED 开关，PB0 下降沿中断 + 20ms 软件消抖。
- 任务4：代码已提交到本仓库。

## 硬件与接线

- 开发板：STM32F103C8T6 最小系统板
- 烧录器：ST-Link V2，SWD 接口
- LED：PA0 → 330Ω → LED 长脚 → LED 短脚 → GND
- LED 为高电平点亮
- 按键：PB0 → 按键 → GND
- 按键使用内部上拉，按下为低电平，下降沿触发
- ST-Link 与开发板共地

## 任务说明

### 任务1

工程目录：`第一次作业-任务1/`

核心逻辑：

    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_0);
    HAL_Delay(500);

### 任务2

工程目录：`第一次作业-任务2/`

核心逻辑：

    if ((HAL_GetTick() - last_tick) >= 500U)
    {
        last_tick = HAL_GetTick();
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_0);
    }

不使用 `HAL_Delay`

### 任务3

工程目录：`第一次作业-任务3/`

- PB0 配置为 GPIO_EXTI0
- 上拉输入，下降沿触发
- 中断回调只记录时间戳和标志
- 主循环等待 20ms 后重新读取 PB0，仍为低电平才翻转 LED
- 现象：按一次翻转一次，长按只翻转一次

核心逻辑：

    if (key_pending && ((HAL_GetTick() - key_edge_tick) >= 20U))
    {
        key_pending = 0;

        if (HAL_GPIO_ReadPin(KEY_GPIO_Port, KEY_Pin) == GPIO_PIN_RESET)
        {
            HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_0);
        }
    }

## 构建与烧录

1. 用 Keil MDK-ARM 打开对应任务目录下的 `MDK-ARM/*.uvprojx`。
2. `F7` 编译。
3. `F8` 通过 ST-Link 下载。
4. 按开发板复位键运行。