# 09 — Power Device Losses: Conduction, Switching, Gate Drive (L14, L15, L16)

> **Source:** L14 (2025 version, which has corrections), L15 and L16, including the worked SMPS loss example.
> ⚠️ The **L15 deck repeats the L14 slides in their pre-2025 form**, with the diode/IGBT conduction loss written as $V\cdot I_\text{rms}$. **That is wrong.** Use the corrected L14 (2025) version below, which uses the **average** current.

---

## 1. Non-ideal MOSFET behaviour

| Condition | Behaviour |
|---|---|
| $V_{GS} < V_{th}$ (≈0 V or negative) | OFF: open circuit |
| $V_{GS} > V_{th}$ (a few V) | ON, in the **linear (ohmic) region**: behaves as a resistor $R_{ds,on}$, **in both directions** (D→S or S→D) |
| OFF with negative $v_{DS}$ | The **body diode** conducts S→D, with forward drop $V_f$ |
| $V_{DS} > V_{BR}$ | Breakdown: the device fails |

---

## 2. Loss equations ⭐

$$
P_\text{loss} = P_\text{cond} + P_{sw}
$$

### Conduction loss

| Device | Model | Conduction loss |
|---|---|---|
| **MOSFET** | Resistor $R_{ds,on}$ | $\boxed{P_\text{cond} = R_{ds,on}\,I_{ds,\text{rms}}^2}$ → **RMS** current |
| **Diode** | Constant drop $V_f$ | $\boxed{P_\text{cond} = V_f\,I_\text{avg}}$ → **average** current |
| **IGBT** | Constant drop $V_{CE}$ | $\boxed{P_\text{cond} = V_{CE}\,I_\text{avg}}$ → **average** current |

**Why RMS for one and average for the others?** Loss is $\frac1T\int v\,i\,dt$.

- If $v = Ri$, this becomes $R\cdot\frac1T\int i^2 = RI_\text{rms}^2$.
- If $v = V$ (a constant), it becomes $V\cdot\frac1T\int i = VI_\text{avg}$.

The switching transients are short, so ignore them when computing $I_\text{rms}$. **Do not ignore the ripple**, though: $i_{ds}$ usually has substantial ripple, and that raises $I_\text{rms}$.

### Switching loss

$$
E_{on} = \int_{t_{on}} v_{ds}i_{ds}\,dt, \qquad E_{off} = \int_{t_{off}} v_{ds}i_{ds}\,dt
$$

$$
\boxed{P_{sw} = (E_{on}+E_{off})\,f_{sw}}
$$

- Switching loss is **proportional to $f_{sw}$**.
- Research effort goes into reducing $E_{on}$ and $E_{off}$ (e.g. wide-bandgap devices, soft switching / LLC).

---

## 3. Worked: buck converter conduction losses (L14)

The MOSFET conducts during $DT$, and its current is a **trapezoid**: average $I_o$, ripple $\Delta I_L$. The diode conducts during $(1-D)T$.

**RMS of the MOSFET current.** Split it into DC plus AC.

- The AC part (over the ON interval) is a sawtooth from $-\Delta I_L/2$ to $+\Delta I_L/2$:

$$
\text{RMS}_{AC}^2 = \frac1T\int_0^{DT}\left(\frac{\Delta I_L}{DT}t - \frac{\Delta I_L}{2}\right)^2dt = \frac{\Delta I_L^2D}{12}
$$

- The DC part over the ON interval contributes $I_o^2D$.
- The cross term $\int 2fg\,dt$ is zero, because $f$ has zero mean.

$$
I_{ds,\text{rms}}^2 = D\left(I_o^2 + \frac{\Delta I_L^2}{12}\right)
$$

$$
\boxed{P_{cond,T} = R_{ds,on}\,D\left(I_o^2 + \frac{\Delta I_L^2}{12}\right)}
$$

**Diode (constant $V_f$):**

$$
\boxed{P_{cond,D} = V_f\cdot\frac1T\int_{t_{off}}i_o\,dt = V_fI_o(1-D)}
$$

**The duty ratio decides how the conduction loss is split** between the switch and the diode. A buck with a large step-down (small D) puts most of the loss in the diode, which is the motivation for the synchronous buck.

---

## 4. Synchronous buck/boost and dead time (L15)

Replace the low-side diode (buck) or high-side diode (boost) with a **MOSFET that conducts in reverse** (S→D). This is the same circuit as a half bridge.

- Conduction loss improves because $R_{ds,on}\,i_L < V_f$ (the MOSFET drop is smaller than the diode drop).
- **Dead time:** a few % of the period with **both** MOSFETs OFF, inserted at every transition to avoid shoot-through.
- During dead time the **body diode** of the low-side MOSFET carries $i_L$. Sequence: T2 body diode, then T2 channel, then T2 body diode, then T1.
- At every instant, $V_{ds1} + V_{ds2} = V_{in}$.

### Commutation loop

During a transient, model the circuit as:

- the MOSFET being switched,
- a **complementary diode** (usually the body diode of the other MOSFET),
- a **constant current source $I_o$** (the inductor current does not change during the short transient),
- the DC link $V_{DC}$ and the gate driver $V_{GG}$ through $R_G$.

This single model covers *any* MOSFET transition, on→off or off→on.

---

## 5. Gate drive model (L14)

- The gate driver is a voltage source $V_{GG}$, feeding the gate through $R_G$ (the external $R_\text{on}$ plus the internal $R_{G,FET}$).
- $V_{GS}$ is the voltage on the internal capacitor $C_{GS}$. $R_G$ limits the charge and discharge current, so the **transient of $C_{GS}$ sets the switching speed**.
- $C_{GD}$ (the **Miller capacitance**) and $C_{DS}$ model the coupling between terminals. **$C_{GS}$ and $C_{GD}$ dominate the switching transient.**
- These capacitances are **non-linear** in reality. In particular, $C_{GD}$ is much larger at low $v_{DS}$.
- Datasheet names: $C_{iss} = C_{GS} + C_{GD}$ (input) and $C_{rss} = C_{GD}$ (reverse transfer).

---

## 6. Turn-on sequence ⭐ (L15 and L16)

**Assumptions:** $I_o$ is constant, the diode has no reverse recovery, $C_{GS}$ starts at 0, and $C_{GD}$ starts charged to $V_{DC}$.

| Interval | $v_{GS}$ | $i_D$ | $v_{DS}$ | What's going on | Loss? |
|---|---|---|---|---|---|
| **$t_1$: delay $t_d$** | 0 → $V_{th}$ (RC charge of $C_{iss}$) | 0 | $V_{DC}$ | The channel is not yet formed | No |
| **$t_2$: current rise $t_{ri}$** | $V_{th}$ → $V_{GS,I_o}$ | 0 → $I_o$ | **Stays at $V_{DC}$** | The diode is still conducting the rest of $I_o$, so it clamps $v_{DS}$ | **Yes** |
| **$t_3$: Miller plateau $t_{fv}$** | **Flat at $V_{GS,I_o}$** | $I_o$ | $V_{DC}$ → ~0 | The diode is off now. All the gate current goes into discharging $C_{GD}$ | **Yes** |
| **$t_4$** | $V_{GS,I_o}$ → $V_{GG}$ | $I_o$ | ≈0 ($R_{ds,on}I_o$) | Fully on; the gate finishes charging | No (conduction only) |

**Why is $v_{DS}$ stuck at $V_{DC}$ during the current rise?** Until the MOSFET carries all of $I_o$, the free-wheeling diode still conducts and holds the switch node, so the MOSFET must block the full $V_{DC}$. This is the key hard-switching insight.

**The Miller plateau** is split into $t_{fv1}$ and $t_{fv2}$ because **$C_{GD}$ is non-linear**. $v_{DS}$ falls quickly at first (small $C_{GD}$ at high $v_{DS}$), then slowly (large $C_{GD}$ at low $v_{DS}$).

### Simplified (linear) model for calculation (L16)

Assume $C_{GD}$ is constant, so $v_{DS}$ changes linearly during the plateau. Use an average gate current in each interval.

**Current-rise interval ($t_2$).** The gate voltage goes from $V_{th}$ to $V_{GS,I_o}$, so use the midpoint:

$$
I_{G2} = \frac{V_{GG} - \tfrac12(V_{GS,I_o}+V_{th})}{R_G}, \qquad t_2 = \frac{(V_{GS,I_o}-V_{th})\,C_{iss}}{I_{G2}}
$$

**Voltage-fall interval ($t_3$).** The gate is clamped at the plateau:

$$
I_{G3} = \frac{V_{GG}-V_{GS,I_o}}{R_G}, \qquad t_3 = \frac{V_{DC}\,C_{rss}}{I_{G3}}
$$

**Turn-on loss.** Each interval is a triangle of power: $V_{DC}\cdot\frac{I_o}{2}$ during $t_2$ and $I_o\cdot\frac{V_{DC}}{2}$ during $t_3$:

$$
\boxed{P_{sw,ON} = \frac{V_{DC}I_o}{2}\cdot\frac{t_2+t_3}{T}}
$$

---

## 7. Turn-off sequence (L16)

This is **essentially the reverse of turn-on**. Use the same equivalent circuits with the correct driving voltage (the gate driver goes to 0) and initial conditions.

| Interval | What happens |
|---|---|
| $t_{d(off)}$ | $v_{GS}$ falls from $V_{GG}$ to the plateau $V_{GS,I_o}$, with time constant $\tau_2 = R_G(C_{gd2}+C_{gs})$. No loss |
| $t_{rv2}$, $t_{rv1}$ (**Miller plateau**) | $v_{GS}$ is flat. **$v_{DS}$ rises** to $V_{in}$ while $i_D = I_o$: slowly at first (large $C_{GD}$), then fast. Loss |
| $t_{fi}$ | $v_{DS}$ is clamped at $V_{in}$ (the diode now conducts). $v_{GS}$ falls from the plateau to $V_{th}$ with $\tau_1 = R_G(C_{gd1}+C_{gs})$, and **$i_D$ falls** to 0. Loss |
| After | $v_{GS}$ decays to 0. No loss |

*By analogy with the turn-on model (derived here, not written out on the slides):* the gate discharges through $R_G$ into 0 V.

- Voltage rise: $t_{rv} \approx \dfrac{V_{DC}C_{rss}R_G}{V_{GS,I_o}}$
- Current fall: $t_{fi} \approx \dfrac{(V_{GS,I_o}-V_{th})C_{iss}R_G}{\frac12(V_{GS,I_o}+V_{th})}$

Turn-off is often **slower** than turn-on, because the gate is driven by only $V_{GS,I_o}$ (to 0 V) rather than $V_{GG} - V_{GS,I_o}$.

### Datasheet form (lecturer's convention)

The datasheet times map onto the simplified pattern as follows:

- Turn-on: $t_{d(on)} \to t_2$ and $t_r \to t_3$.
- Turn-off: $t_{d(off)} \to t_6$ and $t_f \to t_7$.

$$
\boxed{P_{sw,ON} = \frac{V_{DC}I_o}{2}\cdot\frac{t_{d(on)}+t_r}{T}, \qquad P_{sw,OFF} = \frac{V_{DC}I_o}{2}\cdot\frac{t_{d(off)}+t_f}{T}}
$$

> **Use this in the exam**, but be aware that it is a teaching simplification. Real datasheets define $t_{d(on)}$ as a *delay* (roughly $t_1$, which is lossless). Stated precisely, the loss happens only while current and voltage overlap, i.e. during the current-rise and voltage-fall times.

---

## 8. Diode reverse recovery (L16)

- During the MOSFET's current rise ($t_{ri}$), the complementary diode current $I_{DF}$ falls.
- When it reaches zero, the diode's **stored charge** must still be removed. This produces a **reverse recovery current $I_{rr}$** (anode ← cathode) lasting $t_{rr} = t_a + t_b$, with charge $Q_{rr} = Q_a + Q_b$.
- The MOSFET must carry **$I_o + I_{rr}$** while still blocking about $V_{in}$. This **adds to the MOSFET turn-on loss**, and it shows up as a current overshoot above $I_o$.
- Mitigations: fast or soft-recovery diodes, SiC Schottky diodes (≈ no $Q_{rr}$), and DCM/BCM or ZVS operation, where the diode current is already zero at turn-on.

---

## 9. Worked: SMPS switch losses (L16 example) ⭐

> An inductive switching circuit with a power MOSFET. $f = 10$ kHz, $D = 50\%$, $V_{dc} = 300$ V, $I_{OFF} = 0$, $I_{ON} = 1$ A, $V_{ON} = 5$ V. Turn-on: current rise $t_r = 0.1$ µs, voltage fall $t_d = 0.1$ µs. Turn-off: delay $t_s = 0.1$ µs, voltage rise $t_g = 0.2$ µs, current fall $t_f = 0.1$ µs.

**(i) Losses.** $T = 100$ µs.

On-state conduction time, which excludes the transitions:

$$
t_{on} = DT - t_r - t_f = 50 - 0.1 - 0.1 = 49.8\ \mu\text{s}
$$

**Conduction** (constant $V_{ON}$, so $V\cdot I$):

$$
P_{cond} = I_{ON}V_{ON}\frac{t_{on}}{T} = 1\times5\times\frac{49.8}{100} = 2.49\ \text{W}
$$

**Turn-on.** The overlap is current rise plus voltage fall:

$$
P_{sw,on} = \frac{V_{dc}I_{ON}}{2}\cdot\frac{t_r+t_d}{T} = 150\times\frac{0.2}{100} = 0.30\ \text{W}
$$

**Turn-off.** The overlap is voltage rise plus current fall. The delay $t_s$ is lossless:

$$
P_{sw,off} = \frac{V_{dc}I_{ON}}{2}\cdot\frac{t_g+t_f}{T} = 150\times\frac{0.3}{100} = 0.45\ \text{W}
$$

$$
\boxed{P_{loss} = 2.49 + 0.30 + 0.45 = 3.24\ \text{W}}
$$

**(ii) Comment.** On-state losses are much larger than switching losses, which means the switching frequency is **too low**. Switching loss scales with $f$, so increasing $f$ raises the total loss. A design that **balances** conduction and switching losses is better.

**(iii) Increasing $f_{sw}$.**

- **Advantage:** smaller transformer, inductors and capacitors, giving lower cost, volume and weight.
- **Disadvantages:**
  1. Higher switching loss and so lower efficiency.
  2. More heat: a bigger heatsink is needed, and reliability suffers.
  3. **EMI** problems (higher $dv/dt$, $di/dt$ and harmonic content).
