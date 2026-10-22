# 06 — Uncontrolled Diode Rectifiers (L8, L9)

> **Source:** L08 (single-phase) and L09 (three-phase plus a worked exam-style question).
> **Ideal AC source** (no source impedance) and ideal diodes throughout. A diode conducts whenever it is forward biased, so **the diode connected to the most positive (or most negative) voltage conducts**.
> $V_s$ = RMS supply voltage, $\omega_s = 2\pi f_s$ = **supply** angular frequency (careful: in this topic $f_s$ is the **supply** frequency, not the switching frequency).

---

## 1. Single-phase half-wave, resistive load

$v_s = \sqrt2V_s\sin\omega_st$. The diode conducts only in the positive half cycle:

$$
v_o = \begin{cases}\sqrt2V_s\sin\omega_st & 0\le\omega_st\le\pi\\ 0 & \pi<\omega_st<2\pi\end{cases}
$$

$$
\boxed{V_o = \frac{1}{2\pi}\int_0^\pi\sqrt2V_s\sin\theta\,d\theta = \frac{\sqrt2}{\pi}V_s = 0.45V_s}
$$

This is poor voltage utilisation. The output contains DC, the fundamental and the even harmonics:

$$
v_o = \frac{\sqrt2V_s}{\pi} + \frac{\sqrt2V_s}{2}\sin\omega_st - \frac{2\sqrt2V_s}{\pi}\sum_{n=2,4,6,\ldots}\frac{\cos n\omega_st}{n^2-1}
$$

---

## 2. Single-phase full-wave bridge, resistive load

D1 and D2 conduct in the positive half cycle, D3 and D4 in the negative half cycle. The output is $|v_s|$:

$$
\boxed{V_o = \frac1\pi\int_0^\pi\sqrt2V_s\sin\theta\,d\theta = \frac{2\sqrt2}{\pi}V_s = 0.9V_s}
$$

$$
v_o = \frac{2\sqrt2V_s}{\pi} - \frac{4\sqrt2V_s}{\pi}\sum_{n=2,4,6,\ldots}\frac{\cos n\omega_st}{n^2-1}
$$

Only **even** harmonics appear, and the **2nd is the largest**.

---

## 3. Full-wave bridge with R-L load

The **average output voltage is unchanged** ($0.9V_s$), because the bridge output voltage is still $|v_s|$. The inductor smooths the **current**.

The dominant ripple is the **2nd harmonic**:

$$
v_{o2} = -\frac{4\sqrt2V_s}{3\pi}\cos2\omega_st
$$

At $2\omega_s$, $L$ dominates, so

$$
\hat I_{o2} = \frac{\hat V_{o2}}{|j2\omega_sL + R|} \approx \frac{4\sqrt2V_s}{3\pi}\cdot\frac{1}{2\omega_sL}
$$

$$
\boxed{\Delta I_{o,pp} = 2\hat I_{o2} = \frac{4\sqrt2V_s}{3\pi\,\omega_sL}}
$$

> ⚠️ **Slide (L8 p5):** the last line writes $\Delta I_o = 2I_{o2} = \frac{4\sqrt2V_s}{3\pi}\cdot\frac{1}{\omega_2L}$ with $\omega_2 = 2\omega_s$. That expression equals $\hat I_{o2}$ itself, so the factor of 2 has been dropped. Peak-to-peak is twice the amplitude, which is the result above.

### AC-side current

With large $L$, $i_o \approx I_o$ (flat), so the supply current is a **square wave of $\pm I_o$**:

$$
i_s = \sum_{n=1,3,5,\ldots}\frac{4I_o}{n\pi}\sin(n\omega_st), \qquad I_{sn,\text{rms}} = \frac{2\sqrt2}{n\pi}I_o
$$

*Derived (useful for PF questions):* $I_{s1,\text{rms}} = 0.9I_o$ and $I_{s,\text{rms}} = I_o$. The fundamental is in phase with $v_s$, so $\text{PF} = 0.9$ and $\text{THD}_i = 48.3\%$.

---

## 4. Full-wave bridge with capacitor filter (the common one)

The diodes conduct only while $|v_s|$ exceeds the capacitor voltage, which is a **short interval near each peak**.

| Interval | What happens |
|---|---|
| $\alpha < \omega t < \beta$ | Diodes ON. The supply charges C and feeds R: $i_D = \sqrt2V_s\left(\omega_sC\cos\omega_st + \frac{\sin\omega_st}{R}\right)$ |
| $\omega t = \beta$ | $i_D$ reaches zero and the diodes turn off: $\beta = \tan^{-1}(-\omega_sRC)$, taken in the **2nd quadrant**, i.e. $\beta = \pi - \tan^{-1}(\omega_sRC)$, just after the peak |
| $\beta<\omega t<\pi+\alpha$ | Diodes OFF. C discharges exponentially into R |
| $\omega t = \pi+\alpha$ | $|v_s|$ catches up with $v_o$ and the next charging pulse starts. **Finding $\alpha$ is not examinable** |

### Simplified ripple ⭐ (examinable; learn the assumptions)

**Assumptions:**

1. $RC \gg \pi/\omega_s$, so the discharge is approximately **linear**.
2. Charging is instantaneous ($\alpha = \beta = \pi/2$), so C discharges for the **whole half cycle**.

$$
\Delta V_o = \sqrt2V_s - \sqrt2V_se^{-\frac{\pi}{2\pi f_sRC}} \approx \sqrt2V_s\left(1 - \left(1-\frac{1}{2f_sRC}\right)\right)
$$

$$
\boxed{\Delta V_o \approx \frac{\sqrt2V_s}{2f_sRC}}
$$

The ripple depends on **both C and the load**. Equivalently $\Delta V_o \approx \dfrac{I_o}{2f_sC}$, which is just $Q = It$ over half a supply period.

### Why the power factor is poor

The current flows in **short, high-amplitude pulses** near the voltage peaks. That current is very distorted (high THD), so $\text{PF} = \cos\phi_1/\sqrt{1+\text{THD}^2}$ is low **even though the pulses are roughly in phase with the voltage**.

**Combined L-C smoothing** (an L between the bridge and C) spreads the current pulses out and improves the PF.

---

## 5. Three-phase half-wave rectifier

This needs the supply **neutral** for the return. The diode on the **most positive phase** conducts, each for 120°:

$$
\boxed{V_o = \frac{1}{2\pi/3}\int_{\pi/6}^{5\pi/6}\sqrt2V_{ph}\sin\theta\,d\theta = \frac{3\sqrt6}{2\pi}V_{ph} = 1.17V_{ph}}
$$

Each phase carries current only in its positive half cycle. That gives a DC component in each phase current and a poor PF.

---

## 6. Three-phase full-wave (six-pulse) bridge ⭐

Six diodes, D1–D6. At any instant, the pair connected across the **highest line voltage** conducts.

- The output is made of **6 segments per cycle**, each $\pi/3$ long. Each line voltage ($v_{ab}, v_{ac}, v_{bc}, v_{ba}, v_{ca}, v_{cb}$) takes its turn.
- Each diode conducts for $2\pi/3$, and each pair for $\pi/3$.
- The sequence starts with $v_{ab}$ for $\pi/6\le\omega t<\pi/2$, then $v_{ac}$ for $\pi/2\le\omega t<5\pi/6$, and so on.

$$
\boxed{V_o = \frac{1}{\pi/3}\int_{\pi/6}^{\pi/2}\sqrt2V_{ph}\left[\sin\theta - \sin\!\left(\theta-\tfrac{2\pi}{3}\right)\right]d\theta = \frac{3\sqrt2}{\pi}V_L = 1.35V_L}
$$

Here $V_L$ is the RMS line voltage. *Peak-to-peak ripple without filtering:* $\sqrt2V_L(1-\cos30°) \approx 13.4\%$ of the peak.

### Output harmonics

- The output repeats 6 times per supply cycle, so harmonics appear only at **6, 12, 18, …** × $f_s$. Higher orders are smaller.
- In general, a **q-phase full-wave** rectifier has **2q pulses** per cycle, with harmonics at $2q, 4q, 6q, \ldots$. For example, a 5-phase rectifier has 10 ripples per 20 ms.

### AC line current (large L, flat $I_o$)

Each line current is a **quasi-square wave**: $+I_o$ for 120°, zero for 60°, $-I_o$ for 120°, zero for 60°. It is an odd function, so only odd harmonics appear. In a three-wire system the **triplen harmonics cancel** (there is no return path):

$$
\boxed{I_n = \frac{2\sqrt3}{n\pi}I_o,\qquad n = 1, 5, 7, 11, 13, 17, \ldots \;(= 6k\pm1)}
$$

The formula gives peak amplitudes. *Derived:* $I_{1,\text{rms}} = \frac{\sqrt6}{\pi}I_o \approx 0.78I_o$ and $I_\text{rms} = \sqrt{2/3}\,I_o \approx 0.816I_o$, so the distortion factor $I_1/I$ is $3/\pi = 0.955$.

---

## 7. Worked exam question (L9 example) ⭐

### Part A: single-phase bridge with C and R, 230 V 50 Hz

**(i) Sketches.** $v_C$ is a sawtooth-like ripple that just touches the $|v_s|$ peaks. $i_s$ is narrow alternating pulses at the voltage peaks.

**(ii) Poor PF.** The input current is discontinuous and flows only while $|v_s| > v_C$. The short, large pulses mean heavy harmonic distortion and so a poor power factor.

**(iii) Ripple.** Use the simplified result above, $\Delta V_o \approx \dfrac{\sqrt2V_s}{2f_sRC}$, with both assumptions stated.

### Part B: three-phase bridge with C, 415 V line, 50 Hz, $I_o$ = 100 A

**(i) C for 5% ripple.** With 6 pulses, C discharges for **1/6 of a cycle** ($\pi/3$) instead of half:

$$
\Delta V_o \approx \frac{\sqrt2V_L}{6f_sRC}
$$

A 5% ripple means the average sits about **2.5% below the peak**:

$$
V_o = 0.975\sqrt2\cdot415 = 572\ \text{V}, \qquad R = \frac{572}{100} = 5.72\ \Omega
$$

$$
\frac{\sqrt2\cdot415}{6\cdot50\cdot5.72\cdot C} = 0.05\sqrt2\cdot415 \;\Rightarrow\; C = 11.65\ \text{mF}
$$

Examiner's notes from the slide:

- Taking $V_o \approx \sqrt2V_L$ (factor 1) is accepted if you state it.
- Using $1.35V_L$ ($3/\pi$ factor) is **wrong**: that is the unfiltered average.

**(ii) Add a series L for 1% ripple.** The main harmonic is at **300 Hz** (the 6th). There, $|Z_C| = 1/(2\pi\cdot300\cdot11.65\text{m}) = 0.045\ \Omega \ll R$, so the load is effectively just C. Order of placement: **L in series, then C across the load.**

The slide sets $|Z_C|/|Z_L| = 1/99$:

$$
L = \frac{99}{36\omega_s^2C} = 2.39\ \text{mH}
$$

> ⚠️ **Check this with your supervisor.** The slide gives no justification for the 1/99 ratio. It treats the ripple at the bridge output as if it were the full peak voltage (100%) and asks for 1% at C. That is the same "1 : 99 divider" idea as Tutorial 1, and it is a conservative design. (Also: the slide's $Z_C$ evaluates to $-j0.045\ \Omega$; the printed "0.0045" is a typo.)

**(iii) Effect of L on the input current.** Without L the input current is very pulsed and the PF is very poor. The DC inductor keeps the current flowing and commutating between diodes. With large L, each line current conducts for **$2\pi/3$ in each half cycle** (the quasi-square wave of §6), which gives much lower distortion and a much better PF.
