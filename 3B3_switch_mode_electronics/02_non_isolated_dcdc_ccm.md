# 02 — Non-Isolated DC-DC Converters in CCM (L2, L7)

> **Source:** L02 slides, L07 Tutorial 1 Q2 and the "unfamiliar converter" slide.
> **Notation:** $V_d$ = input, $V_o$ = output, $D$ = duty ratio of the controlled switch $S$, $T_s = 1/f_\text{sw}$. Capitals mean averages and lower case means instantaneous values (convention only).
> **CCM** = continuous conduction mode: $i_L$ never reaches zero.

---

## The recipe (works for every converter, including ones you haven't seen)

1. **Draw the two sub-circuits**: S ON (for $DT_s$) and S OFF (for $(1-D)T_s$).
2. **Write $v_L$ in each state** (and $i_C$ if you need ripple).
3. **Volt-second balance**: $v_{L,\text{on}}\,DT_s + v_{L,\text{off}}\,(1-D)T_s = 0$, which gives $M(D)$.
4. **Ripple current**: $\Delta i_{L,pp} = \dfrac{|v_{L,\text{on}}|\,DT_s}{L}$, then substitute $V_o = M V_d$.
5. **Ripple voltage**: find the charge $Q_C$ delivered to C in one charging interval, then $\Delta v_{C,pp} = Q_C / C$.
6. **Currents**: use charge balance on C, or lossless power balance $V_d I_d = V_o I_o$.

---

## 1. Buck (step-down) converter

**Topology:** $V_d$ — switch S — node — inductor L — $V_o$ (C ∥ load). Diode from ground to the switch node.

| State | Duration | $v_L$ | Notes |
|---|---|---|---|
| S ON | $DT_s$ | $V_d - V_o$ | $i_L$ rises, diode reverse biased |
| S OFF | $(1-D)T_s$ | $-V_o$ | $i_L$ free-wheels through the diode, falls |

### Voltage ratio

$$
(V_d - V_o)DT_s + (-V_o)(1-D)T_s = 0 \;\;\Rightarrow\;\; \boxed{M(D) = \frac{V_o}{V_d} = D}\quad(D<1)
$$

Lossless: $V_d I_d = V_o I_o \Rightarrow I_o/I_d = 1/D$. Charge balance on C gives $I_L = I_o$.

### Inductor ripple current

$$
\Delta i_{L,pp} = \frac{(V_d - V_o)\,t_\text{on}}{L} = \frac{V_o\,t_\text{off}}{L}
$$

The goal is to express it in the design quantities $V_d$, $f_\text{sw}$ and $L$. Substitute $V_o = DV_d$:

$$
\boxed{\Delta i_{L,pp} = \frac{V_d\,D(1-D)}{f_\text{sw}\,L}}
$$

- It is **maximum at $D = 0.5$**: $\Delta i_{L,pp,\max} = V_d / (4 f_\text{sw} L)$. Use this for worst-case design.
- Higher $L$ or higher $f_\text{sw}$ means lower ripple. The ripple depends on the operating point (on $D$).

### Output capacitor ripple voltage

Assume all the ripple current goes into C and the load takes only the DC: $\Delta i_C = \Delta i_L$. The capacitor current is a triangle of peak $\Delta i_L/2$, and it is **positive for exactly half the period, $T_s/2$, regardless of D**.

The charge is the area of the positive triangle:

$$
Q_C = \frac12 \cdot \frac{\Delta i_{L,pp}}{2} \cdot \frac{T_s}{2} = \frac{\Delta i_{L,pp} T_s}{8}
$$

$$
\boxed{\Delta v_{C,pp} = \frac{Q_C}{C} = \frac{\Delta i_{L,pp}}{8 C f_\text{sw}} = \frac{V_d\,D(1-D)}{8 L C f_\text{sw}^2}}
$$

- In CCM the ripple is **load independent** (no $R$ appears), provided C is adequately large.
- It goes as $1/f_\text{sw}^2$, so raising the frequency is very effective.
- The output filter has corner frequency $f_0 = 1/(2\pi\sqrt{LC})$, so you can also write $\Delta v_{C,pp}/V_o = \tfrac{\pi^2}{2}(1-D)(f_0/f_\text{sw})^2$. *(Standard rearrangement, not on the slides.)*

---

## 2. Boost (step-up) converter

**Topology:** $V_d$ — inductor L — node; switch S from the node to ground; diode from the node to $V_o$ (C ∥ load).

**Principle:** with S closed, L charges from $V_d$. When S opens, L's current must keep flowing, so the inductor voltage adds to $V_d$ and pushes current "uphill" into the higher-voltage output.

| State | Duration | $v_L$ | Notes |
|---|---|---|---|
| S ON | $DT_s$ | $V_d$ | $i_L$ rises; diode off; C alone supplies the load |
| S OFF | $(1-D)T_s$ | $V_d - V_o$ (negative) | $i_L$ falls, delivering current to C and the load |

### Voltage ratio

$$
V_d\,DT_s + (V_d - V_o)(1-D)T_s = 0 \;\;\Rightarrow\;\; \boxed{M(D) = \frac{V_o}{V_d} = \frac{1}{1-D}}
$$

Lossless: $I_o/I_d = 1-D$. The inductor current is the input current: $I_L = I_d = I_o/(1-D)$.

> ⚠️ **Notation trap (L2 slide "Boost – Complete"):** one early slide gets $V_\text{out}/V_\text{in} = 1/D$. On that slide D is the duty of the **upper** switch of a half-bridge (the one conducting during the discharge interval $T_E$). It is the same result with D defined the other way round. Always state your definition of D.

### Inductor ripple current

$$
\Delta i_{L,pp} = \frac{V_d\,t_\text{on}}{L} = \boxed{\frac{V_d\,D}{f_\text{sw}\,L}}
$$

### Output capacitor ripple voltage

During $DT_s$ the diode is off and **C alone supplies $I_o$**. That rectangle of charge sets the ripple:

$$
Q_C = I_o D T_s, \qquad I_o = \frac{V_o}{R} = \frac{V_d}{R(1-D)}
$$

$$
\boxed{\Delta v_{C,pp} = \frac{D I_o}{C f_\text{sw}} = \frac{D V_o}{R C f_\text{sw}} = \frac{D V_d}{R C f_\text{sw}(1-D)}}
$$

- The ripple is **load dependent**: heavier load (smaller $R$) gives **larger** ripple.
  > ⚠️ The L2 slide bullet says "high load (smaller R) lower ripple". That contradicts its own formula, since $\Delta v \propto 1/R$. Trust the formula.
- It goes as $1/f_\text{sw}$ only (not $1/f^2$ as for the buck), because the boost output current is pulsed and C must hold up the load for the whole of $DT_s$.

**Practical note (L2):** a boost can be run with simple hysteretic voltage control. A Schmitt trigger on the measured $V_\text{out}$ turns the switch on and off.

---

## 3. Buck-boost converter (inverting)

**Topology:** switch S from $V_d$ to a node; L from the node to ground; diode from the output to the node (reversed), so **$V_o$ is negative** with respect to the input.

| State | Duration | $v_L$ |
|---|---|---|
| S ON | $DT_s$ | $V_d$ (L charges from the input, output isolated) |
| S OFF | $(1-D)T_s$ | $-V_o$ (L discharges into the output) |

$$
V_d D T_s + (-V_o)(1-D)T_s = 0 \;\;\Rightarrow\;\; \boxed{M(D) = \frac{V_o}{V_d} = \frac{D}{1-D}}\quad\text{(magnitude; polarity inverted)}
$$

- $D < 0.5$ steps down, $D > 0.5$ steps up.
- By analogy with the boost (*derived, not on the slides*): $\Delta i_{L,pp} = V_d D/(f_\text{sw}L)$, $\Delta v_{C,pp} = D I_o/(C f_\text{sw})$, and $I_L = I_o/(1-D)$.
- Input current $I_s = \dfrac{V_o I_o}{V_d} = \dfrac{V_o^2}{R V_d} = \dfrac{1}{R}\dfrac{D^2}{(1-D)^2}V_d$. This is used for the DCM boundary in [03_dcm.md](03_dcm.md).
- The **flyback** converter is an isolated buck-boost; see [07_isolated_dcdc.md](07_isolated_dcdc.md).

---

## 4. Worked: "unfamiliar converter" (L7)

This is the exam pattern: they give you a converter you haven't seen. Two inductors $L_1, L_2$, a coupling capacitor $C_1$ (voltage $V_c$), and output $C_2 \parallel R$. **Hint given: $V_c$ is constant in steady state.**

Apply the recipe: write $v_{L1}$ and $v_{L2}$ in each state and apply volt-second balance to **each inductor separately**.

- **ON:** $V_i = L_1 \dfrac{\Delta I_{L1}}{kT}$ and $V_i = -V_c + L_2\dfrac{\Delta I_{L2}}{kT} + V_o$
- **OFF:** $V_c = L_1\dfrac{\Delta I_{L1}}{(1-k)T}$ and $V_o = L_2 \dfrac{\Delta I_{L2}}{(1-k)T}$

Eliminate the $\Delta I$ terms in three steps.

1. From the OFF equations: $\Delta I_{L1} = \dfrac{V_c(1-k)T}{L_1}$ and $\Delta I_{L2} = \dfrac{V_o(1-k)T}{L_2}$.
2. Substitute into the ON equation for $L_1$: $V_i = V_c\dfrac{1-k}{k}$, so $V_c = \dfrac{k}{1-k}V_i$.
3. Substitute into the ON equation for $L_2$:

$$
V_i = -\frac{k}{1-k}V_i + \frac{1-k}{k}V_o + V_o
\;\Rightarrow\; \frac{V_i}{1-k} = \frac{V_o}{k}
\;\Rightarrow\; \boxed{\frac{V_o}{V_i} = \frac{k}{1-k}}
$$

(The slide has a stray $T$ in the denominator of the $-kV_i/((1-k)T)$ term. It is a typo, because $V_c$ has no $T$ in it.)

**General lesson:** each inductor gives one volt-second equation, and each "constant" capacitor voltage is one unknown. Solve them simultaneously.

---

## 5. Summary table (CCM)

| | Buck | Boost | Buck-boost |
|---|---|---|---|
| $M(D)$ | $D$ | $\dfrac{1}{1-D}$ | $\dfrac{D}{1-D}$ (inverted) |
| $\Delta i_{L,pp}$ | $\dfrac{V_d D(1-D)}{f L}$ | $\dfrac{V_d D}{f L}$ | $\dfrac{V_d D}{f L}$ |
| $\Delta v_{C,pp}$ | $\dfrac{V_d D(1-D)}{8 L C f^2}$ | $\dfrac{D I_o}{C f} = \dfrac{D V_o}{R C f}$ | $\dfrac{D I_o}{C f}$ |
| $I_L$ | $I_o$ | $\dfrac{I_o}{1-D}$ | $\dfrac{I_o}{1-D}$ |
| Ripple load-dependent? | No | Yes | Yes |
