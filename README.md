# Geopolitics

Replication package for:

> **Oil Market Crises, Macro-Fiscal Policy and Sustainable Futures: Distributive
> Conflict in Malaysia's Energy Geopolitics**


A two-player, two-strategy non-cooperative game between Malaysia and the Rest
of the World, evaluated across four oil-price regimes. Payoffs are computed
from a component calibration rather than assigned. Domestic coalition
resistance enters as a regime-dependent capture cost derived from observable
fiscal ratios. A rules-based Green Stabilisation Fund enters as an exogenous
policy instrument.

---

## The model in one line

```
Pi_net_i(s | Omega) = Pi_gross_i(s | Omega) - c_i(Omega) * 1[s_i = S]
```

Equilibria are evaluated on **net** payoffs. Gross payoffs retain their
welfare interpretation. Everything else follows from that distinction.

| Regime | c_M | c_RoW | NE without GSF | NE with GSF |
|---|---|---|---|---|
| Omega = B  Baseline | 8.36 | 7.80 | (T,T) | (T,T) |
| Omega = L  Crisis | 7.38 | 8.42 | (T,T) | (S,S) |
| Omega = H  Boom | 8.38 | 7.25 | (T,S) | (S,S) |
| Omega = W  Stress test | 11.05 | 3.51 | (T,S) | (S,S) |

Omega = W is a stylised, forward-looking stress test. It uses no observed
post-2024 data and is not a forecast.

### Two probabilities, not one

These are separate quantities and the distinction matters:

- **Deterministic factorial.** At the point calibration, the regime-weighted
  probability that (S,S) is the unique equilibrium is **0.000** without the
  fund and **1.000** with it. This is what manuscript Section 4.7 reports.
- **Monte Carlo.** Over 1,000 draws per regime with plus or minus 25 percent
  shocks, `P[(S,S) is a Nash equilibrium]` with the fund on is 0.639 in
  crisis, 0.626 in boom, 1.000 under the stress test and 0.000 at baseline.
  The regime-weighted aggregate is therefore **0.707**, not 1.000. This is
  what Table S9 reports.

Earlier versions of this README conflated the two. See
`docs/MANUSCRIPT_CORRECTIONS.md` item 8.

---

## Quick start

```bash
git clone https://github.com/caceps/Geopolitics.git
cd Geopolitics
pip install -r requirements.txt
./run_all.sh
```

`run_all.sh` verifies every manuscript number, runs the Monte Carlo, runs the
subsidy sensitivity analysis, and regenerates all three computed figures into
`outputs/`.

To check the numbers alone:

```bash
python python/verify.py
```

This asserts 63 published quantities against the model and exits non-zero if
any has drifted. It also regenerates `docs/NUMBER_MAP.md`. Run it after
touching anything in `data/`.

---

## Repository layout

```
Geopolitics/
├── data/                       all inputs, plain CSV, human readable
│   ├── components.csv          32 x 4 component scores, 0 to 20 scale
│   ├── fiscal_inputs.csv       fiscal ratios behind the capture costs
│   ├── regime_parameters.csv   every constant, with provenance
│   └── mc_draws.csv            4,000 rows: 1,000 draws x 4 regimes, seed 42
├── python/
│   ├── model.py                single source of truth for the model
│   ├── verify.py               asserts all 63 manuscript numbers
│   ├── montecarlo.py           robustness over the binding parameters
│   ├── sensitivity_subsidy.py  robustness to the weakest calibration input
│   ├── make_draws.py           regenerates draws for Python and GAMS
│   ├── write_number_map.py     generates docs/NUMBER_MAP.md
│   ├── fig3_net_payoffs.py     Figure 3
│   ├── fig4_thresholds.py      Figure 4
│   └── figS3_sigma_es_map.py   Supplementary Figure S3
├── gams/
│   ├── main_v3.gms             Modules 1 to 10
│   ├── montecarlo_v3.gms       Monte Carlo with side diagnostics
│   ├── sensitivity_v3.gms      sigma by es equilibrium map
│   └── mc_draws_capture.gms    same draws as data/mc_draws.csv
├── docs/
│   ├── DATA_PROVENANCE.md          every input, tiered by how well sourced
│   ├── MANUSCRIPT_CORRECTIONS.md   numbered log of text corrections
│   └── NUMBER_MAP.md               claim to source map, auto-generated
└── outputs/                    generated figures and GAMS reports
```

Four GAMS files and an independent Python implementation. The two produce
identical results. GAMS is the version described in the manuscript; Python is
the version that runs without a licence, so a referee can check the numbers
in under a minute.

---

## Reproducing each display item

| Item | Command |
|---|---|
| Table 2 gross payoffs | `python python/verify.py` |
| Table 3 thresholds and factorial | `python python/verify.py` |
| Section 4.7 Monte Carlo | `python python/montecarlo.py` |
| Subsidy input sensitivity | `python python/sensitivity_subsidy.py` |
| Figure 3 | `python python/fig3_net_payoffs.py` |
| Figure 4 | `python python/fig4_thresholds.py` |
| Supplementary Figure S3 | `python python/figS3_sigma_es_map.py` |
| Full GAMS run | `gams gams/main_v3.gms` |

GAMS writes `results_main_v3.txt`, `results_montecarlo_v3.txt` and
`results_figS3_v3.txt`.

---

## Notes on method

**Why the payoffs are computed, not assigned.** A common criticism of applied
game theory is that payoff values arrive without justification. Here all 32
gross payoffs are generated from four component scores by
`Pi = 0.25(E - I + SB + F)`, and `main_v3.gms` aborts if any recomputed value
drifts from the reported matrix. The component scores are a *decomposition* of
the reported matrix rather than an independent derivation from raw indicators,
so this is a transparency and regression-guard device, not an external
validation. Its value is that a reader can see which dimension drives each
payoff difference.

**Why the robustness analysis targets capture costs.** Equilibria depend only
on payoff differences and capture costs. Perturbing the component scores
cannot change any equilibrium condition, so it measures nothing. The Monte
Carlo therefore shocks `c_M`, `c_RoW` and the GSF pools, which are the
quantities the result rests on.

**Why scale-free ratios are reported.** Three scaling constants
(`kappa_M = 16`, `lambda_d = 0.12`, `kappa_R = 120`) map fiscal ratios into
payoff units. They are fixed across regimes, so all cross-regime variation
comes from observables. Reporting `c / c*` alongside the levels makes every
equilibrium claim invariant to their choice, since the constants cancel from
the ratio.

**Why the crisis fund has resources at all.** The USD 80 trigger never fires
at crisis prices, so the crisis pool is zero. The crisis fund disburses the
stock accumulated in the preceding boom. This counter-cyclical drawdown is the
mechanism the paper argues for, and the crisis flip depends entirely on it.

**What the stress-test flip does and does not show.** The potency parameter
`mu = 0.84` is set by normalisation so that full deployment of the WarShock
pool at the central saving rate exactly neutralises the WarShock capture cost.
The Omega = W flip is therefore guaranteed by construction and carries no
independent evidential weight. Its 213 percent headroom is an artefact of the
anchoring. **The boom flip is the result that carries the argument**, because
it is not anchored: a fund deployed in the boom converts (T,S) windfall
capture into (S,S) at any saving rate above 0.42, and booms are recurrent
events rather than extreme scenarios.

**Known fragility, stated plainly.** The Omega = L result sits 6.8 percent
below the RoW-side flip threshold. At an energy-security factor of 1.154
rather than 1.08 it would not obtain, and no saving rate would recover it,
because the fund reduces `c_M` alone. Supplementary Figure S3 maps this
directly. Omega = H and Omega = W carry 10.3 and 213 percent headroom.

**The weakest input, and what it does not break.** The fuel subsidy figure in
`fiscal_inputs.csv` is the least well sourced input in the model, because
Malaysian subsidy figures vary by a factor of three depending on scope.
`python/sensitivity_subsidy.py` scales it from 0.7x to 3.0x across all four
regimes. Every equilibrium claim survives: the baseline stays at (T,T), and
boom and crisis still flip to (S,S). The boom threshold rises only from 0.420
to 0.548, against a central saving rate of 0.6. The reported `c_M` levels do
move, so Table 3 must be regenerated if the calibration changes. Full
discussion in `docs/DATA_PROVENANCE.md`.

---

## How well sourced is each input?

Inputs are tiered in `docs/DATA_PROVENANCE.md`:

- **Tier A** observed published statistics, traceable to a named document
- **Tier B** period averages and ratios computed by the authors from tier A
- **Tier C** set by argument or normalisation, not estimated

Several central quantities are tier C, including the energy-security factors,
the potency parameter, the scaling constants and the central saving rate. A
tier C input is not a defect. It is a defect only if presented as though it
were tier A, which is why the tiering is published.

---

## Requirements

Python 3.10 or later, with `numpy` and `matplotlib`. GAMS 24 or later for the
`gams/` files; no solver licence is needed, since the model contains no
`SOLVE` statement.

## Citation

See `CITATION.cff`. Please cite the article rather than this repository where
possible.

## Licence

Code is released under the MIT licence (`LICENSE`). Data files carry figures
compiled from the public sources cited in the manuscript and in
`data/regime_parameters.csv`; please cite the original sources.
