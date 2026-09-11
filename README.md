# AMCAS
Course project comparing SRAM and STT-MRAM caches using CACTI, NVSim, Ramulator, and gem5
A five-part evaluation of memory technologies spanning device-level circuit simulation up to full-system performance: NGSpice (bitcell physics) → CACTI (SRAM array modeling) → NVSim (STT-MRAM array modeling) → Ramulator 2.0 (DRAM controller/scheduling) → gem5 (full-system cache/core interaction).

Central Question

Does a device-level improvement (e.g., a denser but slower non-volatile memory cell) actually translate into a system-level win — and under what conditions does it fail to?

Repo Structure
.
├── ngspice/          # SRAM bitcell transient analysis (sram.cir, plots)
├── cacti/            # SRAM array modeling — cache.cfg variants, sweep results
├── nvsim/            # STT-MRAM array modeling — .cell files, cfg variants
├── ramulator/         # DRAM controller experiments — ddr4.yaml, trace files, patch notes
├── gem5/             # Full-system SE-mode configs and stats.txt outputs
└── report.pdf        # Full written report (all tasks, all parts)
Summary of Results
Part A — NGSpice (SRAM bitcell)
Baseline read ΔV: 730 mV (isolated-cell bitline model, 180 fF)
Destructive read failure observed when access-transistor width increased to 0.24 µm
Minimum functional V_DD (25 mV sense-amp offset floor): 0.60 V
ΔV at 85°C: 582.1 mV (vs. baseline) — temperature dominates degradation over the fixed sense-amp offset
Part B — CACTI (SRAM array, 45nm)
Metric	Value
Access time (2MB, ED²)	2.902 ns
Total area	10.658 mm²
Best Ndwl/Ndbl	4/2

Capacity sweep (256kB–16MB) shows accelerating (not flat) access-time growth per doubling — inconsistent with Amrutur & Horowitz's "one gate delay per doubling," attributed to the fixed 4-bank constraint limiting re-partitioning at scale.

Part C — NVSim (STT-MRAM array, 45nm)
Metric	SRAM	STT-MRAM	Ratio
Read latency	2.902 ns	3.674 ns	1.27×
Write latency	~2.9 ns	10.509 ns	3.6×
Area	10.658 mm²	0.682 mm²	15.6× smaller

Write penalty traced to STT-MRAM's spin-transfer-torque switching physics; area advantage traced to 1T-1R cell density. TMR ratio increase (2:1→3:1) showed negligible system-level benefit — sense-amp margin wasn't the binding constraint. Write-current/area coupling confirmed: halving access-transistor width alongside reset current shrinperipheral (not bitcell) area.

Part D — Ramulator 2.0 (DDR4 controller)
Config	Memory cycles	Read latency	Row-buffer hit rate
FRFCFS (baseline)	1,410	52,529 cycles	—
FCFS	3,768 (2.67×)	128,471 cycles	collapsed
ChRaBaRoCo mapping	5,653 (4.01×)	182,629 cycles	improved, but BLP loss dominates
2 channels	705 (2×)	42,357 cycles	—
Part E — gem5 (full-system, GAPBS bfs/sssp, O3CPU)

8MB STT-MRAM L2 (2× hit latency, 4× capacity) did not outperform 2MB SRAM L2 on either kernel — working sets exceed both cache sizes, so the capacity advantage yields no miss-rate reduction, leaving only the latency penalty. Under TimingSimpleCPU (in-order), the memory-latency penalty is fully exposed rather than hidden by out-of-order execution.

Key Takeaway

A device-level win (density, TMR, write current) only converts into a system-level win when it relieves the metric that's actually the bottleneck for the target workload. Three assumptions worth scrutinizing first: (1) NGSpice's isolated-bitline capacitance model, (2) CACTI's fixed 4-bank constraint, (3) gem5's assumption that GAPBS working sets meaningfully interact with the swept cache sizes.

Reproducing

Each subfolder contains the exact config files (.cir, .cfg, .cell, .yaml, .py) used to generate the results above, along with the tool-specific build notes (e.g., a documented source patch for a Ramulator 2.0 LoadStoreTrace completion-condition bug in ramulator/PATCH_NOTES.md).
