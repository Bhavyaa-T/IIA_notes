# 3B3 Switch-Mode Electronics: Notes

Cambridge Part IIA Engineering, 2026–27. Lecturers: Dr Stephan Goetz (L1–8, L10–13) and Prof Teng Long (L9, L14–16).

These notes are rewritten from the 16 lecture decks in [`source_slides/`](source_slides/). Every derivation has been re-done and the worked examples re-checked. Where the slides are wrong or unclear, the notes say so (see **Slide errata** below).

Maths is LaTeX (`$...$`). It renders in VS Code's Markdown preview, Joplin, Obsidian and GitHub.

## Contents

| File | Lectures | Topics |
|---|---|---|
| [01_basics.md](01_basics.md) | L1 | Why switch-mode, switching function, avg/RMS, THD, PF, **volt-second and charge balance**, device overview |
| [02_non_isolated_dcdc_ccm.md](02_non_isolated_dcdc_ccm.md) | L2, L7 | Buck, boost, buck-boost in CCM: $M(D)$, ripple current/voltage, **recipe for unfamiliar converters** (Zeta) |
| [03_dcm.md](03_dcm.md) | L3, L7 | DCM origin, $K$ and $K_{crit}$, DCM transfer ratios for buck, boost, buck-boost |
| [04_inverters_single_phase.md](04_inverters_single_phase.md) | L4, L5 | Half-bridge leg, dead time, H-bridge bipolar/unipolar, Fourier and THD, PWM/SPWM, over-modulation, FET/IGBT/SiC/GaN |
| [05_three_phase_inverter_and_filters.md](05_three_phase_inverter_and_filters.md) | L6, L7 | Three-phase SPWM, modulation depth, superposition at $f_1$ and harmonics, LCL filters, **Tutorial 1 Q1 grid filter design** |
| [06_rectifiers.md](06_rectifiers.md) | L8, L9 | 1-ph and 3-ph diode rectifiers, R/RL/C loads, ripple, AC-side harmonics, PF, **L9 exam-style example** |
| [07_isolated_dcdc.md](07_isolated_dcdc.md) | L10, L11, L13 | Transformer model, $L_M$, dot convention, push-pull, **flyback** (incl. BCM design example) |
| [08_resonant_llc.md](08_resonant_llc.md) | L12, L13 | FHA, series LC and **LLC** gain, Q and k, rectifier equivalent resistance, Tutorial 2 LLC design |
| [09_device_losses.md](09_device_losses.md) | L14–L16 | Conduction/switching loss, RMS of ripple currents, synchronous buck and dead time, Miller plateau, reverse recovery, **SMPS loss example** |
| [formula_sheet.md](formula_sheet.md) | All | One-page revision sheet |

## How to revise from these

1. Learn the **recipe** at the top of 02 (sub-circuits, then volt-second balance, ripple, charge balance). Exam questions often hand you a converter you haven't seen.
2. For each worked example, cover the solution and redo it. The examples are L7 Q1/Q2, L9, L13 flyback/LLC and L16.
3. Learn the **assumptions** that go with each approximation (small ripple, linear capacitor discharge, FHA, constant $I_o$ in transients). Marks are given for stating them.
4. Use PLECS (licence via Moodle) to check examples-paper answers. It is not examinable.

## Slide errata and things to check ⚠️

I checked the equation errors against the rendered slide pages or the slides' original equation objects, not just extracted text. The "simplified" and "check" items are my judgement.

| Where | Slide says | Correct / note |
|---|---|---|
| **L2**, boost ripple slide | "high load (smaller R) lower ripple" | **Backwards.** The slide's own formula $\Delta v = DV_d/(RCf(1-D))$ gives **larger** ripple for smaller R |
| **L2**, "Boost – Complete" | $V_{out}/V_{in} = 1/D$ | Not wrong: there D is the **upper**-switch duty. Elsewhere D is the low-side switch duty and $M = 1/(1-D)$ |
| **L6**, LCL slide | $C_{delta} = 3C_{star}$ | **Backwards**: $C_{delta} = C_{star}/3$. This is consistent with the slide's own statement that delta reduces the required capacitance |
| **L6**, 3-ph SPWM | "harmonics only at integer multiples of $f_s$" | Simplified: they are **sidebands around** $mf_s$ (e.g. $f_s\pm2f_1$) |
| **L7**, Zeta converter | $-\frac{kV_i}{(1-k)T}$ | Stray $T$ (dimensionally wrong). The final answer $k/(1-k)$ is right |
| **L7 Q1**, $L_f$ | $v_{Cf}/v_{Lf} = 1/99$ → 24.75 mH | Accepted answer. Strictly the two voltages are in antiphase, giving 1/101 → 25.25 mH. State your assumption |
| **L8**, R-L load | $\Delta I_o = 2I_{o2} = \frac{4\sqrt2V_s}{3\pi}\frac{1}{\omega_2L}$ | Drops a factor of 2. $\Delta I_{o,pp} = \frac{4\sqrt2V_s}{3\pi\omega_sL}$ |
| **L8**, C filter | $\beta = \tan^{-1}(-\omega_sRC)$ | Take the **2nd-quadrant** value: $\beta = \pi - \tan^{-1}(\omega_sRC)$ |
| **L9**, Part B(ii) | $Z_c = -j0.0045\ \Omega$; $Z_C/Z_L = 1/99$ | $Z_c = -j0.045\ \Omega$ (typo). The 1/99 ratio is not justified on the slide; **ask your supervisor** |
| **L11**, transformer design | "$n_2$ = 4 turns for 1/10 output" | Only at $D = 0.5$ for a flyback |
| **L13**, LLC 20% load | $f_n \approx 1.8$ | The series-LC formula (used for Q = 6) gives ≈1.95. 1.8 is presumably read from the LLC curves |
| **L15 deck** | Diode/IGBT loss $= V\cdot I_{rms}$ | **Wrong** (an old copy of L14). Use $V\cdot I_{avg}$, as in the L14 2025-corrected deck |
| **L16**, datasheet mapping | $t_{d(on)} = t_2$ (current rise) | Lecturer's convention. Real datasheets define $t_{d(on)}$ as a delay. Use the lecture convention in the exam |

Anything labelled *"not on the slides"* or *"derived"* in the notes is my addition (standard results, or sanity checks). It is useful, but it is not lecture content.

## Not yet covered

- Examples papers and past Tripos questions. Add them to this folder and I can work through them and link them to the relevant sections.
- The handwritten tutorial solutions for L7/L13 parts that only exist as figures (sketches).
