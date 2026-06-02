# Classic Op-Amp Amplifier Configurations with Online Circuit Diagrams

This document is a source-linked companion to `Op_Amp_Amplifier_Configurations.md`.
The circuit diagrams here are not hand-drawn for this report; they are linked directly
from online technical/tutorial sources. The equations and comments are written here
for study and cross-checking.

> Note: External image links may depend on the source website remaining available.
> For formal slides or a thesis/report, cite the source page under each figure and
> download images only if the source license or fair-use context allows it.

## Main Sources Used

- Analog Devices, "ADALM1000 SMU Training Topic 17: Basic Op Amp Configurations":  
  https://www.analog.com/en/resources/analog-dialogue/studentzone/studentzone-may-2019.html
- Analog Devices, "Inverting Op Amp":  
  https://www.analog.com/en/resources/glossary/inverting-op-amp.html
- Analog Devices, "Optimizing Precision Photodiode Sensor Circuit Design":  
  https://www.analog.com/en/resources/technical-articles/optimizing-precision-photodiode-sensor-circuit-design.html
- Electronics Tutorials, "Operational Amplifier Summary":  
  https://www.electronics-tutorials.ws/opamp/opamp_8.html
- Electronics Tutorials, "Active Low Pass Filter":  
  https://www.electronics-tutorials.ws/filter/filter_5.html
- Electronics Tutorials, "Sallen and Key Filter":  
  https://www.electronics-tutorials.ws/filter/sallen-key-filter.html
- Electronics Tutorials, "Op-amp Comparator":  
  https://www.electronics-tutorials.ws/opamp/op-amp-comparator.html
- Electronics Tutorials, "Instrumentation Amplifier":  
  https://www.electronics-tutorials.ws/opamp/instrumentation-amplifier.html
- Wikimedia Commons, "Op-Amp Logarithmic Amplifier.svg":  
  https://commons.wikimedia.org/wiki/File:Op-Amp_Logarithmic_Amplifier.svg
- Wikimedia Commons, "Op-Amp Schmitt Trigger.svg":  
  https://commons.wikimedia.org/wiki/File:Op-Amp_Schmitt_Trigger.svg

## Ideal Op-Amp Assumptions

For most linear negative-feedback circuits below, the first-order ideal assumptions are:

- Input currents are approximately zero: `I+ = I- = 0`.
- With negative feedback and no saturation, the input terminals are at nearly equal voltage: `V+ ≈ V-`.
- Output current and voltage are limited by the actual op amp, supply rails, load, bandwidth, slew rate, input/output swing, noise, and stability.

These assumptions are useful for deriving the clean textbook equations, but real designs must check input common-mode range, output swing, gain-bandwidth product, phase margin, resistor noise, bias-current errors, and capacitive loading.

## 1. Voltage Follower / Unity-Gain Buffer

![Unity-gain follower](https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-02.png?la=en&rev=6aafc168c0834531b901be8e58fab563&sc_lang=en)

Source page: Analog Devices, ADALM1000 Topic 17, Figure 2.  
Direct image URL: https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-02.png?la=en&rev=6aafc168c0834531b901be8e58fab563&sc_lang=en

Function: voltage buffering without voltage gain.

Ideal relation:

```text
Vout = Vin
Av = 1
```

Analysis:

The output is fed directly to the inverting input. Negative feedback forces the op amp output to follow the non-inverting input. Its main value is impedance transformation: very high input impedance and low output impedance.

Applications:

- Buffering high-impedance sensors or voltage dividers.
- Isolating one filter stage from another.
- Driving ADC inputs or cables when no voltage gain is needed.

Practical notes:

- Check unity-gain stability; not all op amps are stable at gain of 1.
- Capacitive loads may require an output isolation resistor.

## 2. Non-Inverting Amplifier

![Non-inverting amplifier with gain](https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-08.png?la=en&rev=2b9c180279164cf095758bf36aae665e&sc_lang=en)

Source page: Analog Devices, ADALM1000 Topic 17, Figure 8.  
Direct image URL: https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-08.png?la=en&rev=2b9c180279164cf095758bf36aae665e&sc_lang=en

Alternative summary figure:

![Basic inverting and non-inverting op-amp circuits](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp48.gif?resize=489%2C251)

Source page: Electronics Tutorials, Operational Amplifier Summary.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp48.gif?resize=489%2C251

Function: amplify a voltage without phase inversion.

Ideal relation:

```text
Vout = (1 + Rf/Rg) Vin
Av = 1 + Rf/Rg
```

Analysis:

The input drives the non-inverting terminal. The feedback divider samples `Vout` and returns a fraction to the inverting terminal. Since `V- ≈ V+ = Vin`, the output must rise until the feedback divider equals `Vin`.

Applications:

- General sensor voltage gain.
- High-input-impedance preamplifier stages.
- Offset or reference-buffered single-supply stages.

Practical notes:

- Minimum stable gain depends on the selected op amp.
- The input common-mode range must include the input signal.
- For precision gain, resistor ratio tolerance matters more than absolute resistor values.

## 3. Inverting Amplifier

![Inverting amplifier](https://www.analog.com/en/_/media/analog/en/design-center/glossary/inverting-op-amp.jpg)

Source page: Analog Devices, Inverting Op Amp.  
Direct image URL: https://www.analog.com/en/_/media/analog/en/design-center/glossary/inverting-op-amp.jpg

Alternative ADI training figure:

![Inverting amplifier configuration](https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-05.png?la=en&rev=4bdc4936d4d1437db3745b1d64f9b8ee&sc_lang=en)

Source page: Analog Devices, ADALM1000 Topic 17, Figure 5.  
Direct image URL: https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-05.png?la=en&rev=4bdc4936d4d1437db3745b1d64f9b8ee&sc_lang=en

Function: amplify with 180-degree phase inversion.

Ideal relation:

```text
Vout = -(Rf/Rin) Vin
Av = -Rf/Rin
```

Analysis:

With the non-inverting input grounded, the inverting node is a virtual ground. Input current through `Rin` cannot enter the op amp input, so it flows through `Rf`. The output voltage is whatever value is required to support that feedback current.

Applications:

- Controlled-gain signal inversion.
- Current summing at a virtual-ground node.
- PI/PID error-path scaling and polarity correction.

Practical notes:

- Input impedance is approximately `Rin`.
- Bias-current compensation may use a resistor from `V+` to ground approximately equal to `Rin || Rf`.

## 4. Inverting Summing Amplifier

![Summing amplifier configuration](https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-07.png?la=en&rev=f3e53f9cacf9432f9c8fb2ac84ae71b7&sc_lang=en)

Source page: Analog Devices, ADALM1000 Topic 17, Figure 7.  
Direct image URL: https://www.analog.com/en/_/media/images/analog-dialogue/en/studentzone/5-2019/240559-fig-07.png?la=en&rev=f3e53f9cacf9432f9c8fb2ac84ae71b7&sc_lang=en

Alternative summary figure:

![Differential and summing amplifiers](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp50.gif?resize=482%2C252)

Source page: Electronics Tutorials, Operational Amplifier Summary.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp50.gif?resize=482%2C252

Function: weighted addition of several input voltages with inversion.

Ideal relation:

```text
Vout = -Rf (V1/R1 + V2/R2 + ... + Vn/Rn)
```

If all input resistors are equal to `R`:

```text
Vout = -(Rf/R)(V1 + V2 + ... + Vn)
```

Analysis:

All input currents sum at the virtual-ground inverting node. Because the node voltage is nearly fixed, each input contributes independently according to its resistor value. This makes the circuit a natural analog weighted-sum block.

Applications:

- Audio mixers.
- DAC resistor networks.
- PID summing junctions combining P, I, D, offset, and feed-forward terms.

Practical notes:

- Input channels interact little ideally, but source impedance adds to each input resistor.
- Large numbers of inputs increase noise and parasitic capacitance at the summing node.

## 5. Difference Amplifier / Subtractor

![Differential and summing amplifiers](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp50.gif?resize=482%2C252)

Source page: Electronics Tutorials, Operational Amplifier Summary.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp50.gif?resize=482%2C252

Function: subtract two input voltages and optionally apply gain.

Ideal relation for matched ratios `R2/R1 = R4/R3`:

```text
Vout = (R2/R1)(V2 - V1)
```

For all four resistors equal:

```text
Vout = V2 - V1
```

Analysis:

The circuit combines an inverting path for one input and a non-inverting divider path for the other. Accurate subtraction depends critically on resistor-ratio matching. Poor matching converts common-mode voltage into output error.

Applications:

- Differential-to-single-ended conversion.
- Bridge readout when source impedance is low and common-mode requirements are modest.
- Error-signal generation.

Practical notes:

- CMRR is often resistor-ratio limited.
- For high source impedance or high CMRR, use an instrumentation amplifier instead.

## 6. Instrumentation Amplifier

![Instrumentation amplifier example circuit](https://www.electronics-tutorials.ws/wp-content/uploads/2025/09/opamp169.gif?resize=487%2C311)

Source page: Electronics Tutorials, Instrumentation Amplifier.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2025/09/opamp169.gif?resize=487%2C311

Function: high-input-impedance differential amplification with strong common-mode rejection.

Typical three-op-amp INA relation:

```text
Vout = (1 + 2R/RG)(R4/R3)(V2 - V1)
```

For a unity-gain final difference stage:

```text
Vout = (1 + 2R/RG)(V2 - V1)
```

Analysis:

The first two op amps buffer and amplify each input without loading the source. The final difference amplifier subtracts the two buffered signals. A single gain resistor `RG` often controls the first-stage differential gain.

Applications:

- Wheatstone bridges.
- Strain gauges, thermocouples, biomedical electrodes.
- Precision differential measurements in noisy environments.

Practical notes:

- Use integrated INA ICs for better resistor matching and CMRR.
- Input common-mode range and output swing must be checked simultaneously.

## 7. Transimpedance Amplifier / Photodiode Amplifier

![Simple transimpedance amplifier circuit](https://www.analog.com/en/_/media/analog/en/landing-pages/technical-articles/optimizing-precision-photodiode-sensor-circuit-design/figure1.png?la=en&rev=98a8d846543d4139b7f050e1fbcd4718&sc_lang=en)

Source page: Analog Devices, Optimizing Precision Photodiode Sensor Circuit Design, Figure 1.  
Direct image URL: https://www.analog.com/en/_/media/analog/en/landing-pages/technical-articles/optimizing-precision-photodiode-sensor-circuit-design/figure1.png?la=en&rev=98a8d846543d4139b7f050e1fbcd4718&sc_lang=en

Programmable-gain version:

![Programmable gain photodiode amplifier](https://www.analog.com/en/_/media/analog/en/landing-pages/technical-articles/optimizing-precision-photodiode-sensor-circuit-design/figure7.png?la=en&rev=27cc0b35e3ef40798b6781e211b1f456&sc_lang=en)

Source page: Analog Devices, Optimizing Precision Photodiode Sensor Circuit Design, Figure 7.  
Direct image URL: https://www.analog.com/en/_/media/analog/en/landing-pages/technical-articles/optimizing-precision-photodiode-sensor-circuit-design/figure7.png?la=en&rev=27cc0b35e3ef40798b6781e211b1f456&sc_lang=en

Function: convert input current to output voltage.

Ideal relation:

```text
Vout = -Iin Rf
```

Depending on photodiode polarity and reference choice, the sign may appear positive in a particular schematic.

Analysis:

The op amp holds the summing node near the reference voltage, so the sensor current flows through `Rf`. A feedback capacitor is often added for stability because photodiode capacitance and op amp input capacitance add phase shift.

Applications:

- Photodiode readout.
- Ionization chambers and low-current detectors.
- Current-output sensors.

Practical notes:

- Choose low input bias current for small currents.
- Guarding, leakage control, and PCB cleanliness can dominate precision.
- Stability must be checked with sensor capacitance included.

## 8. Inverting Integrator

![Differentiator and integrator amplifiers](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp51.gif?resize=476%2C215)

Source page: Electronics Tutorials, Operational Amplifier Summary.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp51.gif?resize=476%2C215

Function: output is proportional to the time integral of input voltage.

Ideal relation:

```text
Vout(t) = -(1/RC) ∫ Vin(t) dt + Vout(0)
H(s) = Vout/Vin = -1/(sRC)
```

Analysis:

The input resistor converts input voltage to current. That current charges or discharges the feedback capacitor. The output ramps as needed to keep the inverting input at virtual ground.

Applications:

- PI/PID integral path.
- Ramp generation.
- Analog computation.
- Charge-sensitive amplifiers in modified form.

Practical notes:

- A real integrator usually places a large resistor in parallel with the capacitor to prevent DC saturation.
- Bias current and input offset voltage integrate over time and can drive the output to a rail.

## 9. Inverting Differentiator

![Differentiator and integrator amplifiers](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp51.gif?resize=476%2C215)

Source page: Electronics Tutorials, Operational Amplifier Summary.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp51.gif?resize=476%2C215

Function: output is proportional to the time derivative of input voltage.

Ideal relation:

```text
Vout(t) = -Rf C dVin/dt
H(s) = -s Rf C
```

Analysis:

The input capacitor current is `C dVin/dt`, and this current flows through the feedback resistor. The differentiator emphasizes fast changes and high-frequency content.

Applications:

- Edge detection.
- Lead compensation.
- PID derivative path, usually with high-frequency roll-off.

Practical notes:

- The ideal differentiator is noise-sensitive and often unstable.
- Practical differentiators add a series input resistor and a feedback capacitor to limit high-frequency gain.

## 10. First-Order Active Low-Pass Amplifier

![First-order active low-pass filter with amplification](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/filter-fil20.gif?resize=453%2C259)

Source page: Electronics Tutorials, Active Low Pass Filter.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/filter-fil20.gif?resize=453%2C259

Function: pass low-frequency signals, attenuate high-frequency signals, and optionally provide gain.

Ideal relations:

```text
Af = 1 + R2/R1
fc = 1/(2πRC)
|H(f)| = Af / sqrt(1 + (f/fc)^2)
```

Analysis:

The RC network sets the corner frequency. The op amp prevents load impedance from disturbing the filter and can provide non-inverting gain.

Applications:

- Anti-noise filtering.
- ADC anti-alias prefiltering when higher-order filtering is not required.
- PID derivative-noise suppression and actuator-bandwidth limiting.

Practical notes:

- Filter behavior is limited by op amp gain-bandwidth.
- For precision filters, capacitor tolerance and dielectric type matter.

## 11. Sallen-Key Active Filter

![Sallen-Key high-pass filter example](https://www.electronics-tutorials.ws/wp-content/uploads/2019/05/fil170.gif?resize=486%2C268)

Source page: Electronics Tutorials, Sallen and Key Filter.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2019/05/fil170.gif?resize=486%2C268

Function: second-order active filter topology; can implement low-pass, high-pass, and band-pass variants.

General second-order form:

```text
H(s) = K ω0^2 / (s^2 + (ω0/Q)s + ω0^2)       low-pass form
```

For equal-component unity-gain Sallen-Key low-pass, a common starting point is:

```text
fc ≈ 1/(2πRC)
```

Analysis:

Sallen-Key uses a non-inverting op amp stage with an RC feedback network that shapes the second-order response. Its advantages are simplicity and high input impedance; its disadvantages are sensitivity to component tolerances and op amp limitations, especially at high Q.

Applications:

- Audio filters.
- Lock-in and detector signal conditioning.
- Control-loop bandwidth shaping.

Practical notes:

- High-Q designs are sensitive to resistor/capacitor tolerance.
- The op amp must have enough bandwidth and slew rate at the selected `fc` and gain.

## 12. Comparator-Like Op-Amp Use

![Op-amp comparator circuit](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp103.gif?resize=436%2C202)

Source page: Electronics Tutorials, Op-amp Comparator.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp103.gif?resize=436%2C202

Function: compare an input voltage against a reference and drive the output high or low.

Ideal decision relation:

```text
If V+ > V-: Vout -> positive saturation
If V+ < V-: Vout -> negative saturation
```

Analysis:

This is open-loop operation, not linear amplification. The op amp's very high open-loop gain makes the output saturate for tiny differential input voltages.

Applications:

- Threshold detection.
- Zero-crossing detection.
- Simple square-wave generation.

Practical notes:

- A dedicated comparator is usually better than an op amp for speed, saturation recovery, and logic output compatibility.
- Slow or noisy inputs can cause output chatter unless hysteresis is added.

## 13. Schmitt Trigger / Comparator with Hysteresis

![Op-amp comparator with hysteresis](https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp109.gif?resize=494%2C156)

Source page: Electronics Tutorials, Op-amp Comparator.  
Direct image URL: https://www.electronics-tutorials.ws/wp-content/uploads/2018/05/opamp-opamp109.gif?resize=494%2C156

Public-domain alternative:

![Op-Amp Schmitt Trigger](https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Op-Amp_Schmitt_Trigger.svg/1280px-Op-Amp_Schmitt_Trigger.svg.png)

Source page: Wikimedia Commons, Op-Amp Schmitt Trigger.svg.  
Direct image URL: https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Op-Amp_Schmitt_Trigger.svg/1280px-Op-Amp_Schmitt_Trigger.svg.png

Function: comparator with two switching thresholds.

For a symmetric inverting Schmitt trigger with feedback factor `β`:

```text
VUT ≈ +β Vsat
VLT ≈ -β Vsat
β = R2/(R1 + R2)       depending on exact resistor naming
```

Analysis:

Positive feedback makes the reference threshold depend on the output state. The input must cross a different threshold to switch back, producing hysteresis and preventing chatter.

Applications:

- Cleaning noisy digital transitions.
- Oscillators and relaxation wave generators.
- Threshold detection for slow sensor signals.

Practical notes:

- Threshold equations depend on exact resistor placement and reference voltage.
- Dedicated comparators again tend to outperform general op amps.

## 14. Logarithmic Amplifier

![Op-amp logarithmic amplifier](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0e/Op-Amp_Logarithmic_Amplifier.svg/1280px-Op-Amp_Logarithmic_Amplifier.svg.png)

Source page: Wikimedia Commons, Op-Amp Logarithmic Amplifier.svg.  
Direct image URL: https://upload.wikimedia.org/wikipedia/commons/thumb/0/0e/Op-Amp_Logarithmic_Amplifier.svg/1280px-Op-Amp_Logarithmic_Amplifier.svg.png

Function: produce an output approximately proportional to the logarithm of input current or voltage.

For a diode-connected feedback element:

```text
Id ≈ Is exp(Vd/(nVT))
Vout ≈ -nVT ln(Iin/Is)
```

Analysis:

The exponential diode or transistor junction converts current ratio into voltage. A matched correction diode or transistor can reduce temperature dependence.

Applications:

- Wide-dynamic-range sensor conditioning.
- Optical power measurements.
- Compression before ADC conversion.

Practical notes:

- Temperature compensation is essential.
- Input current polarity and dynamic range must match the log element.

## 15. Precision Rectifier / Absolute-Value Amplifier

![Op-amp precision rectifier](https://upload.wikimedia.org/wikipedia/commons/thumb/6/60/Op-Amp_Precision_Rectifier.svg/1280px-Op-Amp_Precision_Rectifier.svg.png)

Source page: Wikimedia Commons, Op-Amp Precision Rectifier.svg.  
Direct image URL: https://upload.wikimedia.org/wikipedia/commons/thumb/6/60/Op-Amp_Precision_Rectifier.svg/1280px-Op-Amp_Precision_Rectifier.svg.png

Full-wave version:

![Op-amp full-wave precision rectifier](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Op-Amp_Precision_Rectifier_full_wave.svg/1280px-Op-Amp_Precision_Rectifier_full_wave.svg.png)

Source page: Wikimedia Commons, Op-Amp Precision Rectifier full wave.svg.  
Direct image URL: https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/Op-Amp_Precision_Rectifier_full_wave.svg/1280px-Op-Amp_Precision_Rectifier_full_wave.svg.png

Function: rectify signals smaller than a diode drop by placing the diode inside the op amp feedback path.

Ideal half-wave relation:

```text
Vout ≈ Vin      for selected polarity
Vout ≈ 0        for opposite polarity
```

Absolute-value versions combine rectified paths:

```text
Vout ≈ |Vin|
```

Analysis:

The op amp compensates for diode forward drop by increasing its output until the feedback condition is met. This allows rectification of millivolt-level signals, unlike passive diode rectifiers.

Applications:

- Precision AC-to-DC conversion.
- Signal magnitude detection.
- Analog absolute-value blocks.

Practical notes:

- High-frequency performance is limited by op amp slew rate and diode switching.
- Full-wave circuits require careful gain matching.

## Quick Configuration Map

| Configuration | Core Function | Ideal Equation |
|---|---|---|
| Voltage follower | Buffer | `Vout = Vin` |
| Non-inverting amp | Same-polarity voltage gain | `Vout = (1 + Rf/Rg)Vin` |
| Inverting amp | Inverted voltage gain | `Vout = -(Rf/Rin)Vin` |
| Inverting summer | Weighted sum | `Vout = -Rf Σ(Vi/Ri)` |
| Difference amp | Subtraction | `Vout = (R2/R1)(V2 - V1)` |
| Instrumentation amp | Precision differential gain | `Vout = G(V2 - V1)` |
| Transimpedance amp | Current-to-voltage | `Vout = -IinRf` |
| Integrator | Time integral | `H(s) = -1/(sRC)` |
| Differentiator | Time derivative | `H(s) = -sRC` |
| Active low-pass | Gain plus low-pass filtering | `fc = 1/(2πRC)` |
| Sallen-Key filter | Second-order filtering | `H(s)` second-order form |
| Comparator | Threshold decision | `Vout -> rail` |
| Schmitt trigger | Hysteretic threshold | `VUT/LT` set by feedback divider |
| Log amplifier | Logarithmic conversion | `Vout ∝ ln(Iin)` |

## What To Use for the PID v3.9 Report

For the PID circuit report, the most relevant sourced figures are:

1. Inverting amplifier: directly maps to many OP27/OP270 gain stages.
2. Inverting summing amplifier: directly maps to PID summing and output combination sections.
3. Integrator: directly maps to the I path.
4. Differentiator or practical lead/lag differentiator: conceptually maps to the D path, though the actual circuit may use filtered derivative behavior.
5. Active low-pass / feedback capacitor examples: useful for explaining noise limiting and loop stability.
6. Comparator/Schmitt trigger only if discussing lock detection, limit switching, or non-linear switching behavior.
