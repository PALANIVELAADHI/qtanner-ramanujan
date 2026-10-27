# qtanner-ramanujan

Reproducible finite-size benchmark of **quantum Tanner codes** on the bipartite double cover of the
left–right Cayley complex of PSL(2,q), and on the standard bipartite complex of PGL(2,q).
Companion code for *"Small Best-Found Logical Weights in Finite-Size Quantum Tanner Codes on
PSL(2,q) Cayley Complexes: A Reproducible Benchmark"* (Dr.R. Palanivel).

> **All logical weights reported by this code are best-found upper bounds on a distance. None is certified.**
> Local-code robustness (product-expansion) is not computed; the instances are *not* claimed to satisfy the
> hypotheses of the asymptotic distance theorem. Noise is code-capacity.

## Install
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt          # pinned versions used in the paper
python -m pytest -q tests                 # group axioms, commutation H_X H_Z^T = 0, rate bound, logical count
```

## Quick start
```python
import numpy as np
from qtanner.group import PSL
from qtanner.generators import sample_pair, spectral_ratio
from qtanner.localcodes import best_local_code
from qtanner.complex_dc import build_complex
from qtanner.code import QuantumTannerCode
from qtanner.distance import greedy_bound
from qtanner.decode import block_failure

rng = np.random.default_rng(5)
G = PSL(5)                                   # |G| = 60
A, B = sample_pair(G, 6, rng)                # symmetric, generating, disjoint conjugacy classes
CA, CB = best_local_code(6, 4, rng), best_local_code(6, 2, rng)
code = QuantumTannerCode.build(build_complex(G, A, B), CA, CB)   # raises if H_X H_Z^T != 0
print(code.n, code.k, spectral_ratio(G, A)[1])
print("greedy upper bounds:", greedy_bound(code.logical_x, code.rows_x), greedy_bound(code.logical_z, code.rows_z))
print(block_failure(code, p=0.03, shots=200, rng=rng))           # BP+OSD-0, min-sum 0.625, 60 iterations
```

## Layout
| Path | Content |
|---|---|
| `qtanner/group.py` | PSL(2,q), PGL(2,q) with full multiplication tables, conjugacy classes |
| `qtanner/generators.py` | random symmetric generating sets, validity statistics, Ramanujan ratio ρ |
| `qtanner/complex_dc.py`, `complex_bip.py` | squares `[(g,a,b)]=[(agb,a⁻¹,b⁻¹)]`, V0/V1 local views |
| `qtanner/localcodes.py` | random systematic local codes, exact distances, best-of-60 rule |
| `qtanner/code.py` | CSS matrices (X: C_A⊗C_B, Z: σ(C_A^⊥)⊗C_B^⊥), rank, k, logical operators |
| `qtanner/distance.py` | greedy descent, integer program (HiGHS) per logical class |
| `qtanner/decode.py` | BP / BP+OSD-0 / BP+OSD-CS (`ldpc` 2.4.1), failure statistics |
| `qtanner/stats.py` | Wilson, bootstrap, Cliff's δ, Benjamini–Hochberg, stratified permutation test |
| `experiments/` | one script per experiment, `run_all.sh`, `make_figures.py` |
| `data/` | raw JSON written by the experiments |
| `figures/` | figures written by `make_figures.py` |

## Reproducing the paper
```bash
cd experiments
QUICK=1 ./run_all.sh     # minutes; smoke test with tiny sample sizes
./run_all.sh             # ~10 single-core CPU hours; writes data/*.json and figures/*.png
```
Experiment → paper section: `exp_spectral` (spectral screening, draw validity) · `exp_selection`
(best-of-8 selection and selection check) · `exp_milp_all` (all-class integer programs) · `exp_unselected`
(unselected distributions, `--variant bip` for PGL) · `exp_spectral_assoc` (association tests) ·
`exp_localq` (local-code quality) · `exp_extremes` (best vs worst ρ) · `exp_rate_sweep` · `exp_ler`,
`exp_decoder_tuning`, `exp_fixed_weight`, `exp_runtime`, `exp_group_order`, `exp_toric_baseline`.

## Reproducibility notes
* All randomness uses `numpy.random.default_rng` (PCG64) with seeds fixed in the scripts; NumPy is pinned.
* Integer-programming results depend on HiGHS' time-limited search and may differ across versions/machines.
* Seeds in the scripts define the instances; if you change sampler internals, instances change.
* Verify commutation yourself: `QuantumTannerCode.build(..., verify=True)` is the default.

## License
MIT. See `LICENSE`.
