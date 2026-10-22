# 01 — Background, Basics and Analysis Tools (L1)

> **Source:** L01 slides. Most of L1 is motivation (EVs, wind, chargers). The examinable content is the **analysis toolkit** below, and you will use it in every later topic.

---

## 1. Why switch-mode at all?

You want to get 5 V down to 1 V.

| Approach | How | Efficiency |
|---|---|---|
| **Linear** (series resistor / transistor) | Drop 4 V across a lossy element | $\eta = V_\text{out}/V_\text{in} = 20\%$ |
| **Switch-mode** | Chop $V_\text{in}$ with a switch, then low-pass filter the result | Ideally 100% |

An ideal switch dissipates no power: when it is **on**, $v = 0$; when it is **off**, $i = 0$. So $p = vi = 0$ at all times. All the loss in real converters comes from non-idealities (see [09_device_losses.md](09_device_losses.md)).

**The four conversion types**

| Conversion | Name |
|---|---|
| DC → DC | DC-DC converter (buck, boost, buck-boost, flyback, LLC…) |
| DC → AC | **Inverter** |
| AC → DC | **Rectifier** |
| AC → AC | AC-DC-AC (e.g. wind turbine: rectifier + DC link + inverter) |

**Design challenges** (good for "discuss" questions): efficiency/loss management (devices are temperature sensitive), power density (kW/kg, kW/m³), controllability, reliability, cost.

---

## 2. The core idea: switching function + low-pass filter

Model the switch as a **switching function** $s(t) \in \{0, 1\}$. The buck converter's switch node is $s(t)\,V_\text{in}$, and the LC network acts as a voltage divider:

$$
V_\text{out}(j\omega) = \underbrace{\frac{\dfrac{R_\text{load}}{1 + j\omega C R_\text{load}}}{\dfrac{R_\text{load}}{1 + j\omega C R_\text{load}} + j\omega L}}_{\text{LC low-pass divider}} \cdot\; s\,V_\text{in}
$$

- At DC ($\omega = 0$) the divider equals 1, so $\langle V_\text{out}\rangle = \langle s \rangle V_\text{in} = D\,V_\text{in}$.
- At the switching frequency and its harmonics the divider is ≪ 1, so the ripple is filtered out.

**Duty ratio**

$$
D = \frac{t_\text{on}}{T_s}, \qquad t_\text{off} = (1-D)T_s, \qquad f_\text{sw} = \frac{1}{T_s}
$$

---

## 3. Average and RMS

$$
F_0 = \frac{1}{T}\int_0^T f(t)\,dt \qquad\text{(average — used for DC quantities)}
$$

$$
F = \sqrt{\frac{1}{T}\int_0^T f(t)^2\,dt} \qquad\text{(RMS — used for AC quantities and for } I^2R \text{ loss)}
$$

**Handy RMS values** (peak $A$, symmetric about zero; from L14):

| Waveform | RMS |
|---|---|
| Sine | $A/\sqrt2$ |
| Square | $A$ |
| Triangle | $A/\sqrt3$ |
| Sawtooth | $A/\sqrt3$ |

**DC + AC rule** (used constantly in loss calculations): if $g$ is a constant and $f$ has zero mean over $T$, the cross term integrates to zero, so

$$
\text{RMS}_{DC+AC} = \sqrt{DC^2 + \text{RMS}_{AC}^2}
$$

---

## 4. Harmonics, THD and power factor

Write $f(t) = f_1(t) + f_\text{dis}(t)$, where $f_1$ is the fundamental and $f_\text{dis}$ is everything else (DC and all $h \neq 1$). The harmonics are orthogonal, so their RMS values add in quadrature:

$$
F^2 = F_1^2 + \sum_{h\neq1} F_h^2, \qquad F_\text{dis} = \sqrt{F^2 - F_1^2}
$$

$$
\boxed{\text{THD} = \frac{F_\text{dis}}{F_1} = \frac{\sqrt{F^2 - F_1^2}}{F_1} = \sqrt{\sum_{h\neq 1}\left(\frac{F_h}{F_1}\right)^2}}
$$

**Power factor.** Only the fundamentals of $v$ and $i$ at the same frequency carry real power (assumed here). So

$$
\text{PF} = \frac{P}{V_\text{rms} I_\text{rms}} = \frac{V_1 I_1 \cos\phi_1}{V I}
= \frac{\cos\phi_1}{\sqrt{(1+\text{THD}_v^2)(1+\text{THD}_i^2)}}
$$

- With no harmonics, $\text{PF} = \cos\phi_1$ (the *displacement* factor).
- With a stiff sinusoidal grid ($\text{THD}_v = 0$): $\text{PF} = \dfrac{\cos\phi_1}{\sqrt{1+\text{THD}_i^2}}$. Distorted current alone lowers the PF even when it is perfectly in phase. This is the reason rectifiers with capacitor filters have a "poor power factor" (see [06_rectifiers.md](06_rectifiers.md)).

---

## 5. The two steady-state laws ⭐

These two laws are the basis of almost every derivation in the course.

### Inductor volt-second balance

$v_L = L\,\dfrac{di_L}{dt}$. In steady state $i_L(t+T) = i_L(t)$, so

$$
\boxed{\frac{1}{T}\int_{t_1}^{t_1+T} v_L\,dt = 0}
$$

**The average inductor voltage is zero.** On a $v_L$ plot, the positive area A equals the negative area B; equivalently, the net change in flux over a period is zero.

### Capacitor charge balance

$i_C = C\,\dfrac{dv_C}{dt}$. In steady state $v_C(t+T) = v_C(t)$, so

$$
\boxed{\frac{1}{T}\int_{t_1}^{t_1+T} i_C\,dt = 0}
$$

**The average capacitor current is zero.** The net charge over a period is zero.

**How to use them:**

- **Volt-second balance gives the voltage ratio $M(D)$.**
- **Charge balance gives the current relationships** (e.g. average inductor current = load current in a buck).

The **small-ripple approximation** goes with these. Treat $V_o$ and $V_d$ as constant within a period, so $v_L$ is piecewise constant and $i_L$ is piecewise linear:

$$
v_L = L\frac{di_L}{dt} \approx L\frac{\Delta i_L}{\Delta t}
$$

---

## 6. Power semiconductor devices (overview)

| Device | Type | Controlled? | Conducts | Blocks |
|---|---|---|---|---|
| **Diode** | Passive | No: turns on when forward biased, off when current tries to reverse | A → K only | Reverse voltage |
| **IGBT** | Active ($v_{GE}$ controls it) | On and off | C → E only | Forward $v_{CE}$ only. Needs an **anti-parallel free-wheeling diode (FWD)** to carry reverse current |
| **MOSFET** | Active ($v_{GS}$ controls it) | On and off | D → S **and** S → D when on (resistive channel) | Forward $v_{DS}$. When off, reverse current flows through the **body diode** |

Ideal switch: ON means $v = 0$ and current flows; OFF means $i = 0$ and voltage is blocked. The device ratings are the **max blocking voltage**, the **max on-state current** and the switching frequency (kHz–MHz).

More device physics is in [04_inverters_single_phase.md §6](04_inverters_single_phase.md#6-devices-fet-igbt-wide-bandgap-l4).

> **PLECS** simulation is recommended for checking examples-paper work, but it is **not examinable**. The exam is on paper.
