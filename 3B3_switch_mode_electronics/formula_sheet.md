# 3B3 Formula Sheet (one-page revision)

> Derivations and assumptions are in the topic files. **In the exam, state your assumptions**: CCM/DCM, lossless, small ripple, large L or C, and so on.

## Fundamentals

| | |
|---|---|
| Duty | $D = t_{on}/T_s$, $f_{sw} = 1/T_s$ |
| Volt-second balance | $\langle v_L\rangle = 0$ → gives $M(D)$ |
| Charge balance | $\langle i_C\rangle = 0$ → gives the currents |
| THD | $\sqrt{F^2-F_1^2}/F_1$ |
| PF | $\dfrac{\cos\phi_1}{\sqrt{(1+\text{THD}_v^2)(1+\text{THD}_i^2)}}$ |
| RMS of DC + AC | $\sqrt{DC^2+\text{RMS}_{AC}^2}$ |
| RMS: sine / square / triangle / sawtooth | $A/\sqrt2$, $A$, $A/\sqrt3$, $A/\sqrt3$ |

## Non-isolated DC-DC (CCM)

| | Buck | Boost | Buck-boost |
|---|---|---|---|
| $M$ | $D$ | $\frac{1}{1-D}$ | $\frac{D}{1-D}$ (inverted) |
| $\Delta i_{L,pp}$ | $\frac{V_dD(1-D)}{fL}$ | $\frac{V_dD}{fL}$ | $\frac{V_dD}{fL}$ |
| $\Delta v_{C,pp}$ | $\frac{V_dD(1-D)}{8LCf^2}$ | $\frac{DI_o}{Cf}$ | $\frac{DI_o}{Cf}$ |
| $I_L$ | $I_o$ | $\frac{I_o}{1-D}$ | $\frac{I_o}{1-D}$ |

## DCM ($K = 2L/(RT_s)$; DCM if $K<K_{crit}$)

| | $K_{crit}$ | $M_{DCM}$ |
|---|---|---|
| Buck | $1-D$ | $\frac{2}{1+\sqrt{1+4K/D^2}}$ |
| Boost | $D(1-D)^2$ | $\frac{1+\sqrt{1+4D^2/K}}{2}$ |
| Buck-boost | $(1-D)^2$ | $\frac{D}{\sqrt K}$ |

## Inverters

| | |
|---|---|
| Full-bridge square wave | $v_{AB} = \frac{4V_d}{\pi}\sum_{odd}\frac{\sin n\omega t}{n}$; THD = 48.3% |
| Unipolar, zero interval α | multiply each term by $\cos(n\alpha/2)$ |
| Half bridge | $\frac{2V_d}{\pi}\sum_{odd}\frac{\sin n\omega t}{n}$ |
| 1-ph SPWM | $M = \hat V_{AB1}/V_d$, linear for $M\le1$; square-wave limit $4/\pi$ |
| 3-ph SPWM | $M = \hat V_{Ao}/(V_d/2)$; $\hat V_{AB} = \frac{\sqrt3}{2}MV_d$; linear max $0.866V_d$; six-step $\frac{2\sqrt3}{\pi}V_d$ |
| Harmonic current | $I_h = V_h/(\omega_hL)$; the EMF/grid is a short at harmonics |
| Fundamental phasor | $\bar V_1 = j\omega_1L\bar I_1 + \bar E_1$ |
| Star / delta C | $C_\Delta = C_Y/3$ |

## Rectifiers ($V_s$, $V_{ph}$, $V_L$ RMS; $f_s$ = supply frequency)

| | $V_o$ |
|---|---|
| 1-ph half-wave | $\frac{\sqrt2}{\pi}V_s = 0.45V_s$ |
| 1-ph full-wave | $\frac{2\sqrt2}{\pi}V_s = 0.9V_s$ |
| 3-ph half-wave | $\frac{3\sqrt6}{2\pi}V_{ph} = 1.17V_{ph}$ |
| 3-ph full-wave | $\frac{3\sqrt2}{\pi}V_L = 1.35V_L$ |

| | |
|---|---|
| 1-ph, C filter | $\Delta V_o \approx \frac{\sqrt2V_s}{2f_sRC}$ |
| 3-ph, C filter | $\Delta V_o \approx \frac{\sqrt2V_L}{6f_sRC}$ |
| 1-ph, R-L load | 2nd harmonic $\hat V_{o2} = \frac{4\sqrt2V_s}{3\pi}$; $\Delta I_{o,pp} \approx \frac{4\sqrt2V_s}{3\pi\omega_sL}$ |
| 1-ph AC current (large L) | square wave; $I_{n,rms} = \frac{2\sqrt2}{n\pi}I_o$ |
| 3-ph AC current (large L) | $\hat I_n = \frac{2\sqrt3}{n\pi}I_o$, $n = 6k\pm1$ |
| Output harmonics | 1-ph full-wave: $2, 4, 6, \ldots$; q-phase full-wave: $2q, 4q, \ldots$ (3-ph: $6, 12, 18, \ldots$) |

## Isolated

| | |
|---|---|
| Ideal transformer | $v_1/v_2 = n_1/n_2$, $i_1/(-i_2) = n_2/n_1$ |
| Magnetising inductance | $L_M = n_1^2/\mathcal R_c = n_1^2\mu A_c/l_m$; $A_L = L_M/n^2$; $L_{M1}/L_{M2} = (n_1/n_2)^2$ |
| Push-pull | $M = \frac{N_2}{N_1}D$ (D over half-cycle T) |
| Flyback | $M = \frac{N_2}{N_1}\frac{D}{1-D}$ |
| Flux | $\Delta\phi = \frac{V\Delta t}{N}$, $\hat\phi = B_{max}A_c$ |
| Flyback BCM energy | $P = \tfrac12L_mI_{pk}^2f$ |

## Resonant / LLC (FHA)

| | |
|---|---|
| Tank | $\omega_0 = 1/\sqrt{L_rC_r}$, $\omega_n = \omega_s/\omega_0$, $Q = \sqrt{L_r/C_r}/R_s$ |
| Series LC gain | $\lvert M\rvert = \left[1+Q^2(\omega_n-1/\omega_n)^2\right]^{-1/2}$ |
| LLC gain ($L_m = kL_r$) | $\lvert M\rvert = \left[\left(1+\frac1k(1-\frac1{\omega_n^2})\right)^2+Q^2(\omega_n-\frac1{\omega_n})^2\right]^{-1/2}$ |
| Rectifier $R_s$, L filter | $\frac{\pi^2}{8}R$ (×$N^2$ to primary) |
| Rectifier $R_s$, C filter | $\frac{8}{\pi^2}R$ (×$N^2$ to primary) |
| $V_o$, L filter | $\frac{8M}{\pi^2N}V_d$ |
| $V_o$, C filter | $\frac{M}{N}V_d$ |
| Bridge fundamental | $\hat V_{AB1} = 4V_d/\pi$ |

## Losses

| | |
|---|---|
| MOSFET conduction | $R_{ds,on}I_{rms}^2$ |
| Diode / IGBT conduction | $V_fI_{avg}$ / $V_{CE}I_{avg}$ |
| Buck switch | $I_{rms}^2 = D(I_o^2+\Delta I_L^2/12)$ |
| Buck diode | $P = V_fI_o(1-D)$ |
| Switching | $P_{sw} = (E_{on}+E_{off})f$ |
| Linear-overlap approximation | $P_{sw} \approx \frac{V_{DC}I_o}{2}\cdot\frac{t_{ri}+t_{fv}}{T}$ (same form for turn-off) |
| Current-rise interval | $I_{G2} = \frac{V_{GG}-\frac12(V_{GS,I_o}+V_{th})}{R_G}$, $t_2 = \frac{(V_{GS,I_o}-V_{th})C_{iss}}{I_{G2}}$ |
| Voltage-fall interval | $I_{G3} = \frac{V_{GG}-V_{GS,I_o}}{R_G}$, $t_3 = \frac{V_{DC}C_{rss}}{I_{G3}}$ |
| Capacitances | $C_{iss} = C_{GS}+C_{GD}$, $C_{rss} = C_{GD}$ |
