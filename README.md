# Fast Generation of Metrologically Relevant Fock State Mixtures: simulation code

This repository contains the simulation code for the paper

> G. Reina Rivero, M. Morillas-Rozas, A. López-García and J. Cerrillo,
> *Fast Generation of Metrologically Relevant Fock State Mixtures* (2026).

It generates every figure and every simulated number in the paper. Each result is reproducible from one Jupyter notebook. The cached simulation results are included, so the figures can be redrawn in minutes on a laptop.

Archived version: [doi:10.5281/zenodo.24470822](https://doi.org/10.5281/zenodo.24470822)
· Development: <https://github.com/GonReina/MixturesPaper>

---

## Contents

```
paper_figures.ipynb         every figure of the paper (Figs. 1-7), three additional figures (S1-S3)
                            and the numbers quoted with them
supplementary_checks.ipynb  numbers quoted in the text but not plotted (mostly answers to referee comments)
figgen_cache.pkl            cached simulation results used by both notebooks (~7 MB)
figures/                    figures written by the notebooks (the PDFs/PNG used in the manuscript)
development/                two earlier notebooks, kept for the record (older model, see below)
requirements.txt            Python packages
CITATION.cff                citation metadata
```

Both main notebooks are stored **with their outputs**. You can read the figures, the printed numbers and the explanations on GitHub without running anything.

## Quick start

```bash
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab paper_figures.ipynb                        # then Run All
```

Or headless:

```bash
jupyter nbconvert --to notebook --execute --inplace paper_figures.ipynb
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=-1 supplementary_checks.ipynb
```

| run | time |
|---|---|
| `paper_figures.ipynb` with the shipped cache | a few minutes on a laptop (Fig. 6 builds its readout matrices; everything else is read from the cache) |
| `supplementary_checks.ipynb` | about 2–3 h on a laptop (serial, uncached) |
| `paper_figures.ipynb` from an empty cache | tens of CPU-hours; see [Recomputing from scratch](#recomputing-from-scratch) |

Tested with Python 3.12, numpy 2.2, scipy 1.16, matplotlib 3.10 and QuTiP 5.2, on Windows 11 for the cached runs and on Linux for the full parallel runs. No LaTeX installation is needed: the figures use matplotlib's Computer Modern mathtext.

---

## The physics in brief

A trapped ion has a spin, the qubit $|{\uparrow}\rangle,|{\downarrow}\rangle$, and one motional mode of frequency $\nu$. The aim is to prepare a **mixture of Fock states**. The earlier *selective population trapping* (SPT) protocol, Delakouras, Rodríguez and Cerrillo, [Quantum Sci. Technol. **9**, 015006 (2023)](https://doi.org/10.1088/2058-9565/ad01d7), does this with weak red-sideband pulses ($\Omega\ll\nu$). The paper drives the carrier resonantly, with zero detuning and $\Omega=\nu$. Starting from a Doppler-cooled ion, this prepares the target two orders of magnitude faster than weak-driving SPT, with a three times lower error. Each cycle of the **resonant sequence** is

$$Y(\pi/2)\;\to\;X(f\tau)\;\to\;Y(-\pi/2)\;\to\;\text{spin reset }\mathcal L_\Gamma\;\to\;\text{free evolution }t_d .$$

- **$X$ pulse.** At $\Omega=\nu$ it couples $|-,n\rangle\leftrightarrow|+,n-1\rangle$ resonantly, with strength $\propto\eta\sqrt n$. A pulse of length $\tau=2\pi/(\eta\Omega\sqrt{n_0})$ makes a whole number of turns only for $n=n_0m^2$, so those levels are dark.
- **Reset.** It moves the population of every other level down by one quantum per cycle, until it reaches a dark level.
- **Target.** Starting from a thermal state, the motion converges to
  $$\rho_{\rm tr}=\sum_m p_{\rm tr}(m)\,|n_0m^2\rangle\langle n_0m^2|,\qquad p_{\rm tr}(m)=\sum_{n_0m^2\le n<n_0(m+1)^2}p_{\rm th}(n).$$
  This mixture beats the standard quantum limit for displacement sensing.
- **Calibration.** The counter-rotating (anti-Jaynes–Cummings) term of the $X$ pulse limits the fidelity at larger $\eta$. A **timing calibration**, a slightly longer pulse $f\tau$ and a short free evolution $t_d$ after each reset, lowers the error of a Doppler-cooled ion ($\bar n\simeq10.5$) by up to a factor of three, to 2.5% at $\eta=0.05$. It also keeps the displacement-sensing Fisher information above the standard quantum limit up to $\eta=0.1$.

### Model used for every figure

| quantity | value |
|---|---|
| units | $\hbar=\nu=1$; physical units use $\nu=2\pi\times1$ MHz |
| initial state | Doppler-cooled thermal state, $k_BT=\hbar\Gamma_D/2=11\nu$ ($\Gamma_D=22\nu$, the $^{40}$Ca$^+$ $S_{1/2}$–$P_{1/2}$ line), $\bar n\simeq10.5$ |
| Fock-space truncation | $N=100$ levels (thermal weight above $N$: $10^{-4}$; $N=80$ and $N=100$ agree to $5\times10^{-4}$) |
| cycles per preparation | $R=80$ |
| $X$ pulse | $H_X=\nu a^\dagger a+\tfrac\Omega2(\sigma^+D(i\eta)+{\rm h.c.})$, $\Omega=\nu$, duration $f\tau$; exact lab-frame propagator, no Lamb–Dicke or rotating-wave approximation |
| $Y$ pulses | $H_Y=\nu a^\dagger a+\tfrac{\Omega_Y}2(e^{i\phi}\sigma^+D(i\eta)+{\rm h.c.})$ for $\pi/(2\Omega_Y)$; $\phi=-\pi/2$ for $Y(\pi/2)$, and $\phi=+\pi/2$ (phase-flipped) for $Y(-\pi/2)$ |
| spin reset | Lindblad evolution with $H=\nu a^\dagger a$ and jumps $\sigma^-e^{i\eta' c\,x}$ ($x=a+a^\dagger$), dipole pattern $W(c)=\tfrac38(1+c^2)$, for $20/\Gamma$ (complete) |
| reference pulses and reset | $\Omega_Y=100\nu$, $\Gamma=1000\nu$, $\eta'=\eta$ |
| $^{40}$Ca$^+$ model (Fig. 7) | $\Omega_Y=2\nu$ or $\nu$; 854-nm quench reset with $\Gamma_{\rm eff}=3\nu$, 393-nm recoil $\eta'=0.18$, 854-nm kick $0.854\eta$, 6.5% decay back to $D$ |
| figure of merit | trace distance between Fock populations, $D=\tfrac12\sum_n\lvert p_n-P^{\rm tr}_n\rvert$ |

---

## `paper_figures.ipynb`: the figures

The notebook reads top to bottom:
1. a parameter block, the single source of truth;
2. the building blocks of the model: pulses, reset, cycle iteration, calibration;
3. one section per figure.

Each section explains what is computed before the code. The printed lines under each figure are the numbers quoted in the text.

| paper | file in `figures/` | content |
|---|---|---|
| Fig. 1 | `figure_1.png` | concept: (A) dressed-state ladder, (B) pulse sequence, (C) classical picture (no simulation) |
| Fig. 2 | `figure_2.pdf` | initial and final populations for $\eta=0.01$–$0.2$ |
| Fig. 3 | `figure_3_plus_trace.pdf` | weak-driving SPT vs resonant sequence: (A) vs $\Omega$, (B) vs $\eta$ |
| Fig. 4 | `figure_preparation.pdf` | (A) error vs preparation time, (B) error vs initial $\bar n$ |
| Fig. 5 | `figure_5.pdf` | timing calibration: (A) $(f,t_d)$ map at $\eta=0.05$, (B) error vs $\eta$ |
| Fig. 6 | `figure_6.pdf` | metrology: Fisher information and gain over the SQL |
| Fig. 7 | `figure_ca.pdf` | $^{40}$Ca$^+$: (A) error vs $\eta$ with realistic $Y$ pulses and quench reset, (B) calibration errors in $\Omega$, pulse length and $\nu$ |

Three additional figures are not in the paper. They support statements made in its text, and Fig. 7 is built from their results:

| figure | file in `figures/` | content | supports |
|---|---|---|---|
| S1 | `figure_S1_pulse_shaping.pdf` | pulse shaping: envelopes, plateau and chirp, $\delta\Omega(t)$ profiles | Sec. III B: shaping does not improve on the square pulse |
| S2 | `figure_S2_tolerances.pdf` | (A) calibration errors, as Fig. 7(B); (B) laser detuning (noise and heating printed) | Sec. IV |
| S3 | `figure_S3_implementation.pdf` | (A) $^{40}$Ca$^+$ models vs $\eta$, as Fig. 7(A); (B) dependence on $\Omega_Y$ | Sec. IV |

### Numerical methods

- **Pulses.** Pulses are exact propagators $e^{-iHt}$ on the $2N$-dimensional spin $\otimes$ motion space, computed by eigendecomposition.
- **Reset.** The reset is applied to $\rho$ directly instead of through a $(2N)^2\times(2N)^2$ superoperator, which would need about 25 GB at $N=100$. The recoil average is diagonal in the eigenbasis of $x$. The integral over the emission time uses Gauss–Legendre quadrature: 48 nodes for the fast reset, and at least 120 for the slow $^{40}$Ca$^+$ reset, which the notebook checks against 1.5 times as many. This agrees with the dense superoperator to about $10^{-12}$.
- **Timing calibration.** For each $\eta$, $(f,t_d)$ is first optimised on a grid, $f=1+\eta\times\{-0.4,\dots,0.4\}$ times $t_d=2\pi k/16$, and then refined locally (`calibrate_many`).
- **Noise and heating.** Rabi-frequency noise is averaged with 9-point Gauss–Hermite quadrature. Heating is the Lindblad bath $g\,\mathcal D[a]+g\,\mathcal D[a^\dagger]$.

### The cache

`figgen_cache.pkl` stores every expensive result under a key made of three parts:
- the physical parameters ($k_BT$, $N$, $R$, $\Omega_Y$, $\Gamma$, and the reset model);
- the construction of $Y(-\pi/2)$;
- every argument of the run.

Changing any of these in the parameter block therefore never reuses stale data. **If you change the physics in a way the keys do not capture**, for example by editing a Hamiltonian or the reset, delete the cache.

The cache is written atomically and saved every few minutes, so an interrupted run resumes where it stopped.

### Recomputing from scratch

Independent runs are spread over `NPROC` worker processes with the `fork` start method, which needs Linux or macOS; on Windows the notebook runs serially. The last full revision recomputed about 30 CPU-hours of results in about one hour on 32 cores. On a server:

```bash
mv figgen_cache.pkl figgen_cache.old.pkl              # or keep it to reuse what is still valid
NPROC=32 jupyter nbconvert --to notebook --execute --inplace \
    --ExecutePreprocessor.timeout=-1 paper_figures.ipynb
```

`NPROC` and `SMOKE_TEST` are read when the first code cell runs, so set them in the environment of the kernel. With nbconvert, set them on the command line as above.

To check the code end to end in about a minute, run the smoke test, which uses $N=30$, $R=6$ and tiny grids, with its cache and figures in a temporary folder:

```bash
SMOKE_TEST=1 jupyter nbconvert --to notebook --execute --output smoke.ipynb paper_figures.ipynb
```

---

## `supplementary_checks.ipynb`: numbers not shown in a figure

This notebook loads the model by executing the code cells of `paper_figures.ipynb` above the `# ==== FIGURES` marker, so the two cannot drift apart. It then reproduces:

| section | result | paper |
|---|---|---|
| 1 | Removing the motional coherences after every reset destroys the calibration gain (0.025 → 0.065 at $\eta=0.05$); a timing jitter of 0.1 rad per cycle is harmless | Sec. III B |
| 2 | The prepared state is displaced by about half a photon recoil, but only partly coherently; an end kick $D(i\eta/2)$ lowers the error only from 0.025 to 0.021 | Sec. II B |
| 3 | A tilted 729-nm beam couples to a radial mode, whose thermal motion spreads $\Omega$: 0.9% rms for a 3 MHz mode (error 0.025 → 0.070), 0.3% for 5 MHz (unchanged) | not in the paper (reply to the referees) |
| 4 | Magnetic field perpendicular to the trap axis (needed for $\Delta m=-2$ with the beam along the axis): the 393-nm emission pattern lowers the error slightly (0.149 → 0.147 at $\eta=0.097$), so the paper's model is conservative | not in the paper (reply to the referees) |
| 5 | The error peaks when the full cycle lasts a whole number of trap periods (a stroboscopic resonance). This explains why commensurate $\eta$ are maxima of the nominal error, the bands of Fig. 5(A) and the narrow peaks of Fig. 7(B) | Sec. III B |

Section 5 also writes `figures/supp_cycle_resonance.pdf`. This figure is not in the paper.

---

## `development/`: earlier notebooks

These notebooks record how the paper's analysis developed. They are kept, with their original conclusions, because the paper's discussion grew out of them. Each starts with a note on how its model differs from the paper's. **Their numbers are not the paper's numbers.**

| notebook | what it shows | how its model differs from the paper's |
|---|---|---|
| `improved_pulse_sequence.ipynb` | where the leakage of the resonant sequence comes from (the counter-rotating term, by deleting it from the exact Rabi model), and the first tests of the refined sequence (pulse-length refocusing plus a correction term) | $\bar n=1$–$5$, $N=16$, RF kicks, exact inverse $Y^\dagger$, two-channel reset of $4/\Gamma$, recoil-free $\tfrac\delta2\sigma_x$ term |
| `pulse_shaping.ipynb` | why amplitude envelopes do not help the resonant sequence, while detuning-type corrections do | same as above |

Run them from inside `development/`.

---

## Conventions

- **Spin basis.** The spin basis is ordered $\{|{\uparrow}\rangle,|{\downarrow}\rangle\}$, with $\sigma^+=|{\uparrow}\rangle\langle{\downarrow}|$. The reset pumps to $|{\downarrow}\rangle$.
- **$Y$-pulse sign.** The $Y(\pi/2)$ pulse is $\tfrac{\Omega_Y}{2}(-i\sigma^+D+i\sigma^-D^\dagger)$. It maps $|{\downarrow}\rangle$ to the dressed state $|-\rangle$; the opposite sign maps it to $|+\rangle$ and destroys trapping.
- **$Y(-\pi/2)$.** This is the phase-flipped pulse an experiment applies, not the exact inverse of $Y(\pi/2)$. The two differ because the trap evolution during the pulse does not change sign.
- **Names.** "Weak-driving SPT protocol" is the earlier scheme. "Resonant sequence" is the new one: "nominal" for $f=1$, $t_d=0$, and "calibrated" after the timing calibration.

## Citation

If you use this code, please cite the paper and this archive; `CITATION.cff` contains the metadata.

```bibtex
@misc{reinarivero_fockmixtures_code,
  author = {Reina Rivero, Gonzalo and Morillas-Rozas, Marcel and L{\'o}pez-Garc{\'i}a, Alberto and Cerrillo, Javier},
  title  = {Simulation code for ``Fast Generation of Metrologically Relevant Fock State Mixtures''},
  year   = {2026},
  doi    = {10.5281/zenodo.XXXXXXX},
  url    = {https://github.com/GonReina/MixturesPaper}
}
```

## License
MIT License. See `LICENSE` for details.

## Acknowledgements

The authors acknowledge support from grant CNS2023-144994 funded by MICIU/AEI/10.13039/501100011033 and by "ERDF/EU", and from the European Union project C-QuENS (Grant No. 101135359).

Research Group of Quantum Technologies, Universidad Politécnica de Cartagena (member of EUT+), Spain.
