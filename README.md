# 背景

研0

**纯软基础**：学过C/C++ ，linux，linux系统编程网络编程，Qt/Qml

**转嵌软（MCU + linux）**：目前学了51单片机，部分数电知识，正在学习stm32

课程：江协科技stm32



# Keil μVision 5 代码补全时候卡死

**解决方法**：关掉电脑上的杀毒软件、电脑管家、安全卫士等等（关掉之后还是会有卡顿，但是不至于卡死）

这idea真垃圾，但是现在懒得换，等学完江科的就去换成vscode

# 基础知识

## STM32 

一类 32 位的 ARM 微控制器（MCU），使用 ARM 公司设计的 Cortex-M 内核

## ARM是什么

ARM既指ARM公司，也指ARM处理器内核

ARM公司是全球领先的半导体知识产权（IP）提供商，全球超逾95%的智能手机和平板电脑都采用ARM架构

ARM公司设计ARM内核，半导体厂商完善内核周边电路并生产芯片

ARM公司设计**A系列，R系列，M系列**内核

## 我的STM32

芯片：STM32F103C8T6

主频：72MHz

存储：20KB SRAM（RAM） + 64KB Flash（ROM）

## 总线是什么

总线（Bus） = 电子系统内部用来“传输数据”的高速通道

不同芯片内部模块之间、模块和存储器之间、模块与外设之间，都需要通过总线通信

## 串口是什么

https://zhuanlan.zhihu.com/p/1915485932267771322

**串口（Serial Port）= 一种按“位（bit）”顺序发送数据的通信方式。数据一位挨一位地传输，而不是一字节同时传输。**

常见的串口协议：

| 类型              | 全称                                                    | 用途                           |
| ----------------- | ------------------------------------------------------- | ------------------------------ |
| **UART**          | Universal Asynchronous Receiver/Transmitter             | MCU 与传感器通信、调试信息输出 |
| **USART**         | Universal Synchronous/Asynchronous Receiver/Transmitter | UART + SPI 特性                |
| **RS-232**        | PC 上传统串口接口                                       | 老设备、工业设备               |
| **RS-485**        | 工业设备远距离通信                                      | 电机控制、PLC、仪表            |
| **TTL UART**      | 微控制器串口                                            | Arduino、STM32、51、ESP32      |
| **Bluetooth SPP** | Serial Port Profile                                     | 蓝牙模拟串口，用于无线通信     |

## 时钟是什么

**一句话理解**

- 在电子工程里，时钟 = 芯片里所有电路运行的节奏信号，不是用来看时间的，是“节拍器”，不是“钟表”

在音乐里见过节拍器吧？嘀、答、嘀、答......节奏越快，演奏越快

**单片机内部的数字电路也是一样的：
每个功能都需要一个节奏来驱动它运行。**

例如：

- CPU 每个时钟周期执行一步指令
- UART 每个时钟周期发送一位
- I2C、SPI 的协议节奏也靠时钟
- ADC 每个时钟周期采样
- 定时器用时钟计数
- USB 必须固定 48MHz 时钟才能通信

==所以“时钟”在 MCU 里其实是一种节奏信号==

# 开始开发

## 环境搭建

- Kei5 MDK + 软件支持包

- STLINK驱动 ：**ST-Link 是 STM32 官方的“下载器 + 调试器”**，它能让电脑把程序写入 STM32，也能实时调试单片机内部运行情况，就像给 STM32 插上一根“数据和调试的 USB 线”。

> 我的电脑上是**DAP mini，是一种便宜、小巧、通用的 ARM 调试器（SWD 下载器 + 调试器），**
>  它使用 **CMSIS-DAP 协议**，是 ARM 官方标准协议，因此兼容性很好。它的作用类似于 **ST-Link / J-Link**，但更便宜、通用性更强，==不需要安装驱动==

- USB转串口驱动：是让电脑识别 CH340 芯片为 COM 口的驱动程序

## 开发方式

- 寄存器
- 库函数==（选择该方式进行开发）==
- HAL库

## startup_stm32f10x_xx.s 文件是什么

这些都是 STM32F10x 系列的**启动文件**（Startup File / 启动汇编）

它们负责 MCU 上电后最核心的事情：

1. 负责中断向量表（Vector Table）
   - Reset_Handler
   - HardFault_Handler
   - SysTick_Handler
   - 各种外设中断入口

2. 负责设置堆栈指针 SP

3. 调用 SystemInit()，再跳转到 main()

> 不同的启动文件对**不同 Flash 容量**的芯片做了微调，我选择startup_stm32f10x_md.s

## 连接STM32最小系统板和DAPmini

| STM32 引脚 | 名称     |
| ---------- | -------- |
| GND        | 地线     |
| SWCLK      | 调试时钟 |
| SWDIO      | 调试数据 |
| 3V3        | 电源     |

| DAPmini引脚              | 作用                 |
| ------------------------ | -------------------- |
| GND (Pin7/Pin8)          | 地线（必须接）       |
| SWDCLK (Pin5)            | SWD 时钟线           |
| SWDIO (Pin9)             | SWD 数据线           |
| 3.3V (Pin3) or 5V (Pin1) | 给目标板供电（可选） |

==调试器选择：CMSIS-DAP Debugger==

## 新建工程的步骤

1. 建立工程文件夹，新建工程，选择型号
2. 工程文件夹里建立Start（启动文件的选择由芯片的型号决定）、Library、User等文件夹，复制固件库里面的文件到工程文件夹
3. 工程里对应建立Start、Library、User等同名称的分组，然后将文件夹内的文件添加到工程分组里
4. 工程选项--->C/C++，Include Paths内声明所有包含头文件的文件夹工程选项
5. 工程选项--->C/C++，Define内定义`USE_STDPERIPH_DRIVER`
6. 工程选项--->Debug，下拉列表选择对应调试器`CMSIS-DAP Debugger`；Settings，Flash Download里勾选Reset and Run

# 如何看懂"寄存器方式"的代码

## 第一步：先搞清楚“这个寄存器属于哪个外设”

比如：

- `RCC->APB2ENR` → RCC 模块
- `GPIOC->CRH` → GPIO 模块
- `USART1->BRR` → 串口模块

## 第二步：打开 STM32F10xxx参考手册（中文）.pdf，跳到对应外设章节

目录一般是这样：

- 7.3 GPIO
- 7.4 RCC
- 8.x USART
- 9.x SPI

## 第三步：在这一章里找“寄存器列表”

会有一张表：列出这个外设所有寄存器和偏移地址

找到要的那个，比如 `GPIOx_CRH`

## 第四步：看该寄存器的 **位定义表**

比如：

| Bit   | Name                    | Description   |
| ----- | ----------------------- | ------------- |
| 23:20 | CNF13[1:0], MODE13[1:0] | Portx.13 配置 |

## 第五步：把十六进制数拆成二进制/按位理解

例如：

```
GPIOC->CRH = 0x00300000;
```

- 算出它哪几位是 1
- 对照位表对应到含义上

# GPIO

GPIO = General Purpose Input/Output（ 通用输入输出） ：**“通用输入输出端口”，就是 MCU 上最基本、最常用、能读能写的“引脚”**

- **功能1：**输出模式下可控制端口输出高低电平，用以驱动LED、控制蜂鸣器、模拟通信协议输出时序等

- **功能2：**输入模式下可读取端口的高低电平或电压，用于读取按键输入、外接模块电平信号输入、ADC电压采集、模拟通信协议接收数据等

## GPIO基本结构

https://www.cnblogs.com/Sharemaker/p/17107060.html

## GPIO 外设模块

GPIOA / GPIOB / GPIOC 是“GPIO 外设模块”

每个 GPIO 外设模块内部包含一组寄存器（ODR/IDR/CRL/CRH/BSRR/BRR 等）

通过 `GPIOx->寄存器` 来操作引脚

一个外设模块（如`GPIOB`） = 一组寄存器的集合

- 配置类寄存器：决定外设的模式与能力
- 控制类寄存器：实际驱动外设动作
- 输入类寄存器：读取外设状态 

# LED点灯例子

## 所需知识

STM32（尤其 F103）内部有三个外设总线：

```
AHB：高速外设（DMA、存储器）
APB1：低速外设（USART2、TIM2、I2C1…）
APB2：高速外设（GPIO、USART1、ADC1…）
```

外设挂在 AHB / APB1 / APB2 三条外设总线上，总线是 STM32 外设运行的“交通道路”

STM32 的总线（AHB/APB1/APB2）`时钟`本来就一直在工作，STM32 外设必须先开时钟才能工作

- 所以`LED 点灯 = 开启 GPIOA 外设模块的时钟 + 配置 PA0 为推挽输出 + 拉低 PA0 的电平`（我使用的是 GPIOA模块 与 PA0引脚）

时钟的开启和复位由 **RCC (Reset and Clock Control)** 模块控制

> RCC 是 STM32 芯片内部的一个独立硬件模块，负责整个芯片的时钟与复位管理，并为所有外设提供时钟，任何外设在使用前必须先通过 RCC 打开对应时钟

- 所以**开启/关闭具体外设的时钟**的函数的开头都是RCC

```c
  void RCC_AHBPeriphClockCmd(uint32_t RCC_AHBPeriph, FunctionalState NewState);
  void RCC_APB2PeriphClockCmd(uint32_t RCC_APB2Periph, FunctionalState NewState);
  void RCC_APB1PeriphClockCmd(uint32_t RCC_APB1Periph, FunctionalState NewState);
```

==注意我的LED灯设备中，短引脚是负极，长引脚是正极==

## 推挽输出 VS 开漏输出

**推挽输出GPIO_Mode_Out_PP**：引脚能主动输出 **高电平** 和 **低电平**

**开漏输出GPIO_Mode_Out_OD**：引脚只能主动输出 **低电平**，高电平需要 **外部电阻拉起来**

# GPIO常用的输入输出标准库函数

## 输出

```c
void GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);

void GPIO_ResetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);

void GPIO_WriteBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, BitAction BitVal);

void GPIO_Write(GPIO_TypeDef* GPIOx, uint16_t PortVal);
```
## 读取输入
```c
//读取 PA0~PA15 所有输入
uint16_t GPIO_ReadInputData(GPIO_TypeDef* GPIOx);

//读取指定引脚输入电平
uint8_t GPIO_ReadInputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
```

## 读取输出寄存器的输出状态

```c
uint16_t GPIO_ReadOutputData(GPIO_TypeDef* GPIOx);

uint8_t GPIO_ReadOutputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
```

# GPIO外设模块直接控制引脚电平的寄存器

## ODR（Output Data Register）

### 直接控制 GPIO 输出电平

操作例子：

```
GPIOA->ODR |= (1 << 5);    // PA5 输出高电平
GPIOA->ODR &= ~(1 << 5);   // PA5 输出低电平
```

会直接改变引脚电平

> 不推荐用于单个位操作（有读改写风险）

## BSRR（Bit Set/Reset Register）

### 最推荐控制引脚电平的寄存器

- 低 16 位：置 1（设为高电平）
- 高 16 位：置 0（设为低电平）

例如：

```
GPIOA->BSRR = (1 << 5);        // PA5 = 1
GPIOA->BSRR = (1 << (5 + 16)); // PA5 = 0
```

原子操作

> 标准库 GPIO_SetBits / GPIO_ResetBits / GPIO_WriteBit 都基于 BSRR

##  BRR（Bit Reset Register）

### 用于清零对应的引脚（输出低电平）

```
GPIOA->BRR = (1 << 5); // PA5 = 0
```

专门用于复位（置 0）

# OLED显示屏

> 跟着视频做之后显示屏不亮，将GPIOB的引脚8和9从**开漏输出**修改到**推挽输出**后解决

# 中断

中断是STM32 在运行主程序时，被“外部事件”或“内部事件”打断，跳去执行一段专门的函数（中断服务函数），执行完再回到原来位置继续跑

## 中断分类

### 外部中断（EXTI）

来自 MCU 外部的信号，例如：

- **按键按下（PA0）**
- 传感器信号
- 引脚电平变化（上升沿/下降沿）

经典例子：

```
PA0 按键按下 → 触发 EXTI0 → 执行 EXTI0_IRQHandler()
```

https://www.cnblogs.com/Xa-L/p/17552096.html****

### 内部外设中断

来自 STM32 内部硬件的事件，例如：

- **定时器溢出（TIM2 update）**
- **串口收到数据（USART1 RXNE）**
- **ADC 转换完成（ADC1 EOC）**
- DMA 完成
- SPI 通信事件
- I2C 事件

## NVIC（中断控制器）

### 中断要由 **NVIC（Nested Vectored Interrupt Controller）** 管理：

- 管理优先级（共16个优先级）
- 管理开/关
- 决定哪个中断先执行

### **NVIC 用 4 位优先级字段**

-  高 n 位 = 抢占优先级（决定能否中断其他中断）

- 低 4-n 位 = 响应优先级（决定排队顺序）

-  抢占优先级越小 → 越能抢占别人

-  抢占优先级相同时 → 响应优先级小的先执行

-  两者都相同时 → 按中断号排序执行

> **抢占优先级**：决定谁能“打断谁”，用于中断嵌套
>
> **响应优先级**：决定在同一抢占优先级下，谁先执行（排队顺序）

# EXTI与AFIO

## EXTI 是什么

EXTI 全称：

> **External Interrupt/Event Controller
>  外部中断/事件控制器**

它的作用：

> **把 GPIO 引脚的电平变化（上升沿、下降沿）转换成中断事件**

简单理解：

- GPIO 引脚变化 → 送给 EXTI
- EXTI 判断是否满足触发条件
- 满足 → 产生 EXTI 中断
- NVIC 处理中断

## EXTI 的输入来源是什么？

STM32F103 总共有 16 条 EXTI 线：

```
EXTI0 ~ EXTI15
```

每一条 EXTI 线 **可以接某一个 GPIO 引脚**：

- EXTI0 ← PA0 / PB0 / PC0 / PD0（四选一）
- EXTI1 ← PA1 / PB1 / PC1 / PD1（四选一）
- EXTI2 ← PA2 / PB2 / PC2 / PD2（四选一）
   …

> STM32 的 EXTI 线是 **按引脚序号，而不是端口** 进行索引的。

例如：

- PA0 → EXTI0
- PA1 → EXTI1
- PB0 → EXTI0
- PC0 → EXTI0

## EXTI 本身不能决定“用哪个 GPIO 引脚”

这就是 **AFIO 的作用**。

## AFIO 是什么？

AFIO = Alternate Function I/O
 **复用功能配置模块**

它是用来：

> **把某个 GPIO 引脚连接映射到某条 EXTI 线上的。**

也就是说：

> **AFIO 决定 GPIOx.y → 接到 EXTIy。**

## AFIO 怎么实现 GPIO → EXTI 的映射？

在 AFIO 模块里，有一个寄存器：

```
AFIO->EXTICR[0..3]
```

每个 EXTICR 寄存器控制 4 条 EXTI 线：

```
EXTICR0 → EXTI0, EXTI1, EXTI2, EXTI3
EXTICR1 → EXTI4~EXTI7
EXTICR2 → EXTI8~EXTI11
EXTICR3 → EXTI12~EXTI15
```

每条EXTI 线 4 位，用于选择端口：

| 数值 | 意义       |
| ---- | ---------- |
| 0000 | 选择 GPIOA |
| 0001 | 选择 GPIOB |
| 0010 | 选择 GPIOC |
| 0011 | 选择 GPIOD |

让 **PB0 触发 EXTI0**：

```c
AFIO->EXTICR[0] &= ~(0xF << 0);  // 清 EXTI0 的 4 位
AFIO->EXTICR[0] |=  (0x1 << 0);  // 选择 PB0（0001）
```

让 **PB1 触发 EXTI1**：
```c
AFIO->EXTICR[0] &= ~(0xF << 4);  // 清 EXTI1 的 4 位
AFIO->EXTICR[0] |=  (0x1 << 4);  // 选择 PB1（00010000）
```

## EXTI 与 AFIO 的关系一句话总结：

> **EXTI 是中断“引擎”（检测边沿 + 产生中断请求）**
>
> **AFIO 是 GPIO 到 EXTI 的“路由器”**

![image-20251128003824974](./assets/image-20251128003824974.png)

# EXTI与NVIC

> **EXTI 是中断“引擎”（产生中断请求）**
>
> **NVIC 是中断“调度器 / 管理中心”**

NVIC 接受到 EXTI 的中断请求后，会根据优先级调度各个请求

当轮到一个请求，例如轮到 EXTI0 时，就会跳转执行 `EXTI0_IRQHandler()` —— 而这个函数是程序员定义的

```c
void EXTI0_IRQHandler(void)
{
    if (EXTI_GetITStatus(EXTI_Line0) != RESET)
    {
        // 用户代码，例如翻转 LED
        EXTI_ClearITPendingBit(EXTI_Line0);
    }
}

```

