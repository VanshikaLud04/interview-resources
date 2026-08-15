# EE Basics — Q&A

## Q. "How does EE help you in software?"
**A:** 1) Signal Processing → ML feature extraction (FFT for audio, convolutions for images). 2) Control Systems → PID controllers in ArduPilot robotics. 3) Digital Electronics → CPU architecture, bitwise ops, memory management. 4) Power Constraints → Edge AI optimization (Focus Lock: entropy-based CPU optimization for YOLOv8).

---

## Circuit Theory

### Q. Ohm's Law, KVL, KCL?
- **Ohm's:** V = IR
- **KCL:** Total current entering a junction = total current leaving (conservation of charge).
- **KVL:** Sum of voltages around any closed loop = zero (conservation of energy).

### Q. Thevenin's vs Norton's?
- **Thevenin:** Any linear circuit → single voltage source + series resistance.
- **Norton:** Any linear circuit → single current source + parallel resistance.

---

## Semiconductor Devices: BJT, MOSFET, IGBT

### Q. How does an NPN BJT work?
**A:** An NPN BJT has an n-type emitter, thin p-type base, and n-type collector. In forward-active operation, the base-emitter junction is forward biased and the base-collector junction is reverse biased. Electrons injected from the emitter cross the thin base; the collector electric field sweeps most of them into the collector. A small base current controls a much larger collector current (`I_C ≈ beta I_B` in the simple model).

**Regions:**
- **Cutoff:** base-emitter not forward biased; essentially off.
- **Forward active:** amplification region; `I_C` is mainly controlled by base drive / `V_BE`.
- **Saturation:** both junctions forward biased; switch is on but no longer in linear amplification.

**Follow-up: Why is the base thin and lightly doped?** So few emitter electrons recombine in the base; most reach the collector, producing current gain.

### Q. How does an NMOS MOSFET work?
**A:** A MOSFET is voltage-controlled. When `V_GS` exceeds threshold voltage, the electric field through the insulated gate oxide creates an inversion channel between source and drain. Current can then flow from drain to source. Because the gate is insulated, ideal DC gate current is nearly zero.

**Regions:**
- **Cutoff:** `V_GS < V_TH`; channel absent.
- **Triode/linear:** channel behaves roughly like a controllable resistor; common for an on-switch with low `R_DS(on)`.
- **Saturation:** pinched near the drain; current depends mainly on `V_GS`; common in analogue amplification models.

**Follow-up: Why is it good for switching?** Gate drive needs charge, not sustained DC current; a fully enhanced device can have low on-resistance and switch efficiently at high frequency.

### Q. How does an IGBT work?
**A:** An IGBT combines a MOS gate input with bipolar conduction in the power path. A positive gate voltage forms a MOS channel that enables carrier injection through the device, giving low conduction loss at high voltage/current. It is normally used as a power switch, not a small-signal amplifier.

**Key trade-off:** IGBTs are attractive in high-voltage/high-current applications (motor drives, inverters, EV power electronics) but are generally slower to turn off than MOSFETs because stored minority carriers create a tail current.

### Q. BJT vs MOSFET vs IGBT?

| Device | Control | Best mental model | Typical strength | Main limitation |
|---|---|---|---|---|
| BJT | current-driven base (with exponential `V_BE` relation) | current amplifier / switch | gain, analogue circuits | continuous base-drive current, slower power switching |
| MOSFET | voltage-driven insulated gate | field-effect switch | fast switching, low gate DC power | conduction loss rises with `R_DS(on)` at high voltage |
| IGBT | voltage-driven MOS gate + bipolar power path | high-power switch | high voltage/current, low conduction loss | slower turn-off / tail current |

### Q. How would you choose a MOSFET or IGBT for a converter?
**A:** Start with bus voltage, current, switching frequency, thermal budget, and cost. For lower-to-medium voltage and high switching frequency, a MOSFET is often preferred because it switches quickly. At higher voltage/current where conduction loss dominates and switching frequency is lower, an IGBT can be a better fit. I would compare datasheet switching energy, on-state loss, gate-drive requirements, cooling, and safe-operating-area—not choose only by device label.

### Q. What is a flyback diode and why is it needed with an inductive load?
**A:** Inductor current cannot stop instantly. When a relay/motor switch turns off, the inductor can create a large voltage spike to preserve current. A properly oriented flyback diode gives that current a safe path and protects the switch. In higher-performance power stages, diode recovery and switching losses also matter.

### Q. What is thermal runaway?
**A:** A temperature rise can increase device losses or current, creating more heat and a feedback loop. Prevent it with correct derating, heat sinking, current limiting, thermal design, and device selection. The exact risk and mitigation differ by device and circuit.

**EE-to-software bridge:** These device questions are usually testing first-principles reasoning: identify control input, state/operating region, loss/failure mode, and the trade-off. Use the same structure in software answers.

---

## Digital Electronics

### Q. Combinational vs Sequential circuits?
- **Combinational:** Output depends ONLY on current input. No memory. (Gates, Mux, Decoders, Adders)
- **Sequential:** Output depends on current input AND previous state. Has memory/clocks. (Flip-flops, Registers, Counters)
- **CS Bridge:** Sequential circuits = foundation of RAM and CPU registers.

### Q. What's a Multiplexer?
**A:** Data selector. Multiple inputs → routes one to output based on select lines. Hardware `switch` statement.

---

## Signals & Systems

### Q. What does Fourier Transform do?
**A:** Decomposes time-domain signal into constituent frequencies. Amplitude-over-time → amplitude-over-frequency.
**AI Bridge:** Can't train neural nets on raw audio waveforms. FFT → spectrogram → CNN.

### Q. Nyquist Sampling Theorem?
**A:** To reconstruct a continuous signal from samples: $f_s \ge 2f_{max}$.

### Q. Convolution?
**A:** Math operation on two functions producing a third showing how one modifies the other.
**AI Bridge:** In CNNs (YOLOv8/Focus Lock), convolution = sliding filter over image to extract features like edges and textures.

---

## Control Systems

### Q. Open-loop vs Closed-loop?
- **Open:** Output has no effect on control (toaster runs 2 min regardless).
- **Closed:** Uses feedback to adjust (AC stops when room hits target temp).

### Q. PID Controller?
**A:** Proportional (reacts to current error) + Integral (accumulation of past errors, fixes steady-state) + Derivative (rate of change, dampens oscillations).
**Resume Bridge:** ArduPilot ROVs use PID loops for stability despite water currents.
