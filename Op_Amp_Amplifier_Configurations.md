# Classic Op-Amp Amplifier Configurations

本文整理经典由 operational amplifier 构成的 amplifier / signal-conditioning configurations。内容包括电路示意图、功能、理想数学关系、简要分析、应用与实际设计注意事项。

> 约定：除非特别说明，下面公式均基于理想 op amp 假设：输入电流近似为 0；在负反馈线性工作区内，`V+ ≈ V-`；输出没有饱和；op amp 带宽、slew rate、输入/输出摆幅满足要求。

---

## 0. Sources and Reliability Notes

本整理主要参考以下一手或权威资料：

- Texas Instruments, **AN-20: An Applications Guide for Op Amps**: <https://www.ti.com/lit/an/snoa621c/snoa621c.pdf>
- Texas Instruments, **Analog Engineer's Circuit Cookbook: Op Amps / Amplifiers**: <https://e2e.ti.com/cfs-file/__key/communityserver-discussions-components-files/14/CircuitCookbook_2D00_OpAmps.pdf>
- Texas Instruments, **TI Precision Labs - Op Amps**: <https://www.ti.com/video/series/precision-labs/ti-precision-labs-op-amps.html>
- Analog Devices, **Op Amp Applications Handbook**, edited by Walt Jung: <https://www.analog.com/op_amp_handbook>
- Analog Devices, **A Designer's Guide to Instrumentation Amplifiers**: <https://www.analog.com/en/resources/technical-books/dh-designers-guide-to-instrumentation-amps.html>
- Analog Devices, **AN-649: Active Filter Design Tool / active filter topologies**: <https://www.analog.com/en/resources/app-notes/an-649.html>

这些资料在具体电路命名、实际设计限制、instrumentation amplifier、active filter 等方面互相印证。本文中的公式是经典理想模型；实际电路还必须检查 datasheet 中的 common-mode range、output swing、gain-bandwidth product、phase margin、input bias current、input offset voltage、noise、load capacitance 等。

---

## 1. Common Ideal Op-Amp Rules

在负反馈线性区内：

```text
I+ ≈ 0
I- ≈ 0
V+ ≈ V-
```

因此分析大多数经典 op-amp 电路时，核心步骤是：

```text
1. 找出 V+ 的电压。
2. 令 V- ≈ V+。
3. 在反相输入节点写 KCL。
4. 解出 Vout 与输入之间的关系。
```

实际中，理想规则失效的常见原因包括：

- 输出接近电源轨导致 saturation。
- 输入 common-mode voltage 超出允许范围。
- op amp 的 gain-bandwidth 不够。
- 输入 offset voltage 被高增益或积分器放大。
- input bias current 在大电阻上产生额外误差。
- 负载电容造成相位裕度不足。

---

## 2. Voltage Follower / Unity-Gain Buffer

### Configuration

```text
                 ┌─────────┐
Vin ────────────▶│ +       │
                 │   OpAmp │────── Vout
          ┌─────▶│ -       │
          │      └─────────┘
          │
          └────────────────┘
```

### Function

电压跟随器不提供电压增益，但提供阻抗变换：

- 输入阻抗非常高。
- 输出阻抗较低。
- 能隔离前级高阻信号源和后级负载。

### Equation

```text
Vout = Vin
Av = 1
```

### Analysis

由于输出直接反馈到反相输入端：

```text
V- = Vout
V+ = Vin
V- ≈ V+
```

所以：

```text
Vout ≈ Vin
```

### Applications

- Sensor buffer
- Reference voltage buffer
- ADC input driver, when bandwidth/load are appropriate
- Monitor output buffer
- Isolating a potentiometer or high-impedance node

### Practical Notes

不是所有 op amp 都 unity-gain stable。使用 follower 前必须确认 datasheet 中写明 stable at gain of 1 或 unity-gain stable。

---

## 3. Non-Inverting Amplifier

### Configuration

```text
                 ┌─────────┐
Vin ────────────▶│ +       │
                 │   OpAmp │────── Vout
          ┌─────▶│ -       │
          │      └─────────┘
          │
          ├── Rf ──────────┐
          │                │
          └── Rg ── GND    │
                           │
                         Vout
```

更直观地说，反相端接收输出分压：

```text
Vout ── Rf ──●── Rg ── GND
             │
             └── V-
```

### Function

非反相放大器提供正增益，不反相。输入阻抗很高，因此适合放大高阻信号源。

### Equation

```text
Vout = (1 + Rf/Rg) Vin
Av = 1 + Rf/Rg
```

### Analysis

负反馈使：

```text
V- ≈ V+ = Vin
```

而 `V-` 是输出分压：

```text
V- = Vout · Rg/(Rf + Rg)
```

所以：

```text
Vin = Vout · Rg/(Rf + Rg)
Vout = Vin · (Rf + Rg)/Rg
Vout = (1 + Rf/Rg) Vin
```

### Applications

- 高阻传感器电压放大
- 参考电压放大
- ADC 前级
- 不希望反相的模拟信号链

### Practical Notes

- 最小闭环增益为 1。
- 噪声增益等于信号增益。
- 输入 common-mode range 必须覆盖 `Vin`。
- 单电源系统中常需要把 `Rg` 接到中点参考 `Vref`，而不是 ground。

---

## 4. Inverting Amplifier

### Configuration

```text
Vin ── Rin ──●─────────────┐
             │             │
             │      ┌─────────┐
             └─────▶│ -       │
                    │   OpAmp │────── Vout
GND ───────────────▶│ +       │
                    └─────────┘
                         ▲
                         │
                       Rf│
                         │
                         └──────── Vout
```

### Function

反相放大器提供负增益，输出相位相对输入反转 180 degrees。输入阻抗主要由 `Rin` 决定。

### Equation

```text
Vout = -(Rf/Rin) Vin
Av = -Rf/Rin
```

### Analysis

`V+` 接地，因此：

```text
V- ≈ 0
```

这个节点称为 virtual ground。因为 op amp 输入电流近似为 0，流过 `Rin` 的电流必须流过 `Rf`：

```text
Iin = Vin/Rin
If = (0 - Vout)/Rf
Iin = If
```

所以：

```text
Vin/Rin = -Vout/Rf
Vout = -(Rf/Rin) Vin
```

### Applications

- 精确反相 gain stage
- Audio / signal conditioning
- Weighted summing amplifier 的基础
- Active filter 的基础
- 虚地点电流输入电路

### Practical Notes

- 输入阻抗约为 `Rin`，不像非反相放大器那样很高。
- 反相节点是 virtual ground，不是真正 ground，不能当电源地使用。
- 可实现小于 1 的电压增益，例如 `Rf < Rin`。

---

## 5. Inverting Summing Amplifier

### Configuration

```text
V1 ── R1 ──┐
V2 ── R2 ──┤
V3 ── R3 ──┼──●───────────┐
...        │  │           │
Vn ── Rn ──┘  │    ┌─────────┐
              └───▶│ -       │
                   │   OpAmp │────── Vout
GND ──────────────▶│ +       │
                   └─────────┘
                        ▲
                        │
                      Rf│
                        │
                        └──────── Vout
```

### Function

把多个输入电压按权重相加，并反相输出。它是 analog mixer、offset injection、PID summing node 中非常常见的结构。

### Equation

```text
Vout = -Rf (V1/R1 + V2/R2 + ... + Vn/Rn)
```

若所有输入电阻相同 `R1 = R2 = ... = R`：

```text
Vout = -(Rf/R)(V1 + V2 + ... + Vn)
```

### Analysis

反相节点近似 virtual ground。每一路输入电流为：

```text
Ii = Vi/Ri
```

所有输入电流加到反馈电阻：

```text
If = Σ Ii
```

因此输出是加权和的负值。

### Applications

- Analog signal mixer
- DAC weighted summing
- PID/error signal summing
- Offset + signal addition
- Audio mixing
- Function generator / scan / modulation injection

### Practical Notes

- 每一路输入阻抗约为对应的 `Ri`。
- 适合多个信号源相加，因为每个源通过电阻隔离。
- 若要高精度加权，电阻匹配和温漂很重要。

---

## 6. Non-Inverting Summing / Averaging Amplifier

### Configuration

一种常见结构是先用电阻网络产生加权平均，再用非反相放大器放大：

```text
V1 ── R1 ──┐
V2 ── R2 ──┤
V3 ── R3 ──┼──●──▶ op amp +
...        │  │
Vn ── Rn ──┘  │
              Rb
              │
             GND

                 ┌─────────┐
weighted node ──▶│ +       │
                 │   OpAmp │────── Vout
          ┌─────▶│ -       │
          │      └─────────┘
          ├── Rf ───── Vout
          └── Rg ───── GND
```

### Function

输出与输入加权平均同相。相比 inverting summing amplifier，它没有 virtual-ground summing node，因此输入之间可能互相影响。

### Equation

加权节点：

```text
Vx = (V1/R1 + V2/R2 + ... + Vn/Rn) / (1/R1 + 1/R2 + ... + 1/Rn + 1/Rb)
```

输出：

```text
Vout = (1 + Rf/Rg) Vx
```

若所有 `Ri` 相等且无 `Rb`，理想高输入阻抗下：

```text
Vx ≈ (V1 + V2 + ... + Vn)/n
```

### Applications

- Averaging multiple DC signals
- Combining same-phase control voltages
- Sensor averaging when source impedances are controlled

### Practical Notes

TI 的单电源 op amp 资料中也提醒，non-inverting summing circuits 可以做，但通常不如 inverting summer 简洁、隔离好。多个输入源之间会通过电阻网络互相加载。

---

## 7. Difference Amplifier / Subtractor

### Configuration

```text
V1 ── R1 ──●─────────────┐
           │             │
           │      ┌─────────┐
           └─────▶│ -       │
                  │   OpAmp │────── Vout
V2 ── R3 ──●─────▶│ +       │
           │      └─────────┘
           R4          ▲
           │           │
          GND          R2
                       │
                       └──────── Vout
```

### Function

输出两个输入的差值，并抑制共同存在于两个输入上的 common-mode voltage。

### Equation

若电阻比匹配：

```text
R2/R1 = R4/R3
```

则：

```text
Vout = (R2/R1)(V2 - V1)
```

若四个电阻满足：

```text
R1 = R3
R2 = R4
```

则：

```text
Vout = (R2/R1)(V2 - V1)
```

### Analysis

反相端看到来自 `V1` 和 `Vout` 的加权关系；非反相端看到 `V2` 的分压。只有在电阻比精确匹配时，common-mode rejection 才好。

### Applications

- Measuring voltage difference between two nodes
- Ground-shifted sensor signal extraction
- Current-sense resistor voltage measurement
- Bridge sensor readout, if source impedance is acceptable

### Practical Notes

- CMRR 强烈依赖电阻匹配。
- 输入阻抗不是很高，可能加载信号源。
- 高精度 differential measurement 通常用 instrumentation amplifier 或 integrated difference amplifier。

---

## 8. Instrumentation Amplifier: Three-Op-Amp INA

### Configuration

经典 three-op-amp instrumentation amplifier：

```text
              R        R
V1 ──▶ +A1  ─/\/\──●──/\/\─┐
        │          │       │
        │         Rg       │
        │          │       │
V2 ──▶ +A2  ─/\/\──●──/\/\─┘
              R        R

A1 and A2 are non-inverting input buffers/gain stages.
Their outputs feed a difference amplifier:

A1out ── R1 ──●────▶ -A3
              │
             R2 feedback from Vout

A2out ── R3 ──●────▶ +A3
              │
             R4 to reference/ground

                      A3 ─── Vout
```

### Function

Instrumentation amplifier 用来高精度放大两个输入之间的差值，同时提供：

- very high input impedance,
- adjustable differential gain,
- high common-mode rejection,
- often a reference pin to shift output baseline.

### Equation

对于经典对称 three-op-amp INA：

```text
Vout = G (V2 - V1) + Vref
```

第一阶段增益常见形式：

```text
G1 = 1 + 2R/Rg
```

若第二阶段 difference amplifier 为 unity gain：

```text
G = 1 + 2R/Rg
```

若第二阶段还有差分增益 `G2`：

```text
G = G1 · G2
```

### Analysis

前两个 op amp 以非反相形式接入两个输入，因此输入阻抗高。`Rg` 调节两个前级之间的差分增益。最后一级 difference amplifier 去掉 common-mode component。

### Applications

- Strain gauge / Wheatstone bridge
- Thermocouple / RTD front-end
- Biomedical electrode signals
- Low-level sensor measurements
- Current-sense with high common-mode voltage, if ratings allow

### Practical Notes

- 分立 op amp + discrete resistors 可以搭建 INA，但 CMRR 很难做到很高。
- 工业和高精度测量通常选 integrated instrumentation amplifier，因为内部电阻匹配更好。
- 注意输入 common-mode range 会随 gain 和 supply voltage 变化。

---

## 9. Transimpedance Amplifier, TIA / Current-to-Voltage Amplifier

### Configuration

```text
              photodiode / current source
                      │
                      ▼ Iin
                    ●─────────────┐
                    │             │
                    │      ┌─────────┐
                    └─────▶│ -       │
                           │   OpAmp │────── Vout
Vref / GND ───────────────▶│ +       │
                           └─────────┘
                                ▲
                                │
                              Rf│
                                │
                                └──────── Vout

Often Cf is placed in parallel with Rf for stability:

              Rf
Vout ───────/\/\/\──────●
              │         │
              └── Cf ───┘
```

### Function

把输入电流转换为输出电压。最经典应用是 photodiode amplifier。

### Equation

若非反相端接地：

```text
Vout = -Iin Rf
```

若非反相端接参考电压 `Vref`：

```text
Vout = Vref - Iin Rf
```

含反馈电容时，反馈阻抗为：

```text
Zf = Rf || 1/(s Cf)
```

因此：

```text
Vout(s) - Vref = -Iin(s) Zf
```

### Analysis

反相端被保持在 virtual ground 或 `Vref`，因此 photodiode 端电压稳定，有利于线性和速度。输入电流几乎全部流过反馈阻抗。

### Applications

- Photodiode readout
- PMT / current-output sensor readout
- Ionization chamber current measurement
- Electrochemical sensor front-end

### Practical Notes

- `Rf` 越大，电流-电压增益越高，但带宽和稳定性越难。
- Photodiode capacitance、op amp input capacitance、`Rf` 和 op amp GBW 决定 stability。
- `Cf` 常用于补偿和限制带宽。
- 低噪声 TIA 设计要认真计算 Johnson noise、current noise、voltage noise。

---

## 10. Transconductance Amplifier / Voltage-to-Current Converter

### Basic Inverting Current Output Concept

```text
Vin ── Rin ──●─────────────┐
             │             │
             │      ┌─────────┐
             └─────▶│ -       │
                    │   OpAmp │────── drives load/current path
Vref/GND ──────────▶│ +       │
                    └─────────┘
```

更常见的实用形式是让 op amp 驱动一个晶体管或 MOSFET，使采样电阻上的电压等于输入控制电压：

```text
                 ┌─────────┐
Vin ────────────▶│ +       │
                 │   OpAmp │──── gate/base ── transistor ── load
          ┌─────▶│ -       │
          │      └─────────┘
          │
          ●── Rsense ── GND
          │
      feedback voltage = Iout Rsense
```

### Function

把输入电压转换为受控输出电流。

### Equation

对于 sense resistor feedback 结构：

```text
V- ≈ V+ = Vin
```

而：

```text
V- = Iout Rsense
```

因此：

```text
Iout = Vin / Rsense
```

### Applications

- LED current driver
- Laser diode current modulation, with appropriate safety design
- Coil / actuator current driver
- Sensor excitation current source

### Practical Notes

- op amp 输出可能需要驱动晶体管，因为普通 op amp 不能提供大电流。
- 必须检查 compliance voltage：输出电流需要足够电压余量跨过 load、transistor 和 sense resistor。
- 对激光二极管等敏感负载必须加限流、软启动、保护和故障模式分析。

---

## 11. Inverting Integrator

### Configuration

```text
Vin ── R ──●──────────────┐
           │              │
           │       ┌─────────┐
           └──────▶│ -       │
                   │   OpAmp │────── Vout
GND / Vref ───────▶│ +       │
                   └─────────┘
                        ▲
                        │
                        C
                        │
                        └──────── Vout
```

Practical version often adds a large resistor `Rf` in parallel with `C`:

```text
Vout ── C ──●
       │    │
       Rf   │
       │    │
       └────┘
```

### Function

输出输入信号的积分，并反相。

### Equation

理想 integrator：

```text
Vout(s) / Vin(s) = -1/(R C s)
```

时间域：

```text
Vout(t) = Vout(0) - (1/RC) ∫ Vin(t) dt
```

对于常数输入：

```text
dVout/dt = -Vin/(RC)
```

### Analysis

输入电阻把输入电压转换为电流：

```text
I = Vin/R
```

该电流流入反馈电容：

```text
I = C d(0 - Vout)/dt
```

所以输出斜率与输入电压成正比。

### Applications

- PI/PID controller I term
- Ramp generator
- Analog computation
- Charge integration
- Active filters

### Practical Notes

- 纯积分器会把 input offset 和 bias current 积分到饱和。
- 常加并联 `Rf` 给 DC feedback path，限制低频增益。
- 电容漏电和介质吸收会影响精度。

---

## 12. Inverting Differentiator

### Configuration

```text
Vin ── C ──●──────────────┐
           │              │
           │       ┌─────────┐
           └──────▶│ -       │
                   │   OpAmp │────── Vout
GND ──────────────▶│ +       │
                   └─────────┘
                        ▲
                        │
                        R
                        │
                        └──────── Vout
```

Practical differentiator usually adds limiting components:

```text
input:  Vin ── Rin ── C ──●
feedback: Vout ── Rf || Cf ──●
```

### Function

输出输入信号的微分，并反相。

### Equation

理想 differentiator：

```text
Vout(s) / Vin(s) = -R C s
```

时间域：

```text
Vout(t) = -R C · dVin/dt
```

### Analysis

输入电容电流：

```text
I = C dVin/dt
```

该电流流过反馈电阻：

```text
Vout = -I R
```

所以：

```text
Vout = -RC dVin/dt
```

### Applications

- Edge detection
- Analog derivative term in control systems
- Wave-shaping
- High-pass active filtering

### Practical Notes

- 理想 differentiator 高频增益无限增长，容易放大噪声并不稳定。
- 实际电路必须限制高频增益。
- 在 PID 中 D term 常被省略或强烈滤波，因为实验噪声会被 derivative 放大。

---

## 13. First-Order Active Low-Pass Amplifier

### Configuration: Inverting Low-Pass

```text
Vin ── Rin ──●─────────────┐
             │             │
             │      ┌─────────┐
             └─────▶│ -       │
                    │   OpAmp │────── Vout
GND ───────────────▶│ +       │
                    └─────────┘
                         ▲
                         │
                  Rf ────┤
                         │
                  Cf ────┤   Rf || Cf feedback
                         │
                         └──────── Vout
```

### Function

低频按反相增益放大，高频被反馈电容衰减。

### Equation

反馈阻抗：

```text
Zf = Rf || 1/(s Cf) = Rf / (1 + s Rf Cf)
```

传递函数：

```text
Vout/Vin = -Zf/Rin
          = -(Rf/Rin) · 1/(1 + s Rf Cf)
```

截止频率：

```text
fc = 1/(2π Rf Cf)
```

### Applications

- Noise limiting
- Anti-alias prefilter, if order is sufficient
- Control-loop bandwidth limiting
- Audio/sensor signal smoothing

### Practical Notes

这是一个一阶低通。若需要更陡的 roll-off，通常级联多个一阶级，或使用 Sallen-Key / multiple-feedback 二阶滤波器。

---

## 14. First-Order Active High-Pass Amplifier

### Configuration: Inverting High-Pass

```text
Vin ── C ── Rin ──●─────────┐
                  │         │
                  │  ┌─────────┐
                  └─▶│ -       │
                     │   OpAmp │────── Vout
GND ────────────────▶│ +       │
                     └─────────┘
                          ▲
                          │
                         Rf
                          │
                          └────── Vout
```

### Function

阻断 DC，放大高频/交流成分。

### Equation

输入阻抗：

```text
Zin = Rin + 1/(sC)
```

传递函数：

```text
Vout/Vin = -Rf/Zin
          = -(Rf/Rin) · (s Rin C)/(1 + s Rin C)
```

截止频率：

```text
fc = 1/(2π Rin C)
```

### Applications

- AC coupling
- Removing DC offset
- Edge/waveform emphasis
- Audio high-pass stages

### Practical Notes

如果后级还需要确定 DC operating point，必须提供偏置路径。单电源电路中常把 op amp 非反相端接到 `Vref`。

---

## 15. Sallen-Key Active Filter

### Configuration: Unity-Gain Low-Pass Example

```text
Vin ── R1 ──●── R2 ──●────▶ +OpAmp
            │        │       │
            C1       C2      │
            │        │       │
           GND      GND      │
                             │
                 ┌─────────┐ │
                 │   OpAmp │─┴── Vout
          ┌─────▶│ -       │
          │      └─────────┘
          └──────── Vout
```

### Function

Sallen-Key 是经典二阶 active filter topology，常用作 low-pass 或 high-pass。op amp 通常作为 non-inverting buffer 或 non-inverting gain stage。

### Generic Second-Order Low-Pass Form

```text
H(s) = H0 · ω0^2 / (s^2 + (ω0/Q)s + ω0^2)
```

其中：

- `ω0` 是 natural frequency。
- `Q` 决定峰化和 damping。
- `H0` 是 passband gain。

### Applications

- 二阶 low-pass / high-pass filters
- Anti-alias filters
- Audio tone shaping
- Sensor bandwidth limiting

### Practical Notes

Analog Devices 的 active filter 文档指出，Sallen-Key 与 multiple-feedback topology 的 op amp 参数敏感度不同。Sallen-Key 中 op amp 通常作为 amplifier/buffer，而 multiple-feedback 结构中 op amp 更像 integrator，因此对 open-loop gain 和带宽更敏感。

---

## 16. Multiple-Feedback, MFB Filter Amplifier

### Configuration: Conceptual Inverting Low-Pass / Band-Pass Family

```text
                    C2
             ┌─────||─────┐
             │            │
Vin ── R1 ──●─────────────●────▶ -OpAmp
            │             │
            C1            R2
            │             │
           GND          Vout

                 ┌─────────┐
GND ────────────▶│ +       │
                 │   OpAmp │────── Vout
          node ─▶│ -       │
                 └─────────┘
```

Exact component placement depends on low-pass, high-pass, or band-pass design.

### Function

MFB filters use an inverting op-amp topology with multiple feedback paths to realize second-order filter functions.

### Generic Form

For a second-order filter:

```text
H(s) = numerator(s) / (s^2 + (ω0/Q)s + ω0^2)
```

The numerator depends on whether it is low-pass, high-pass, or band-pass.

### Applications

- Precision low-pass / band-pass filters
- Higher-Q filters
- Cascaded active filter sections

### Practical Notes

ADI notes that MFB uses the op amp like an integrator and is more dependent on op amp parameters than Sallen-Key. The op amp open-loop gain should remain comfortably above the required closed-loop response near the cutoff/resonant frequency.

---

## 17. Charge Amplifier

### Configuration

```text
charge source / piezo sensor
          │
          ▼ Qin
        ●──────────────┐
        │              │
        │       ┌─────────┐
        └──────▶│ -       │
                │   OpAmp │────── Vout
Vref/GND ──────▶│ +       │
                └─────────┘
                     ▲
                     │
                     Cf
                     │
                     └──────── Vout

Often Rf is placed in parallel with Cf to provide DC leakage path.
```

### Function

把输入电荷转换为电压。常用于 piezoelectric sensors。

### Equation

理想情况下：

```text
Vout - Vref = -Qin/Cf
```

若以电流形式输入：

```text
Iin = dQin/dt
Vout(s) - Vref = -Iin(s)/(s Cf)
```

### Applications

- Piezoelectric accelerometers
- Charge-output sensors
- Radiation or ionization detectors

### Practical Notes

- 必须提供 DC path，例如并联很大的 `Rf`，否则输入偏置电流和漏电会使输出饱和。
- 低漏电电容、PCB 清洁、guarding 对精密 charge amplifier 很重要。

---

## 18. Logarithmic Amplifier

### Configuration

```text
Vin ── R ──●──────────────┐
           │              │
           │       ┌─────────┐
           └──────▶│ -       │
                   │   OpAmp │────── Vout
GND/Vref ─────────▶│ +       │
                   └─────────┘
                        ▲
                        │
                     diode/BJT
                        │
                        └──────── Vout
```

### Function

利用 diode/BJT 的指数 I-V 关系，把输入电流或电压转换为对数输出。

### Idealized Equation

对于 BJT feedback 的简化模型：

```text
Ic = Is exp(Vbe/VT)
```

反相节点电流约为：

```text
Iin = Vin/R
```

输出大致为：

```text
Vout ≈ -VT ln(Iin/Is)
```

其中：

```text
VT = kT/q ≈ 25.85 mV at room temperature
```

### Applications

- Wide dynamic range measurement
- Analog compression
- Optical intensity measurement over many decades

### Practical Notes

- 强烈依赖温度。
- 需要温度补偿和匹配器件。
- 小信号和大信号端都受 offset、leakage、saturation 限制。

---

## 19. Antilog / Exponential Amplifier

### Configuration

```text
Vin ── diode/BJT ──●────────┐
                   │        │
                   │ ┌─────────┐
                   └▶│ -       │
                     │   OpAmp │────── Vout
GND/Vref ───────────▶│ +       │
                     └─────────┘
                          ▲
                          │
                          R
                          │
                          └────── Vout
```

### Function

产生与输入电压指数相关的输出。

### Idealized Equation

若输入控制 BJT/diode 电流：

```text
I ≈ Is exp(Vin/VT)
```

反馈电阻把该电流变成电压：

```text
Vout ≈ -R Is exp(Vin/VT)
```

### Applications

- Analog multiplication/division with log/antilog pairs
- Exponential control in audio synthesizers
- Dynamic-range expansion

### Practical Notes

和 log amplifier 一样，温度补偿非常关键。

---

## 20. Precision Rectifier / Absolute-Value Amplifier

### Configuration: Conceptual Half-Wave Precision Rectifier

```text
                  diode
             ┌────|>|────┐
             │           │
Vin ── R ──●─┘    ┌─────────┐
           │      │ -       │
           └─────▶│   OpAmp │──── internal drive
GND ─────────────▶│ +       │
                  └─────────┘

output is taken through diode/feedback network depending on polarity.
```

### Function

用 op amp 补偿 diode forward drop，实现小信号整流。普通二极管整流在几十毫伏信号下误差很大；precision rectifier 可处理远小于 diode drop 的信号。

### Idealized Relationship

半波整流：

```text
Vout = max(0, Vin)
```

或反相版本：

```text
Vout = max(0, -Vin)
```

全波 absolute-value amplifier：

```text
Vout = |Vin|
```

### Applications

- AC signal measurement
- Envelope detection
- Absolute-value analog computation
- RMS-related front-end circuits

### Practical Notes

- 高频性能受 op amp slew rate 和 diode switching 限制。
- 需要注意 op amp 在 diode 截止时是否进入饱和，饱和恢复会变慢。

---

## 21. Comparator-Like Op-Amp Use

### Configuration

```text
Vin ─────────────▶ +OpAmp
Vref ────────────▶ -OpAmp

                 ┌─────────┐
Vin ────────────▶│ +       │
                 │   OpAmp │────── Vout high/low
Vref ───────────▶│ -       │
                 └─────────┘
```

### Function

比较两个电压，输出接近高/低电源轨：

```text
Vout high if Vin > Vref
Vout low  if Vin < Vref
```

### Important Note

这不是线性 amplifier configuration，而是开环或正反馈开关应用。很多 op amp 可以临时当 comparator 用，但这通常不是最佳实践。

### Applications

- Threshold detection
- Zero crossing detector
- Simple level detector

### Practical Notes

- 专用 comparator 通常更适合此用途。
- Op amp 可能有慢饱和恢复、输入保护限制、输出逻辑电平不兼容等问题。
- 若加入 hysteresis，则变成 Schmitt trigger。

---

## 22. Schmitt Trigger / Comparator with Hysteresis

### Configuration: Non-Inverting Schmitt Trigger

```text
Vin ─────────────▶ +OpAmp
                  │
                  │
             ┌─────────┐
             │ +       │
             │   OpAmp │────── Vout
Vref node ──▶│ -       │
             └─────────┘

Vout ── R1 ──●── R2 ── Vref/GND
             │
             └── threshold node
```

### Function

通过正反馈设置两个不同阈值：

```text
upper threshold
lower threshold
```

输入上升和下降时的切换点不同，从而抑制噪声导致的反复翻转。

### Applications

- Noisy threshold detection
- Square-wave cleanup
- Switch debouncing
- Relaxation oscillator building block

### Practical Notes

和 comparator-like use 一样，专用 comparator 通常比普通 op amp 更合适。

---

## 23. Quick Comparison Table

| Configuration | Main function | Ideal equation |
|---|---|---|
| Voltage follower | buffer | `Vout = Vin` |
| Non-inverting amplifier | positive voltage gain | `Vout = (1 + Rf/Rg)Vin` |
| Inverting amplifier | negative voltage gain | `Vout = -(Rf/Rin)Vin` |
| Inverting summer | weighted sum | `Vout = -Rf Σ(Vi/Ri)` |
| Non-inverting summer | same-phase average/sum | `Vout = (1 + Rf/Rg)Vx` |
| Difference amplifier | subtract two voltages | `Vout = (R2/R1)(V2 - V1)` |
| Instrumentation amplifier | precision differential gain | `Vout = G(V2 - V1) + Vref` |
| Transimpedance amplifier | current to voltage | `Vout = Vref - Iin Rf` |
| Voltage-to-current | voltage to current | `Iout = Vin/Rsense` |
| Integrator | integrate input | `Vout/Vin = -1/(RCs)` |
| Differentiator | differentiate input | `Vout/Vin = -RCs` |
| Active low-pass | gain + low-pass filtering | `-(Rf/Rin)/(1+sRfCf)` |
| Active high-pass | gain + high-pass filtering | `-(Rf/Rin)(sRinC)/(1+sRinC)` |
| Sallen-Key filter | second-order active filter | `H0ω0²/(s²+(ω0/Q)s+ω0²)` |
| MFB filter | inverting second-order filter | topology-dependent |
| Charge amplifier | charge to voltage | `Vout - Vref = -Q/Cf` |
| Log amplifier | logarithmic compression | `Vout ≈ -VT ln(Iin/Is)` |
| Antilog amplifier | exponential output | `Vout ≈ -RIs exp(Vin/VT)` |
| Precision rectifier | ideal-like rectification | `Vout = max(0,Vin)` or `|Vin|` |

---

## 24. Choosing the Right Configuration

### Need high input impedance?

Use:

```text
voltage follower
non-inverting amplifier
instrumentation amplifier
```

Avoid basic inverting amplifier if the signal source cannot drive `Rin`.

### Need exact weighted sum?

Use:

```text
inverting summing amplifier
```

It gives clean current summing at a virtual-ground node.

### Need subtract two signals?

Use:

```text
difference amplifier
```

For precision/high impedance:

```text
instrumentation amplifier
```

### Need photodiode or current-output sensor readout?

Use:

```text
transimpedance amplifier
```

Check stability and feedback capacitor carefully.

### Need PID-style I or D behavior?

Use:

```text
integrator for I term
differentiator for D term, but strongly bandwidth-limit it
```

### Need frequency shaping?

Use:

```text
first-order active filters
Sallen-Key filters
multiple-feedback filters
```

### Need small-signal rectification?

Use:

```text
precision rectifier
```

not a bare diode.

---

## 25. Practical Design Checklist

Before finalizing any op-amp amplifier circuit:

```text
1. Is the op amp stable at the intended closed-loop gain?
2. Is the input common-mode voltage inside the allowed range?
3. Can the output swing to the required voltage under load?
4. Is gain-bandwidth product sufficient?
5. Is slew rate sufficient?
6. Are input bias current and input offset voltage acceptable?
7. Are resistor thermal noise and tolerance acceptable?
8. Is there a DC path for input bias currents?
9. Are capacitive loads isolated or compensated?
10. Is power-supply decoupling placed close to the IC?
11. In single-supply circuits, is the signal biased around a valid Vref?
12. Does the circuit fail safely if the input is disconnected?
```

For precision analog work, the schematic equation is only the first layer. The final circuit behavior is set by the interaction of the ideal topology, component tolerances, op amp nonidealities, layout, grounding, decoupling, and the external source/load.

