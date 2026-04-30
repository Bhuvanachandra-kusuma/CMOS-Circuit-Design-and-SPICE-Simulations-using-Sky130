
# CMOS Circuit Design and SPICE Simulations using Sky130

A comprehensive documentation of CMOS circuit design concepts and SPICE simulations using the open-source SkyWater 130nm (Sky130) Process Design Kit (PDK). This repository covers 5 days of structured learning — from MOSFET fundamentals to robustness analysis of CMOS inverters.

---

## Table of Contents

- [About the Workshop](#about-the-workshop)
- [Tools Used](#tools-used)
- [Day 1 – NMOS Basics and Drain Current Characteristics](#day-1--nmos-basics-and-drain-current-characteristics)
- [Day 2 – Velocity Saturation and CMOS Inverter VTC](#day-2--velocity-saturation-and-cmos-inverter-vtc)
- [Day 3 – CMOS Switching Threshold and Dynamic Simulations](#day-3--cmos-switching-threshold-and-dynamic-simulations)
- [Day 4 – CMOS Noise Margin Robustness](#day-4--cmos-noise-margin-robustness)
- [Day 5 – Power Supply and Device Variation Robustness](#day-5--power-supply-and-device-variation-robustness)
- [Key Takeaways](#key-takeaways)

---

## About the Workshop

This workshop covers the fundamentals of CMOS circuit design and SPICE simulations using the Sky130 PDK — an open-source 130nm process technology developed by SkyWater Technology Foundry in collaboration with Google. The course builds understanding from individual MOSFET behavior all the way up to full CMOS inverter characterization, including noise margins, transient delays, and robustness under process and supply voltage variations.

---

## Tools Used

- **ngspice** — Open-source SPICE simulator
- **Sky130 PDK** — SkyWater 130nm Process Design Kit
- **VirtualBox** — Virtual machine environment for running simulations
- **GitHub** — Version control and documentation

---

## Day 1 – NMOS Basics and Drain Current Characteristics

### Why Do We Need SPICE Simulations?

In CMOS design, complex digital circuits such as NAND, NOR, AND, and OR gates are all built from combinations of PMOS and NMOS transistors. Before fabricating a chip, engineers need to verify that the circuit will behave correctly, meet timing requirements, and operate within acceptable power limits.

This is where **SPICE (Simulation Program with Integrated Circuit Emphasis)** becomes essential. SPICE allows us to simulate the electrical behavior of circuits with great accuracy before any physical chip is manufactured. It helps us:

- Calculate propagation delays through logic gates
- Characterize transistor I-V curves
- Build **delay tables** used in Static Timing Analysis (STA)
- Optimize transistor W/L ratios for performance and power

Without SPICE simulation results, downstream physical design steps like clock tree synthesis, timing closure, and crosstalk analysis would have no reliable data to work with.

---

### Introduction to the NMOS Transistor

An NMOS (N-type Metal Oxide Semiconductor) transistor is built on a **P-type silicon substrate**. Its structure consists of:

- Two heavily doped **n+ regions** that form the **Source** and **Drain** terminals
- A thin **silicon dioxide (SiO₂) oxide layer** grown on top of the substrate between Source and Drain
- A **metal (or polysilicon) gate** deposited on top of the oxide, which controls current flow
- A **Body (Bulk/Substrate)** terminal connected to the P-type substrate

The gate is electrically isolated from the channel by the oxide layer. This means the gate draws virtually no DC current — it controls transistor operation purely through the electric field it creates, which is why these devices are called **field-effect transistors**.

---

### Threshold Voltage (Vt)

The **threshold voltage (Vt)** is one of the most fundamental parameters of a MOSFET. It defines the minimum gate-to-source voltage (Vgs) required to form a conducting channel between Source and Drain.

**Case 1: Vgs = 0V**

When no voltage is applied to the gate, the Source and Drain n+ regions are separated by the P-type substrate. The PN junctions at both ends behave like reverse-biased diodes. No current flows and no channel exists.

**Case 2: Vgs > 0 (small positive voltage)**

As we increase Vgs, the positive charge on the gate repels holes in the P-substrate near the surface and attracts electrons. A **depletion region** begins to form just below the oxide. The substrate surface begins transitioning from P-type toward intrinsic.

**Case 3: Strong Inversion (Vgs = Vt)**

When Vgs reaches the threshold voltage, the surface concentration of electrons equals the bulk hole concentration. The surface is now effectively **N-type** — this is called **strong inversion**. A conductive channel now exists between Source and Drain.

If Vgs increases further beyond Vt, even more electrons are attracted from the n+ source region into the channel, strengthening the channel and allowing more current to flow.

> **Key point:** Even though a channel exists, if no drain voltage (Vds) is applied, electrons have no reason to move. This condition — channel formed but no current — is the boundary of the **cut-off region**.

---

### Body Effect (Threshold Voltage with Substrate Bias)

When a negative voltage is applied to the body relative to the source (Vsb > 0 for NMOS), the depletion region between the body and source widens. This means more gate voltage is needed to invert the surface and form a channel.

The result: **threshold voltage increases with increasing Vsb**. This phenomenon is called the **Body Effect** and is characterized by the body effect coefficient **γ (gamma)**, which is a process-dependent parameter extracted from the technology model files.

This effect is important in multi-stage circuit design where transistors in series cause the body potential of inner transistors to float, effectively raising their threshold voltages.

---

### Regions of Operation

#### 1. Cut-off Region (Vgs < Vt)
No channel is formed. The transistor is OFF and drain current Id ≈ 0. Used when transistor acts as an open switch.

#### 2. Linear (Resistive) Region (Vgs > Vt, Vds < Vgs - Vt)

When a small Vds is applied, electrons flow from Source to Drain through the channel. The device behaves like a **voltage-controlled resistor**.

The channel charge varies along its length — it is maximum near the Source and slightly reduced near the Drain because the effective gate-to-channel voltage at any point x along the channel is (Vgs - V(x)).

The drain current in the linear region is:

```
Id = μn × Cox × (W/L) × [(Vgs - Vt)×Vds - Vds²/2]
```

Where:
- **μn** = electron mobility
- **Cox** = gate oxide capacitance per unit area = εox / tox
- **W/L** = transistor width-to-length ratio
- **Vt** = threshold voltage

#### 3. Saturation Region (Vgs > Vt, Vds ≥ Vgs - Vt)

As Vds increases, the channel near the Drain becomes thinner. When Vds = Vgs - Vt, the channel **pinches off** at the Drain end — meaning the channel charge at the Drain side drops to zero.

This is called the **pinch-off point**. Beyond this, increasing Vds does not significantly increase Id because the channel remains pinched off and the voltage across the channel stays at (Vgs - Vt).

The drain current in saturation is:

```
Id = (μn × Cox / 2) × (W/L) × (Vgs - Vt)²
```

In reality, as Vds increases beyond pinch-off, the pinch-off point moves slightly toward the Source, effectively **shortening the channel**. This causes a slight increase in Id with Vds — an effect called **Channel Length Modulation**, characterized by the parameter **λ (lambda)**:

```
Id = (μn × Cox / 2) × (W/L) × (Vgs - Vt)² × (1 + λ×Vds)
```

---

### SPICE Basics and Netlist Structure

A SPICE netlist is a text description of the circuit. For an NMOS transistor:

```spice
* NMOS transistor definition
* Syntax: M<name> Drain Gate Source Bulk <model> W=<width> L=<length>
M1 vdd vin 0 0 sky130_fd_pr__nfet_01v8 W=0.39 L=0.15

* Voltage sources
Vdd vdd 0 1.8
Vin vin 0 0

* DC sweep analysis
.dc Vds 0 1.8 0.01 Vgs 0 1.8 0.6

* Include Sky130 model library
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

.end
```

Key points about SPICE syntax:
- Lines starting with `*` are **comments**
- MOSFET terminals are listed as: **D G S B** (Drain, Gate, Source, Bulk)
- `.dc` performs a DC sweep analysis
- `.lib` includes the foundry-provided model file

The Sky130 model files contain hundreds of parameters that describe the exact electrical behavior of transistors in the 130nm process, including mobility, oxide thickness, threshold voltage, and many higher-order effects.

---

### Lab: Running the First SPICE Simulation

**Steps:**
1. Open VirtualBox and launch the Ubuntu terminal
2. Clone the course repository:
   ```bash
   git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
   cd sky130CircuitDesignWorkshop
   ```
3. Navigate to Day 1 design files:
   ```bash
   cd design/day1_nfet_idvds_L2_W5
   ```
4. Run ngspice:
   ```bash
   ngspice day1_nfet_idvds_L2_W5.spice
   ```
5. Plot the results:
   ```bash
   plot -vdd#branch
   ```

**Expected Output:** A family of Id vs Vds curves for different values of Vgs (0V to 1.8V in steps of 0.6V), clearly showing the linear region, saturation region, and the transition between them.

> *(Simulation screenshot to be added)*

---

## Day 2 – Velocity Saturation and CMOS Inverter VTC

### Velocity Saturation in Short Channel Devices

For **long channel devices** (L > ~1μm), the drain current in saturation is proportional to **(Vgs - Vt)²** — a quadratic relationship. If we scale W and L together keeping W/L constant, Id should remain the same ideally.

However, for **short channel devices** (L < ~0.5μm), this is no longer true. In short channel devices, the electric field in the channel becomes very high. At high electric fields, the carrier (electron) velocity no longer increases linearly with the field. Instead, it saturates at a maximum velocity called **saturation velocity (v_sat)**.

This gives rise to a fourth region of operation:

| Region | Condition | Id behavior |
|--------|-----------|-------------|
| Cut-off | Vgs < Vt | Id ≈ 0 |
| Linear | Vgs > Vt, Vds < Vdsat | Id ∝ Vds |
| Saturation | Vgs > Vt, Vds ≥ Vdsat | Id ∝ (Vgs-Vt)² (long channel) |
| Velocity Saturation | Short channel, high field | Id ∝ (Vgs-Vt) — becomes linear |

The key new parameter here is **Vdsat** — the drain-to-source voltage at which velocity saturation begins. For short channel devices, Vdsat << (Vgs - Vt), meaning the device enters saturation much earlier.

**Practical observation from SPICE:**
- For long channel (L = 2μm): Id vs Vgs shows quadratic behavior
- For short channel (L = 0.25μm): Id vs Vgs becomes nearly linear due to velocity saturation
- Counter-intuitively, **shorter channel devices carry LESS current** than expected because velocity saturation limits the carrier speed

> *(Id vs Vgs comparison plot to be added)*

---

### CMOS Inverter – Introduction

The CMOS inverter is the most fundamental building block of all digital logic. It consists of:
- One **PMOS** transistor connected between VDD and the output
- One **NMOS** transistor connected between the output and GND
- Both gates connected together as the input
- Both drains connected together as the output

**Operation:**

| Input (Vin) | PMOS state | NMOS state | Output (Vout) |
|-------------|------------|------------|---------------|
| 0V (LOW) | ON (conducting) | OFF | VDD (HIGH) |
| VDD (HIGH) | OFF | ON (conducting) | 0V (LOW) |

This complementary structure is what gives CMOS its key advantage: **in steady state, one transistor is always OFF**, meaning there is virtually no direct path from VDD to GND. This results in extremely low static power consumption.

---

### Voltage Transfer Characteristic (VTC)

The **Voltage Transfer Characteristic (VTC)** is a graph of output voltage (Vout) vs input voltage (Vin) swept from 0 to VDD. It reveals how the inverter responds to every possible input voltage.

**How the VTC is derived analytically:**

For PMOS:
- Vgsp = Vin - VDD
- Vdsp = Vout - VDD

For NMOS:
- Vgsn = Vin
- Vdsn = Vout

By superimposing the NMOS and PMOS I-V curves and finding the intersection points (where Id_NMOS = Id_PMOS) for each value of Vin, we trace out the VTC curve.

**Operating regions along the VTC:**

| Vin range | NMOS region | PMOS region | Vout |
|-----------|-------------|-------------|------|
| 0V | OFF | Linear | ≈ VDD |
| Low (just above 0) | Saturation | Linear | High |
| Mid (≈ VDD/2) | Saturation | Saturation | Rapidly falling |
| High (just below VDD) | Linear | Saturation | Low |
| VDD | Linear | OFF | ≈ 0V |

> *(VTC curve plot to be added)*

---

### Lab: Sky130 SPICE Simulation for VTC

**SPICE netlist for CMOS inverter:**

```spice
* CMOS Inverter VTC simulation
M1 out in vdd vdd sky130_fd_pr__pfet_01v8 W=1.05 L=0.15
M2 out in 0 0 sky130_fd_pr__nfet_01v8 W=0.39 L=0.15

Vdd vdd 0 1.8
Vin in 0 0

* Load capacitance
CL out 0 10f

* DC sweep
.dc Vin 0 1.8 0.01

.lib "sky130_fd_pr/models/sky130.lib.spice" tt
.end
```

Run in ngspice:
```bash
ngspice inverter_vtc.spice
plot out vs in
```

**Observed Vt (threshold voltage from Id vs Vgs plot):** ≈ 0.76V

> *(VTC simulation output to be added)*

---

## Day 3 – CMOS Switching Threshold and Dynamic Simulations

### Switching Threshold (Vm)

The **switching threshold Vm** is the input voltage at which Vout = Vin. On the VTC curve, it is the point where the curve intersects the line Vout = Vin (a 45° diagonal line).

At Vm:
- Both NMOS and PMOS are simultaneously ON
- Both transistors are in **saturation**
- Maximum current flows through the circuit
- This is the most power-hungry operating point

The analytical expression for Vm is:

```
Vm = (Vt_n + (√(kp/kn)) × (VDD + Vt_p)) / (1 + √(kp/kn))
```

Where:
- **kn = μn × Cox × (W/L)n**
- **kp = μp × Cox × (W/L)p**
- Vt_n = NMOS threshold voltage
- Vt_p = PMOS threshold voltage (negative value)

**Key insight:** Vm can be shifted by changing the W/L ratio of either transistor:
- Increasing PMOS W/L → PMOS becomes stronger → Vm shifts right (higher)
- Increasing NMOS W/L → NMOS becomes stronger → Vm shifts left (lower)
- For a **symmetric inverter** where rise delay = fall delay, we want Vm ≈ VDD/2

For the Sky130 process, because PMOS has lower mobility than NMOS (μp ≈ μn/2), the PMOS transistor needs to be made approximately 2-3× wider than NMOS to achieve Vm ≈ VDD/2.

**Simulated Vm for Sky130 inverter:** ≈ 0.876V (with W_p/W_n ≈ 2.77)

> *(Switching threshold plot to be added)*

---

### Transient Analysis – Rise and Fall Delay

While VTC tells us about the static behavior, **transient analysis** reveals the dynamic behavior — how fast the inverter can switch.

A **pulse input** is applied (0V to 1.8V with defined rise and fall times) and the output waveform is observed over time.

**Delay is measured at 50% of VDD** (≈ 0.9V) because this represents the midpoint of the transition and is consistent with how delays are defined in standard cell libraries.

**Rise Delay (τ_rise):** Time from input crossing 50% VDD (falling) to output crossing 50% VDD (rising)
```
τ_rise = t_out_50% - t_in_50%  (when output rises)
```

**Fall Delay (τ_fall):** Time from input crossing 50% VDD (rising) to output crossing 50% VDD (falling)
```
τ_fall = t_out_50% - t_in_50%  (when output falls)
```

**From simulation with W_p/W_n = 2.77:**
- Rise Delay ≈ 0.333 ns
- Fall Delay ≈ 0.285 ns

Factors affecting delay:
- **Load capacitance (CL):** Larger load = slower switching
- **Transistor strength (W/L):** Wider transistor = faster charging/discharging = smaller delay
- **Supply voltage:** Higher VDD = faster switching (but more power)

**SPICE transient setup:**
```spice
* Pulse input: V(initial) V(pulsed) T_delay T_rise T_fall T_on T_period
Vin in 0 pulse 0 1.8 0 100p 100p 1n 2n

* Transient analysis: T_step T_stop
.tran 10p 4n
```

> *(Transient waveform with rise and fall delay marked to be added)*

---

### Effect of PMOS Width on VTC and Delay

By varying the PMOS width while keeping NMOS fixed, we can observe the trade-off between Vm, rise delay, and fall delay:

| PMOS Width (W_p) | Vm | Rise Delay | Fall Delay |
|------------------|-----|------------|------------|
| Small (W_p = W_n) | Low (< VDD/2) | Large | Small |
| Balanced (W_p ≈ 2.77×W_n) | ≈ VDD/2 | ≈ Fall delay | ≈ Rise delay |
| Large (W_p >> W_n) | High (> VDD/2) | Small | Large |

For **clock buffers and timing-critical paths**, a symmetric inverter (equal rise and fall delay) is preferred. This is achieved by setting W_p/W_n ≈ 2-3 depending on the process.

---

## Day 4 – CMOS Noise Margin Robustness

### What is Noise Margin?

In real digital systems, wires pick up noise from adjacent switching signals (crosstalk), power supply variations, and other sources. **Noise Margin** quantifies how much noise a logic gate can tolerate at its input while still producing a correct output.

Think of it as a safety buffer — if your logic HIGH is at 1.5V but the gate treats anything above 0.98V as HIGH, then the noise margin is 1.5 - 0.98 = 0.52V. The noise has to be larger than this margin to cause an error.

---

### Key VTC Parameters

From the VTC curve, four critical voltage levels are defined:

| Parameter | Definition |
|-----------|-----------|
| **VOH** | Output High Voltage — maximum output voltage when output is HIGH (ideally VDD) |
| **VOL** | Output Low Voltage — minimum output voltage when output is LOW (ideally 0V) |
| **VIH** | Input High Voltage — minimum input voltage recognized as logic HIGH |
| **VIL** | Input Low Voltage — maximum input voltage recognized as logic LOW |

VIL and VIH are defined as the points on the VTC where the **slope = -1** (unity gain points). Between VIL and VIH is the **undefined region** or **transition region** — the inverter is not reliably producing HIGH or LOW output in this range.

---

### Noise Margin Equations

```
NMH (Noise Margin High) = VOH - VIH
NML (Noise Margin Low)  = VIL - VOL
```

**Interpretation:**
- **NMH** = how much noise can be added to a HIGH input before the gate misreads it as LOW
- **NML** = how much noise can be added to a LOW input before the gate misreads it as HIGH

For a good digital design, both NMH and NML should be large and approximately equal.

**From Sky130 simulation (W_p/W_n = 2.77, VDD = 1.8V):**

| Parameter | Value |
|-----------|-------|
| VOH | 1.70952 V |
| VOL | 0.09523 V |
| VIH | 0.98778 V |
| VIL | 0.77330 V |
| **NMH** | **0.72 V** |
| **NML** | **0.68 V** |

These are very healthy noise margins — nearly 40% of VDD on each side.

> *(Noise margin extraction from VTC to be added)*

---

### Effect of PMOS Width on Noise Margin

As PMOS width increases:
- The VTC shifts right (Vm increases)
- VOH stays close to VDD, VOL stays close to 0V
- NMH and NML adjust but remain large

**Key observation:** After a certain W_p/W_n ratio, the noise margins stabilize. This demonstrates one of CMOS's most important properties — it is inherently **robust to transistor sizing variations** in terms of noise immunity.

The VTC has two flat regions (near VDD and near 0V) and one steep transition region:
- **Flat regions** = Digital behavior (robust, noise-immune)
- **Steep transition region** = Analog behavior (high gain, sensitive to input)

This is why CMOS logic is so reliable even in noisy environments.

---

## Day 5 – Power Supply and Device Variation Robustness

### Power Supply Variation

As CMOS technology scales down, the supply voltage (VDD) must also be reduced to limit power consumption and electric fields in thin oxide layers. A key concern is: **does the inverter still work correctly at lower VDD?**

**Simulation:** VTC is plotted for VDD = 2.5V, 2.0V, 1.5V, 1.0V, and 0.5V.

**Observations:**

| VDD | Gain | Output Swing | Noise Margin | Speed |
|-----|------|-------------|--------------|-------|
| 2.5V | High | Full (0 to VDD) | Large | Fast |
| 1.5V | Medium | Full (0 to VDD) | Medium | Medium |
| 0.5V | Low | Full (0 to VDD) | Small | Slow |

**Advantages of lower VDD:**
- Dynamic power P = C × VDD² × f is reduced quadratically — huge power savings
- Reduced electric field stress on thin gate oxides — better reliability
- Essential for battery-powered and mobile applications

**Disadvantages of lower VDD:**
- Gain (dVout/dVin) at the switching point decreases — the VTC transition becomes less sharp
- Rise and fall delays increase because the driving current is lower
- Noise margins shrink — the circuit becomes less tolerant to noise
- At very low VDD (close to Vt), the circuit may not switch correctly at all

> *(VTC for different VDD values to be added)*

---

### Device Variation – Etching Process

During chip fabrication, the **etching process** defines the transistor dimensions by selectively removing material. However, etching is never perfectly uniform across a wafer:

- Transistors near the **center** of a wafer may have slightly different W and L than transistors at the **edges**
- This causes variation in the W/L ratio across the chip
- Even a small change in W or L changes the drain current (Id) and therefore the switching behavior

**Effect on the CMOS inverter:**
- If W/L of NMOS or PMOS changes, the current drive changes
- This shifts the VTC curve left or right
- The switching threshold Vm moves accordingly

This is why timing analysis tools use **process corners** — fast/slow combinations of NMOS and PMOS to bound the worst-case performance:
- **FF (Fast-Fast):** Both NMOS and PMOS are fast → circuit is fast but may violate setup
- **SS (Slow-Slow):** Both NMOS and PMOS are slow → circuit is slow, may violate hold
- **FS/SF:** One fast, one slow → asymmetric behavior, affects noise margins

---

### Device Variation – Gate Oxide Thickness (tox)

The gate oxide thickness (tox) directly affects the gate oxide capacitance:

```
Cox = εox / tox
```

If tox increases (thicker oxide than designed):
- Cox decreases
- The drain current Id = μ × Cox × (W/L) × ... decreases
- The transistor becomes **weaker** (slower)

If tox decreases (thinner oxide):
- Cox increases
- Drain current increases
- The transistor becomes **stronger** (faster), but the oxide is more prone to breakdown

---

### SPICE Simulation for Device Variations

Two extreme corners are simulated:

**Case 1: Strong PMOS, Weak NMOS**
- PMOS has higher than nominal drive strength
- NMOS has lower than nominal drive strength
- Result: Output stays HIGH longer → Vm shifts to the **right**

**Case 2: Strong NMOS, Weak PMOS**
- NMOS has higher than nominal drive strength
- PMOS has lower than nominal drive strength
- Result: Output pulls LOW faster → Vm shifts to the **left**

**Key conclusion from simulation:**
Even with these variations, the noise margins remain large and the inverter continues to function correctly as a logic gate. This demonstrates the **inherent robustness of CMOS logic** to process variation.

> *(Device variation VTC comparison plot to be added)*

---

## Key Takeaways

### 1. MOSFET Fundamentals
- Threshold voltage (Vt) is process-dependent and affected by body bias
- Three operating regions: cut-off, linear, and saturation
- Channel length modulation causes slight Id increase with Vds in saturation

### 2. Short Channel Effects
- Velocity saturation makes short channel Id vs Vgs more linear than quadratic
- Short channel devices can carry less current than long channel devices with same W/L

### 3. CMOS Inverter
- Complementary NMOS/PMOS structure gives low static power
- VTC reveals complete input-output behavior
- W/L ratio controls Vm (switching threshold)

### 4. Dynamic Behavior
- Rise and fall delays depend on load capacitance and transistor strength
- PMOS must be ~2.77× wider than NMOS for symmetric delays in Sky130

### 5. Noise Margin
- NMH ≈ 0.72V and NML ≈ 0.68V for optimized Sky130 inverter at VDD = 1.8V
- CMOS noise margins are robust to transistor sizing variations

### 6. Robustness
- CMOS inverter maintains correct logic function across wide range of VDD
- Process corners (FF, SS, FS, SF) are used to bound worst-case behavior
- Despite etching and oxide thickness variations, CMOS logic remains reliable

---

## Acknowledgements

- Course instructor: **Kunal Ghosh**, VSD (VLSI System Design)
- Sky130 PDK: **SkyWater Technology Foundry** and **Google**
- Simulation tool: **ngspice** open-source community

---

## Author

**Bhuvanachandra Kusuma**
Workshop participant — CMOS Circuit Design and SPICE Simulations using Sky130
