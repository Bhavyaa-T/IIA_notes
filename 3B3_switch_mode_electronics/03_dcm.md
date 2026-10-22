# 03 — Discontinuous Conduction Mode (L3, L7)

> **Source:** L03 slides and L07 Tutorial 1 Q2 (buck-boost DCM).
> **DCM** means the inductor current falls to zero and **stays at zero** for part of each period. The diode prevents it from going negative.

---

## 1. Where DCM comes from

In CCM the average inductor current is set by the **load**, while the ripple is set by **$L$, $f$ and $D$** and does not depend on the load. Make the load lighter (bigger $R$) and the whole triangle slides down. Eventually the trough touches zero: that is the **boundary (critical) conduction mode, BCM**. Any lighter, and the current would have to go negative, which the diode blocks. The result is **DCM**.

**Condition for DCM:** the average current is less than half the peak-to-peak ripple.

$$
I_\text{avg} < \Delta i_L \qquad\text{where}\quad \Delta i_L = \tfrac12 \Delta i_{L,pp}\;\text{(peak, i.e. half of p-p)}
$$

### The conduction coefficient

$$
\boxed{K = \frac{2L}{R\,T_s}}
$$

$K$ depends only on the **components and load** ($L$, $T_s$, $R$). Each converter has a **critical function $K_\text{crit}(D)$**:

$$
K < K_\text{crit}(D) \;\Rightarrow\; \text{DCM} \qquad\qquad K > K_\text{crit}(D) \;\Rightarrow\; \text{CCM}
$$

If $K > \max_D K_\text{crit}$, the converter is **always CCM** whatever the duty ratio.

---

## 2. Boundary derivations

### Buck

$I_o = \dfrac{DV_d}{R}$ and $\Delta i_L = \dfrac{D(1-D)V_d T_s}{2L}$.

$$
\frac{DV_d}{R} < \frac{D(1-D)V_dT_s}{2L} \iff \frac{2L}{RT_s} < 1-D \qquad\Rightarrow\qquad \boxed{K_\text{crit}^\text{buck}(D) = 1-D}
$$

$\max K_\text{crit} = 1$ (as $D \to 0$), so $K > 1$ guarantees CCM.

### Boost

Compare the average **input** current with the ripple, because the inductor sits at the input. Using $V_o = V_d/(1-D)$:

$$
I_d = \frac{V_o I_o}{V_d} = \frac{V_d}{(1-D)^2R}, \qquad \Delta i_L = \frac{V_dDT_s}{2L}
$$

$$
\frac{V_d}{(1-D)^2 R} < \frac{V_dDT_s}{2L} \qquad\Rightarrow\qquad \boxed{K_\text{crit}^\text{boost}(D) = D(1-D)^2}
$$

The maximum is at $D = 1/3$, where $K_\text{crit} = 4/27 \approx 0.148$. *(Calculus check, not on the slides.)*

### Buck-boost (L7 Q2)

$I_s = \dfrac{1}{R}\dfrac{k^2}{(1-k)^2}V_1$. At the boundary the ON-interval current is a right triangle of height $\Delta I_{L,pp} = V_1kT/L$, so its average over $T$ is

$$
I_s = \frac{1}{T}\cdot\frac12\cdot\frac{V_1 kT}{L}\cdot kT = \frac{V_1 k^2 T}{2L}
$$

Set the two expressions for $I_s$ equal:

$$
\boxed{K_\text{crit}^\text{buck-boost}(k) = (1-k)^2}
$$

---

## 3. DCM voltage transfer ratio: the three-interval method

DCM has three intervals in each period:

| Interval | Duration | What's happening |
|---|---|---|
| $D_1T_s$ (= $DT_s$) | Switch ON | $i_L$ rises from **zero** |
| $D_2T_s$ | Diode ON | $i_L$ falls to zero |
| $D_3T_s$ | Both OFF | $i_L = 0$, so $v_L = 0$ |

There is a new unknown, $D_2$, so two equations are needed:

1. **Volt-second balance** on L. This gives $M$ in terms of $D$ and $D_2$.
2. **Charge balance** on C: average diode (or inductor) current = load current $V_o/R$. The current waveform is a **triangle starting from zero**, peak $i_\text{pk} = \dfrac{v_{L,\text{on}}DT_s}{L}$, so the area is easy.

Then eliminate $D_2$.

### Buck in DCM

**Volt-second balance:**

$$
D(V_d - V_o) + D_2(-V_o) + 0 = 0 \;\Rightarrow\; V_o = V_d\frac{D}{D+D_2}
$$

**Charge balance.** Here $\langle i_L\rangle = I_o$, and $i_L$ is a triangle of base $(D+D_2)T_s$ and height $i_\text{pk} = \frac{(V_d-V_o)DT_s}{L}$:

$$
\frac{V_o}{R}T_s = \frac12\cdot\frac{V_d-V_o}{L}DT_s\cdot(D+D_2)T_s
$$

**Eliminate $D_2$** using $D + D_2 = D/M$:

$$
K M^2 + D^2 M - D^2 = 0 \;\Rightarrow\; \boxed{M(D,K) = \frac{2}{1+\sqrt{1+\dfrac{4K}{D^2}}}}
$$

Check: when $K \to K_\text{crit} = 1-D$, $M \to D$ (the CCM value). ✓

### Boost in DCM

**Volt-second balance:**

$$
DV_d + D_2(V_d - V_o) = 0 \;\Rightarrow\; V_o = V_d \frac{D+D_2}{D_2}
$$

**Charge balance.** Here $\langle i_D\rangle = I_o$. The diode conducts only during $D_2T_s$, as a triangle of height $i_\text{pk} = \frac{V_d DT_s}{L}$:

$$
\frac12\cdot\frac{V_d}{L}DT_s\cdot D_2T_s = \frac{V_o}{R}T_s \;\Rightarrow\; D_2 = \frac{KM}{D}
$$

**Eliminate $D_2$.** With $M = 1 + D/D_2 = 1 + \dfrac{D^2}{KM}$:

$$
M^2 - M - \frac{D^2}{K} = 0 \;\Rightarrow\; \boxed{M(D,K) = \frac{1+\sqrt{1+\dfrac{4D^2}{K}}}{2}}
$$

### Buck-boost in DCM (L7 Q2)

The slide writes the second interval as $dT$.

**Volt-second balance:**

$$
kTV_1 - dTV_2 = 0 \;\Rightarrow\; \frac{V_2}{V_1} = \frac{k}{d}
$$

**Input-current balance.** The average input current is $I_s = \dfrac{V_2^2}{RV_1}$. From the waveform it is also $\dfrac{1}{T}\cdot\dfrac12\cdot\dfrac{V_1kT}{L}\cdot kT$:

$$
\frac{V_1 k^2 T}{2L} = \frac{V_1}{R}\frac{k^2}{d^2} \;\Rightarrow\; d = \sqrt{\frac{2L}{RT}} = \sqrt K
$$

$$
\boxed{M(k,K) = \frac{k}{\sqrt K}}
$$

Note that here the diode-conduction fraction is $d = \sqrt K$. **It depends only on the load and L, not on $k$.**

**Key insight:** in DCM the buck-boost (and so the flyback) behaves as a **constant-power source**. Each cycle dumps $\tfrac12 L i_\text{pk}^2$ into the output, whatever the load, so $V_o$ depends on $R$.

---

## 4. Summary

| | $K_\text{crit}(D)$ | $M$ in DCM | CCM limit |
|---|---|---|---|
| Buck | $1-D$ | $\dfrac{2}{1+\sqrt{1+4K/D^2}}$ | $D$ |
| Boost | $D(1-D)^2$ | $\dfrac{1+\sqrt{1+4D^2/K}}{2}$ | $\dfrac{1}{1-D}$ |
| Buck-boost | $(1-D)^2$ | $\dfrac{D}{\sqrt K}$ | $\dfrac{D}{1-D}$ |

All use $K = 2L/(RT_s)$, with DCM when $K < K_\text{crit}$.

**What DCM means physically**

- The output voltage becomes **load dependent** (bad for open-loop regulation).
- Peak currents are higher for the same average current, giving higher RMS current and higher conduction loss.
- But turn-on happens at **zero current**: there is no diode reverse recovery and turn-on loss is lower.
