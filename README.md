# 基于 STM32F103C8T6 的智能手表设计 —— 功能与创新点

## 一、项目简介

本智能手表以 STM32F103C8T6 为主控，集成 0.96 寸 SSD1306 OLED 显示屏、MPU6050 六轴陀螺仪、HC-05 蓝牙模块及 EC11 旋转编码器，采用 FreeRTOS 多任务实时调度架构，实现了传感器数据采集、无线传输、人机交互及锂电池独立供电等完整功能。同时配套 Flask Web 上位机，支持 PC 端实时数据可视化。

---

## 二、功能总览

### 2.1 主界面 — 时间日期与步数

实时显示当前时间、日期、累计步数。支持通过蓝牙发送时间同步帧校准日期时间。

![](picture/01_home.jpg)

### 2.2 多级菜单系统

按压编码器进入 MENU 主菜单，旋转翻页，短按进入子菜单，长按返回上级。10 秒无操作自动返回主界面。

![](picture/02_menu.jpg)

### 2.3 STEPS — 计步与健康估算

基于 MPU6050 加速度计的峰值检测 + 动态阈值算法统计步数，实时估算卡路里消耗和运动距离。

![](picture/03_steps.jpg)

### 2.4 SENSOR — 传感器数据

显示三轴加速度（X/Y/Z）、俯仰角（Pitch）、温度、电池电压，数据以 200Hz 频率采集。

![](picture/04_sensor.jpg)

### 2.5 CLOCK — 秒表与倒计时

**Clock 子菜单：**

![](picture/05_clock_menu.jpg)

**秒表**（开始 → 计时 → 暂停 → 复位）：

| 未开始 | 计时中 | 暂停 |
|:---:|:---:|:---:|
| ![](picture/06_timer_idle.jpg) | ![](picture/07_timer_run.jpg) | ![](picture/08_timer_pause.jpg) |

**倒计时**（编码器设置时间 → 运行 → 结束）：

| 设置中 | 过程中 | 结束 |
|:---:|:---:|:---:|
| ![](picture/09_countdown_set.jpg) | ![](picture/10_countdown_run.jpg) | ![](picture/11_countdown_end.jpg) |

### 2.6 FLASHLIGHT — 手电筒

OLED 屏幕全白最大亮度，作为应急照明使用。

| 手电筒页面 | 演示效果 |
|:---:|:---:|
| ![](picture/12_flash_page.jpg) | ![](picture/13_flash_demo.jpg) |

### 2.7 SETTING — 系统设置

**设置主页面：**

![](picture/14_setting.jpg)

**亮度调节**（5 档 + 进度条动画）：

| 最低 | 最高 |
|:---:|:---:|
| ![](picture/15_brightness_low.jpg) | ![](picture/16_brightness_high.jpg) |

**蓝牙开关：**

| 关闭状态 | 开启状态 |
|:---:|:---:|
| ![](picture/17_bt_off.jpg) | ![](picture/18_bt_on.jpg) |

**关于页面**（多字体中英文显示）：

![](picture/21_about.jpg)

### 2.8 蓝牙无线传输

HC-05 以 1 Hz 频率持续发送文本格式的传感器数据包（加速度、温度、步数等），手机端使用任意蓝牙串口 APP 即可查看。支持通过手机发送时间同步帧校准手表时间。

![](picture/19_phone_data.jpg)

### 2.9 自动休眠与唤醒

主界面 5 秒无操作自动熄屏，编码器旋转或按键按下立即唤醒。

![](picture/20_sleep.jpg)

### 2.10 PC 上位机

基于 Python Flask 的 Web 端实时传感器数据可视化面板，浏览器即开即用，支持 `/stream on`、`/ping`、`/mpu` 等诊断命令。

![](picture/22_flask.png)

### 2.11 锂电池供电

TP4056 充电管理 + AMS1117 稳压 + 3.7V 锂电池，实现可充电独立供电，无需外接电源即可运行。

---

## 三、创新点分析

以下对照老师提供的 Phase 1–5 设计文档，列出本项目超出标准要求的创新功能。

### 3.1 秒表与倒计时

本组独立开发了完整的秒表（开始/计时/暂停/复位）和倒计时器（编码器设置时间/运行/结束），验证了 FreeRTOS 多任务架构下状态机与 OLED 刷新协作的可行性。

| Clock 子菜单 | 计时-未开始 | 计时-过程中 | 计时-暂停 |
|:---:|:---:|:---:|:---:|
| ![](picture/05_clock_menu.jpg) | ![](picture/06_timer_idle.jpg) | ![](picture/07_timer_run.jpg) | ![](picture/08_timer_pause.jpg) |

| 倒计时-设置中 | 倒计时-过程中 | 倒计时-结束 |
|:---:|:---:|:---:|
| ![](picture/09_countdown_set.jpg) | ![](picture/10_countdown_run.jpg) | ![](picture/11_countdown_end.jpg) |

### 3.2 手电筒模式

本组将 OLED 屏幕全白点亮（128×64 全白）作为应急照明，软硬件结合的创意应用，几乎零额外成本。

![](picture/13_flash_demo.jpg)

### 3.3 亮度调节 + 进度条动画

本组实现了亮度调节，菜单中显示可视化进度条（实心矩形逐档填充），提升了人机交互体验。

![](picture/16_brightness_high.jpg)

### 3.4 自动休眠 + 自动回主页

本组完整实现了低功耗策略：主界面 5 秒无操作自动熄屏，其他页面 10 秒无操作自动返回主界面，编码器旋转或按键按下立即唤醒/恢复。

![](picture/20_sleep.jpg)

### 3.5 蓝牙开关控制

本组在 SETTING 菜单内增加了蓝牙开关功能，关闭时停止数据发送，开启时恢复 1Hz 传输，兼顾省电与隐私。

![](picture/17_bt_off.jpg)

### 3.6 卡路里/距离估算

本组基于步数实时估算卡路里消耗（约 0.04 kcal/步）和运动距离（约 0.7 m/步），将原始传感器数据转化为用户可理解的健康指标。

![](picture/03_steps.jpg)

### 3.7 中英文混合显示

本组突破性地使用 Python PIL 从系统字体按需生成中文列优先格式字模，并在关于页面展示了中英文混合显示效果。

![](picture/21_about.jpg)

### 3.8 锂电池供电 + 充电

本组完成了 TP4056 充电管理 + AMS1117 稳压 + 3.7V 锂电池的完整供电链路，可脱离 USB 线独立运行演示。

### 3.9 Flask Web 上位机

本组开发了基于 Python Flask 的浏览器端实时传感器数据可视化面板，支持 `/stream on`、`/ping`、`/mpu` 等诊断命令，无需安装任何客户端软件。

![](picture/22_flask.png)
