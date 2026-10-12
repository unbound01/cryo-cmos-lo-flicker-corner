# Flicker-corner tolerance of a PLL-based qubit LO

A small, reproducible analysis of how large a **1/f³ (flicker) phase-noise corner** a cryo-CMOS
local oscillator can have before it breaks the frequency-noise (FN) budget of a PLL-stabilised
qubit-control LO. It extends the budget of Patra et al. (JSSC 2018, Eqs. 7-9) with a 1/f³ term and
re-analyses that paper's measured 4 K data.

> **Status: unreviewed analysis, not peer reviewed.** Digitized from published figures
> (about ±0.3 dB per point). "No prior work found" means *not found in the six papers listed below*,
> not a survey of the literature.

## Results (rows marked † need your own copy of the Patra paper; see `data/README.md`)

| Quantity | Value |
|---|---|
| α_max (f_PLL = 300 kHz, 8.3 kHz to 10 MHz, 1.9 kHz rms) | **0.1842 Hz²/Hz** (Patra: < 0.2) |
| In-band PN / PN at 10 MHz at α_max | −116.9 / −147.3 dBc/Hz (Patra: −115 / −147; the 2 dB in-band gap is unexplained but changes α_max by < 0.6%) |
| Corner tolerance f_c,max at α = 0.17 / 0.15 / 0.10 (fixed floor) | 233 / 637 / 2353 kHz (continuous floor: 213 / 582 / 2149 kHz) |
| Upper band limit f_b = 1 / 2 / 5 / 10 MHz | α_max = 2.26 / 1.00 / 0.38 / 0.18 Hz²/Hz (equivalent f_R = 0.41 / 0.81 / 2.03 / 4.05 MHz) |
| Gong 2022 oscillator (β = 0.041, f_C = 1.3 MHz) | FN = 1.09 kHz; f_c,max ≈ 9 MHz (about 7× margin) |
| † Fit to Patra's 4 K data (Fig. 22) | α ≈ 0.14 Hz²/Hz, f_c ≈ 4.1 MHz (range 0.09–0.15 / 4.1–6.5 MHz); free-running FN 3.1–3.5 kHz vs Patra's 3.4 kHz |
| † Same oscillator through a PLL | 2.1–2.5 kHz at 1 MHz bandwidth; 1.9 kHz needs 2.9–3.6 MHz |

**Main findings**

1. Eq. 7 depends only on α and K = α·f_c, so f_c,max(α) is a hyperbola in (α, f_c).
2. For Patra's own measured oscillator at 6.3 GHz, the claim that a PLL of ≤ 1 MHz "can easily" reach
   1.9 kHz is **not supported** by this model.
3. K rises 10–30× on cooling for that chip, so a rising corner is not benign there (Gong 2022 reports
   roughly constant flicker for a different design).
4. The 1.9 kHz figure is the qubit's own frequency noise (van Dijk 2019, Table I), not a
   fidelity-derived oscillator limit.

Full write-up: [`docs/report.md`](docs/report.md).

## Quick start

```bash
pip install -r requirements.txt
python src/budget.py                 # baseline checkpoints + figures/budget_figures.png
python tests/test_budget.py          # 8 model tests (or: pytest); 3 data tests are skipped until step below

# optional, needs YOUR copy of the Patra et al. PDF (not included):
python scripts/digitize_patra_figs.py --pdf PATH/TO/CryoCMOS_BPatra.pdf --out data/patra_digitized.json
python scripts/fit_patra_data.py     # fits + figures/patra_fit.png (git-ignored)
python tests/test_budget.py          # now 11 tests
```

Tested with Python 3.12, numpy 2.4, scipy 1.17, matplotlib 3.10.

## Repository layout

```
src/budget.py                    model: FN budget, closed form, f_c,max, alpha_max; checkpoint printout
scripts/fit_patra_data.py        fit (alpha, f_c) to Patra Figs 22/24/25; 3.4 kHz and PLL checks
scripts/digitize_patra_figs.py   re-extract the data from YOUR copy of the paper (optional)
data/README.md                   how to generate patra_digitized.json locally (file not distributed)
tests/test_budget.py             regression tests (checkpoint values, units; fit tests need the local data)
docs/report.md                   the report
figures/                         generated figures
```

## Model in one paragraph

Oscillator L(Δf) = α/Δf²·(1 + f_c/Δf) (single-sideband). A brick-wall PLL at f_PLL replaces the
oscillator noise below f_PLL with a flat floor, either α/f_PLL² (*fixed*) or α(1 + f_c/f_PLL)/f_PLL²
(*continuous*). FN² = 2∫Δf² L dΔf from 8.3 kHz to 10 MHz. Units are Hz throughout; α = L·Δf²,
and the one-sided FN spectrum is 2α. The conventions were cross-checked against van Dijk 2019
(Table I) and Ball et al. 2016 (SI Eqs. 80–83, 112).

## Data provenance and limitations

- The digitized points are **not included**. They are derived from figures of a paper copyrighted by IEEE,
  and IEEE says that reproducing its graphics and tables normally requires permission; I found no IEEE
  statement covering digitized data points, so the repo errs on the safe side (not legal advice).
  Generate the file locally from your own copy with `scripts/digitize_patra_figs.py`; it is git-ignored.
  The overlay images the script can write, and `figures/patra_fit.png`, redraw the paper's curves: do not publish them.
- `scripts/digitize_patra_figs.py` needs poppler's `pdftoppm` and OpenCV. Its pixel calibrations are specific
  to the 13-page IEEE Xplore PDF (pages 9 and 10 at 300 dpi).
- Results marked † in the table above and in the report are my own derived numbers (fit parameters, FN values)
  with the source cited; the underlying points are about ±0.3 dB.
- Main limitations: brick-wall PLL model; the data start at 100 kHz (about 31% of the 4 K
  free-running FN² is extrapolated, but the PLL results use only data above 300 kHz); the fixed
  floor is discontinuous at f_PLL (hence the continuous variant); 1.9 kHz is a heuristic budget, not
  a measurement or a fidelity limit; fits at 165 K and 300 K are weakly constrained. See report §6.
- The closed-form-versus-quad test checks the algebra; it is not a scientific result.

## References

1. B. Patra et al., "Cryo-CMOS circuits and systems for quantum computing applications," *IEEE J. Solid-State Circuits* 53(1), 309–321, 2018.
2. J. Gong et al., "A cryo-CMOS oscillator with an automatic common-mode resonance calibration for quantum computing applications," *IEEE Trans. Circuits Syst. I* 69(12), 4810–4822, 2022.
3. J. Gong et al., "A cryo-CMOS PLL for quantum computing applications," *IEEE J. Solid-State Circuits* 58(5), 1362–1375, 2023.
4. J. P. G. van Dijk et al., "Impact of classical control electronics on qubit fidelity," *Phys. Rev. Applied* 12, 044054, 2019 (arXiv:1803.06176).
5. H. Ball, W. D. Oliver, M. J. Biercuk, "The role of master clock stability in quantum information processing," *npj Quantum Inf.* 2, 16033, 2016 (arXiv:1602.04551).
6. L. Lei et al., "Cryogenic control and readout integrated circuits for solid-state quantum computing," arXiv:2410.15895.

## License

Code: MIT, © 2026 Krishna Gupta (see `LICENSE`). Digitized data: not distributed, see above.
