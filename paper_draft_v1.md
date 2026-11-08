# Where Does Interleaving Slicing and Tree Reconfiguration Help? A Paired-Intervention Study on Naturally Generated QPE Tensor Networks

**Draft v1, 2026-11-08. Working draft, not peer reviewed.** All numbers come from frozen raw evidence (Appendix A). Any statement without a cited run is marked as interpretation.

## Abstract

Contraction planners for quantum-circuit tensor networks have to keep every intermediate tensor below a memory capacity K. They do this by slicing indices and re-optimising the contraction tree. Recent work (ArcTN) reports that interleaving slicing with subtree reconfiguration lowers total cost on average compared with slicing a fixed tree afterwards. What has not been measured is *where* that gain comes from on a given network. We test one specific mechanism with paired, same-base interventions on naturally generated quantum phase estimation (QPE) networks. The mechanism is that restoring a slice is blocked by capacity on the original tree and becomes possible only after the tree is changed.

We ran cotengra's native hyper-optimiser, with slicing and reconfiguration enabled, on a fixed, pre-registered 12-network matrix: N ∈ {48, 64, 96} qubits × two phase seeds × two representations, lightcone with and without ADCRS simplification, at K = 2^28 elements. Eleven of the twelve returned trees are capacity-sensitive: the returned tree needs its slices to fit within K, and the same tree without slices exceeds K.

On the five ADCRS rows we then ran an exact finite-domain six-arm comparison on the actual returned tree, with an independent integer audit: incumbent, slice-only, tree-only, joint, and the two strict alternations. The joint arm restored a slice in only 1 of 14 candidate domains. In that domain total work fell by 14.16% and the slice count dropped from 2 to 1. Elsewhere, joint gains were at most 5.34% and came almost entirely from tree changes, and on N ≥ 64 the arms were indistinguishable to within 0.03%. A pre-search capacity signal did not choose better domains than native ordering.

On the representations without ADCRS, slicing inflated the native returned tree's total work by up to 2^3438 times over the same tree unsliced. These rows were beyond our exact certificate machinery.

The capacity-restoration mechanism exists but is rare on these networks. We also compare our own bounded joint planner (26-full) against ArcTN with Dynamic slicing at the same cap: ArcTN is 26–31× cheaper on the two larger structures, so the planner's earlier advantage over cotengra and OMECO does not extend to ArcTN. We report this as a scoped negative result and identify the representation, not the slicing heuristic, as the dominant factor.

## 1. Introduction

Large quantum-circuit tensor networks can usually be contracted only by slicing: fixing the values of a set of indices and summing over independent sub-contractions. This trades memory for repeated work. Hyper-optimised planners such as cotengra [Gray & Kourtis 2021] search over contraction trees and choose slices under a size limit. ArcTN [arXiv:2610.31554] showed that alternating slice selection with local subtree reconfiguration ("Dynamic slicing") reduces total sliced FLOPs by 31.99% (geometric mean) relative to slicing a fixed tree. It reports aggregate gains only and gives no per-instance account of why.

One plausible explanation is *capacity-blocked restoration*: on the original tree some intermediate result exceeds K once a slice is removed, so the slice cannot be dropped, while a different local tree keeps all intermediates within K and makes the restoration feasible. If this mechanism dominated, it would suggest targeted search, for example prioritising regions where restoration is blocked only by fixed nodes. It would also predict which networks benefit.

We test the mechanism directly with interventions on fixed bases rather than with correlations to graph metrics. Our contributions:

1. A pre-registered, fully isolated, provenance-pinned screen of 12 naturally generated QPE networks, establishing that 11 of the returned trees are capacity-sensitive (§3).
2. An exact, independently audited six-arm decomposition on the actual returned bases of the five ADCRS rows, separating slice-only, tree-only and joint effects (§4).
3. Evidence that capacity-blocked restoration is rare on these networks (1/14 domains) and that a pre-search capacity signal does not beat native ordering (§4).
4. A measurement that the representation, lightcone with versus without ADCRS, changes the sliced cost of native returned trees by hundreds to thousands of orders of magnitude in base 2 (§3.3).
5. A same-cap, same-scorer comparison of our bounded joint planner (26-full) with ArcTN Dynamic slicing, showing that ArcTN is 26–31× better on the larger QPE structures (§5).

## 2. Setup

**Networks.** QPE circuits with an inexact phase at N ∈ {48, 64, 96} total qubits and phase seeds 10 and 20 are exported as scalar expectation networks. Each is exported in two representations, lightcone with ADCRS simplification (L_ADCRS) and lightcone without it (L_no_ADCRS). Each of the 12 rows had its own fresh generation inside the pinned container. The native run then received that generation's exact raw bytes, verified by SHA-256.

**Capacity.** K = 2^28 elements bounds the largest single binary contraction result. It does not bound bytes or total live memory.

**Native planner.** cotengra 0.8.3 `HyperOptimizer`, configuration C0_dev60: methods greedy, kahypar and labels; max_repeats 128; random optlib; minimize flops; slicing and subtree reconfiguration toward target size K; max_time 60 s. max_time is a soft limit: one slicing trial on the largest network ran for 4252 s. Seeds and configuration were fixed before any results were seen.

**Outcome definitions.** For the native returned tree T with slices S, we compute exact integer scores (total work, per-slice work, multiplicity, largest result) for (T, S) and for (T, ∅) with an independent iterative scorer. HIT_THIS_BASE means S ≠ ∅, peak(T, S) ≤ K and peak(T, ∅) > K. NULL means the planner returned no slices. Timeouts, scoring exceptions and incomplete captures are never counted as NULL. These definitions apply to the returned tree only. They make no claim that every possible tree needs slicing.

**Isolation and evidence.** Every command ran once in a pinned Linux image: network none, read-only root, one CPU, 3 GiB, an unprivileged UID, and individually mounted read-only sources. Raw stdout and stderr bytes were hashed. Container stop, removal, OOM status and exit status were recorded per command. Infrastructure failures (one host-memory gate and two 900 s slot timeouts) were closed with immutable failure receipts. Those rows were re-run under fresh identities with longer slots, and the scientific configuration was not changed.

## 3. Screen results

### 3.1 Capacity sensitivity

| ord | network | outcome | contractions | slices | log₂ peak(T,∅) | native search (s) |
|---|---|---|---:|---:|---:|---:|
| 0 | N48-s10 ADCRS | NULL | 569 | 0 | 23.0 | 69 |
| 1 | N48-s10 no_ADCRS | HIT | 3740 | 94 | 72.0 | 93 |
| 2 | N48-s20 ADCRS | HIT | 569 | 2 | 30.0 | 64 |
| 3 | N48-s20 no_ADCRS | HIT | 3740 | 170 | 70.0 | 62 |
| 4 | N64-s10 ADCRS | HIT | 1017 | 3 | 31.0 | 82 |
| 5 | N64-s10 no_ADCRS | HIT | 6524 | 183 | 99.0 | 90 |
| 6 | N64-s20 ADCRS | HIT | 1017 | 1 | 29.0 | 79 |
| 7 | N64-s20 no_ADCRS | HIT | 6524 | 2131 | 169.0 | 1342 |
| 8 | N96-s10 ADCRS | HIT | 2273 | 28 | 56.0 | 144 |
| 9 | N96-s10 no_ADCRS | HIT | 14396 | 737 | 142.0 | 760 |
| 10 | N96-s20 ADCRS | HIT | 2273 | 19 | 47.0 | 98 |
| 11 | N96-s20 no_ADCRS | HIT | 14396 | 3578 | 166.0 | 4252 |

On every HIT row, peak(T, S) is exactly 2^28.

### 3.2 Seeds and repeats are not independent samples

The two seeds share circuit structure, and the two representations share a parent circuit. The matrix therefore holds 6 parent circuits and 12 network instances, not 12 independent observations. cotengra's internal RNG chain was not controlled by a caller seed, so the returned trees are single realisations. Repeat runs could return different trees.

### 3.3 Representation dominates sliced cost

For ADCRS rows, the sliced and unsliced total work of the returned tree agree to within 0.05%: slicing is nearly free. For no_ADCRS rows, the returned tree's sliced total work far exceeds its own unsliced cost:

| ord | log₂ W(T,∅) | log₂ W(T,S) | slices |
|---|---:|---:|---:|
| 1 | 84.1 | 132.4 | 94 |
| 3 | 91.0 | 216.8 | 170 |
| 5 | 120.0 | 223.4 | 183 |
| 7 | 174.4 | 2169.3 | 2131 |
| 9 | 171.0 | 780.4 | 737 |
| 11 | 178.0 | 3616.2 | 3578 |

These sliced costs are astronomically beyond any feasible contraction. Our interpretation: without ADCRS the native planner is forced into extremely deep trees (depth up to 4069), and the trees it returns satisfy K only by slicing almost every large index. The capacity constraint is met in form, not in any practical sense. For these networks, the most consequential "intervention" is the representation, not the slicing heuristic.

## 4. Same-base mechanism decomposition (ADCRS rows)

### 4.1 Method

For each ADCRS HIT row we take the actual returned tree as the incumbent I. The pre-J candidate catalogue for that tree is computed from the incumbent alone, before any optimised answer exists. It consists of local regions with at most 10 frontier and 6 editable nodes, with features computed over at most 64 slice masks. From the catalogue we select the two top native-ordered domains. Row 2 had enough candidates to also select one top capacity-signal domain and one domain from a fixed-seed permutation. Within each domain, an exact dynamic program gives the optimum of:

- **S**: change slices only, with the tree fixed;
- **F**: change the region tree only, with slices fixed;
- **J**: change both jointly;
- **FS/SF**: strict alternation, stopping at the first non-improving step.

A fixed-node capacity certificate checks whether nodes outside the editable region alone make restoring a slice infeasible. Every arm is re-scored by an independent integer implementation in a separate process.

### 4.2 Results

All five rows completed; the integer audits matched and every certificate returned UNREJECTED_BY_FIXED_NODES. Each entry is the reduction in total work relative to I.

| ord / domain | S | F | J | FS = SF | slices I→J |
|---|---:|---:|---:|---:|---|
| 2 native1 | 0.02% | 3.73% | 3.76% | 3.76% | 2→2 |
| 2 native2 | 0.02% | 14.13% | **14.16%** | 14.16% | **2→1** |
| 2 signal1 | 0.02% | 3.73% | 3.76% | 3.76% | 2→2 |
| 2 seeded | 0.02% | 5.32% | 5.34% | 5.34% | 2→2 |
| 4 (both) | 0.00% | 0.00% | 0.00% | 0.00% | 3→3 |
| 6 (both) | 0.02% | 0.00% | ≤0.03% | ≤0.03% | 1→1 |
| 8 (both) | 0.00% | 0.00% | 0.00% | 0.00% | 28→28 |
| 10 (both) | ≤0.01% | 0.00% | ≤0.02% | ≤0.02% | 19→19 |

### 4.3 Findings

1. **Capacity-blocked restoration is real but rare.** Only one of 14 domains, row 2 native2, shows the joint arm reducing the slice count. That domain's 14.16% gain is the largest observed.
2. **Tree changes, not slice changes, carry the gains.** In every domain where J improves on I by more than 0.1% (the four row-2 domains), F alone captures at least 99% of the improvement, while S alone contributes about 0.02%. The sub-0.05% gains on rows 6 and 10 come from slice changes instead and are negligible in size.
3. **Strict alternation matches the joint optimum here.** FS and SF reached J in every domain. This holds for these small domains. Equal-cost plateau moves and other tie policies were not explored.
4. **Native bases on N ≥ 64 are locally saturated.** For rows 4, 6, 8 and 10, all arms agree within 0.03%. Within these domains, the native returned tree is effectively locally optimal.
5. **No signal advantage.** The capacity-signal domain on row 2 equals native1. The fixed-seed domain is slightly better. On this evidence a pre-search capacity signal would not help choose regions.

## 5. External comparison: our planner versus ArcTN

**Background.** Before this study, our bounded joint planner (26-full: deterministic elimination initialisation followed by a capacity-bounded local joint slice/tree refinement) had been compared on the same fresh-QPE structures with cotengra and OMECO. Against its own common base it was better on all three structures (1.36–7.07×). Against OMECO TreeSA-8 it won on N64 and N96 and lost on N48. Against cotengra it won 2 of 4 completed pairs, and cotengra timed out on both N96 descriptors. That cotengra configuration was weak (4 trials earlier, a 128-repeat cap later). ArcTN had not been compared.

**Protocol.** We ran ArcTN (`arctn_schedule`, presets Light and Heavy, seed 0, `target_size = 2^28`, `slicing_mode = "dynamic"`, FLOPs-only objective, rate criterion off, `max_time` 3600 s) on the three structures in the same pinned image under 1 CPU and 3 GiB. ArcTN's returned path and slice set were re-scored with the same exact integer scorer used throughout this paper. That scorer first reproduced the saved 26-full totals exactly on all three structures, and ArcTN's own reported sliced cost agrees with ours to four significant digits.

**Results (total sliced work at K = 2^28; lower is better).**

| structure | 26-full | ArcTN Light | ArcTN Heavy | 26-full / Heavy |
|---|---:|---:|---:|---:|
| N64 (1018 tensors) | 1.450×10^12 | 4.82×10^10 | 4.71×10^10 | 30.8× |
| N96 (2274 tensors) | 1.659×10^17 | 1.07×10^16 | 6.34×10^15 | 26.2× |
| N48 (570 tensors) | 1.177×10^9 | — | — | rerun pending |

ArcTN planning took 43 s (Light) and 96 s (Heavy) on N64, and 234 s and 561 s on N96. 26-full took roughly 70–90 s. ArcTN Light is still 15–30× better than 26-full. On N48, both ArcTN runs aborted in our wrapper on an over-strict consistency check (ArcTN reported no slices needed); the check was removed and the rerun is pending.

**Interpretation.** On these QPE structures, a general-purpose planner with Dynamic slicing beats our QPE-targeted planner by more than an order of magnitude. Together with §4, this suggests that 26-full's earlier external advantage came from comparing against weaker baselines, not from a QPE-specific capacity mechanism. It is consistent with the finding that local tree quality, which ArcTN's search improves broadly, matters more than capacity-blocked slice restoration.

## 6. Discussion

**What this does and does not say about Dynamic slicing.** Our domains are small and local, and we test one tree at a time. The aggregate gains reported for alternating schemes may come from larger moves, from many small tree improvements summed over many regions, or from different networks. Our result narrows the explanation for these QPE networks: the specific capacity-restoration mechanism contributes little, and local tree quality matters more.

**Representation as the main lever.** The ADCRS versus no_ADCRS contrast is far larger than any effect we measured inside the planner, by hundreds to thousands of orders of magnitude (base 2) in sliced cost. For QPE expectation networks, applying ADCRS simplification before planning matters much more than how slicing and reconfiguration are interleaved. We did not certify the export lineage of the two representations end to end (§6), so this is an observed difference between pipelines, not a proven equivalence-preserving rewrite.

**Not a positive heuristic.** We do not propose a new planner. The evidence argues against building a QPE-specific capacity-signal ordering on top of these bases, and our existing QPE-targeted planner does not beat ArcTN (§5).

## 7. Limitations

- Six parent circuits, one circuit family (inexact-phase QPE), a single K, and a single native configuration. No held-out family was evaluated.
- No_ADCRS rows were outside our exact certificate machinery: the per-domain certificate evidence would be gigabytes. The mechanism is therefore unmeasured there, not shown to be absent.
- Native search is a single realisation per row, with no caller-controlled seed and no repeats. Hash randomisation in isolated Python was not controlled.
- Production provenance of the export pipeline is incomplete. A scalable equivalence certificate was written but not run.
- Planning-cost claims only. No numerical contraction was performed and no runtime or memory speedup is claimed.
- A planned Stage 3 equal-time root-ordering comparison on an earlier QPE70 network was not run. It was registered as a descriptive, likely null, diagnostic.
- The ArcTN comparison uses one seed per preset, a single CPU (the ArcTN paper used 8 cores), and 26-full numbers from an earlier run with a different time budget; it is not a strictly equal-time comparison. N48 is pending.

## 8. Reproducibility

Each row's raw generation and native receipts, acceptance records, container inspect and cleanup receipts, and the six-arm evidence files are retained with SHA-256 pins. Failed attempts are kept alongside, with their own immutable closeouts. The Appendix lists the paths.

## Appendix A. Evidence index

Public code and data: [qpe-tn-benchmark](https://github.com/Ferrari-72/qpe-tn-benchmark) (networks, native plans, exact costs), [tn-exact-scorer](https://github.com/Ferrari-72/tn-exact-scorer) (scorer, ArcTN re-scoring and runs), [tn-slice-mechanism](https://github.com/Ferrari-72/tn-slice-mechanism) (six-arm analysis and results). The paths below refer to the authors' full research workspace, which retains raw bytes, hashes, container receipts and failed attempts; they are listed for traceability.

- Screen receipts: `results/qpe_trajectory_renewed_20261106_v{32,48,49,59,60,64,65,66,67,68,69}/` (row → run in §3.1; rows 4 and 6 via retained receipts `paper/qpe_original12_r49_ordinal4_retained_v1/`, `paper/qpe_original12_r60_ordinal6_retained_v1/`).
- Closeouts: `paper/qpe_capacity_screen_v{48,59,64..69}_closeout.committed.json` (completed); `v49, v54, v60` (failed, retained).
- Mechanism evidence: `results/qpe_mechanism_v3_ord{2,4}/`, `results/qpe_mechanism_v3a_ord{6,8,10}/`; summary `results/qpe_mechanism_v3_summary_20261108.txt`; record `paper/qpe_stage2_mechanism_results_20261108.md`.
- Failure and prevention log: `paper/qpe_stage2_error_register_v1.md`.
- ArcTN comparison: `arctn_qpe_compare/RESULTS.md`, per-run JSON in `arctn_qpe_compare/runs/`, wrapper `arctn_qpe_compare/run_arctn_qpe.py`; earlier 26-full / cotengra / OMECO results `paper/qpe_fresh_complete_results_v1.md`.

## References

- J. Gray and S. Kourtis, "Hyper-optimized tensor network contraction," Quantum 5, 410 (2021); arXiv:2002.01935.
- ArcTN: "Efficient Tensor-Network Contraction Planning for Quantum Circuit Simulation," arXiv:2610.31554.
- X. Liu et al., "Lifetime-based Optimization for Simulating Quantum Circuits on a New Sunway Supercomputer," arXiv:2205.00393.
- "A Case Study on LLM-Usage for Contraction Order Optimization in Tensor Networks," arXiv:2606.01975.
