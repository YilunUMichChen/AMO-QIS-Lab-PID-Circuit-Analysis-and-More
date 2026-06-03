# PID v3.9 Schematic Direct Circuit Description

Source: `PID_GDrive/PID_V3_9.sch`

This document is a direct functional translation of the EAGLE schematic file. It is based on the schematic XML contents: part names, part values, BNC connectors, switch names, schematic text notes, and net-to-pin connectivity. It intentionally stays close to what the `.sch` file proves, and marks interpretation where the front-panel wiring or physical NIM-box implementation is needed.

## 1. High-Level Structure

The schematic itself divides the circuit into five labeled regions:

1. `Input and Preamps`
2. `Offset`
3. Main PID path around `SUM`, `INT`, `OPP`, and `OPOUT`
4. `Current Lock section`
5. `Power supply and filtering`

At the system level, the circuit takes an analog error/input signal, conditions it through preamplifier stages, forms proportional and integral correction terms, sums these terms with optional external modulation/offset inputs, and drives an output BNC. A separate current-lock/current-output path generates `C_OUT` using the current-lock proportional chain.

The main signal flow visible in the schematic is:

```text
IN1 BNC
  -> GIN1 preamp
  -> PREIN preamp
  -> SUM stage
  -> branches to:
       MONITOR / ERR output
       INT integrator path
       OPP proportional path
       C_POL_OP / C_PROP / C_FLOAT current-lock path
  -> OPOUT final summing/output amplifier
  -> OUT BNC
```

The final output stage `OPOUT` also accepts three extra summing inputs:

```text
FN_GEN  -> R95 -> OPOUT summing node
GRAT_OFF -> R94 -> OPOUT summing node
SUM_IN3 -> R96 -> OPOUT summing node
```

The schematic note explicitly states that these are intended as:

- `FN_GEN`: function-generator input.
- `GRAT_OFF`: grating offset, using a front-panel potentiometer.
- `SUM_IN3`: extra input, reserved just in case.

The same note recommends placing front-panel switches between the BNC inputs and the board inputs, wired locally at the front panel, to avoid long wire runs from the board to the switch and back.

## 2. Major Active Devices and External Interfaces

| Schematic name | Device/value in `.sch` | Functional role |
|---|---:|---|
| `GIN1` | OP27Z | First input amplifier after `IN1` |
| `PREIN` | OP27Z | Second preamplifier / adjustable gain stage |
| `SUM` | OP27Z | Main summing/error-processing amplifier |
| `MONITOR` | OP27Z | Monitor/error output buffer or amplifier feeding `ERR` |
| `INT` | AD711N | Integrator stage |
| `OPP` | OP27Z | Proportional path amplifier |
| `OPOUT` | OP27Z | Final output summing amplifier |
| `C_POL_OP` | OP27Z | Current-lock polarity/pre-conditioning amplifier |
| `C_PROP` | OP27Z | Current-lock proportional amplifier |
| `C_FLOAT` | OP27Z | Current output / floating output driver for `C_OUT` |
| `5V` | OP270 | Dual op-amp used in offset/reference section |
| `IC4` | REF02Z | Precision +5 V reference |
| `7815`, `7915` | 78xx/79xx regulators | Positive/negative supply regulation |

External connectors and board pads:

| Name | Type | Role |
|---|---|---|
| `IN1` | BNC-to-board | Main input/error source into the preamp chain |
| `ERR` | BNC-to-board | Monitor/error output from `MONITOR` path |
| `OUT` | BNC-to-board | Main PID output |
| `C_OUT` | BNC-to-board | Current-lock/current output |
| `FN_GEN` | BNC-to-board | External function-generator summing input |
| `GRAT_OFF` | BNC-to-board | External grating-offset summing input |
| `SUM_IN3` | BNC-to-board | Spare external summing input |
| `+5V_OUT`, `-5V_OUT` | 1x2 pin headers | Reference voltage outputs |
| `SUPPLY+`, `SUPPLY-`, `SUPPLY_GND` | wirepads | Supply input pads |
| `ALT_SUPPLY+`, `ALT_SUPPLY-` | wirepads | Alternate supply pads through `U$1` |
| `+15V`, `-15V`, `+GND`, `-GND` | wirepads | Regulated supply output / distribution pads |

## 3. Input and Preamps

### 3.1 `IN1` input connector

The main signal enters through `IN1`, a BNC-to-board connector. The BNC signal pin is net `N$14`, which connects to:

```text
IN1 SIGNAL -> R3 pin 1
```

The BNC shield pins connect to net `N$22`, tied through the input grounding network:

```text
IN1 GND1/GND2 -> R2 pin 1 -> local ground/shield network
```

This means the schematic treats `IN1` as a single-ended coaxial input referenced to circuit ground.

### 3.2 `GIN1`: first input amplifier

`GIN1` is an OP27 stage. Its output net is `N$65`, which connects:

```text
GIN1 OUT -> R4 pin 2 -> R35 pin 2 -> C34 pin 2
```

`C34` has value `2.2pF*`, so it is likely a small high-frequency compensation/stability capacitor around or near the first gain stage. The exact gain of `GIN1` depends on the connections around `R3`, `R4`, `R5`, and `R2`; from the net connectivity it is an op-amp conditioning stage placed immediately after the input BNC.

Functional description:

```text
V_GIN1_OUT = A_GIN1(s) * V_IN1
```

where `A_GIN1(s)` is the first-stage gain/conditioning transfer function. The schematic clearly shows this as an OP27-based preamplifier, but the exact closed-loop expression should be verified from the graphical schematic because several local input/grounding elements participate.

### 3.3 `PREIN`: second preamp / adjustable gain stage

`PREIN` is also OP27Z. Its inverting input net `N$72` connects:

```text
GIN1 OUT -> R35 -> PREIN -IN
GPREIN pin 1 -> PREIN -IN
```

The `PREIN` output net `N$60` connects:

```text
PREIN OUT -> R56 pin 2 -> R28 pin 2
```

The gain potentiometer `GPREIN` is a 100 k potentiometer. Its pins 2 and 3 are tied to `R56 pin 1` on net `N$97`, while pin 1 is at the inverting input. This makes `GPREIN` part of the feedback/gain-setting path for the `PREIN` stage.

Approximate inverting-stage interpretation:

```text
V_PREIN_OUT ~= - (Z_feedback_PREIN / R35) * V_GIN1_OUT
```

where `Z_feedback_PREIN` is controlled by the `GPREIN`/`R56` network. The `+IN` input is referenced to ground through `R50`, so `PREIN` is best described as an adjustable inverting gain stage.

## 4. Offset and Reference Section

### 4.1 Precision reference

`IC4` is a REF02Z precision voltage reference. Its output net `N$18` connects:

```text
IC4 VO -> R31 pin 2
       -> R22 pin 2
       -> +5V_OUT pins 1 and 2
```

Therefore `+5V_OUT` is directly derived from the REF02 output node.

`IC4` is powered from `+15V` and locally decoupled by `C5 = 100 nF`.

### 4.2 Offset potentiometer and OP270 section

The dual op-amp part `5V` is an OP270. It has gates `A`, `B`, and power gate `P`.

Observed nets:

```text
REF02 +5 V node N$18 -> R22 -> 5V(A) -IN
5V(A) OUT -> R58/R9 -> -5V_OUT
O potentiometer, 20k, participates in the offset/reference path
5V(B) appears configured as a buffer/follower around JP1 and the offset node
```

The net `N$67` connects:

```text
5V(A) OUT -> R58 pin 1 -> R9 pin 2 -> -5V_OUT pins 1 and 2
```

This indicates that the circuit creates a negative reference/output node, labeled `-5V_OUT`, using the OP270 section. The `O` potentiometer is a 20 k offset control connected between reference-related nodes and the OP270 buffer path.

Functional description:

```text
+5V_OUT = +5 V reference from REF02
-5V_OUT = buffered/inverted reference from OP270 stage
V_offset = adjustable fraction/sign derived from the reference network
```

The exact offset transfer depends on the front-panel use of `O` and `JP1`, but the schematic clearly implements a local precision reference and adjustable offset source.

## 5. Main Summing Stage: `SUM`

`SUM` is an OP27Z. Its non-inverting input is tied to ground through `R13`, so it functions as an inverting summing/gain stage.

The `SUM` inverting input net is `N$16`:

```text
R10 pin 1
R28 pin 1
GSUM pin 1
SUM -IN
```

Its output net is `N$21`:

```text
SUM OUT
R65 pin 2
R14 pin 1
R11 pin 1
R25 pin 1
C_POL switch contacts
```

`R28 = 20k` brings the `PREIN` output into `SUM`. `R10` brings in a signal from `JP1`. `GSUM` is a 100 k potentiometer with pins 2 and 3 tied to `R65 pin 1`; together `GSUM` and `R65` form the feedback/gain-adjusting network.

Approximate transfer:

```text
V_SUM_OUT ~= - Zf_SUM * (V_PREIN_OUT / R28 + V_JP1 / R10 + ...)
```

where:

```text
Zf_SUM ~= feedback impedance set by GSUM and R65
```

This stage is the central error/summing node before the PID branches. It feeds:

- `MONITOR` through `R14`
- `INT` through `R11` and `I`
- `OPP` through `R25` / proportional path
- `C_POL_OP` and current-lock path through the `C_POL` switch network

## 6. Monitor / Error Output Path

`MONITOR` is an OP27Z. It receives the `SUM` output through `R14`:

```text
SUM OUT net N$21 -> R14 -> MONITOR +IN
```

Its output net `N$24` connects:

```text
MONITOR OUT -> R26 pin 1 -> R15 pin 2
```

The `ERR` BNC signal net `N$64` connects:

```text
ERR SIGNAL -> R26 pin 2
```

So the monitor output is sent to the `ERR` connector through `R26`.

`MONITOR` uses the common OP27 offset-null network:

```text
R82 = 1k trim pot
R84, R85 = 4.5k
```

Functional description:

```text
V_ERR ~= A_MONITOR * V_SUM_OUT
```

where the exact gain/sign depends on the `R14/R15` arrangement. Since `SUM OUT` enters `MONITOR +IN`, this path appears intended as a monitor/buffer or non-inverting scaling stage for observing the internal error/sum signal.

## 7. Integral Path: `INT`

`INT` is an AD711N op amp used as the integrator.

Input to the integrator:

```text
SUM OUT net N$21 -> R11 -> I potentiometer pin 1
```

The integrator summing node is net `N$82`, connected to:

```text
INT -IN
I pins 2 and 3
C7 pin 2
C8/C9/C10/C11 pin 1
CL pins 2 and 3
R24 pin 1
S4 O
```

The integrator output net is `N$37`, connected to:

```text
INT OUT
C7 pin 1
R23 pin 1
S4 P
S1 switch common side for multiple capacitor selections
```

The schematic also shows switch `S1` selecting among capacitors:

```text
C8, C9, C10, C11 and C7 are connected around the INT feedback/summing network through S1 contacts.
```

This means the integrator has switch-selectable feedback capacitance/time constant. `I = 20k` adjusts integral gain, and `CL = 20k` participates in the clamp/limit or integrator leakage/reset path through `R59` and `S1/S4`.

Approximate integrator relation:

```text
V_INT_OUT(s) ~= - 1 / (R_I_eff * C_INT_eff * s) * V_SUM_OUT(s)
```

where:

```text
R_I_eff  = effective resistance set mainly by R11 and I
C_INT_eff = selected feedback capacitance through S1/C7-C11
```

The integrator output enters the final output summing node through `R23`:

```text
INT OUT -> R23 -> OPOUT -IN summing node N$1
```

Therefore:

```text
Contribution at OPOUT from INT ~= - (Zf_OPOUT / R23) * V_INT_OUT
```

## 8. Proportional Path: `OPP`

`OPP` is an OP27Z proportional amplifier. This is the source of the earlier ambiguity around the label "P/Grating Path": the `.sch` file itself names the op amp `OPP`, and its associated potentiometer is `P = 100k`. The grating-offset input is separate and enters the final `OPOUT` summing node through `GRAT_OFF` and `R94`.

Input/output connectivity:

```text
SUM OUT net N$21 -> R25 -> OPP -IN net N$40
OPP OUT net N$39 -> R30 -> OPOUT summing node N$1
P potentiometer and R1 form the adjustable proportional feedback path
S3 switches part of the proportional path
S1/R36 provide an additional switched connection into the OPOUT summing node
```

Important nets:

```text
N$40: S3 P, OPP -IN, R25 pin 2, P pin 1
N$80: P pins 2/3, R1 pin 1
N$39: OPP OUT, R1 pin 2, R30 pin 1, S3 O, S1 -1 pin 2
```

This is consistent with an adjustable inverting proportional stage:

```text
V_OPP_OUT ~= - (R_P_eff / R25) * V_SUM_OUT
```

where `R_P_eff` is set by the `P` potentiometer and associated `R1/S3` feedback path.

Then `OPP` contributes to the final output through `R30`:

```text
Contribution at OPOUT from OPP ~= - (Zf_OPOUT / R30) * V_OPP_OUT
```

Because `OPOUT` is also inverting, the final sign of the proportional contribution depends on the two-stage inversion and switch state.

## 9. Final Output Summing Stage: `OPOUT`

`OPOUT` is an OP27Z configured as the final inverting summing amplifier. Its non-inverting input is grounded through `R32 = 2.2k`.

The main summing node is net `N$1`, connected to:

```text
OPOUT -IN
R30 pin 2       from OPP proportional output
R23 pin 2       from INT output
R94 pin 2       from GRAT_OFF
R95 pin 2       from FN_GEN
R96 pin 2       from SUM_IN3
R36 pin 1       switched auxiliary path from S1
G_OUT pin 1     output gain feedback network
```

The output net `N$53` connects:

```text
OPOUT OUT -> R46 pin 1 -> OUT BNC signal through R46
OPOUT OUT -> R78 pin 2 -> G_OUT feedback network
```

`G_OUT = 100k` with `R78 = 2.2k` forms the adjustable final feedback/output-gain network. `R46 = 560` is a series output resistor before the `OUT` BNC.

The final output can be summarized as:

```text
V_OUT_PRE_SERIES ~= - Zf_OUT * (
    V_OPP_OUT / R30
  + V_INT_OUT / R23
  + V_GRAT_OFF / R94
  + V_FN_GEN / R95
  + V_SUM_IN3 / R96
  + V_AUX_SWITCHED / R36
)

V_OUT_BNC ~= V_OUT_PRE_SERIES after R46 series isolation
```

With `R94 = R95 = R96 = 10k`, the three external summing inputs enter with equal nominal summing weights before the final gain factor.

## 10. Current Lock Section

The current-lock section is the upper-right block in the schematic. It includes:

```text
C_POL_OP -> C_PROP -> diode/filter network -> C_FLOAT -> C_OUT
```

### 10.1 Current polarity/pre-conditioning: `C_POL_OP`

`C_POL_OP` is OP27Z. It is connected to the main `SUM OUT` net through the `C_POL` switch:

```text
SUM OUT net N$21 -> C_POL switch contacts
C_POL -> R16/R17 -> C_POL_OP inputs
```

`C_POL_OP` output net `N$29` connects:

```text
C_POL_OP OUT -> R19 feedback -> R20 pin 1
```

The switched polarity arrangement suggests that `C_POL` selects the sign/polarity of the current-lock input before it goes to `C_PROP`.

Approximate role:

```text
V_C_POL_OUT = +/- A_CPOL * V_SUM_OUT
```

where the sign is selected by `C_POL`.

### 10.2 Current proportional amplifier: `C_PROP`

`C_PROP` is OP27Z. Its `+IN` is referenced through `R27 = 2.2k` to ground. Its `-IN` is net `PC_OUT`, connected to:

```text
R20 pin 2
C_PROP -IN
PC pins 2 and 3
C_ON/OFF switch P
```

Its output net `N$41` connects:

```text
C_PROP OUT
R37 pin 2
R63 pin 2
C_ON/OFF switch O
```

The `PC = 10k` potentiometer and `R37 = 2.2k` form the adjustable feedback path. `C_ON/OFF` switches the current proportional path on/off.

Approximate proportional relation:

```text
V_C_PROP_OUT ~= - (R_PC_eff / R20) * V_C_POL_OUT
```

where `R_PC_eff` is controlled by `PC` and the switch state.

### 10.3 Clamp/filter network

After `C_PROP`, the signal passes through `R63 = 1k` to net `N$42`, which contains:

```text
R63 pin 1
D5 1N4148
D6 1N4148
C12 = 100pF
R88 pin 1
```

`D5` and `D6` are connected to ground around this node, and `C12` is also tied to ground. Functionally, this network provides fast protection/limiting and high-frequency filtering before the floating/current output driver.

### 10.4 Current output driver: `C_FLOAT`

`C_FLOAT` is OP27Z. It drives `C_OUT`.

Key nets:

```text
N$45: C_FLOAT +IN, R86 pin 2, R89 pin 1
N$51: C_FLOAT -IN, R88 pin 2, R90 pin 1
N$52: C_FLOAT OUT, R90 pin 2, C_OUT SIGNAL
```

`R86`, `R88`, `R89`, and `R90` are all 10 k. This symmetry indicates a differential/floating-output style driver or unity-gain differential stage around `C_FLOAT`.

The output relation is approximately:

```text
V_C_OUT ~= A_C_FLOAT * V_C_PROP_FILTERED
```

with `A_C_FLOAT` near unity if the 10 k resistor network is balanced. `R90` also participates in feedback and output drive to the `C_OUT` BNC.

## 11. External Summing Inputs

The schematic explicitly creates three BNC-to-board inputs that feed the final output summing node:

```text
GRAT_OFF SIGNAL -> R94 10k -> OPOUT summing node N$1
FN_GEN SIGNAL   -> R95 10k -> OPOUT summing node N$1
SUM_IN3 SIGNAL  -> R96 10k -> OPOUT summing node N$1
```

Since these enter the inverting input of `OPOUT`, each contributes:

```text
V_OUT contribution ~= - (Zf_OUT / 10k) * V_external
```

The front-panel note in the schematic says:

- `FN_GEN` is meant for function-generator injection.
- `GRAT_OFF` is meant for grating offset using a front-panel pot.
- `SUM_IN3` is spare.
- Switches may be inserted at the front panel before these board inputs.

This is important: the schematic captures the board-level summing inputs, but not necessarily every front-panel switch/pot wiring detail.

## 12. Power Supply and Filtering

The supply section includes:

```text
7815 positive regulator
7915 negative regulator
D3/D4 1N4001 protection diodes
C3 = 0.33uF
C36 = 2uF
C2/C4 = 0.1uF local polarized capacitors
many 100nF decoupling capacitors C13-C35
```

Supply input and output pads:

```text
SUPPLY+  -> V+ regulator input path
SUPPLY-  -> V- regulator input path
SUPPLY_GND -> supply ground
ALT_SUPPLY+ / ALT_SUPPLY- -> alternate supply route through U$1
+15V, -15V, +GND, -GND -> board distribution pads
```

The many 100 nF capacitors are distributed across the op-amp supply rails. Their purpose is local high-frequency decoupling:

```text
+15V rail -> 100nF -> GND
-15V rail -> 100nF -> GND
```

Most OP27/AD711 stages also have offset-null trim networks:

```text
1k trim pot + two 4.5k resistors around OP27 ON1/ON2 and supply pins
```

These trim networks are not signal gain controls; they are for op-amp offset/null adjustment.

## 13. Functional PID Interpretation

The schematic implements a PID-like analog controller, although the board labels emphasize proportional and integral functions more clearly than a standalone derivative section.

The dominant visible control terms are:

```text
P term: SUM OUT -> OPP -> R30 -> OPOUT
I term: SUM OUT -> INT -> R23 -> OPOUT
External offsets/modulation: FN_GEN, GRAT_OFF, SUM_IN3 -> OPOUT
Current-lock branch: SUM OUT -> C_POL_OP -> C_PROP -> C_FLOAT -> C_OUT
```

A compact system-level expression is:

```text
V_OUT(s) ~= K_OUT * [
    K_P * V_SUM(s)
  + K_I/s * V_SUM(s)
  + K_FN * V_FN_GEN(s)
  + K_GRAT * V_GRAT_OFF(s)
  + K_EXT * V_SUM_IN3(s)
]
```

where the signs depend on the number of inverting stages and switch positions. In terms of schematic controls:

```text
K_P   is adjusted mainly by P and the OPP feedback path.
K_I   is adjusted by I and the selected integrator capacitance through S1/C7-C11.
K_OUT is adjusted by G_OUT around OPOUT.
K_GRAT, K_FN, and K_EXT are set by R94/R95/R96 and OPOUT feedback.
```

The current-output path can be summarized as:

```text
V_C_OUT(s) ~= K_C_FLOAT * K_C_PROP * K_C_POL * V_SUM(s)
```

where:

```text
K_C_POL  includes the C_POL switch sign.
K_C_PROP is adjusted by PC and C_ON/OFF.
K_C_FLOAT is set by the balanced 10k C_FLOAT network.
```

## 14. Direct Schematic Conclusions

1. `OPP` is the proportional amplifier stage. It is not the grating-offset input. The grating-offset input is `GRAT_OFF`, entering `OPOUT` through `R94`.
2. `FN_GEN` is an external function-generator injection input summed directly into the final output amplifier through `R95 = 10k`.
3. `OPOUT` is the final summing amplifier. Its inverting input net `N$1` collects the proportional path, integral path, external function generator, grating offset, spare summing input, and a switched auxiliary path.
4. `INT` uses switch-selectable capacitors, so the integral time constant is user-selectable.
5. The `C_OUT` path is separate from the main `OUT` path and is generated by the current-lock chain.
6. The schematic includes board-level BNC inputs and pads, but some NIM-box front-panel wiring is intentionally external and must be confirmed from photos or the physical box.

## 15. Items That Need Physical/Panel Confirmation

The `.sch` file alone does not fully specify:

1. Which front-panel switch labels correspond to `S1`, `S3`, `S4`, `C_ON/OFF`, and `C_POL`.
2. Whether `GRAT_OFF` is always connected to a front-panel pot or can be switched/bypassed.
3. The exact human-facing labels of `P`, `I`, `CL`, `PC`, `G_OUT`, `GPREIN`, `GSUM`, and `O`.
4. Whether all optional external BNC inputs are populated on the physical NIM box.
5. The exact mechanical mapping between PCB pads and front-panel wiring.

Those points should be resolved by cross-checking `PID_V3_9.brd`, the NIM-box photos, and continuity/physical wiring observations.

