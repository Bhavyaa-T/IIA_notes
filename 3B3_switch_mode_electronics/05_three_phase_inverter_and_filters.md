# 05 — Three-Phase Inverter, Filters and Tutorial 1 Q1 (L6, L7)

> **Source:** L06 (three-phase inverter, SPWM, superposition, LCL) and L07 Tutorial 1 Q1 (single-phase grid inverter filter design).

---

## 1. Three-phase two-level inverter

Three half-bridge legs (A, B, C) across $V_d$.

- The **DC midpoint "o"** is a *virtual* reference used for analysis. It is not physically accessible.
- The load neutral is normally not available either, so only **line voltages** are physically available.
- Each "switch" can be many devices in series or parallel switching together, and can be a MOSFET or an IGBT. SiC MOSFETs use their body diode, so no separate free-wheeling diode is needed.

---

## 2. Three-phase SPWM

Three sinusoidal references 120° apart, each compared with a **common carrier**.

- **Leg (pole) voltage** $v_{Ao}$ switches between $+V_d/2$ and $-V_d/2$.
- **Line voltage** $v_{AB} = v_{Ao} - v_{Bo}$ takes the levels $+V_d$, 0, $-V_d$.

**Harmonics.** The slides say they are at integer multiples of $f_s$. *More precisely,* they appear as sidebands around $mf_s$. In the line voltage, the components exactly at $mf_s$ (common to all three legs) cancel, leaving sidebands such as $f_s \pm 2f_1$ and $2f_s \pm f_1$.

### Modulation depth

$$
\boxed{M = \frac{\hat V_{Ao}}{V_d/2}}, \qquad 0 < M \le 1 \text{ (linear)}
$$

Line and phase magnitudes are related by $\hat V_{LL} = \sqrt3\,\hat V_{ph}$, so

$$
\boxed{M = \frac{2}{\sqrt3}\frac{\hat V_{AB}}{V_d}} \qquad\Rightarrow\qquad 0 < \hat V_{AB} \le \frac{\sqrt3}{2}V_d \approx 0.866V_d\;\text{(linear limit)}
$$

### Over-modulation range

$$
\frac{\sqrt3}{2}V_d < \hat V_{AB} \le \frac{2\sqrt3}{\pi}V_d \approx 1.10V_d
$$

The upper limit is **six-step (square-wave) operation**: each leg is a square wave of $\pm V_d/2$, so the phase fundamental peak is $\frac{4}{\pi}\frac{V_d}{2}$ and the line peak is $\sqrt3\times$ that.

> Exam trap: $M$ is defined on the **phase** (pole) voltage relative to $V_d/2$. If the question gives a **line** voltage, convert it with $\sqrt3$ first.

---

## 3. Inverter driving an AC load: superposition

**Model:** inverter, series inductor $L$ (a filter, or the motor's leakage inductance), then a sinusoidal EMF $e$ (motor back-EMF or the grid).

Because the circuit is linear, **analyse each frequency separately** and add the results.

### At the fundamental $f_1$

Phasor equation:

$$
\boxed{\bar V_{Ao1} = j\omega_1L\,\bar I_{A1} + \bar E_{A1}}
$$

with

- $e_{An1} = E_{An1}\sin\omega_1t$,
- $v_{Ao1} = V_{Ao1}\sin(\omega_1t - \delta)$, where $\delta$ is the load (power) angle,
- $i_{A1} = I_{A1}\sin(\omega_1t - \phi - \delta)$.

Draw the phasor diagram: $\bar V$ is the vector sum of $\bar E$ and the inductor drop $j\omega L\bar I$, which is perpendicular to $\bar I$.

### At a harmonic $h$ (e.g. $2f_s$)

The EMF is purely sinusoidal at $f_1$, so it is a **short circuit at harmonics**. Only the inductor remains:

$$
\bar V_{Ao,h} = j\omega_hL\,\bar I_{A,h} \;\Rightarrow\; |I_h| = \frac{|V_h|}{\omega_hL}
$$

The harmonic current lags the harmonic voltage by 90°. This is the **basis of every ripple-current / filter calculation**.

---

## 4. LCL filter for grid connection

Inverter → $L_c$ (converter side) → shunt $C$ → $L_g$ (grid side) → grid.

- The **grid** is modelled as an ideal AC source in series with an inductor.
- $L_g$ can be the grid's internal inductance alone or grid plus added inductance. $L_c$ and the capacitors belong to the inverter.
- The capacitors can be connected in **star** or **delta**. Delta needs **less capacitance** for the same effect.

**Star–delta equivalence** (from $Z_Y = Z_\Delta/3$ and $Z = 1/(j\omega C)$):

$$
\boxed{C_\text{star} = 3\,C_\text{delta}\quad\Leftrightarrow\quad C_\text{delta} = \frac{C_\text{star}}{3}}
$$

> ⚠️ **Slide error (L6, last slide):** it prints $C_\text{delta} = 3C_\text{star}$. That is backwards, and it contradicts the slide's own sentence that "delta connection will effectively reduce the required capacitance". Delta capacitors also see the line voltage ($\sqrt3\times$ higher), so they need a higher voltage rating.

---

## 5. Worked example: Tutorial 1 Q1 (single-phase grid inverter) ⭐

> A battery (600 V) feeds a single-phase full-bridge inverter with **bipolar SPWM**, $f_s = 5$ kHz, and an LC filter $L_f$, $C_f$ to a 230 V, 50 Hz grid. The grid has **10% inductive impedance** at 50 Hz, rating 8.5 kW, unity PF at the point of connection, lossless converter. Grid limits at the connection point: **1% ripple voltage** and **10% ripple current**. Find $L_f$, $C_f$ and $M$.

### Step 1: the grid inductance (at 50 Hz)

"10% inductive impedance" means the grid's internal reactance is 10% of the **base impedance**:

$$
Z_\text{base} = \frac{V_g^2}{P} = \frac{230^2}{8500} = 6.22\ \Omega
$$

$$
L_g = \frac{0.1\,Z_\text{base}}{2\pi f_1} = \frac{0.622}{2\pi\cdot50} = 1.98\ \text{mH} \approx 2\ \text{mH}
$$

Rated current: $I_{g1} = P/V_g = 8500/230 = 37$ A.

### Step 2: $C_f$ from the current limit (at 5 kHz)

The lowest harmonic is at $f_s$ = 5 kHz, and the grid voltage is a short circuit there. The ripple current from the inverter splits between $C_f$ and $L_g$. The limit says only 10% may reach the grid, so 90% goes into $C_f$:

$$
\frac{i_{cs}}{i_{gs}} = \frac{|Z_{Lg}|}{|Z_{Cf}|} = \omega_s^2L_gC_f = \frac{90}{10} = 9
$$

$$
C_f = \frac{9}{\omega_s^2 L_g} = \frac{9}{(2\pi\cdot5000)^2\cdot2\times10^{-3}} = 4.56\ \mu\text{F}
$$

### Step 3: $L_f$ from the voltage limit (at 5 kHz)

The ripple at the connection point is the voltage across $C_f \parallel L_g$. Since $\omega_s^2L_gC_f = 9$:

$$
Z_\parallel = \frac{j\omega_sL_g}{1-\omega_s^2L_gC_f} = \frac{j\omega_sL_g}{1-9}, \qquad |Z_\parallel| = \frac{\omega_sL_g}{8} = 7.85\ \Omega
$$

The slide takes $|v_{Cf}|/|v_{Lf}| = 1/99$, i.e. 1% across the parallel branch and 99% across $L_f$:

$$
\omega_sL_f = 99\,|Z_\parallel| \;\Rightarrow\; L_f = \frac{99}{8}L_g = 24.75\ \text{mH}
$$

> **Subtlety (fine to mention in the exam):** $Z_\parallel$ is *capacitive* (negative), while $L_f$ is inductive. The two voltages are therefore in **antiphase**, so $|v_s| = |v_{Lf}| - |v_{Cf}|$ and strictly the ratio is $1/101$, giving $L_f = \frac{101}{8}L_g = 25.25$ mH. The difference is 2%. The slide's version is the expected answer; say which assumption you are using.

### Step 4: modulation depth (at 50 Hz)

At 50 Hz, $|Z_{Cf}| \gg |Z_{Lf}|$, so treat $C_f$ as an **open circuit**. With unity PF at the grid, $\bar I_{g1}$ is in phase with $\bar V_{g1}$, and the drop on $L_f$ is at 90°:

$$
V_1^2 = V_{g1}^2 + (\omega_1L_fI_{g1})^2 = 230^2 + (2\pi\cdot50\cdot24.75\text{m}\cdot37)^2 \;\Rightarrow\; V_1 = 368\ \text{V (rms)}
$$

$$
M = \frac{\hat V_1}{V_d} = \frac{\sqrt2\cdot368}{600} = \boxed{0.867}
$$

**State your assumptions:** $C_f$ is open circuit at 50 Hz, the converter is lossless, and $V_g$ is measured at the connection point (so $L_g$ is not in the 50 Hz loop).
