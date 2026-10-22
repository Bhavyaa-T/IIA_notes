# 04 — Bridge Converters, Single-Phase Inverters and SPWM (L4, L5)

> **Source:** L04 (bridge converter, devices) and L05 (PWM, SPWM).

---

## 1. The half-bridge leg: the basic building block

Two switches (T1 high side, T4 low side), each with an anti-parallel diode, across $V_{dc}$. The midpoint A is the output.

| T1 | T4 | Current path for inductive load $i_L > 0$ (out of A) | $v_A$ |
|---|---|---|---|
| ON | OFF | Through T1 (inductor charging) | $V_{dc}$ |
| OFF | OFF | **Through D4** (free-wheeling) | 0 |
| OFF | ON | (for $i_L<0$) through T4 | 0 |
| OFF | OFF | (for $i_L<0$) **through D1** | $V_{dc}$ |

**Rules:**

- The high and low sides switch **complementarily**.
- They must **never both be ON**: that is a shoot-through, a short circuit of the DC link.
- Both may be OFF. The current then still flows, but through an anti-parallel diode, and the **direction of the current** decides which diode.

### Dead time

Real switches take time to turn off, so a short **dead time** with both switches OFF is inserted at every transition to prevent an accidental overlap (shoot-through). During the dead time the load current free-wheels through a diode. See [09_device_losses.md](09_device_losses.md) for the synchronous-buck version.

---

## 2. H-bridge (full bridge): bipolar switching, square wave

Two legs, A (T1/T4) and B (T3/T2), with $v_{AB} = v_A - v_B$.

**Bipolar switching:** the diagonal pairs switch together.

- T1 + T2 ON gives $v_{AB} = +V_d$.
- T3 + T4 ON gives $v_{AB} = -V_d$.
- Equal positive and negative intervals, so there is **no DC offset**.

### Fourier series of the square wave

It is an odd, half-wave symmetric function ($f(t) = -f(t+T/2)$), so **only odd harmonics** appear:

$$
\boxed{v_{AB} = \frac{4V_d}{\pi}\sum_{n=1,3,5,\ldots}\frac{\sin(n\omega_s t)}{n}}
$$

- RMS of $v_{AB}$: $V_d$ (it is a square wave).
- Fundamental peak: $\hat V_{AB1} = \dfrac{4V_d}{\pi} \approx 1.27V_d$. Fundamental RMS: $\dfrac{4V_d}{\sqrt2\pi} \approx 0.90V_d$.

**THD of the square wave:**

$$
\text{THD} = \frac{\sqrt{V_d^2 - \left(\frac{4V_d}{\sqrt2\pi}\right)^2}}{\frac{4V_d}{\sqrt2\pi}} = \sqrt{\frac{\pi^2}{8}-1} = \boxed{48.3\%}
$$

### Inductive (R-L) load

The current lags the voltage. Just after each voltage reversal the current still flows in the old direction, so it is carried by the **diodes** (e.g. D1/D2 conduct before T1/T2 take over). This returns energy to the DC source. The conduction sequence each half cycle is **diode pair, then transistor pair**, alternating between pairs.

---

## 3. Unipolar switching (quasi-square wave)

The diagonals are **not** synchronised. Both upper switches (T1, T3) ON, or both lower (T4, T2) ON, gives a **zero state**, $v_{AB} = 0$. The output takes three levels: $+V_d$, $0$, $-V_d$.

Insert a zero interval of width $\alpha$ around each zero crossing:

$$
\boxed{v_{AB} = \frac{4V_d}{\pi}\sum_{n=1,3,5,\ldots}\frac{\sin(n\omega t)\cos\!\left(\frac{n\alpha}{2}\right)}{n}}, \qquad 0\le\alpha\le\pi
$$

- Increasing $\alpha$ **reduces the fundamental**, which gives voltage control without PWM.
- *Bonus (not on the slides):* choosing $\alpha$ so that $\cos(n\alpha/2) = 0$ eliminates harmonic $n$. For example, $\alpha = \pi/3$ removes the 3rd harmonic.

---

## 4. Half-bridge inverter

Only one leg. The load connects between A and the **midpoint of a split DC capacitor bank**, so the output is $\pm V_d/2$:

$$
v_{AB} = \frac{2V_d}{\pi}\sum_{n=1,3,5,\ldots}\frac{\sin(n\omega t)}{n}
$$

| Half bridge vs full bridge | |
|---|---|
| ✅ | Fewer switches |
| ❌ | Needs access to the DC midpoint |
| ❌ | **High loss**: the load current flows through the DC capacitors |
| ❌ | **Half the voltage utilisation** of a full bridge |

---

## 5. Pulse-width modulation (PWM)

Compare a **modulating (reference) signal $v_m$** with a **carrier $v_{cr}$** (a triangle or sawtooth with peak 1, i.e. 100%):

$$
v_m > v_{cr} \Rightarrow \text{switch ON}, \qquad v_m < v_{cr} \Rightarrow \text{switch OFF}
$$

- Carrier frequency = **switching frequency** $f_s$ (in linear modulation).
- Frequency of $v_m$ = **output fundamental frequency** $f_1$.
- For a DC-DC buck, $v_m$ is a DC value and $D = v_m$.

### Why sinusoidal PWM?

A square-wave inverter has THD 48.3%, and its lowest harmonics (3rd, 5th, …) sit **close to $f_1$**, so they are very hard to filter. SPWM pushes the unwanted harmonics up to **around $f_s$ and its multiples**, far from $f_1$, where a small LC filter removes them. This is why filters in real converters (inductors and film/electrolytic capacitors) are physically small relative to the power.

### Single-phase bipolar SPWM

Compare $v_m$ (a sine) with one triangle carrier:

- $v_m > v_{cr}$: S1 and S2 ON, so $v_{AB} = +V_d$.
- $v_m < v_{cr}$: S3 and S4 ON, so $v_{AB} = -V_d$.

Two output levels.

### Modulation index (depth)

$$
\boxed{M = \frac{\hat V_{AB1}}{V_d}}, \qquad v_{AB1} = MV_d\sin(2\pi f_1 t)
$$

**Linear modulation:** $0 \le M \le 1$, and the fundamental is proportional to $M$.

**Spectrum for bipolar SPWM** (e.g. $M = 0.8$): the fundamental at $f_1$, then harmonic clusters at $f_s$, $f_s \pm 2f_1$, $2f_s \pm f_1$, $3f_s$, $3f_s \pm 2f_1$, …

### Over-modulation ($M > 1$)

The reference exceeds the carrier peak, so some carrier cycles are skipped. The fundamental grows **non-linearly**, and lower-order harmonics reappear. In the limit $M \to \infty$ the output becomes the square wave, with $\hat V_{AB1} = \frac{4}{\pi}V_d \approx 1.27V_d$.

### Single-phase unipolar SPWM

Each leg has its own comparison: leg A compares $v_m$ and leg B compares $-v_m$ with the same carrier.

- $v_m > v_{cr}$: S1 ON, so $v_{AN} = V_d$. Otherwise S4 ON and $v_{AN} = 0$.
- $-v_m > v_{cr}$: S3 ON, so $v_{BN} = V_d$. Otherwise S2 ON and $v_{BN} = 0$.
- $v_{AB} = v_{AN} - v_{BN}$ takes **three levels** ($+V_d$, 0, $-V_d$).

**Bipolar vs unipolar SPWM.** The slide is a figure only; these are the standard conclusions.

| | Bipolar | Unipolar |
|---|---|---|
| Output levels | 2 ($\pm V_d$) | 3 ($+V_d, 0, -V_d$) |
| Voltage steps | $2V_d$ | $V_d$ (lower $dv/dt$, less ripple) |
| First harmonic cluster | around $f_s$ | around $2f_s$ (the $f_s$ cluster cancels between legs) |
| Effective ripple frequency | $f_s$ | $2f_s$, so a smaller filter is needed |
| Fundamental | $MV_d$ | $MV_d$ (same) |

---

## 6. Devices: FET, IGBT, wide-bandgap (L4)

**MOSFET (field-effect transistor):**

- Current flows drain → source through a channel. The channel is induced in a p-doped region when $V_{GS} > V_{th}$: the gate is a capacitor, and positive $V_{GS}$ attracts electrons that carry current.
- It is off for $V_{GS} < V_{th}$ (typically 0 V or negative).
- It can conduct in reverse too. Most MOSFETs have a **body diode** (source tied to the p-region), which conducts in reverse when the channel is off.
- **Unipolar** device: no stored minority charge, so switching is fast.

**IGBT:**

- Problem with the FET: blocking a high voltage needs a long **drift region** to keep the field below breakdown. That raises the on-resistance and stores charge.
- Fix: add a counter-doped p-layer at the drain. The extra PN junction blocks high voltage and injects carriers (conductivity modulation), so a shorter drift region suffices.
- The result is **bipolar**: a hybrid of a BJT and a FET, with a low on-state voltage at high voltage and current.
- Like all bipolar devices, the stored charge slows switching (tail current).
- It blocks only forward voltage, so it needs an **anti-parallel diode**.

**Wide-bandgap (SiC, GaN) vs Si** have:

- Higher breakdown field (≈10× Si), so a thinner drift region and lower $R_{ds,on}$ at a given voltage.
- Larger bandgap, so they operate at higher temperature.
- Higher electron saturation velocity.
- Higher thermal conductivity (SiC).

The upshot is **much faster switching and lower switching loss**. A SiC FET switches far faster than a Si IGBT, which allows a higher $f_s$ and smaller passives (e.g. the Tesla Model 3 traction inverter uses SiC MOSFETs, with no separate free-wheel diode; the body diode is used).

**Power modules** package several dies (transistor and diode) as a half-bridge or a full three-phase bridge, e.g. the Infineon HybridPACK.
