# advanced-grid-pll

Grid synchronisation studies built around the **first-harmonic-tracker PLL (FHT-PLL)**: the quadrature
and sequence signals are produced by a harmonic-tracker observer and fed to a standard dq loop. The
repository compares it, on the same input and at the same control rate, with the PLLs used in the other
pwr-control models: the single-phase Park (SOGI-type) PLL, the plain three-phase dq-PLL and the DDSRF-PLL.

Two Simulink/Simscape benches, one single-phase and one three-phase, each with its own README that gives the
model structure, the full parameter list, the design check of every loop and the results.

| Folder | Bench | What it shows |
|---|---|---|
| [`single_phase_dqPLLs_analysis`](single_phase_dqPLLs_analysis) | No power stage: one test voltage with known angle and frequency feeds three single-phase PLLs (Park PLL, FHT-PLL, DDSRF-PLL) at ts = 125 µs | Six test cases (2021 reference test, +30° phase jump, 50 → 49 Hz step, DC offset, EN 50160 harmonics, sag to 0.2 pu) with lock time, offset, ripple and peak error of each PLL |
| [`three_phase_dqPLLs_analysis`](three_phase_dqPLLs_analysis) | 690 V / 1 MW grid emulator with Dyn11 transformer and the LCL filter of a 250 kW AFE (converter side open); control at 4 kHz with double update | dq-PLL (Simulink and C-Caller), DDSRF-PLL and FHT-PLL through the asymmetric fault of VDE-AR-N 4110 test 25.4, feeding the FRT and sequence blocks of the library |

## Prerequisites

- MATLAB with Simulink, Simscape and Simscape Electrical.
- The companion [library](https://github.com/pwr-control/library) repository (setup functions, C-Caller
  PLL and tracker code, grid emulator and AFE control blocks). The `init_model.m` of each bench puts it on
  the path by itself (`private/library_setup.m`): clone the two repositories side by side, or set the
  environment variable `PWR_CONTROL_LIBRARY` to the library folder; if neither is found the library is
  cloned next to this repository.

## How to use

1. Open the bench folder and run `init_model.m` **from that folder**: it fills the struct `param`, prints the
   design check of the PLL loops, opens the model and checks its `param.*` references and From/Goto tags.
2. Simulate the `.slx` (`single_phase_pll_analysis.slx` or `frt_dqpll.slx`). In the single-phase bench
   `run_test_cases(1:6)` runs all the test cases and prints one error table per case.
3. Post-process with `plotting_results_plls.m` (both benches) and the other `plotting_results_*.m` scripts.
   Results (`.mat`) and figures are ignored by git.

## Design notes

The derivations, the linear models checked against the simulation and the recommended changes are in
[pwr-control/repositories-documentation](https://github.com/pwr-control/repositories-documentation):

- [`repos_doc/single_phase_lock_loop`](https://github.com/pwr-control/repositories-documentation/tree/main/repos_doc/single_phase_lock_loop): the single-phase quadrature generators and PLLs;
- [`repos_doc/three_phase_plls_comparison`](https://github.com/pwr-control/repositories-documentation/tree/main/repos_doc/three_phase_plls_comparison): dq-PLL, DDSRF-PLL and FHT-PLL under the asymmetric grid fault.

## Status

Both benches were moved to the `param` init in November 2026 and re-run in MATLAB R2026b. The PLL blocks are
kept as they are in the AFE and inverter controls they come from; the changes suggested by the design check
(Park PLL integral gain, undamped trackers, frequency integrator starting at 1 pu, fault gains of the
dq-PLL) are listed in the bench READMEs and not applied yet. Open point: the trackers are tuned at 50 Hz,
adaptive tuning on the PLL frequency still needs its own design.
