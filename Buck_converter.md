# ⚡ Buck Converter Simulation in LTspice

> Design and transient simulation of a step-down (buck) DC-DC converter using a MOSFET switch, Schottky diode, LC filter and resistive load — **12 V input, 35 % duty cycle, 2.5 kHz switching frequency.**

**Author:** Arpit Jain
**Tool:** LTspice XVII / 24
**Analysis:** Transient (`.tran 5m`)

---

## 📑 Table of Contents

1. [Introduction](#-introduction)
2. [Buck Converter Theory](#-buck-converter-theory)
3. [Circuit Description](#-circuit-description)
4. [Design Parameters](#-design-parameters)
5. [Simulation Results](#-simulation-results)
   - [Result 1 – Gate Pulse & Output Voltage](#result-1--gate-pulse--output-voltage)
   - [Result 2 – Inductor Current](#result-2--inductor-current)
   - [Result 3 – Output Voltage Ripple (Cursor Measurement)](#result-3--output-voltage-ripple-cursor-measurement)
   - [Result 4 – Zoomed View of One Switching Cycle](#result-4--zoomed-view-of-one-switching-cycle)
6. [Result Summary Table](#-result-summary-table)
7. [Discussion: Why Is Vout Higher Than D × Vin?](#-discussion-why-is-vout-higher-than-d--vin)
8. [How to Run the Simulation](#-how-to-run-the-simulation)
9. [Repository Structure](#-repository-structure)
10. [Possible Improvements](#-possible-improvements)
11. [Conclusion](#-conclusion)

---

## 📌 Introduction

A **buck converter** is a switching DC-DC regulator that produces an output voltage **lower** than its input voltage with high efficiency. It is widely used in battery-powered devices, microcontroller supplies, LED drivers, and point-of-load regulators.

This project simulates a buck converter in **LTspice** and studies:

- Gate drive and output voltage behaviour
- Inductor current waveform
- Output voltage ripple
- Start-up transient and steady-state operation

---

## 📖 Buck Converter Theory

### Working principle

A buck converter has two operating states in every switching period **T**:

| State | Switch (M1) | Diode (D1) | What happens |
|-------|-------------|------------|--------------|
| **ON** (0 → D·T) | Closed | Reverse biased (off) | Source feeds the inductor and load. Inductor current rises and energy is stored in the magnetic field. |
| **OFF** (D·T → T) | Open | Forward biased (on) | Inductor keeps current flowing through the diode (freewheeling). Stored energy is released to the load and current falls. |

The capacitor **C1** smooths the pulsating inductor current so the load sees an almost-DC voltage.

### Key equations

**Ideal output voltage (continuous conduction mode, CCM):**

```
Vout = D × Vin
```

**Duty cycle:**

```
D = Ton / T
```

**Inductor current ripple (CCM):**

```
ΔIL = (Vin − Vout) × D × T / L
```

**Output voltage ripple (CCM):**

```
ΔVout = ΔIL / (8 × f × C)
```

**Boundary between CCM and DCM (critical inductance):**

```
L_crit = (1 − D) × R × T / 2
```

If `L < L_crit`, the inductor current falls to zero during each cycle and the converter works in **Discontinuous Conduction Mode (DCM)**, where `Vout ≠ D × Vin`.

---

## 🔌 Circuit Description

![Buck converter schematic](images/01_schematic.png)

*Figure 1 – LTspice schematic of the buck converter.*

| Component | Part / Value | Role |
|-----------|--------------|------|
| **Vin** | 12 V DC | Input source |
| **M1** | Si7336ADP (N-channel MOSFET) | High-side switch |
| **V1** | PULSE(0 10 0 1u 1u 0.14m 0.4m) | Gate drive signal |
| **D1** | 1N5819 (Schottky) | Freewheeling diode |
| **L2** | 100 µH | Energy-storage inductor |
| **C1** | 100 µF | Output filter capacitor |
| **R1** | 10 Ω | Load resistor |

**Gate pulse definition:** `PULSE(Vinitial=0 Von=10 Tdelay=0 Trise=1u Tfall=1u Ton=0.14m Tperiod=0.4m)`

---

## 🧮 Design Parameters

| Parameter | Value |
|-----------|-------|
| Input voltage (Vin) | 12 V |
| Gate drive amplitude | 10 V |
| Switching period (T) | 0.4 ms |
| Switching frequency (f) | 2.5 kHz |
| On-time (Ton) | 0.14 ms |
| Duty cycle (D) | 0.14 / 0.4 = **0.35 (35 %)** |
| Inductor (L) | 100 µH |
| Capacitor (C) | 100 µF |
| Load (R) | 10 Ω |
| Ideal Vout (D × Vin) | 12 × 0.35 = **4.2 V** |
| Critical inductance (L_crit) | 0.65 × 10 × 0.4 ms / 2 = **1.3 mH** |
| Simulation time | 5 ms |

> ⚠️ Since L = 100 µH is far below L_crit = 1.3 mH, the converter is expected to operate in **DCM** (confirmed in Result 2).

---

## 📊 Simulation Results

### Result 1 – Gate Pulse & Output Voltage

![Gate pulse and output voltage](images/02_gate_and_output_voltage.png)

*Figure 2 – Gate drive V(n004) in green and output voltage V(n003) in magenta, 0 – 5 ms.*

**Explanation**

- The **green trace** is the 0 – 10 V PWM gate signal with a 0.4 ms period (2.5 kHz), giving 12 – 13 pulses in 5 ms.
- The **magenta trace** is the output voltage. It rises from 0 V during start-up, **overshoots to about 8.1 V** at roughly 0.2 ms, then settles into a steady ripple by around 1 ms.
- In steady state the output swings between approximately **5.1 V and 6.5 V**, with an average near **5.7 V**.
- The output rises while the switch is on (inductor charging the capacitor) and falls while the switch is off (capacitor discharging into the load) — the characteristic **sawtooth-like ripple** of a buck converter.

---

### Result 2 – Inductor Current

![Inductor current](images/03_inductor_current.png)

*Figure 3 – Inductor current I(L2), 0 – 5 ms.*

**Explanation**

- At start-up the capacitor is empty, so the inductor current surges to a **peak of about 7 A** in the first pulse (inrush while charging C1).
- In steady state, the current in each cycle **ramps up to about 2.3 – 2.4 A** and then **falls back to 0 A**, staying at zero for part of the period.
- This "triangle then flat zero" shape proves the converter is operating in **Discontinuous Conduction Mode (DCM)**, as predicted by `L < L_crit`.
- Average inductor current ≈ ½ × 2.4 A × (≈0.2 ms / 0.4 ms) ≈ **0.6 A**, which agrees with the load current `Vout / R ≈ 5.7 / 10 ≈ 0.57 A`.

---

### Result 3 – Output Voltage Ripple (Cursor Measurement)

![Output ripple with cursors](images/04_output_ripple_cursors.png)

*Figure 4 – Output voltage V(n003) with cursors placed on one minimum and one maximum.*

| Measurement | Cursor 1 (min) | Cursor 2 (max) | Difference |
|-------------|----------------|----------------|------------|
| Time | 2.4234 ms | 2.5711 ms | 147.7 µs |
| V(n003) | 5.0879 V | 6.4598 V | **1.3718 V** |

**Explanation**

- The cursors give a **peak-to-peak output ripple of ≈ 1.37 V**.
- The rising interval between minimum and maximum is ≈ **148 µs**, which is close to the 140 µs on-time of the switch — the output climbs only while energy is being delivered.
- *(Note: the 6.77 kHz "frequency" displayed in the cursor box is just the inverse of the cursor spacing, not the switching frequency. The true switching frequency is 2.5 kHz.)*
- The ripple is large (~24 % of the average output) because C1 is relatively small and the switching frequency is low.

---

### Result 4 – Zoomed View of One Switching Cycle

![Zoomed output rise](images/05_output_zoomed_rise.png)

*Figure 5 – Zoomed plot of V(n003) between 2.40 ms and 2.56 ms.*

| Measurement | Cursor 1 | Cursor 2 | Difference |
|-------------|----------|----------|------------|
| Time | 2.4216 ms | 2.5674 ms | 145.8 µs |
| V(n003) | 5.0878 V | 6.4695 V | **1.3817 V** |

**Explanation**

- Zooming in shows the output sitting flat at its minimum (≈ 5.09 V), then rising smoothly to its maximum (≈ 6.47 V) over ≈ 146 µs, and beginning to fall again.
- The flat start corresponds to the interval where the inductor current is zero and the capacitor alone supplies the load.
- The rise has a smooth, rounded profile because the inductor current ramps linearly and is integrated by the capacitor.
- The result (1.38 V ripple) matches the earlier measurement (1.37 V), confirming consistent steady-state behaviour.

---

## 📋 Result Summary Table

| Quantity | Value |
|----------|-------|
| Switching frequency | 2.5 kHz |
| Duty cycle | 35 % |
| Ideal Vout (CCM formula) | 4.2 V |
| **Simulated Vout (average)** | **≈ 5.7 V** |
| Output voltage maximum | ≈ 6.47 V |
| Output voltage minimum | ≈ 5.09 V |
| **Peak-to-peak ripple** | **≈ 1.37 – 1.38 V** |
| Start-up overshoot | ≈ 8.1 V |
| Inductor peak current (start-up) | ≈ 7 A |
| Inductor peak current (steady state) | ≈ 2.4 A |
| Average load current | ≈ 0.57 A |
| Conduction mode | **DCM** |

---

## 🔍 Discussion: Why Is Vout Higher Than D × Vin?

The simulated average output (~5.7 V) is higher than the ideal CCM value of 4.2 V. Two effects explain this:

1. **Discontinuous conduction mode.** The CCM formula `Vout = D × Vin` only holds when inductor current never reaches zero. In DCM the output voltage becomes higher than `D × Vin` for the same duty cycle, since the load gets energy from the inductor and capacitor without the diode clamping the switch node for the full off-time.

2. **Gate drive limitation of the high-side N-MOSFET.** The gate source V1 is referenced to ground, so the MOSFET only conducts while `Vgs = 10 V − Vsource` stays above the threshold voltage. The switch node therefore cannot rise all the way to 12 V; it is limited to roughly 8 V, which is consistent with the ≈ 8.1 V start-up peak seen in Figure 2. The *effective* input is lower than 12 V, which pulls the result down from the DCM value you would get with a full 12 V.

The final ≈ 5.7 V is the combined outcome of these effects plus small diode and MOSFET conduction losses. (Treat this explanation as the likely cause; it can be verified by probing the switch node voltage `V(n001)` or by using a floating/bootstrapped gate drive and comparing.)

---

## ▶️ How to Run the Simulation

1. Install [LTspice](https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html).
2. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/<your-repo-name>.git
   ```
3. Open `Buck_Converter.asc` in LTspice.
4. Click **Run** (▶) — the `.tran 5m` directive is already set.
5. Click on nodes/components to plot:
   - `V(n004)` – gate pulse
   - `V(n003)` – output voltage
   - `I(L2)` – inductor current
6. Use **Ctrl + left-click** on the waveform name to place measurement cursors.

---

## 🗂 Repository Structure

```
Buck-Converter-LTspice/
│
├── README.md
├── Buck_Converter.asc          # LTspice schematic
├── images/
│   ├── 01_schematic.png
│   ├── 02_gate_and_output_voltage.png
│   ├── 03_inductor_current.png
│   ├── 04_output_ripple_cursors.png
│   └── 05_output_zoomed_rise.png
└── LICENSE
```

---

## 🚀 Possible Improvements

- **Increase L to ≥ 1.3 mH** so the converter runs in CCM and `Vout` approaches the expected 4.2 V.
- **Increase C1 or switching frequency** (e.g. 50 – 100 kHz) to reduce the ~1.4 V ripple dramatically.
- **Use a proper gate driver** (bootstrap or P-channel high-side switch) so the MOSFET can switch the full 12 V.
- **Add closed-loop control** (PWM controller with feedback) to regulate the output at a fixed target voltage.
- **Replace the diode with a synchronous MOSFET** to improve efficiency.
- Add **efficiency, load-step and line-step** analyses.

---

## ✅ Conclusion

A buck converter was designed and simulated in LTspice with Vin = 12 V, D = 35 %, f = 2.5 kHz, L = 100 µH, C = 100 µF and R = 10 Ω. The simulation demonstrated correct step-down behaviour: the 12 V input was converted to an average output of about **5.7 V**, with a **1.37 V peak-to-peak ripple** and a start-up overshoot of about 8.1 V. The inductor current showed a 2.4 A steady-state peak and returned to zero each cycle, confirming **discontinuous conduction mode**, which is expected since the 100 µH inductor is well below the 1.3 mH critical value.

The output was higher than the ideal `D × Vin = 4.2 V` because of DCM operation and the limited gate drive of the ground-referenced high-side MOSFET. The results also show that a low switching frequency and small filter components give large ripple. For a practical design, increasing the inductance, raising the switching frequency, adding a proper gate driver and using closed-loop control would give a lower-ripple, well-regulated output.

Overall, the project verifies the fundamental operation of the buck converter — switching, energy storage in the inductor, freewheeling through the diode and smoothing by the output capacitor — and highlights the key design trade-offs among inductance, capacitance, frequency and conduction mode.

---

## 👤 Author

**Arpit Jain**
*Power Electronics – Buck Converter Simulation Project*

⭐ If you find this useful, consider starring the repository!
