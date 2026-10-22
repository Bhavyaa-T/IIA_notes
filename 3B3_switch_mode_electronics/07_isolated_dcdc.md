# 07 — Isolated DC-DC: Transformers, Push-Pull, Flyback (L10, L11, L13)

> **Source:** L10 (transformer model, push-pull), L11 (flyback) and L13 Tutorial 2 Q1 (flyback design).

---

## 1. Why isolate?

The chain is DC → (inverter) → AC → **high-frequency transformer** → AC → (rectifier) → DC.

- **Galvanic isolation**: both sides can be grounded independently, which matters for safety.
- **Large step ratios** come cheaply from the turns ratio, instead of extreme duty cycles.
- Transformer size and weight scale roughly as **$1/f$**, so a high frequency gives a small transformer.

---

## 2. Transformer model (magnetic circuit)

The core has mean length $l_m$, cross-section $A_c$ and permeability $\mu_c$.

| Quantity | Formula |
|---|---|
| MMF | $\mathcal F = ni$, and in total $\mathcal F_c = n_1i_1 + n_2i_2$ |
| Reluctance | $\mathcal R_c = \dfrac{l_m}{\mu_cA_c}$ |
| "Ohm's law" | $\mathcal F_c = \mathcal R_c\,\phi_c$ |
| Faraday | $v_1 = n_1\dfrac{d\phi_c}{dt}$, $v_2 = n_2\dfrac{d\phi_c}{dt}$ |

### Ideal transformer ($\mathcal R_c = 0$)

Then $0 = n_1i_1 + n_2i_2$, so

$$
\boxed{\frac{v_1}{v_2} = \frac{n_1}{n_2}, \qquad \frac{i_1}{-i_2} = \frac{n_2}{n_1}}
$$

The minus sign comes from **both currents being defined into the dots**.

### Dot convention

- All dotted terminals have the **same voltage polarity** at the same instant.
- Current **into** a dot on one side corresponds to current **out of** the dot on the other side.
- Current into both dots means their MMFs (fluxes) add.

### Magnetising inductance ($\mathcal R_c \ne 0$)

Combine Faraday's law with $\mathcal R_c\phi_c = n_1i_1+n_2i_2$:

$$
v_1 = \frac{n_1^2}{\mathcal R_c}\frac{d}{dt}\left(i_1 + \frac{n_2}{n_1}i_2\right) = L_M\frac{di_M}{dt}
$$

$$
\boxed{L_M = \frac{n_1^2}{\mathcal R_c} = \frac{n_1^2\mu_cA_c}{l_m}}, \qquad i_M = i_1 + \frac{n_2}{n_1}i_2
$$

**Model:** $L_M$ in **parallel** with an ideal transformer. The ideal-transformer symbol does **not** include $L_M$; the inductance must be drawn separately.

- $L_M$ can be referred to either side using the **impedance reflection rule**: $L_{M(1)}/L_{M(2)} = (n_1/n_2)^2$.
- **With one winding open circuit, the other behaves as a plain inductor $L_M$**, referred to the side that is not open. This idea drives the flyback analysis.
- The **inductance factor** $A_L = L_M/n^2 = 1/\mathcal R_c$ is given by core datasheets.

### Leakage inductance and the full model

Flux that links only one winding is called **leakage flux**. It is modelled as a **series** inductance on each side. The full equivalent circuit is: winding resistance and leakage $L_{l1}$, then $L_M$ (plus core-loss R) in shunt, then the ideal transformer, then $L_{l2}$ and winding resistance.

---

## 3. Push-pull converter

**Topology:** a centre-tapped primary with two switches, Q1 and Q2, each driving half the primary from $V_1$. On the secondary, diodes D1 and D2 (centre-tapped full-wave rectifier) feed an output L-C filter.

**Assumptions:** $L_M$ is very large (ideal transformer).

**Switching pattern:**

- Q1 ON, then both OFF, then Q2 ON, then both OFF, and repeat.
- **Q1 and Q2 are never ON together.** Their ON intervals are equal, which prevents a DC flux build-up and so core saturation.
- **D is defined as $t_{ON}/T$, where $T$ = one ON interval + one both-OFF interval**, i.e. **half** the full Q1–Q2 cycle.

**Operation:**

- Q1 ON: $v_s = +\frac{N_2}{N_1}V_1$ at the rectifier, so the inductor sees $\frac{N_2}{N_1}V_1 - V_2$.
- Q2 ON: the same, with the opposite transformer polarity, rectified.
- Both OFF: $v_s = 0$. The inductor current **splits between D1 and D2** ($i_L = i_{D1} + i_{D2}$, each about $I_2/2$), so the transformer flux stays put.

Volt-second balance on L over $T$ gives

$$
\boxed{M = \frac{V_2}{V_1} = \frac{N_2}{N_1}D}
$$

The output is **buck-derived**: a buck with a $\frac{N_2}{N_1}$ gain.

> ⚠️ **Watch the definition of D.** With the lecturer's $T$ (half the switching cycle), $D\le1$ and $M = \frac{N_2}{N_1}D$. Many textbooks define D over the **full** Q1+Q2 period $T_s = 2T$, giving $M = 2D\frac{N_2}{N_1}$ with $D \le 0.5$. In the exam, use the question's definition and say so.

The output inductor sees ripple at **twice** the individual switch frequency.

---

## 4. Flyback converter ⭐

### Origin: an isolated buck-boost

Take the inverting buck-boost converter and replace the inductor L with the **magnetising inductance $L_m$** of a transformer. Then reverse the diode and the secondary dot, so the output is **positive**.

**The flyback "transformer" is really a coupled inductor**: it stores energy in ON and releases it in OFF. In practice the core is gapped to store energy without saturating.

**Real example:** a 45 W laptop charger. 100–240 V ac → bridge rectifier + $C_d$ → flyback MOSFET + transformer → diode + $C_o$ → 14.58 V, 3.05 A.

### Switching intervals

| | **Q ON** ($DT$) | **Q OFF** ($(1-D)T$) |
|---|---|---|
| Diode | Reverse biased (dot polarity) | Forward biased |
| Transformer | Secondary **open**, so $L_m$ is seen on the **primary** | Primary **open**, so $L_m$ is seen on the **secondary** |
| Voltage on $L_m$ | $v_{Lm} = V_1$ | $v_{Lm} = -V_2$ (referred to the secondary) |
| $L_m$ | **Charging** from the input, $\phi_c$ rises | **Discharging** into C and the load, $\phi_c$ falls |
| Load supplied by | Output capacitor alone | Diode current $i_D$ (which also recharges C) |
| Currents | $i_s = i_m$ | $i_D = \frac{N_1}{N_2}i_m$ |

### Voltage ratio (flux balance)

Faraday's law gives $d\phi/dt = v/N$:

- **ON:** $\phi_c(t) = \phi_1 + \dfrac{V_1}{N_1}t$, so $\hat\phi_c = \phi_2 = \phi_1 + \dfrac{V_1}{N_1}DT$.
- **OFF:** $\phi_c$ falls back to $\phi_1 = \phi_2 - \dfrac{V_2}{N_2}(1-D)T$.

Equate the rise and fall:

$$
\frac{V_1}{N_1}DT = \frac{V_2}{N_2}(1-D)T \;\;\Rightarrow\;\; \boxed{M(D) = \frac{V_2}{V_1} = \frac{N_2}{N_1}\cdot\frac{D}{1-D}}
$$

This is the buck-boost result times the turns ratio.

**Current levels:**

- Primary current ramps $I_1 \to I_2$ during ON, with $I_{1,2} = N_1\phi_{1,2}/L_M$ and $I_2 = I_1 + \dfrac{V_1DT}{L_m}$.
- Diode current ramps from $\frac{N_1}{N_2}I_2$ down to $\frac{N_1}{N_2}I_1$ during OFF.
- Capacitor current: $i_C = i_D - I_o$, which is $-I_o$ during ON.

### Why run in BCM? (L13 Q1(ii))

In **boundary conduction mode** the flux starts at **zero** at the beginning of ON and returns to **zero** at the end of OFF ($\phi_1 = 0$). For a given energy transfer, this **minimises the peak flux** and so avoids core saturation. *A further benefit (not on the slide):* the MOSFET turns on at zero current, with no diode reverse recovery.

---

## 5. Designing a flyback transformer (L11 example)

The core is a U100/57/25 + I100/25/25 ferrite pair, with $A_L = L_M/n_1^2 = 1/\mathcal R_c = \mu_cA_c/l_m = 6700$ nH/turn².

- For ~10 mH: $n_1 = \sqrt{10\text{m}/6700\text{n}} = 38.6$, so take **40 turns** (10.7 mH).
- For $V_o = V_\text{in}/10$: $n_2 = 4$ turns. *This only holds at $D = 0.5$, where $D/(1-D) = 1$. In general $n_2/n_1 = M(1-D)/D$.*
- Check the permeability: $\mu = A_L\,l_m/A_c = 6700\text{n}\times0.245/645\mu = 2.55\times10^{-3}$ H/m, i.e. $\mu_r \approx 2000$. This is consistent with **3C90 ferrite** ($\mu_r \approx 2100$). (The slide's arithmetic gives ≈2060, a rounding difference.)

---

## 6. Worked: L13 Tutorial 2 Q1, flyback in BCM ⭐

> $V_1 = 150$ V, $V_2 = 50$ V, $D = 0.5$, $P = 100$ W, $N_1 = 60$. Ferrite saturates at 0.45 T, $A_c$ = 100 mm², **40% saturation margin**. Find $N_2$, $f_{sw}$ and $L_m$ (lossless).

**$N_2$ from M(D):**

$$
N_2 = \frac{V_2(1-D)N_1}{V_1D} = \frac{50\cdot0.5\cdot60}{150\cdot0.5} = \boxed{20}
$$

**Switching period from the flux limit.** BCM means the flux starts from zero, so the peak flux is $\phi_2 = B_mA_c$ with $B_m = 0.6\times0.45 = 0.27$ T:

$$
\phi_2 = 0.27\times100\times10^{-6} = 27\ \mu\text{Wb}
$$

The flux falls from $\phi_2$ to 0 during OFF, under $-V_2$ across $N_2$:

$$
\phi_2 = \frac{V_2(1-D)T}{N_2} \;\Rightarrow\; T = \frac{N_2\phi_2}{V_2(1-D)} = \frac{20\cdot27\mu}{50\cdot0.5} = 21.6\ \mu\text{s}, \qquad \boxed{f = 46.3\ \text{kHz}}
$$

**Peak primary current from the load current.** $I_o = P/V_o = 2$ A. The diode current is a triangle of peak $\frac{N_1}{N_2}I_2$ lasting $(1-D)T$:

$$
I_o = \frac1T\cdot\frac12\cdot\frac{N_1}{N_2}I_2\cdot(1-D)T \;\Rightarrow\; I_2 = \frac{2I_oN_2}{N_1(1-D)} = \frac{2\cdot2\cdot20}{60\cdot0.5} = 2.67\ \text{A}
$$

**Magnetising inductance (primary).** During ON, $V_1 = L_m\dfrac{I_2 - 0}{DT}$:

$$
L_m = \frac{V_1DT}{I_2} = \frac{150\cdot0.5\cdot21.6\mu}{2.67} = \boxed{608\ \mu\text{H}}
$$

✅ **Sanity checks** (do these in the exam; they take 20 s):

- Energy per cycle: $P = \tfrac12L_mI_2^2f = 0.5\cdot608\mu\cdot2.67^2\cdot46.3\text{k} \approx 100$ W ✓
- Flux: $L_m = N_1\phi_2/I_2 = 60\cdot27\mu/2.67 = 607\ \mu$H ✓

---

## 7. Safety and design rules (L11, not examinable but useful)

- **Functional isolation** (e.g. the level shift for high-side gate drives) is different from **safety isolation** (making a touch-safe low voltage).
- **Creepage** is the distance along a surface; **clearance** is the distance through air. Both depend on the expected pollution level and can be set in PCB layout tools. The standard is IEC 60664.
- Isolation is tested at test voltages well **above** the operating voltage.
