# Energy-Aware Path Following for Electric Vehicles

Companion code for:

> M. Sabaa and M. Emam, *Closing the Regenerative-Energy Gap in Learned Path
> Following: Preview-Augmented Reinforcement Learning versus NMPC for
> Electric Vehicles in the Frenet Frame*.

A unified Frenet-frame benchmark comparing **NMPC**, **PPO**, a
gain-scheduled Ackermann state-feedback controller (**PID-SF**), a
**Stanley** geometric baseline, and a dynamic-model **LQR**, all sharing one
validated EV energy model (VT-CPEM) with explicit regenerative braking.

**Main result.** With a purely local observation, the learned policy recovers
only 9.3 % of traction energy against NMPC's 59.9 %. Adding a short
*preview* of upcoming curvature and reference speed, together with a reward
term computed from the same VT-CPEM mechanical power, raises recovery to
47.9 % - closing **76 % of the gap** - while simultaneously *improving*
tracking from 0.459 m to 0.314 m.

---

## Repository contents

```
.
├── notebook.ipynb          # Full study: run top-to-bottom to reproduce everything
├── generate_figures.py     # Plots all manuscript figures from figure_data.npz
├── figure_data.npz         # Exported simulation arrays (lets you skip the full run)
├── requirements.txt
├── models/
│   ├── ppo_pathfollow.zip      # Winner policy (preview + energy reward), seed 42
│   ├── ppo_pathfollow_dyn.zip  # Winner policy fine-tuned on the dynamic plant
│   └── ppo_base_8d.zip         # Baseline policy (8-D observation, heuristic reward)
└── results/
    └── nmpc_performance_data.npy   # Per-step NMPC solve times
```

---

## Reproducing the results

### Option A - regenerate the figures only (seconds)

Every figure in the paper is plotted from `figure_data.npz`, which contains
the measured simulation arrays:

```bash
pip install -r requirements.txt
python generate_figures.py
```

The script **does not synthesise data**. If `figure_data.npz` is absent it
exits with an error rather than producing a plot.

### Option B - full reproduction (~6 h on a quad-core CPU)

Open `notebook.ipynb` and run all cells top to bottom. This trains the PPO
policies (a 2x2 ablation, ten seeds, an energy-weight sweep, a curriculum
ablation, and a dynamic-plant fine-tune) and executes every NMPC, LQR,
PID-SF and Stanley experiment. The final cell writes `figure_data.npz`.

For a ~10 min smoke test that exercises every code path with reduced
budgets, set `FAST_TEST = True` in the training section.

---

## Environment note

Two independent version incompatibilities affect `import do_mpc`:

1. **NumPy >= 2.0** removed the alias `Inf`, which `casadi.tools` still
   imports. The notebook restores it before importing `do_mpc`, so NumPy
   does **not** need to be downgraded. (On Kaggle, downgrading NumPy breaks
   the preinstalled stack with `No module named 'numpy.strings'`.)
2. **CasADi >= 3.8** removed `casadi.tools.SX`, which `do-mpc` requires.
   The install cell pins `casadi==3.6.7`.

After running the install cell, **restart the kernel** before running the
rest of the notebook.

---

## Notes on the implementation

* **Shared plant.** All controllers are simulated on the *same* nonlinear
  Frenet kinematic model, so no controller benefits from a simplified plant.
* **Energy accounting.** Traction and regeneration are separated by the sign
  of mechanical power `P = F*v`, not of acceleration; the latter
  misclassifies mild decelerations and inflates the recovered fraction.
* **Timing.** NMPC is reported both via do-mpc/IPOPT and as a JIT-compiled
  pure-CasADi formulation. The compiled solver meets the 100 ms budget, so
  the honest speed comparison is ~40x rather than three orders of magnitude.
  Dedicated embedded solvers would narrow this margin further.
* **Dynamic validation.** A dynamic single-track model with linear tyres
  (slip angles clipped at 0.12 rad, ten RK4 sub-steps per control period,
  v <= 8 m/s) and an LQR baseline on the lateral-error state.

---

## Citation

```bibtex
@article{Sabaa2026EnergyAware,
  author  = {Sabaa, Mohamed and Emam, Mostafa},
  title   = {Closing the Regenerative-Energy Gap in Learned Path Following:
             Preview-Augmented Reinforcement Learning versus {NMPC} for
             Electric Vehicles in the {Frenet} Frame},
  journal = {TBD},
  year    = {2026}
}
```

## License

Released under the MIT License - see `LICENSE`.
