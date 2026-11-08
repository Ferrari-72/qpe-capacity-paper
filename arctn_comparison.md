# ArcTN vs 26-full on fresh QPE (K = 2^28), 2026-11-08

Working notes; not peer reviewed.

## Setup

- Networks: `results/qpe_fresh_20261105T054446Z_33fbb46a/generated/qpe-N{48,64,96}-s10.json`. These are the same three structures as the 26-full / cotengra / OMECO runs; the s20 descriptors share each structure hash.
- ArcTN: `tnlab-benchmark:v5-kahypar-arctn`. Called as `arctn_schedule(preset, seed=0, target_size=2**28, slicing_mode="dynamic", max_time=3600, use_ssa=True, flops_weight=1, read_write_weight=0, rate_enabled=False)`, the paper's FLOPs protocol with its Dynamic slicing at our cap.
- Container: 1 CPU (cpuset0), 3 GiB, network none, read-only root, uid 65534.
- Scoring: all numbers below are total sliced work, recomputed by the same iterative exact integer scorer used in the Stage2 screen (`qpe_j26_trace.collect_native_exact_summary`, r64 runtime). The scorer reproduced the saved 26-full totals exactly for N48/N64/N96: 1176946032, 1450023385088, 165852330825089024. ArcTN's own `sliced_log10_flops_total` agrees with ours to 4 significant digits.

## Results (total sliced work, lower is better)

| network | 26-full | ArcTN Light | ArcTN Heavy | cotengra (s10) | OMECO (s10) |
|---|---:|---:|---:|---:|---:|
| N48 (570 tensors) | 1,176,946,032 | pending | pending | 1,085,980,896 | 611,927,036 |
| N64 (1018) | 1,450,023,385,088 | 48,219,537,704 | **47,076,422,592** | 5,300,464,403,873,792 | 10,753,436,663,808 |
| N96 (2274) | 165,852,330,825,089,024 | 10,706,042,090,422,272 | **6,337,058,448,605,184** | timeout | 6.71×10^30 |

| network | 26-full / ArcTN Heavy | ArcTN time (Light / Heavy) | 26-full time |
|---|---:|---|---|
| N64 | **30.8×** worse | 43 s / 96 s | ~77 s |
| N96 | **26.2×** worse | 234 s / 561 s | ~71 s |

All ArcTN plans have peak exactly 2^28 (within cap); N64 uses 3 slices and N96 uses 20.

## Reading

On N64 and N96, ArcTN Heavy with Dynamic slicing beats 26-full by about 26–31× in total sliced work. ArcTN Light also beats 26-full by 15–30×. 26-full's earlier advantage on these networks was against cotengra (4-trial / 128 cap) and OMECO TreeSA-8; it does not hold against ArcTN. ArcTN used more planning time on N96 (561 s vs ~71 s), but Light at 234 s is still 15× better.

## Pending / caveats

- N48: two ArcTN runs aborted on an over-strict consistency assertion in our wrapper. ArcTN reported `sliced=True` with an empty `sliced_legs`, i.e. no slicing needed. The assertion has been removed; `run_n48.ps1` reruns both.
- One seed per configuration. Single CPU, whereas the ArcTN paper used 8 cores.
- The earlier 26-full numbers come from a different runner and time budget; this is not a strictly equal-time comparison.
- Raw per-run JSON: `arctn_qpe_compare/runs/`. Logs: `run_all_attempt2.log`.
