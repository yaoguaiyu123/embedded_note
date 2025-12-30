# 背景

学习过了江科大的stm32标准库

hal库课程：`keysking`

[课程文档](https://docs.keysking.com)

开发板：`正点原子 STM32F407 探索者`（STM32F407ZGT6）

仿真器：`CMSIS-DAP`（烧录、调试程序，虚拟串口）

# 开发环境搭建

## CubeIDE安装

安装教程：https://docs.keysking.com/docs/stm32/getting-started/getting-started_InstalCube

CubeIDE怎么调大工具栏的图标：

- https://blog.csdn.net/qq_39669243/article/details/139335163    造成UI坐标错误
- `XXXXXX\STM32CubeIDE_1.19.0\STM32CubeIDE\stm32cubeide.ini`
- 通过更改这个文件，在末尾添加`-Dswt.autoScale=160`实现更好的缩放

CubeIDE修改深色主题：https://blog.csdn.net/qq_41650023/article/details/127531997

## 登录 ST 账号

如果没有代理，很难登录，我这里使用代理登录，

- 在windows配置好代理（clash等工具）
- 打开`stm32cubeIDE`

- 菜单栏 **Window** -> **Preferences**

- 搜索 **Network**，找到 **General -> Network Connections**

- 将 `Active Provider` 从 `Native` 改为 **Manual**
- 如果windows早已配置了代理的话，这边会自动检测完成，最后直接点击`apply and close`即可

https://docs.keysking.com/docs/stm32/FAQ/login/#-%E5%A6%82%E4%BD%95%E7%99%BB%E5%BD%95

## 新建项目

新建stm32 project，选择`STM32F407ZGT6`芯片，接着都采用默认设置，随后IDE会下载固件包、配置工具链、自动生成 `main.c`、`startup_xxx.s` 等基础文件结构

给PF9 PF10 配置为低电平，点亮板载LED灯

## 配置烧录/调试工具为DAP

### 创建一个配置文件

1. 在左侧 **Project Explorer**（项目资源管理器）中，**右键点击你的工程名** -> **New** -> **File**

2. 文件名填：`dap.cfg`，点击 **Finish**

3. 在打开的空白文件中，复制粘贴下面这两行代码：

   Tcl

   ```
   source [find interface/cmsis-dap.cfg]
   source [find target/stm32f4x.cfg]
   ```

4. 按 `Ctrl + S` 保存

### 修改调试配置

1. 点击顶部菜单栏的 **Run** -> **Debug Configurations...**
2. 在左侧列表找到 **STM32 C/C++ Application**，点击下面的你的工程名
3. 在右侧点击 **Debugger**（调试器）选项卡
4. 找到 **Debug Probe**（调试探头）这一项，默认选的是 `ST-LINK (ST-LINK GDB Server)`
   - **请把它修改为：`OpenOCD`**
5. 修改后，页面下方会出现 **Configuration Script**（配置脚本）区域：
   - 勾选 **User Defined**（用户定义）
   - 点击 **Browse...**，在弹出的窗口中找到并选中你刚才创建的 `dap.cfg` 文件
6. 点击右下角的 **Apply**，然后点击 **Debug**



## 烧录BUG 

![image-20251229154849213](.\assets\image-20251229154849213.png)

表现为跳出如图的界面，然后板子上的程序运行一会之后停止，复位之后则正常运行

修改`dap.cfg`为
```cfg
# 1. 选择调试器接口 (CMSIS-DAP)
source [find interface/cmsis-dap.cfg]

# 2. 显式指定使用 SWD 接口
transport select swd

# 3. 选择目标芯片
source [find target/stm32f4x.cfg]

# 4. 降低调试速度
adapter speed 1000

# 5. 复位策略
reset_config none
```

烧录不会在弹出报错窗口，但是板子需要**手动复位**之后才运行程序



## 代码常驻自动补全

[STM32CubeIDE 代码自动补全设置_stm32cubeide 自动补全-CSDN博客](https://blog.csdn.net/wyn3514/article/details/139274725)

> 我试了之后发现不行，还是继续用 Alt + \ 吧

# CubeIDE项目文件介绍

## stm32f4xx_hal_conf.h

- **它的角色**：配置文件
- **通俗解释**： 这是公司的**采购清单**。STM32F407 功能非常多（有 ADC, CAN, I2C, SPI, UART...），但你的项目可能只需要用一个串口。 为了不把没用的代码都编译进去（浪费空间和时间），我们需要在这里划勾。
  - 打勾（Enable）了的模块，编译器才会去编译对应的驱动文件。
  - 没打勾的，编译器直接忽略。

## stm32f4xx_hal.h

- **它的角色**：总头文件

- **通俗解释**： 这是公司的**总经理**。任何想用 HAL 库的人，只要找他（Include 他）就行了，不需要单独去联系下面的部门（单独引用具体的 uart.h 或 gpio.h）。 总经理会拿着上面的“采购清单” (`conf.h`)，把所有启用的部门经理（头文件）都叫过来开会。

- **它做了什么？**

  1. 它包含了 `stm32f4xx_hal_conf.h`（先看清单）。

  2. 根据清单，自动帮你

     ```c
      #include "stm32f4xx_hal_uart.h"
      #include "stm32f4xx_hal_gpio.h" 
      // ...其他头文件
     ```

  3. 它还定义了全公司通用的规则，比如 `HAL_StatusTypeDef` (OK, ERROR, BUSY) 这种返回值类型。

## stm32f4xx_hal_msp.c

- **它的角色**：板级支持包硬件初始化

- **通俗解释**： 这是最容易让人晕的地方。它负责**脏活累活（硬件层面的连线）**。

  **举个栗子：初始化串口 (UART)**

  - **HAL_UART_Init (在 hal_uart.c 里)**：这是**“软件经理”**。他负责设置波特率是 115200，停止位是 1，校验位是 None。他在配置寄存器的逻辑。
  - **HAL_UART_MspInit (在 hal_msp.c 里)**：这是**“装修队长”**。软件经理在干活前，会把装修队长叫来：“哎，我要用 UART1，你去帮我把线接好”。 装修队长（MSP）负责：
    1. **开时钟**：把 UART1 的电闸拉上去 (`__HAL_RCC_USART1_CLK_ENABLE`)。
    2. **接引脚**：把芯片的 PA9 引脚连到 UART1 的 TX 功能上，PA10 连到 RX 上 (`HAL_GPIO_Init`)。
    3. **配中断**：如果要用中断，负责把 NVIC 里的中断线接通。

- **为什么一定要把逻辑和硬件分开？** 为了**移植性**！

  - `HAL_UART_Init` 是通用的，换个芯片代码不用变。
  - `HAL_UART_MspInit` 是**跟你的板子绑定的**。如果你换了一块板子，串口没接在 PA9/PA10，而是接在 PB6/PB7，你只需要改 MSP 文件里的引脚配置，不需要改主逻

# STM32 HAL 库移植正点原子 LCD 驱动

## 1.  准备工作

正点原子例程通常基于 Keil (GB2312 编码)，直接复制到 CubeIDE (UTF-8) 会乱码，将以下文件转码之后在复制到CubeIDE项目中，

- `lcd.c`, `lcd_ex.c` (放入 `Core/Src`)
- `lcd.h`, `lcdfont.h` (放入 `Core/Inc`)
- `sys.c`, `sys.h` (用于位带操作，可选，若保留需同样转码)

## 2.  CubeMX 配置

我们需要 HAL 库的 FSMC 底层文件 (`stm32f4xx_hal_sram.c`)，但不需要 CubeMX 生成的初始化逻辑。

- **Connectivity -> FSMC**:
  - **Bank**: `NOR Flash/PSRAM/SRAM/ROM/LCD 4` (对应 NE4/PG12)
  - **Memory Type**: `SRAM`
  - **Data Width**: **16 bits** 
  - **Address**: 任意非 0 值 (如 10 bits，仅为了解锁 Data Width 选项)
- **System Core -> GPIO**:
  - 配置 LED 引脚 (PF9, PF10) 为 `Output Push Pull` (用于心跳检测)。
- **Connectivity -> USART1**:
  - 配置为 `Asynchronous` (用于后续 printf 调试)。
- **生成代码**：点击 Generate Code。

## 3.  代码移植与冲突解决

### 解决“重定义”

- **FSMC 初始化冲突**：
  - 在 `stm32f4xx_hal_msp.c` 中，**注释掉** `HAL_SRAM_MspInit()` 函数。
  - *原因*：原子的 `lcd.c` 已经手写了包含 GPIO 配置的 MSP 初始化，保留原子的。
- **LCD 扩展驱动包含**：
  - 在 `lcd.c` 中，**删除/注释** `#include "lcd_ex.c"`。
  - *原因*：CubeIDE 会自动编译项目内所有 .c 文件，重复 include 会导致重复编译。

### 解决“延时函数” (这一步是可选的)

- **抛弃原子 `delay.c`**：删除 `delay.c` 和 `delay.h`，统一使用 HAL 库时钟。
- **替换 `lcd.c` / `lcd_ex.c` 中的延时**：
  - 引入头文件：`#include "main.h"`
  - 替换毫秒：全局替换 `delay_ms` -> `HAL_Delay`。
  - 补全微秒：手动实现一个简单的 `lcd_delay_us()`，并将代码中的 `delay_us` 替换之。
  - **公开接口**：在 `lcd.h` 中声明 `void lcd_delay_us(uint32_t us);` 供 `lcd_ex.c` 调用。

### 避坑 (==每次生成代码都要检查==)

- 在 `main.c` 中，**注释掉 `MX_FSMC_Init();`**
  - 原子的 `lcd_init()` 内部会重新配置 FSMC 寄存器。如果让 CubeMX 先初始化一遍，可能会导致引脚时序被覆盖或冲突
- 在 `stm32f4xx_hal_msp.c` 中，**注释掉`HAL_SRAM_MspInit()`**整个函数
  - 原子哥觉得 CubeMX 生成的代码太啰嗦或者为了兼容旧板子，自己在 `lcd.c` 里也写了一个 `HAL_SRAM_MspInit`（他也想负责初始化 FSMC 的引脚），但是与`stm32f4xx_hal_msp`中的`HAL_SRAM_MspInit`冲突了

# CubeIDE中的项目重用

假设现在的项目叫 `demo02`，想把它复制成 `demo03`

## 1.复制与清理

**复制文件夹**：在 Windows 文件管理器里，把 `demo02` 文件夹复制一份，重命名为 `demo03`

**清理垃圾**：进入 `demo03` 文件夹，**删除** `Debug` 和 `Release` 文件夹。（这些是编译生成的临时文件，删了可以减小体积，而且新项目需要重新编译，留着反而容易出 Bug）

## 2.关键文件改名

**重命名 .ioc 文件**：

- 把文件夹里的 `demo02.ioc` 重命名为 **`demo03.ioc`**。

**修改 .project 文件 **：

- 找到文件夹里的 **`.project`** 文件
- 用 **vscode** 打开它。
- 找到 `<name>demo02</name>` 这一行
- 把它改成：`<name>demo03</name>`

> **为什么要改 .project？** CubeIDE 不是靠文件夹名字识别项目的，而是靠这个文件里的 `<name>` 标签。如果不改，当你导入新项目时，IDE 会提示：“工作空间里已经有一个叫 demo02 的项目了”，导致导入失败。

## 3.导入新项目

- 打开 STM32CubeIDE

- 点击 **File** -> **Import**

- 选择 **General** -> **Existing Projects into Workspace**，点击 Next

- **Select root directory**：选择你刚刚改好的 `demo03` 文件夹

- **Projects** 列表里应该会出现 `demo03`

- 点击 **Finish**

# 串口

- 轮询
- 中断
- DMA

# 将一个裸机项目升级到FreeRTOS

### 1.CubeMX 配置 (基建工程)

1. **修改时基 (Timebase Source) **
   - **操作**：`System Core` -> `SYS` -> `Timebase Source`。
   - **修改**：将默认的 `SysTick` 改为 **`TIM1`** (或其他定时器)。
   - **原因**：FreeRTOS 必须独占 SysTick 用来做任务调度的心跳，HAL 库的延时函数就需要用别的定时器，否则会冲突死机。
2. **启用 FreeRTOS**
   - **操作**：`Middleware` -> `FREERTOS` -> `Interface` 选择 **`CMSIS_V1`**。
3. **规划与创建任务 (Tasks)**
   - **操作**：在 `Tasks and Queues` 标签页下点击 `Add`。
   - **优先级**：通常设为 `osPriorityNormal`。

### 2.逻辑搬家 

这是“大脑移植”手术，把原来的“单线程思维”转变为“多线程思维”。

1. **拆分 `while(1)`**
   - **裸机做法**：一个大 `while(1)` 里顺序执行 `检测按键` -> `刷屏` -> `闪灯`。
   - **RTOS做法**：
     - 按键和闪灯逻辑 -> 搬进 **`Led_Task_Entry`** 的死循环。
     - 刷屏和串口处理 -> 搬进 **`Gui_Task_Entry`** 的死循环。
2. **替换延时函数**
   - **查找**：`HAL_Delay(ms)`
   - **替换**：**`osDelay(ms)`**
   - **原理**：`HAL_Delay` 是傻等（阻塞 CPU），`osDelay` 是休息（释放 CPU 给别的任务用）。**任务的死循环里必须包含至少一个 `osDelay`**，否则低优先级任务会饿死。

