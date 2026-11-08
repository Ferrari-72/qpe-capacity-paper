# qpe-capacity-paper

Draft paper and result summaries for:

**Where Does Interleaving Slicing and Tree Reconfiguration Help? A Paired-Intervention Study on Naturally Generated QPE Tensor Networks**

**Status: working draft, not peer reviewed.** The findings are a scoped, mostly negative result.

## Findings in brief

1. On 12 naturally generated QPE networks (N = 48/64/96, two phase seeds, lightcone with and without ADCRS), 11 of the plans returned by cotengra are capacity-sensitive at K = 2^28.
2. Capacity-blocked slice restoration, the most intuitive explanation for why interleaving slicing and reconfiguration helps, occurred in only 1 of 14 exactly analysed domains. Gains came almost entirely from tree changes.
3. Representation dominates. Without ADCRS, sliced plans cost up to 2^3438 times their own unsliced cost. They meet the cap in form but are not feasible in practice.
4. Our own capacity-bounded joint planner, which beat weaker cotengra and OMECO configurations on QPE, is 26–31× worse than ArcTN with Dynamic slicing on the larger structures.

## Files

| file | content |
|---|---|
| `paper_draft_v1.md` | English draft |
| `results_summary_zh.md` | full results summary (Chinese) |
| `mechanism_results_zh.md` | six-arm mechanism results (Chinese) |
| `arctn_comparison.md` | ArcTN comparison details |

## Code and data

- Networks and native plans: [qpe-tn-benchmark](https://github.com/Ferrari-72/qpe-tn-benchmark)
- Exact scorer and ArcTN re-scoring: [tn-exact-scorer](https://github.com/Ferrari-72/tn-exact-scorer)
- Mechanism analysis: [tn-slice-mechanism](https://github.com/Ferrari-72/tn-slice-mechanism)

## License

Text: CC BY 4.0. Any code snippets: MIT.
