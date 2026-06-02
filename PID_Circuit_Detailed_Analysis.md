# PID Circuit 详细分析笔记

> 基于已上传的 PID circuit schematic、PCB 丝印、README / testing procedure 的综合整理。  
> 这份笔记的目标不是重新设计 PCB，而是把这个老实验室 PID/NIM 模块拆成可理解、可焊接、可测试、可汇报的功能模块。

---

## 0. 一句话总览

这块板习惯上被叫做 **PID board**，但从 README 和 schematic 看，它实际上更接近：

```text
PI analog lock circuit + offset control + gain stages + monitor/output interfaces
```

也就是说，它主要包含：

- **P：比例控制路径**
- **I：积分控制路径**
- **Offset：锁点/偏置调节**
- **Monitor：中间信号监测**
- **Current / Grating lock 输出接口**
- **NIM 供电与电源稳压**

它没有真正独立的 D 微分路径。实验用途通常是把一个 spectroscopy / photodiode / error signal 转换成 correction voltage，再送到外部 actuator，例如 laser current driver、piezo driver、grating actuator 等。

---

## 1. 整体信号流

可以先按下面这条主线理解：

```text
External Error Signal
        ↓
GIN1：输入级 / 极性选择 / 初步信号处理
        ↓
PREIN：前级增益调节
        ↓
SUM：误差信号 + offset / extra inputs 求和
        ↓
分支：
    1. MONITOR：给示波器看的监测输出
    2. P path：比例路径
    3. INT：积分路径
        ↓
OPP / OPOUT：输出合成与输出驱动
        ↓
External Actuator
```

如果用于激光锁定，闭环可理解为：

```text
laser / cavity / spectroscopy signal
        ↓
photodiode / error signal
        ↓
PID board
        ↓
correction voltage
        ↓
laser current driver / piezo / grating
        ↓
laser frequency changes
        ↓
signal returns to lock point
```

---

## Schematic Reference

![PID schematic page 1](PID_GDrive/pid39sch01.png)

![PID schematic page 2](PID_GDrive/pid39sch02.png)

---

## 2. Power Supply / Regulators 电源模块

### 2.1 功能

电源模块的作用是把 NIM crate 或外部供电转换成稳定的 analog rails，给所有运放和 reference 使用。

典型结构是：

```text
NIM ±24 V 或外部 supply
        ↓
positive regulator / negative regulator
        ↓
+15 V / -15 V
        ↓
op-amp, reference, analog stages
```

schematic 中能看到类似：

- 正电压 regulator，例如 7815 类
- 负电压 regulator，例如 7915 类
- 供电保护二极管，例如 1N4001
- 大量 100 nF 去耦电容
- 电解电容或较大电容用于低频滤波

### 2.2 为什么必须有 regulators

运放需要稳定、低噪声、极性正确的供电。NIM 背板给的电源可能是较高电压，例如 ±24 V，不能直接给所有 analog IC 使用。因此板上先进行稳压。

### 2.3 去耦电容的作用

每个 IC 附近的 100 nF capacitor 不是信号电容，而是 **decoupling capacitor**。它连接在 supply rail 和 ground 之间，用来：

- 降低供电噪声
- 提供快速瞬态电流
- 防止 op-amp 高频振荡
- 让每一级运放看到稳定的局部电源

### 2.4 测试重点

在插 IC 前必须先测：

```text
+15 V rail 是否正确
-15 V rail 是否正确
GND 是否连续
regulator 是否发热异常
是否存在 + rail / - rail / GND 短路
```

**规则：先不插 IC，先测电源。**

---

## 3. 5V Reference / Offset Reference 模块

### 3.1 功能

schematic 中有 `REF02Z` 一类 reference chip，用来产生稳定的 +5 V reference。这个 +5 V 通常不是作为主电源，而是作为 **精密参考电压**。

后续通过 OP270 或其他 op-amp、电阻、电位器网络，可以生成：

- +5 V reference
- -5 V reference
- 可调 offset
- 给 summing amplifier 的 DC bias

### 3.2 为什么 PID 需要 offset

在理想控制里，error signal = 0 时系统在锁点。但真实实验中你可能希望：

- 锁在谱线峰的一侧
- 锁在某个 dispersive error signal 的零点
- 人为移动锁定点
- 补偿 detector / electronics 的 DC 偏置

所以 offset 模块的作用是：

```text
把 error signal 的“目标零点”移动到实验需要的位置
```

### 3.3 Offset 与 op-amp offset trim 的区别

要区分两个概念：

#### A. 实验 offset knob

这是用户故意加的 offset，用来选择锁点。

#### B. op-amp offset trim

这是运放自身 pin 1 / pin 8 的 offset-null 网络，用来消除运放内部失调误差。

两者不同：

```text
实验 offset：改变锁定目标
op-amp offset trim：校正电子学误差
```

---

## 4. Op-Amp Offset Trim：pin 1 / pin 8

### 4.1 它是什么

很多精密运放，例如 OP27，除了正常的：

```text
+ input
- input
output
positive supply
negative supply
```

之外，还有 `offset trim` 或 `offset null` 引脚，常见是 pin 1 和 pin 8。

真实运放内部晶体管不完全匹配，即使：

```text
V+ = V-
```

输出也可能不为 0。这叫 input offset voltage。

### 4.2 为什么 pin 1 / pin 8 外面有一圈电阻/电位器

这不是主信号 feedback，而是 **offset-null network**。它一般把一个 potentiometer 接到 pin 1 / pin 8，并从 supply rail 引入一个可调微小偏置。

作用：

```text
人为注入一个很小的反向修正量，
抵消运放内部 offset。
```

### 4.3 为什么 positive supply pin 7 会拉线到 potentiometer

对于某些 op-amp，datasheet 推荐 offset trim network 需要从 +Vs 或 -Vs 取一个参考。schematic 中看到 pin 7 positive supply 拉到一圈电阻/电位器，通常就是给 offset null 网络提供可调参考。

这条线不是 signal path，而是辅助校准路径。

### 4.4 在 PID 中的实际意义

如果不校正运放 offset，多级放大和积分会把很小的 DC error 放大，造成：

- output 不在 0 V
- integrator 自己慢慢 ramp
- actuator 被推向饱和
- 锁点漂移
- 难以判断问题来自实验信号还是电路本身

所以 offset trim 的作用是让每一级在零输入时尽量输出零。

---

## 5. GIN1：输入级

### 5.1 功能

`GIN1` 是外部信号进入 PID board 后的第一级处理。它通常负责：

- 接收外部 error signal
- 设置输入极性
- 初步放大或缓冲
- 把外部信号转换成内部控制电路可用的节点

它是整条反馈链的入口。

### 5.2 为什么 polarity 很重要

反馈系统必须是负反馈：

```text
系统偏高 → correction 让系统降低
系统偏低 → correction 让系统升高
```

如果输入或输出 polarity 错了，会变成正反馈：

```text
系统偏高 → correction 让系统更高
```

结果是：

- 输出冲到 rail
- lock 失效
- 可能出现振荡

### 5.3 测试方法

用 function generator 输入小幅三角波或正弦波：

```text
Input: 100 mVpp ~ 1 Vpp triangle wave
Check: GIN1 output 是否存在
Check: 极性是否符合 switch 状态
Check: 是否饱和到 ±15 V
```

---

## 6. PREIN：前级增益模块

### 6.1 功能

`PREIN` 是输入信号正式进入 summing stage 前的 gain stage。它决定 error signal 在后续 PI 控制前的幅度。

作用：

```text
error signal 太小 → 放大
error signal 太大 → 缩小
```

### 6.2 电路类型

从 schematic 结构看，PREIN 类似 op-amp gain stage，常见是反相放大或可调反相放大。

典型关系：

```text
V_PREIN = - G_PRE · V_IN
```

其中 `G_PRE` 由固定电阻和外接 potentiometer / trimmer 决定。

### 6.3 为什么需要外接 potentiometer

在不同实验中，error signal 斜率和幅度不同。PREIN gain 需要可调，才能让后续 SUM / INT / OPOUT 工作在线性范围内。

### 6.4 测试方法

```text
Input: triangle wave
Expected:
    输出仍为 triangle wave
    可能反相
    幅度随 GPRE pot 改变
Failure:
    无输出 → IC/供电/接线问题
    幅度固定不变 → pot wiring 问题
    波形削顶 → gain 太大或输入太大
```

---

## 7. SUM：求和放大模块

### 7.1 功能

`SUM` 是整个 PID 的核心节点之一。它把多个输入合成：

```text
processed error signal
+ offset
+ grating offset
+ extra external input
+ function generator / modulation input
```

最终得到一个内部控制信号。

### 7.2 为什么用 summing amplifier

实验控制中经常需要同时加入：

- error signal
- manual offset
- scan signal
- external modulation
- grating offset
- current offset

如果每个都单独接到 actuator 很混乱，所以在 SUM 处统一加权求和。

### 7.3 数学形式

典型反相求和器：

```text
V_SUM = -Rf · (V1/R1 + V2/R2 + V3/R3 + ...)
```

如果输入电阻不同，每个输入的权重也不同。

### 7.4 实验意义

SUM 决定了：

- 锁定点
- 是否叠加扫描信号
- 外部调制是否进入 loop
- 后续 P/I 路径看到的误差信号

### 7.5 测试方法

分别测试：

```text
只输入 error signal → 看 SUM 输出
只调 offset → 看 SUM 输出是否上下平移
同时输入 error + offset → 看是否线性叠加
```

---

## 8. MONITOR：监测输出模块

### 8.1 功能

`MONITOR` 通常是 buffer / voltage follower，用来把某个内部节点送到前面板 BNC，让示波器观察。

它不应该显著影响主电路。

### 8.2 为什么需要 buffer

如果直接把示波器或外部仪器接到内部高阻节点，可能改变电路行为。buffer 提供高输入阻抗和低输出阻抗，使监测不加载前级。

### 8.3 测试方法

```text
MONITOR output 应该跟被监测节点一致
如果 monitor 有波形但主输出没有，说明前级可能正常、后级有问题
如果 monitor 已经异常，问题在前级
```

---

## 9. INT：积分器模块

### 9.1 功能

`INT` 是 PI 控制中的 I 路径。它对误差信号进行时间积分：

```text
V_INT ∝ - ∫ V_error dt
```

### 9.2 为什么需要积分

比例控制 P 只根据当前误差给 correction。它响应快，但可能留下 steady-state error。积分 I 会持续累积长期偏差，直到误差被消除。

### 9.3 典型电路结构

op-amp integrator 通常是：

```text
input resistor → inverting input
feedback capacitor from output to inverting input
non-inverting input at ground/reference
```

传递函数：

```text
Vout(s) / Vin(s) = -1 / (R C s)
```

其中 RC 决定积分强度。

### 9.4 Switches / capacitors

schematic 中 INT 附近有多个 switch 和 capacitor，说明积分时间常数可以切换。不同电容/电阻组合对应不同响应速度。

- 大 C 或大 R：积分慢
- 小 C 或小 R：积分快

### 9.5 实验注意

积分太强会导致：

- overshoot
- oscillation
- lock hunting
- 输出撞 rail

积分太弱会导致：

- 长期漂移消不掉
- 锁点慢慢偏移

### 9.6 测试方法

对 integrator 测试不一定用 sine wave，推荐：

```text
输入一个小 DC step 或方波
观察 INT 输出是否 ramp
切换 S1/S3/S4，看 ramp slope 是否变化
```

---

## 10. P Path / OPP：比例路径或输出前级

### 10.1 功能

`OPP` 从命名和所在位置看，属于比例/输出前级相关模块。它可能负责：

- P 路径放大
- 极性调整
- 与 I 路径合成前的缓冲
- 输出前 conditioning

具体功能应结合 schematic 中 OPP 周围的输入、feedback、output net name 确认。

### 10.2 与 INT 的区别

```text
P path：对当前 error 立即响应
I path：对 error 的时间累积响应
```

实验调 lock 时通常先用 P 找到正确 polarity 和大致响应，再逐渐加入 I。

---

## 11. OPOUT：输出级

### 11.1 功能

`OPOUT` 是最终输出运放模块，用来把前面合成的控制信号输出到外部。

它后面可能接：

- front panel output BNC
- current lock output
- grating lock output
- actuator modulation input

### 11.2 输出串联电阻

schematic 中能看到输出附近有串联电阻，例如约几百欧量级。它通常用于：

- 保护 op-amp 输出
- 隔离长同轴电缆电容
- 降低振荡风险
- 限制短路电流

### 11.3 接外部 actuator

OPOUT 输出不是直接驱动大功率负载，而是给外部 driver 的 modulation input：

```text
PID output voltage
        ↓
laser current driver / piezo driver
        ↓
physical change
```

---

## 12. Current Lock Section

### 12.1 功能

Current Lock section 用来控制激光电流。PID 输出通过这一部分送到 laser current driver 的 modulation input。

### 12.2 C_ON/OFF

`C_ON/OFF` 是 current lock on/off 控制。

作用：

```text
ON  → current feedback enabled
OFF → current feedback disconnected / disabled
```

调试时通常先 OFF，确认信号和极性后再 ON。

### 12.3 C_POL

`C_POL` 是 current lock polarity switch。

作用是改变反馈方向。它非常重要，因为：

```text
正确 polarity → negative feedback → lock
错误 polarity → positive feedback → runaway / oscillation
```

### 12.4 C_PROP / C_POL_OP / C_FLOAT

这些名字看起来像电容/补偿相关模块，主要作用不是 DC gain，而是 loop dynamics：

- 限制高频响应
- 改善相位裕度
- 防止输出级振荡
- 调整 current lock 响应速度
- 滤除噪声

其中 `C_FLOAT` 可能表示一个 floating capacitor，即不一定一端接地，而是跨接在两个信号节点之间。

### 12.5 调试逻辑

```text
1. 先断开 current lock
2. 看 output polarity
3. 用示波器确认 correction voltage 方向
4. 再打开 C_ON/OFF
5. 如果系统发散，先换 C_POL 或降低 gain
```

---

## 13. Grating Lock / Grating Output

### 13.1 功能

Grating lock 通道通常用于控制 ECDL grating 或 piezo，从而调节激光频率。相对于 current lock：

- grating / piezo 响应较慢
- 调谐范围较大
- 更适合慢漂移补偿

### 13.2 与 current lock 的区别

```text
Current lock:
    fast, small range, affects laser current and frequency

Grating / piezo lock:
    slower, larger range, controls cavity/grating geometry
```

实际实验可能同时用：

- current channel 做快速 correction
- grating/piezo channel 做慢速 drift compensation

---

## 14. Switches：S1/S3/S4 等

### 14.1 PCB 上矩形 + 圆形标志

如果丝印上有矩形轮廓、里面一个圆形标志，而旁边真正有三个小 pad，通常矩形/圆形是机械或方向标识，三个小圆才是 switch 的焊盘。

### 14.2 三个 pad 的意义

很多开关是 SPDT：

```text
pin 1 --- option A
pin 2 --- common
pin 3 --- option B
```

不同位置决定 common 接到哪一边。

### 14.3 在 PID 中的作用

这些 switch 可能用于：

- enable / disable integrator
- bypass integrator
- select integration capacitor
- reverse polarity
- turn current lock on/off
- switch output routing

具体每个 S 的功能要看 schematic 上它连接到哪个 net。

---

## 15. Potentiometer / Trimmer Daughter Board

### 15.1 为什么有板外小板

README 提到 single-width NIM 空间很紧。很多 potentiometer 或 trimmer 可能不直接焊在主 PCB 上，而是：

```text
主 PCB header / holes
        ↓ flying wires
silver daughter board / front panel pots
        ↓
可调 gain / offset / time constant
```

### 15.2 100k trimmer 的作用

你看到的蓝色方块标注 `T18 100k` 很可能是 100 kΩ trimmer potentiometer，用来调：

- GPRE gain
- GSUM gain
- output gain
- offset
- integration rate
- compensation setting

### 15.3 如何确认每个 trimmer 接哪里

用万用表 continuity mode 建立 wiring map：

```text
Trimmer pin 1 → PCB which label?
Trimmer pin 2 / wiper → PCB which label?
Trimmer pin 3 → PCB which label?
```

并记录成表：

| Trimmer | Value | PCB label | Function | Notes |
|---|---:|---|---|---|
| T18 | 100k | unknown | likely gain/offset | continuity needed |

---

## 16. BNC / Front Panel I/O

### 16.1 BNC 的电气含义

BNC connector 有：

```text
center pin = signal
outer shell = shield / ground
```

schematic 中圆形接 GND 的符号通常是 BNC / coax connector 相关符号。

### 16.2 可能的前面板接口

这类 PID NIM module 通常有：

- error signal input
- monitor output
- current output
- grating output
- scan / extra input
- offset / gain knobs
- lock on/off switches
- polarity switches

### 16.3 外部连接原则

板子输出不能随便直接接负载。通常接到外部 driver 的 modulation input：

```text
PID voltage output → laser current driver modulation input
PID voltage output → piezo driver input
PID voltage output → grating control input
```

---

## 17. Jumpers / 3-pin Headers

### 17.1 常见 3-pin 结构

PCB 上的 1-2-3 三孔可能是：

- external potentiometer connector
- jumper selector
- switch connector
- routing selector

典型 potentiometer：

```text
pin 1 --- end A
pin 2 --- wiper
pin 3 --- end B
```

典型 jumper：

```text
short 1-2 → mode A
short 2-3 → mode B
```

### 17.2 如何判断

不要靠猜，按以下顺序：

```text
1. 在 schematic 找 label
2. 看 pin 1/2/3 分别连到哪里
3. 用万用表 continuity 验证 PCB 实物
4. 记录实际 wiring
```

---

## 18. 独立电容与特殊电容标记

### 18.1 普通去耦电容

通常接在 supply 和 ground 之间：

```text
+15V → C → GND
-15V → C → GND
```

### 18.2 feedback / compensation capacitor

如果电容跨接在 op-amp output 和 input 之间，或跨接两个 signal nodes，它属于反馈/补偿网络。

作用可能是：

- 积分
- 低通滤波
- 限制带宽
- 改善稳定性

### 18.3 白方形 / 空心方形 footprint

PCB 上不同外形可能表示不同封装或 optional part：

- box film capacitor
- ceramic capacitor
- optional capacitor position
- unpopulated footprint
- alternate package footprint

是否安装要看 BOM、schematic note 或已有版本照片。

---

## 19. 二极管

### 19.1 常见作用

PID 板上的二极管可能用于：

- 电源反接保护
- supply rail discharge protection
- output clamp
- integrator reset / discharge
- relay/switch transient protection

### 19.2 如何判断

看二极管连接位置：

| 连接位置 | 可能功能 |
|---|---|
| supply rail 附近 | 电源保护 |
| output 附近 | 输出限幅 / 保护 |
| integrator capacitor 附近 | 快速放电 / reset |
| switch 附近 | transient protection |

---

## 20. 推荐的学习与分析顺序

不要按 PCB 空间位置学，而是按功能流学：

```text
1. Power supply / regulators
2. 5V reference and offset
3. Op-amp offset trim
4. GIN1 input stage
5. PREIN gain stage
6. SUM summing amplifier
7. MONITOR buffer
8. INT integrator
9. OPP / P path
10. OPOUT output stage
11. Current lock and grating lock outputs
12. Switches, jumpers, pots, BNC wiring
```

每个模块填下面的表：

| Item | Notes |
|---|---|
| Block name |  |
| Input node |  |
| Output node |  |
| Main IC |  |
| Key R/C components |  |
| Adjustable component |  |
| Function |  |
| Expected test result |  |
| Failure symptoms |  |

---

## 21. Assembly Plan：组装步骤

### 21.1 准备阶段

```text
1. 找到 schematic
2. 找到 BOM / order list
3. 找到 PCB silkscreen / board file
4. 确认 off-board components：pots, switches, BNC, daughter board
5. 建立 wiring map
```

### 21.2 焊接顺序

推荐：

```text
1. 电阻
2. 小电容
3. 二极管，注意方向
4. IC sockets
5. regulator
6. 大电容
7. headers / connectors
8. switches / BNC / off-board wires
9. 最后插 IC
```

### 21.3 关键规则

```text
不要一开始插 IC。
先测电源，再插 IC。
```

---

## 22. Testing Procedure：测试流程

### 22.1 Power test

不插 IC：

```text
Measure +15V
Measure -15V
Measure +5V reference if possible
Check GND continuity
Check no rail short
```

### 22.2 Reference / offset test

```text
Measure REF02 +5V
Measure generated -5V if present
Turn offset pot
Check offset range and smoothness
```

### 22.3 Stage-by-stage signal test

用 function generator：

```text
Input: triangle wave, small amplitude
Check GIN1
Check PREIN
Check SUM
Check MONITOR
Check INT
Check OPOUT
```

### 22.4 Integrator test

```text
Input: small DC step or square wave
Expected: output ramps
Change integration switch/capacitor
Expected: ramp slope changes
```

### 22.5 Lock output test

先不接真实 laser：

```text
Check current output polarity
Check grating output polarity
Toggle C_ON/OFF
Toggle C_POL
Confirm output behavior
```

---

## 23. 测试记录模板

```text
Date:
Board version:
Power supply:
ICs installed? yes/no

Stage:
Input signal:
Expected output:
Measured output:
Gain:
Offset:
Switch positions:
Result: PASS / FAIL
Notes:
```

示例：

```text
Stage: PREIN
Input: 500 mVpp triangle, 1 kHz
Expected: inverted triangle, adjustable amplitude
Measured:
Switch / pot:
Conclusion:
```

---

## 24. 汇报展示结构

推荐 presentation 结构：

```text
1. Motivation: why need an analog PID lock circuit
2. System overview: experiment → error signal → PID → actuator
3. Board architecture: power, reference, input, sum, P/I, output
4. Detailed stage analysis
5. Off-board wiring and NIM integration
6. Assembly plan
7. Testing procedure
8. Troubleshooting and expected issues
9. Next steps
```

最重要的一页应该是系统框图：

```text
Error Signal → GIN1 → PREIN → SUM → P/I → OPOUT → Actuator
```

---

## 25. 常见故障与排查

| 现象 | 可能原因 |
|---|---|
| 没有输出 | IC 未供电、IC 方向错、输出没接、stage 前级无信号 |
| 输出饱和到 +15/-15 V | gain 太大、offset 太大、反馈断路、polarity 错 |
| offset knob 没效果 | pot wiring 错、reference 没有、SUM 输入没接 |
| monitor 正常但 output 异常 | 后级 OPP/OPOUT 问题 |
| integrator 自己 ramp | input offset、op-amp offset、漏电、switch 状态错 |
| 一接 actuator 就发散 | polarity 错、gain 太大、actuator bandwidth 不匹配 |
| 高频振荡 | compensation cap 缺失、长 BNC 电缆、电源去耦差 |
| 噪声大 | ground/shield 问题、供电噪声、输入线太长 |

---

## 26. 当前还需要确认的信息

为了把这份 note 变成完全可焊接版本，还需要补齐：

```text
1. BOM / parts order list
2. 每个 potentiometer 的阻值和连接 pinout
3. 每个 switch 的实际 ON/OFF 方向
4. C_PROP / C_POL_OP / C_FLOAT 是否装、装多大
5. front panel BNC 对应 PCB 哪个 net
6. daughter board 上每个 trimmer 对应哪个 PCB label
7. NIM connector 的 supply pinout
8. 已有成品板照片，用来对照实际 wiring
```

---

## 27. 最核心理解

这块板不是一个“单纯焊元件就完事”的 PCB，而是一个完整的实验控制模块：

```text
主 PCB：
    固定 analog PI control topology

off-board pots / switches：
    提供可调 gain, offset, polarity, integration time

front panel BNC：
    提供实验输入输出接口

NIM box：
    提供机械结构、屏蔽和标准供电

external actuator：
    真正把 PID correction voltage 变成物理变化
```

最终你需要掌握的不只是“每个电阻多大”，而是：

```text
每一级在信号链中的功能，
每个可调元件改变什么，
每个测试点应该看到什么，
系统失效时该从哪里开始查。
```
