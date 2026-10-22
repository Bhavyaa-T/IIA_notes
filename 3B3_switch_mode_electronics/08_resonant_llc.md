# 08 — Resonant and LLC DC-DC Converters (L12, L13)

> **Source:** L12 (series LC and LLC resonant converters) and L13 Tutorial 2 Q2–Q3 (equivalent rectifier resistance, LLC design).
> **Method:** **first-harmonic approximation (FHA).** The tank filters the square wave, so treat everything as sinusoidal at the switching frequency and use phasors.

---

## 1. Series resonant tank driven by a square wave

A full bridge at **50% duty with bipolar switching** produces a square wave $v_{AB} = \pm V_d$, with fundamental peak $\hat V_{AB1} = \dfrac{4V_d}{\pi}$ (harmonics $\frac{4V_d}{3\pi}$, $\frac{4V_d}{5\pi}$, …). It drives a series $L_r$-$C_r$ into a resistance $R_s$.

$$
Z_r = j\omega L_r + \frac{1}{j\omega C_r}, \qquad \omega_0 = \frac{1}{\sqrt{L_rC_r}}\;\;(Z_r = 0 \text{ at resonance})
$$

At $\omega_s = \omega_0$, the tank passes the fundamental fully and blocks the harmonics. The current is **almost sinusoidal**: $\hat I_{1ac} = \dfrac{4V_d}{\pi R_s}$.

Compare a plain series L: its current is **triangular**.

### Normalised frequency and the nature of the load

$$
\omega_n = \frac{\omega_s}{\omega_0}, \qquad Z_r = j\sqrt{\frac{L_r}{C_r}}\left(\omega_n - \frac{1}{\omega_n}\right)
$$

| $\omega_n$ | $Z_r$ | Load seen by the bridge |
|---|---|---|
| $=1$ | 0 | Resistive |
| $>1$ | Positive (inductive) | **Inductive**: the current lags |
| $<1$ | Negative (capacitive) | Capacitive: the current leads |

### Why resonant? Reducing switching loss

- A sinusoidal current is **small at the switching instants**, much smaller than with a triangular current, so the switching loss is lower.
- In practice, operate **slightly above resonance** ($\omega_n$ slightly > 1). The load is then slightly inductive and the current slightly lags the voltage.
- *Why lagging helps (the standard explanation, not spelled out on the slides):* at the instant a switch is turned on, the current is still flowing through its **anti-parallel diode**. The switch therefore turns on at **zero voltage (ZVS)**, so the turn-on loss is nearly zero.

---

## 2. Series LC resonant DC-DC converter

The circuit is: full bridge, then $C_r$ and $L_r$, then a transformer $N:1$, then a diode bridge, then an output filter ($L_o$, $C_o$), then R.

**Control:** the duty cycle is fixed at 50%. The output is set by the **switching frequency** (adjusted from slightly above $f_0$ upwards) and by the turns ratio.

### Equivalent AC resistance of the rectifier ⭐ (L12 + L13)

Replace the rectifier and load with an equivalent resistance $R_s$ (as seen at the fundamental). The answer depends on **what kind of output filter** there is:

| Output filter | Rectifier input behaves like | $v_{2ac}$ | $i_{2ac}$ | $R_s$ (secondary) |
|---|---|---|---|---|
| **Inductor** ($L_o$ large) | A current sink | Sinusoid | Square wave $\pm I_o$ | $\boxed{\dfrac{\pi^2}{8}R}$ |
| **Capacitor only** ($C_o$ large) | A voltage sink | Square wave $\pm V_o$ | Sinusoid | $\boxed{\dfrac{8}{\pi^2}R}$ |

**Derivation, inductor filter.** $V_o$ is the average of $|v_{2ac}|$, so $V_o = \frac{2}{\pi}\hat V_{2ac}$. The fundamental of the square current is $\hat I = \frac{4I_o}{\pi}$. Hence

$$
R_s = \frac{\hat V_{2ac}}{\hat I_{2ac,1}} = \frac{\pi V_o/2}{4I_o/\pi} = \frac{\pi^2}{8}R
$$

**Derivation, capacitor filter.** The fundamental of the square voltage is $\frac{4V_o}{\pi}$. The average of $|i_{2ac}|$ is $I_o$, so $\hat I = \frac{\pi I_o}{2}$. Hence

$$
R_s = \frac{4V_o/\pi}{\pi I_o/2} = \frac{8}{\pi^2}R
$$

**Referred to the primary** (turns $N:1$): multiply by $N^2$. For example, with a capacitor filter, $R_s = \dfrac{8N^2}{\pi^2}R$.

> L13's advice: "take care of the source type of the AC input". That means: work out whether the rectifier is fed by a current-like or a voltage-like source.

### Overall voltage transfer

Let $M = \dfrac{\hat V_{1ac}}{\hat V_{AB1}}$ be the **tank gain**. Then:

- **Inductor output filter:** $V_o = \dfrac{2}{\pi}\hat V_{2ac} = \dfrac{2}{\pi}\cdot\dfrac1N\cdot M\cdot\dfrac{4V_d}{\pi} = \boxed{\dfrac{8M}{\pi^2N}V_d}$. L12 writes this with $N = 1$ as $\frac{8M}{\pi^2}V_d$.
- **Capacitor output filter (L13):** $V_o = \boxed{\dfrac{M}{N}V_d}$. The $\pi$ factors cancel exactly.

### Gain of the series LC tank

$$
M_{LC} = \frac{R_s}{R_s + j\sqrt{\frac{L_r}{C_r}}\left(\omega_n - \frac{1}{\omega_n}\right)} = \frac{1}{1+jQ\left(\omega_n-\frac{1}{\omega_n}\right)}
$$

$$
\boxed{|M_{LC}| = \frac{1}{\sqrt{1+Q^2\left(\omega_n-\frac{1}{\omega_n}\right)^2}}}, \qquad \boxed{Q = \frac{\omega_0L_r}{R_s} = \frac{1}{R_s\omega_0C_r} = \frac{\sqrt{L_r/C_r}}{R_s}}
$$

- $Q$ is the quality factor. It **depends on the load**: a heavy load means small $R_s$ and so **high Q**.
- $|M| \le 1$ always, with the peak at $\omega_n = 1$. The gain is controlled by moving $\omega_n$ above 1.
- **Problem:** at light load (low Q) the curve is very flat, so a **huge frequency range** is needed to regulate.

---

## 3. LLC resonant converter

Same circuit, but now the transformer's **magnetising inductance $L_m$ is deliberately used**. It appears in **parallel with $R_s$** across the transformer primary. Define

$$
L_m = k\,L_r
$$

Then

$$
M_{LLC} = \frac{R_s\,\|\,j\omega_sL_m}{R_s\,\|\,j\omega_sL_m + j\sqrt{\frac{L_r}{C_r}}\left(\omega_n-\frac1{\omega_n}\right)}
$$

$$
\boxed{|M_{LLC}| = \frac{1}{\sqrt{\left[1+\dfrac1k\left(1-\dfrac{1}{\omega_n^2}\right)\right]^2 + Q^2\left(\omega_n-\dfrac{1}{\omega_n}\right)^2}}}
$$

**Derivation** (worth knowing; it is short):

$$
\frac1M = 1 + \frac{Z_r}{R_s} + \frac{Z_r}{j\omega_sL_m}
$$

The two terms are:

- $\dfrac{Z_r}{R_s} = jQ\left(\omega_n - \dfrac1{\omega_n}\right)$
- $\dfrac{Z_r}{j\omega_sL_m} = \dfrac{\omega_0L_r(\omega_n - 1/\omega_n)}{k\,\omega_s L_r} = \dfrac1k\left(1-\dfrac1{\omega_n^2}\right)$

Check the limit: $k\to\infty$ ($L_m \to \infty$) recovers the series LC result. ✓

**Q is defined the same way** ($\sqrt{L_r/C_r}/R_s$). The main resonance is still between $L_r$ and $C_r$ only, and at $\omega_n = 1$ the gain is exactly 1 for **every** load.

### Effect of k

- **Decreasing k squeezes the whole family of gain curves together.** A wide range of Q (load) is covered within a **narrow** frequency band, which gives better control than a large k or a pure LC ($k = \infty$).
- Example from the slides: with a suitable k, all the given Q values are covered for **$1 < f_n < 1.4$** at **40–80%** gain.
- **k cannot be too small:** a small $L_m$ means a large magnetising current (circulating current, which causes conduction loss), and $L_r$ cannot be too large.
- *Not on the slides:* below resonance ($\omega_n < 1$), $L_m$ joins the resonance and the LLC can give **gain > 1**. The plain series LC cannot.

**Application:** the Tesla Model 3 on-board charger (100–240 Vac, 265–405 Vdc, up to 22 kW) uses an LLC stage.

---

## 4. Worked: L13 Tutorial 2 Q2–Q3 ⭐

### Q2: LLC with a capacitor-only output filter

Sketch the full bridge, $C_r$, $L_r$, $L_m$, the transformer $N:1$, the diode bridge and $C_o \parallel R$.

- Primary-referred equivalent load: $R_s = \dfrac{8N^2}{\pi^2}R$.
- Transfer ratio: $V_o = \dfrac{M}{N}V_d$, where M is the tank gain.

Full chain:

$$
V_o = \frac{2}{\pi}\hat I_{2ac}R = \frac{2R}{\pi}\cdot N\cdot\frac{M\hat V_{AB1}}{R_s} = \frac{2R}{\pi}\cdot N\cdot M\cdot\frac{4V_d}{\pi}\cdot\frac{\pi^2}{8RN^2} = \frac{M}{N}V_d
$$

### Q3: find Q and R at full load

> At full load, $f_n = 1.15$ gives an output that is **50% of the at-resonance value**, so $|M| = 0.5$. $C_r = 0.47\ \mu$F, $L_r = 47\ \mu$H, $N = 10$.

**Q.** Using the series-LC gain formula:

$$
\frac{1}{1+Q^2(1.15 - 1/1.15)^2} = 0.25 \;\Rightarrow\; Q\cdot0.280 = \sqrt3 \;\Rightarrow\; Q \approx 6.2 \approx \boxed{6}
$$

**R.** Here $\sqrt{L_r/C_r} = \sqrt{47\mu/0.47\mu} = 10\ \Omega$, so $R_s = 10/Q = 1.67\ \Omega$ (primary-referred). Then

$$
R = \frac{\pi^2}{8N^2}R_s = \frac{9.87}{800}\cdot1.67 = \boxed{20.5\ \text{m}\Omega}
$$

(The very small R is because of $N = 10$.)

### New $f_n$ at 20% load

20% of full load means R is 5× larger, so $R_s$ is 5× larger and **$Q$ is 5× smaller: $Q = 1.2$**. To keep $|M| = 0.5$:

$$
1.2\left(f_n - \frac1{f_n}\right) = \sqrt3 \;\Rightarrow\; f_n - \frac1{f_n} = 1.443 \;\Rightarrow\; f_n \approx 1.95
$$

> ⚠️ **The slide answer is "≈ 1.8"**, which was presumably read from the LLC gain-curve figure (it depends on k). The series-LC formula used for Q gives ≈ 1.95. If you meet this in the exam: say which model you use and be consistent. The lesson is the same either way: **light load means low Q, and so a much higher $f_n$ is needed** for the same gain. That is exactly the weakness LLC (with a smaller k) is designed to reduce.
