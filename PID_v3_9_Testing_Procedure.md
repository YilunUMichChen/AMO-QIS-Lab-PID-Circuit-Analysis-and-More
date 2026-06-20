# PID v3.9 Circuit Testing Procedure

This note is a practical bring-up and debugging procedure for the PID v3.9
circuit. It is based on the handwritten testing notes, Xinyu Feng's PI
calibration note, the schematic-level analysis, and the current interpretation
of the NIM-box wiring. The goal is to verify the board in a controlled order:
power first, then references and offsets, then signal stages, then PI behavior,
then actuator-facing outputs.

The most important rule is:

$$
\text{Never install the ICs before the power rails and regulator outputs have been verified.}
$$

## 1. Scope and Test Philosophy

The board should be tested stage by stage. Do not start by connecting the laser
PZT or current actuator. First prove that each internal signal is sane when the
input is a known waveform from a function generator.

The basic signal chain discussed for the PID/PD-box debugging is:

```text
spectroscopy input -> pre-gain -> offset / bias summing
                   -> P and I branches
                   -> summing amplifier
                   -> final output
```

In the schematic names used in this note, this corresponds approximately to
`IN1 -> GIN1 -> PREIN -> SUM -> MONITOR`, with `SUM` feeding the `OPP`, `INT`,
and current-lock branches.

The final output stage `OPOUT` also accepts auxiliary external inputs:

```text
FN_GEN   -> R95 -> OPOUT summing node
GRAT_OFF -> R94 -> OPOUT summing node
SUM_IN3  -> R96 -> OPOUT summing node
```

These auxiliary terms are useful for modulation, grating-offset injection, and
future spare summing input use.

A key meeting-level debugging point is headroom: the bias or scan voltage before
a later op-amp output stage should not be allowed to approach the supply rails.
One reported failure mode was a relevant bias node approaching roughly `15 V`,
which is too large for the following op-amp stage to process usefully. The
testing procedure should therefore check both waveform shape and DC level at
each stage.

## 2. Required Equipment

- NIM crate or verified bench supply connection for the board power.
- Digital multimeter with continuity and DC-voltage modes.
- Oscilloscope with at least two channels.
- Function generator.
- BNC cables and a BNC T connector.
- Small insulated screwdriver for trim pots and front-panel pots.
- Dummy load or high-impedance scope input for output testing.
- Board schematic and a printed switch/pot map.

Recommended initial function-generator signals:

- Stage-by-stage gain/sign tests: a small zero-centered square wave, typically
  `0.2 Vpp` to `0.5 Vpp` at first.
- Larger signal-chain stress tests, only after basic gain/sign checks pass:
  `1 Vpp` to `2 Vpp`, centered at `0 V`.
- Integrator tests: zero-centered square wave.

The zero-centered square wave matters. A square wave with DC offset will be
integrated as a nonzero average input and can drive the integrator into a rail.
Use oscilloscope DC coupling when checking offsets; AC coupling can hide the
very DC shift that the offset knob is supposed to create.

## 3. Preflight Inspection

Before applying power:

1. Confirm that no ICs are installed in the sockets.
2. Inspect solder joints, socket pins, jumper pins, and front-panel wiring.
3. Check for accidental shorts between `+15 V`, `-15 V`, and ground.
4. Confirm BNC shells are tied to the intended ground reference.
5. Check board ground continuity to the NIM box/chassis ground where intended.
6. Identify all front-panel controls and map them to schematic names.
7. Identify `JP1`, the offset-routing jumper.
8. Identify the internal switches associated with `S1`, especially `S1-1`,
   `S1-3`, and `S1-4`.
9. Inspect capacitor footprints that use multiple pad options. Confirm each
   capacitor is installed in the intended pad pair.
10. Verify that the expected IC types are available and separated:
    `OP27/OP27G`, `OP270`, `AD711`, and `REF02` are not interchangeable.
11. Identify any small capacitors placed across feedback/gain stages. These are
    usually for high-frequency rolloff and stability, not for the main
    integrator time constant.

Known practical risk from the notes: the negative regulator path previously
showed a fault, and replacing the regulator fixed the rail. Treat the `-15 V`
rail as a first-class check, not an afterthought.

## 4. Phase 1: Power-Only Test, No ICs Installed

### Purpose

Verify that the raw NIM supply, regulators, switch wiring, and local bypassing
are correct before any IC can be damaged.

### Procedure

1. Confirm again that all IC sockets are empty.
2. Connect the NIM power input.
3. Verify the expected raw supply pins from the NIM crate, typically around
   `+24 V` and `-24 V` before regulation.
4. Apply power.
5. Measure the regulator inputs.
6. Measure the regulator outputs:

$$
V_{+15} \approx +15~\mathrm{V}
$$

$$
V_{-15} \approx -15~\mathrm{V}
$$

7. Check the power-switch output after the regulators.
8. Check the local rail points distributed around the schematic.
9. Touch-test only cautiously: no regulator or component should heat rapidly.
10. Turn power off before installing ICs.

### Expected Results

| Node | Expected result |
|---|---|
| Raw positive supply | positive NIM supply before regulator |
| Raw negative supply | negative NIM supply before regulator |
| Regulated positive rail | about `+15 V` |
| Regulated negative rail | about `-15 V` |
| Ground | continuous with intended board/NIM ground |

### If This Fails

Do not install ICs. Debug in this order:

1. NIM crate power and pinout.
2. Power connector orientation.
3. Regulator part number and orientation.
4. Diode orientation around the regulators.
5. Shorts on the `+15 V` or `-15 V` rails.
6. Power switch wiring.

## 5. Phase 2: IC Installation Check

### Purpose

Install active components only after the supply rails are proven safe.

### Procedure

1. Turn off and unplug power.
2. Install each IC in the correct socket.
3. Verify package orientation and pin 1 direction.
4. Carefully distinguish:
   - `OP27` or `OP27G`: single precision op amp.
   - `OP270`: dual op amp, used in the reference/offset section.
   - `AD711`: used for the integrator section.
   - `REF02`: precision `+5 V` reference.
5. Apply power again.
6. Recheck `+15 V` and `-15 V`.
7. Check for excessive supply current or fast heating.
8. If any chip heats quickly, power down immediately and inspect orientation,
   socket placement, and local shorts.

## 6. Phase 3: Reference, Offset, and JP1 Test

### Purpose

Verify the local precision references and offset network before injecting the
offset into the main SUM stage.

### Initial Jumper State

Set `JP1` between pins `1` and `2`. In this state, the top-left
reference/offset section is isolated from the rest of the SUM path.

### Procedure

1. Power the board.
2. Measure the `REF02` output:

$$
V_{\mathrm{REF02}} \approx +5~\mathrm{V}
$$

3. Measure the OP270-derived negative reference:

$$
V_{-5} \approx -5~\mathrm{V}
$$

4. Check the board output points labeled for `+5 V OUT` and `-5 V OUT`.
5. Rotate the offset pot slowly across its range.
6. Confirm the offset voltage changes smoothly without jumps or dead spots.
7. Check that the buffer/follower output tracks the selected offset voltage.

After this test passes, move `JP1` to pins `2` and `3` to connect the offset
source into the `SUM` stage.

### What the Offset Is Adjusting

The offset adds a controllable DC term to the internal error/summing signal. In
a real saturated-absorption or laser-lock setup, the raw error signal may not be
perfectly centered at zero. The offset lets the user move the lock point and
compensate for DC imbalance before the PI controller acts.

If the offset circuit is not broken but the offset appears to have no visible
effect, the most likely explanations are measurement configuration and gain
allocation rather than a failed offset source. First check that `JP1` is in the
state that actually routes the offset into `SUM`. Then check that the scope is
DC-coupled and that the probe is placed at `SUM`, `MONITOR`, or a later node,
not before the offset is injected.

The lab debugging with Xinyu also showed a more subtle but important case: the
offset range can be real but small compared with the waveform entering through
`PREIN`. If `GPREIN` is too large, the error-signal contribution dominates the
`SUM` node. If `GSUM` or later output gain is too small, the resulting DC shift
can be hard to see on the oscilloscope. A simplified model is:

$$
V_{\mathrm{SUM}}
\approx
-R_{\mathrm{fb,SUM}}
\left(
\frac{V_{\mathrm{PREIN}}}{R_{\mathrm{PREIN}}}
+
\frac{V_{\mathrm{offset}}}{R_{\mathrm{offset}}}
\right)
$$

The relative visibility of the offset is therefore set by both the actual
offset range and the signal contribution entering the same summing node. During
tuning, it can be useful to reduce `GPREIN` and increase `GSUM` in opposite
directions so that the monitor amplitude remains usable while the error-signal
amplitude becomes comparable to the available offset adjustment range.

This offset/bias path also needs a headroom check. Meeting notes flagged a
failure mode in related PID/PD boxes where the bias before a later op-amp output
stage could become too large, approaching the rail-scale range around `15 V`.
That is not useful control range; it risks saturating the next stage. Some early
gain stages were estimated to be reasonable, with several gains near `1`, one
variable gain stage roughly in the `1` to `5` range, and one relevant offset
range around `+/-0.8 V`. The practical lesson is to keep the earlier bias/scan
range modest and recover output range later with controlled gain.

If a measured bias range is too large, the proposed fix is attenuation before
the sensitive op-amp input, for example by adding series/summing resistors on
the order of `100 kOhm` where appropriate. The goal is:

```text
reduce scan/bias range -> avoid op-amp saturation -> increase later gain if needed
```

## 7. Phase 4: Stage-by-Stage Signal Test

### Purpose

Verify the gain, sign, and routing of the input chain using a known waveform.

### Setup

Use a BNC T connector:

```text
Function generator -> T connector -> scope channel 1
                               |
                               +-> PID input IN1
```

Use scope channel 2 to probe each stage output.

Recommended input:

$$
V_{\mathrm{IN}}(t) = \text{small zero-centered square wave}
$$

The square wave is preferred for the main debug pass because its amplitude,
polarity, clipping, and DC shift are easy to see. A triangle wave can still be
used later as a smooth signal-chain check, but the square wave is the clearer
choice for finding the stage where a fault first appears.

### GIN1

Probe `GIN1` output.

Expected:

```text
square wave in, square wave out
sign depends on the polarity switch
```

The schematic/report interpretation is:

$$
V_{\mathrm{GIN1}} \approx S_{\mathrm{GIN1}} G_{\mathrm{GIN1}} V_{\mathrm{IN}}
$$

where `S_GIN1` is the selected polarity sign.

### PREIN

Probe `PREIN` output.

Expected:

```text
PREIN is inverted relative to GIN1.
Gain is adjustable with GPREIN.
For initial testing, set the gain near -1 if possible.
```

Approximate relationship:

$$
V_{\mathrm{PREIN}} \approx -G_{\mathrm{PREIN}} V_{\mathrm{GIN1}}
$$

The notes describe a practical adjustable range on the order of `0.1x` to `5x`,
but the exact range should be confirmed on the physical unit.

### SUM

Probe `SUM` output.

Expected:

```text
SUM is inverted relative to PREIN.
SUM gain is adjustable.
After JP1 is moved to pins 2 and 3, the offset contribution should be visible.
```

Approximate relationship:

$$
V_{\mathrm{SUM}}
\approx
-\left(
k_{\mathrm{pre}}V_{\mathrm{PREIN}}
+k_{\mathrm{off}}V_{\mathrm{offset}}
+\cdots
\right)
$$

At this stage, record the DC level as well as the waveform amplitude. A clean
triangle wave riding on a large DC bias is still a problem if it pushes the next
op-amp close to its input or output range. The test should therefore answer two
questions:

1. Is the waveform sign/gain correct?
2. Is the bias small enough that the following `OPP`, `INT`, and `OPOUT`
   stages retain useful headroom?

### MONITOR

Probe the monitor/error output.

Expected:

$$
V_{\mathrm{MONITOR}} \approx V_{\mathrm{SUM}}
$$

The monitor is intentionally placed after `SUM` because it shows the actual
conditioned internal error signal that will drive the P, I, and current-lock
branches.

Small capacitors in preamp or feedback paths should be interpreted separately
from the large integrator capacitors. A small feedback capacitor reduces
high-frequency gain, suppresses noise amplification, and improves stability. It
does not set the main low-frequency integral action in the same way as the
selected `INT` feedback capacitor.

### Square-Wave Unity-Gain Calibration

For gain calibration, use two oscilloscope channels:

```text
CH1 = local input of the stage being tested
CH2 = local output of that stage
```

For ordinary amplifier stages, define:

$$
G_{\mathrm{stage}}
=
\frac{V_{\mathrm{out,pp}}}{V_{\mathrm{in,pp}}}
$$

Unity gain means:

$$
\left|G_{\mathrm{stage}}\right| \approx 1
$$

The sign may be negative for an inverting stage. For example, if a stage is
designed to invert, the correct unity setting is:

$$
V_{\mathrm{out}} \approx -V_{\mathrm{in}}
$$

Recommended calibration order:

1. `GIN1`: adjust polarity/gain so the output square wave has the expected sign
   and the same peak-to-peak amplitude as `IN1`.
2. `PREIN`: adjust `GPREIN` until `PREIN` has the intended sign and the same
   peak-to-peak amplitude as the local input being compared.
3. `SUM`: set offset near the desired center, then adjust `GSUM` until the
   square-wave amplitude is known and useful. The offset knob should mainly
   move the waveform up or down, not change its peak-to-peak amplitude.
4. `OPOUT`: inject a square wave through a known final summing input and adjust
   `G_OUT` for the intended output scaling.
5. `OPP`: short or disable the integrator, turn on the lock path, and adjust
   the P control until the `OPP` square-wave amplitude matches the selected
   reference amplitude.

For the final output stage, there is no single `G_OUT` setting that makes every
possible input path unity gain, because the input resistors are different. For
the 10 kOhm auxiliary inputs `FN_GEN`, `GRAT_OFF`, and `SUM_IN3`, unity gain
requires approximately:

$$
R_{\mathrm{fb,out}} \approx 10~\mathrm{k\Omega}
$$

For the P path with `S1-1` open, the `OPP` signal enters through `R30 = 30
kOhm`, so the same output-gain setting gives only about one third of the
amplitude:

$$
\frac{10~\mathrm{k\Omega}}{30~\mathrm{k\Omega}} \approx \frac{1}{3}
$$

This matches Xinyu's PI calibration note: when `OPP` is calibrated to unity and
`S1-1` is open, the output terminal should show an inverted signal with
amplitude around one third of the input used for the final 10 kOhm output-gain
calibration.

The integrator is different. It cannot be calibrated by forcing
`Vout_pp/Vin_pp = 1` for a square wave, because a square-wave input should
produce a ramp or triangle-like output. For the integral branch, check the ramp
slope:

$$
\frac{dV_{\mathrm{INT}}}{dt}
=
-\frac{V_{\mathrm{SUM}}}{R_{\mathrm{I,eff}}C_{\mathrm{I,eff}}}
$$

If an integral unity-gain condition is needed, it must be defined at a chosen
frequency:

$$
\left|H_I(j\omega)\right|
=
\frac{1}{\omega R_{\mathrm{I,eff}}C_{\mathrm{I,eff}}}
$$

so unity at frequency `f` requires:

$$
R_{\mathrm{I,eff}}C_{\mathrm{I,eff}}
=
\frac{1}{2\pi f}
$$

## 8. Phase 5: Proportional and Integral Behavior

### Purpose

Verify that the P and I branches produce the expected qualitative behavior
before connecting an actuator.

### P-Only Test

Use a zero-centered square wave or triangle wave input. Disable or short the
integrator path for initial P-only testing if needed.

Expected P behavior:

```text
input square wave -> amplified square wave
input triangle wave -> amplified triangle wave
shape mostly preserved
amplitude and sign depend on gain/polarity settings
```

For the proportional path:

$$
V_{\mathrm{OPP}} \approx -G_{\mathrm{OPP}} V_{\mathrm{SUM}}
$$

The notes indicate `OPP` has a negative adjustable gain, roughly `0.2` to `10`.
More explicitly, the `OPP` stage is an inverting amplifier with:

$$
R_{\mathrm{in,P}} = R25 = 10~\mathrm{k\Omega}
$$

and feedback:

$$
R_{\mathrm{fb,P}} = R1 + R_P
$$

where:

$$
R1 = 2.2~\mathrm{k\Omega}, \qquad R_P = 0\text{--}100~\mathrm{k\Omega}
$$

Therefore:

$$
G_{\mathrm{OPP}}
=
-\frac{R1+R_P}{R25}
=
-0.22\text{ to }-10.22
$$

If the final output stage is included, the P contribution depends strongly on
`S1-1`. With `S1-1` open, the `OPP` output enters `OPOUT` through:

$$
R30 = 30~\mathrm{k\Omega}
$$

With `S1-1` closed, `R36 = 3 kOhm` is connected in parallel with `R30`, so:

$$
R_{\mathrm{P,out,closed}}
=
R30 \parallel R36
=
\frac{30~\mathrm{k\Omega}\cdot 3~\mathrm{k\Omega}}
{30~\mathrm{k\Omega}+3~\mathrm{k\Omega}}
=
2.727~\mathrm{k\Omega}
$$

The final output feedback is approximately:

$$
R_{\mathrm{fb,out}} = R78 + R_{G\_OUT}
=
2.2~\mathrm{k\Omega}\text{ to }102.2~\mathrm{k\Omega}
$$

Thus the total P-path contribution from `SUM` to `OUT` is approximately:

$$
G_{\mathrm{P,total}}
=
\frac{R_{\mathrm{fb,P}}}{R25}
\frac{R_{\mathrm{fb,out}}}{R_{\mathrm{P,out}}}
$$

where `R_P,out = R30` if `S1-1` is open and
`R_P,out = R30 || R36` if `S1-1` is closed. This gives:

| `S1-1` state | `R_P,out` | Approximate total P gain from `SUM` to `OUT` |
|---|---:|---:|
| open | `30 kOhm` | `+0.016` to `+34.8` |
| closed | `2.727 kOhm` | `+0.177` to `+383` |

The total sign is positive because the signal is inverted by `OPP` and inverted
again by `OPOUT`. The large numerical range is an electrical capability, not a
recommended starting point. Initial lock attempts should use low P and low
`G_OUT`.

### I-Only or I-Dominant Test

Use a zero-centered square wave.

Ideal integrator expectation:

$$
V_{\mathrm{INT}}(t)
=
-\frac{1}{R_{\mathrm{I,eff}}C_{\mathrm{I,eff}}}
\int V_{\mathrm{SUM}}(t)\,dt
+V_{\mathrm{INT}}(0)
$$

Thus:

```text
zero-centered square wave input -> triangle/ramp-like output
```

In the real board, selected feedback capacitors, leakage paths, parallel
resistors, and switch states may make the waveform look more like RC
charging/discharging than a perfect triangle. That is still useful information:
it tells you which effective time constant is active.

The qualitative time constant is:

$$
\tau_{\mathrm{I}} \approx R_{\mathrm{I,eff}}C_{\mathrm{I,eff}}
$$

Larger selected capacitance gives slower integration and smoother output.
Smaller selected capacitance gives faster correction but can be noisier or more
oscillation-prone in a closed loop.

When a resistor is placed across the feedback capacitor, it should not be
treated as merely another ideal-integrator input resistor. It gives the
capacitor a discharge path. This changes the long-time behavior:

```text
ideal integrator:    output can keep accumulating until it rails
leaky integrator:    output slowly relaxes through the parallel resistor
```

In Laplace-domain terms, an ideal capacitive feedback impedance is

$$
Z_C = \frac{1}{sC}
$$

while a capacitor with a parallel discharge resistor is

$$
Z_f = R_{\mathrm{leak}} \parallel \frac{1}{sC}
    = \frac{R_{\mathrm{leak}}}{1+sR_{\mathrm{leak}}C}
$$

Thus changing an input resistor and changing a leakage/discharge resistor can
produce curves that look superficially similar over a limited simulation window,
but they are not the same circuit operation. The input resistor mainly changes
integration rate; the parallel resistor limits long-time drift and saturation.

For the first integrator calibration described in Xinyu's PI note, short the P
branch, turn on the lock path, and turn off the selectable switches in `S1` so
that the default integration capacitance is `1 nF`. Then turn the I-gain pot to
the maximum-resistance setting. For a fixed capacitance, this should produce the
smallest ramp slope:

$$
\left|\frac{dV_{\mathrm{INT}}}{dt}\right|
=
\frac{|V_{\mathrm{SUM}}|}
{R_{\mathrm{I,eff}}C_{\mathrm{I,eff}}}
$$

Increasing either the integration resistance or the integration capacitance
reduces integral gain. Decreasing either one increases integral gain.

### Relevant Internal Switches

The notes highlight these switch functions:

| Switch | Practical role during testing |
|---|---|
| `S1-4` | can bypass/short the integrator for initial output testing |
| `S1-3` | affects the `INT` time constant |
| `S1-1` | affects the `OPP` signal level entering `OPOUT` |

Only one of the internal time-constant/capacitor switch choices should normally
be closed at a time. Once the NIM box is installed in the rack, internal `S1`
settings may no longer be easy to access, so document the chosen states before
mounting the module.

## 9. Phase 6: Current-Lock Section Test

### Purpose

Verify the independent current-lock output path before it is connected to any
laser-current actuator.

The current-lock section may not be used in every laser setup. Historically,
some systems used a separate current-locking NIM box rather than using the
current-feedback branch on this PID board. It is still important to understand
and test the section because current feedback is faster than PZT feedback.
PZT feedback changes the cavity length mechanically and has limited bandwidth;
current modulation can respond faster, help narrow the laser linewidth, and can
be important for high-resolution spectroscopy or precision Rydberg experiments.

The current-lock path is:

```text
SUM -> C_POL_OP -> C_PROP -> diode/filter network -> C_FLOAT -> C_OUT
```

### Procedure

1. Keep the external actuator disconnected.
2. Turn the front-panel current-lock switch off.
3. Probe after `C_FLOAT` and at `C_OUT`.
4. Confirm that the output is held at ground or the intended disabled state.
5. Turn current lock on only after the disabled behavior is correct.
6. Apply a small known input through the main signal chain.
7. Measure the `C_PROP` response.
8. Verify polarity switch behavior through `C_POL_OP`.
9. Verify the output remains bounded and does not rail unexpectedly.

Expected `C_PROP` behavior from the notes:

$$
V_{\mathrm{C\_PROP}} \approx -G_{\mathrm{C}}V_{\mathrm{C\_POL}}
$$

with `G_C` tunable roughly from `0.2` to `1.2`.

## 10. Phase 7: OPOUT, OUT, and Grating-Lock Test

### Purpose

Verify the final grating/PZT-facing output without risking the laser actuator.

The output summing node receives:

```text
OPP path       -> R30 / switched path -> OPOUT
INT path       -> R23 -> OPOUT
FN_GEN input   -> R95 -> OPOUT
GRAT_OFF input -> R94 -> OPOUT
SUM_IN3 input  -> R96 -> OPOUT
```

### Procedure

1. Keep the laser PZT disconnected.
2. Probe `OPOUT` output and the final `OUT` BNC.
3. With the grating-lock switch off, confirm the output is in the expected safe
   or offset-controlled state.
4. Rotate the grating-offset front-panel knob and confirm that `OUT` changes
   smoothly.
5. Inject a small test waveform through `FN_GEN` and confirm it appears at
   `OPOUT` with the expected sign and scale.
6. If `SUM_IN3` is wired, inject a small signal and verify it enters the same
   final summing node.
7. Test `OPP` contribution with the integrator bypassed or disabled.
8. Test `INT` contribution only after its switch state and time constant are
   understood.
9. Adjust `G_OUT` and confirm the output gain range is reasonable.

Expected final summing behavior:

$$
V_{\mathrm{OUT}}
\approx
-Z_{f,\mathrm{OPOUT}}
\left(
\frac{V_{\mathrm{INT}}}{R23}
+\frac{V_{\mathrm{OPP}}}{R30}
+\frac{V_{\mathrm{FN\_GEN}}}{R95}
+\frac{V_{\mathrm{GRAT\_OFF}}}{R94}
+\frac{V_{\mathrm{SUM\_IN3}}}{R96}
\right)
$$

This expression is schematic-level and assumes the relevant switches are in the
states that connect those branches.

## 11. Front-Panel and NIM-Box Integration Checks

The NIM-box wiring is part of the circuit. Verify it explicitly.

Checklist:

1. `IN1` BNC center and shield.
2. `FN_GEN` BNC center and shield.
3. `GRAT_OFF` BNC or pot wiring, especially the path through `R94`.
4. `SUM_IN3` spare input, if populated.
5. `OUT` BNC center and shield.
6. `C_OUT` BNC center and shield.
7. `MONITOR` or `ERR` output.
8. Current-lock switch mapping.
9. Grating-lock switch mapping.
10. Polarity switch mapping.
11. `P`, `I`, `G_OUT`, `GPREIN`, `GSUM`, and offset pot direction.
12. Internal `S1` states before the module is installed in the rack.

The handwritten notes mention a repaired or suspicious grating-offset path.
Therefore, the continuity around `GRAT_OFF` and `R94` should be checked rather
than assumed.

## 12. Expected Waveform Summary

| Test point | Input | Expected output |
|---|---|---|
| `GIN1` | zero-centered square | square, sign selected by polarity switch |
| `PREIN` | `GIN1` square | inverted square, adjustable gain |
| `SUM` | `PREIN` square | inverted/scaled square plus offset DC shift |
| `MONITOR` | internal `SUM` signal | same as `SUM` |
| `OPP` | `SUM` square | proportional inverted/scaled square |
| `INT` | zero-centered square | ramp/triangle or RC-like charging waveform |
| `C_PROP` | current-lock input signal | negative adjustable gain |
| `OPOUT` | P/I/aux inputs | weighted sum into final output |
| `OUT` | `OPOUT` output | final grating/PZT correction output |
| `C_OUT` | `C_FLOAT` output | current-lock correction output |

## 13. Troubleshooting Table

| Symptom | Likely cause | First check |
|---|---|---|
| No output anywhere | no power, wrong IC, bad socket, disconnected BNC | rails and IC orientation |
| Output railed at `+15 V` or `-15 V` | excessive gain, offset too large, wrong polarity, broken feedback | reduce gain, center offset, inspect feedback path |
| Integrator ramps with no intentional input | DC offset, leakage, switch state, op-amp offset | zero input, lock switch, capacitor switch, offset trim |
| MONITOR correct but OUT wrong | issue after `SUM` | `OPP`, `INT`, `OPOUT`, front-panel output wiring |
| Offset knob has no effect | wrong `JP1`, AC coupling, wrong probe node, offset too small compared with PREIN, reference failure | `REF02`, `-5 V`, `JP1`, DC coupling, `SUM`/`MONITOR` |
| Grating offset has no effect | broken `GRAT_OFF` path or `R94` wiring | continuity from front panel to `R94` to `OPOUT` |
| Some switches do nothing visibly | wrong node probed, switch controls another branch, internal switch inaccessible | map switch to schematic and probe both sides |
| Current lock runs away | current polarity wrong or gain too high | current-lock polarity and `C_PROP` gain |
| Oscillation with actuator connected | loop gain too high, time constant too fast, cable/load capacitance | lower P/I gain, choose larger `C`, check shielding |
| High noise | grounding, long front-panel wiring, poor bypassing | BNC shields, rail bypass caps, ground continuity |

## 14. Closed-Loop Laser/PZT Bring-Up

Only do this after the standalone electrical tests pass.

For the likely `780 nm` repumper ECDL lock:

```text
saturated-absorption photodiode/error signal
-> PID input
-> GIN1/PREIN/SUM conditioning
-> P and I correction
-> OPOUT
-> PZT on the ECDL grating
-> laser cavity length/frequency correction
```

The PID circuit does not know the desired optical lock point by itself. The
experimenter chooses it from the spectroscopy signal. First scan the laser
open-loop across the saturated-absorption feature and identify the desired
transition or detuned point. For a dispersive error signal, the natural lock
point is often the zero crossing of the selected feature. For a side-of-fringe
lock, the chosen point may instead be a fixed voltage on a slope.

The offset control then tells the PID which point should count as zero error.
If the desired optical frequency is `nu_0`, choose the offset so that:

$$
V_{\mathrm{raw\ error}}(\nu_0)
+
V_{\mathrm{offset}}
=
0
$$

After this adjustment, the PID will try to enforce:

$$
V_{\mathrm{MONITOR}} \approx 0
$$

at the chosen spectroscopy point.

Practical order:

1. Disconnect or disable the actuator.
2. Scan the laser open-loop and identify the spectroscopy feature or
   zero-crossing that should define the lock point.
3. Use offset to place that selected point near zero on `MONITOR`.
4. Confirm the error signal has the expected slope at the selected point.
5. Start with low proportional gain and the integrator disabled or shorted.
6. Verify loop sign: a small positive frequency perturbation should produce a
   correction that drives the error back toward zero.
7. Enable proportional feedback first.
8. Increase P gain gradually until the error is well suppressed but not
   oscillating. If ringing or oscillation appears, back the P gain down.
9. Add integral action slowly to remove steady-state error and slow drift.
10. Watch for rail saturation, slow wind-up, or low-frequency oscillation.
11. Adjust the selected integrator capacitor/time constant.
12. Record final switch states, pot settings, and measured waveforms.

For the 780 nm repumper ECDL case described by Carlos, the saturated-absorption
photodiode signal is the error signal. The PID output drives the PZT on the
homebuilt ECDL grating. Changing PZT voltage tilts the grating, changes the
external-cavity length, and pulls the laser frequency back toward the selected
spectroscopy lock point. The gain is correct only if closing the loop reduces
the observed error. If the error grows or the output runs away, stop and reverse
the relevant polarity before increasing gain.

During PI tuning, use these rules:

1. If the lock is weak or the error is barely corrected, increase P gain
   cautiously.
2. If the lock rings or oscillates quickly, reduce P gain or output gain.
3. If there is a steady-state offset after P-only lock, enable or increase I.
4. If the locked signal has slow oscillation, reduce integral gain by increasing
   the integration resistance or selected capacitance.
5. If the output slowly rails, reduce I gain, re-center the offset, and check
   for DC bias at `SUM`/`MONITOR`.

The same saturated-absorption signal can also be used to estimate laser
frequency noise during tuning. Place the laser frequency on the side of an
absorption feature, where the spectroscopy slope converts frequency noise into
voltage noise:

```text
laser frequency noise -> sat-spec voltage noise -> scope RMS / FFT estimate
```

If the local spectroscopy slope is known, voltage noise can be converted into
frequency noise:

$$
\delta \nu_{\mathrm{rms}}
\approx
\frac{\delta V_{\mathrm{rms}}}
{\left|dV/d\nu\right|}
$$

Use an oscilloscope time base appropriate to the noise frequencies of interest.
RMS measurements are often sufficient for practical tuning, while Fourier
analysis can reveal acoustic or mechanical resonances. Exact prediction is
difficult because real laser noise depends on mechanical, electronic, thermal,
and optical details.

## 15. Data Log Template

Use one row per test condition.

| Date | Stage | Input waveform | Switch states | Pot settings | Probe node | Expected | Measured | Pass/fail | Notes |
|---|---|---|---|---|---|---|---|---|---|
|  | Power only | none | ICs removed | n/a | `+15 V`, `-15 V` | rails correct |  |  |  |
|  | Reference | none | `JP1 = 1-2` | offset swept | `+5 V`, `-5 V` | refs correct |  |  |  |
|  | Input chain | zero-centered square | polarity set | gains near 1 | `GIN1`, `PREIN`, `SUM` | sign/gain match |  |  |  |
|  | Bias/headroom | square or DC sweep | offset routed | gains low | `SUM`, `OPP`, `OPOUT` inputs | no near-rail bias |  |  |  |
|  | Integrator | zero-centered square | selected `S1` | I gain set | `INT` | ramp/RC waveform |  |  |  |
|  | Output | small signal | lock off/on tested | `G_OUT` swept | `OUT` | bounded output |  |  |  |
|  | PI tuning | real sat-spec error | actuator connected cautiously | low P, slow I | `MONITOR`, `OUT` | error decreases, no oscillation |  |  |  |
|  | Laser-noise check | sat-spec side slope | loop open or controlled | low gain | `MONITOR`, scope trace | voltage noise measurable |  |  |  |

## 16. Acceptance Criteria

The board is ready for cautious closed-loop testing only when:

1. `+15 V`, `-15 V`, `+5 V`, and `-5 V` are verified.
2. ICs are correctly installed and remain cool during operation.
3. Offset generation works and `JP1` routing is understood.
4. `GIN1`, `PREIN`, `SUM`, and `MONITOR` produce expected waveforms.
5. Bias/headroom is checked so no stage drives the next op amp near the rails
   during normal scan or offset operation.
6. P-only behavior is verified.
7. Integrator behavior is verified for the selected internal switch state,
   including any leakage/discharge resistor behavior.
8. `C_OUT` and `OUT` are bounded and respond to the correct controls.
9. Front-panel BNCs, switches, and pots are mapped to schematic names.
10. Grating-offset continuity through `R94` is verified.
11. The final closed-loop sign has been checked using a small, reversible
    perturbation before full lock engagement.
12. The selected optical lock point is identified from the spectroscopy scan
    and is brought to `MONITOR ~= 0` using the offset control before final PI
    tuning.
