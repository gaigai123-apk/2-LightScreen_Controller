## OLED 实时显示 LED 状态 - 功能介绍与使用说明

### 功能概述
- **实时显示**：在 0.96 寸 I2C OLED 上显示 LED 的当前状态（ON / OFF）。
- **串口控制**：通过 USART1 收发命令 `ON` / `OFF` 控制 LED，并同步更新 OLED。
- **HAL 兼容**：工程基于 STM32F1 HAL；OLED 驱动为软件 I2C，已适配 HAL GPIO。
- **中文与 ASCII**：内置 ASCII 字库，支持显示中文（需在字库中添加相应汉字）。

### 软硬件环境
- 芯片：STM32F103C8（BluePill 兼容开发板）
- 编译环境：Keil MDK V5 + ARMCC V5
- 串口：USART1 9600 8N1（PA9/PA10）
- OLED：4 引脚 I2C 版（SSD1306/SH1106 兼容）

### 硬件连接
- OLED 供电：VCC → 3.3V；GND → GND
- OLED I2C：
  - SCL → PB8（开漏 + 上拉）
  - SDA → PB9（开漏 + 上拉）
- 板载 LED：PC13（BluePill 常见，主动低，低电平点亮）

如需修改 OLED 引脚，请在 `Src/OLED.c` 中更改 `GPIOB, GPIO_PIN_8/9`；如需切换 LED 引脚或电平有效性，请在 `Src/gpio.c` 的 `LED_On/Off/IsOn` 中修改。

### 工程文件结构（与本功能相关）
- `Inc/OLED.h`、`Inc/OLED_Data.h`：OLED 外部接口与字库声明
- `Src/OLED.c`：软件 I2C + 显示/绘图实现（HAL 适配）
- `Src/OLED_Data.c`：ASCII/中文/图像字库（已填充完整 ASCII，中文索引为 UTF-8 转义）
- `Src/gpio.c`：LED 引脚初始化与 `LED_On/Off/IsOn` 实现（默认 PC13 主动低）
- `Src/main.c`：初始化 OLED、监听串口命令并在 OLED 上实时显示 LED 状态
- `MDK-ARM/5.uvprojx`：Keil 工程文件

### 快速开始
1) 打开工程
- 使用 Keil 打开 `1-Serial_LED_Controller/MDK-ARM/5.uvprojx`

2) 下载运行
- 连接 ST-LINK，上电后编译并下载程序

3) 串口测试
- 打开串口工具（9600 8N1，COM 口视系统而定）
- 发送 `ON` 或 `OFF`（回车可选）：
  - LED（PC13）相应点亮/熄灭
  - OLED 第一行显示 `LED: ON` 或 `LED: OFF`

### 运行时行为
- 上电后，`main.c` 初始化 OLED 并显示标题 `LED:`，随后根据 `LED_IsOn()` 在局部区域更新 `ON/OFF`。
- 串口接收到 `ON` / `OFF` 命令，调用 `LED_On/Off()` 并更新 OLED 显示。

### 二次开发与自定义
- 修改显示内容
  - 在 `Src/main.c` 的循环中，将 `OLED_ShowString(30, 0, ...)` 替换为自定义字符串或数值显示（如计数、传感器值）。
- 修改 OLED 引脚或 I2C 地址
  - 引脚：`Src/OLED.c` 中的 `GPIO_PIN_8/9`
  - 地址：`OLED_I2C_SendByte(0x78)` 中的 `0x78`（SSD1306 常见写地址），如需改为 `0x7A` 等请同步修改命令与数据发送处。
- 支持 1.3 寸 SH1106
  - 在 `OLED_SetCursor` 中启用 `X += 2` 的注释以补偿列偏移。
- 添加中文字模
  - 在 `Src/OLED_Data.c` 的 `OLED_CF16x16` 末尾，按现有格式添加新汉字索引（UTF-8 转义）和 16x16 点阵数据。

### 常见问题与排查
- OLED 显示乱码
  - 确认 `Src/OLED_Data.c` 的 ASCII 字库完整（已内置完整表）；若是中文乱码，需要将对应汉字加入字库。
  - 确认源码文件保存为 UTF-8（本工程使用 UTF-8 转义字符串避免编码差异）。
- OLED 无显示
  - 检查 3.3V 供电与 GND，I2C 上拉是否正确，SCL/SDA 引脚是否接对。
  - 确认设备地址（默认 0x78）是否匹配屏幕模块丝印/资料。
  - 上电先留给 OLED 充电泵稳定的延时已在 `OLED_GPIO_Init` 中包含，仍异常可适当增加。
- LED 不受控
  - 默认控制板载 PC13，主动低；若你使用外接 LED（如 PA0），请在 `gpio.c` 中改回对应引脚与电平有效性。
- 显示上下/左右颠倒
  - 在 `OLED_Init` 中调整 `0xA1/0xA0`（左右）与 `0xC8/0xC0`（上下）配置。

### 代码片段示例
在任意位置刷新某段文本：
```c
OLED_ShowString(0, 16, "Hello", OLED_6X8);
OLED_UpdateArea(0, 16, 6 * 5, 8); // 局部刷新 5 个字符
```

显示整数/浮点数：
```c
OLED_ShowNum(0, 32, 1234, 4, OLED_6X8);
OLED_ShowFloatNum(0, 40, 3.1415, 1, 3, OLED_6X8);
OLED_Update();
```

### 致谢与许可
- OLED 驱动与字库来源参考江协科技开源实现，已在本工程中进行 HAL 适配与整合。


