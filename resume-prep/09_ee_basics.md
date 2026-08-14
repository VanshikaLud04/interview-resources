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
