# Rebuttal exhibits — upload manifest (2026-05-29)

**Purpose.** Every cited table/figure as a standalone, directly-viewable **cropped PDF**, so each can be uploaded individually to <https://anonymous.4open.science/> and cited by its per-file link in the reviewer responses.

**How these were built.** Table fragments in `../tables/*.tex` (+ two inline tables extracted from `../sections/`) were rendered through `_preamble.tex` (mirrors the paper's macros/colors) with the `preview` package (one table per page, tight-cropped). Source/log in `_src/`; extracted/authored sources in `_extracted/`. To re-render after a table edit, see `_wrap_ALL.tex` (one-pass) — note: pdflatex on this cluster is ~8 min/invocation (CVMFS), so always render in ONE pass, not per-file.

---

## A. Rendered TABLE exhibits → `pdf/`

| exhibit id (paper) | file | what it shows | cited by |
|---|---|---|---|
| **T1** | `T1_70B_headline.pdf` | 70B headline: ODILE 0.01% ASR / 51.5% benign vs Meta-SecAlign, firewall, base | DefG (novelty/MSA) |
| **T2** | `T2_crossmodel_AD.pdf` | Cross-model AgentDojo (incl. CaMeL/Qwen3-Next where present) | FoEk-RtR4, GHCC-RtR2 (models) |
| **T3b** | `T3b_injecagent_crossmodel.pdf` | InjecAgent ASR ≤0.15% every backbone (n=1054/family) | FoEk-RtR1, DefG-RtR3/Q3, GHCC (12% drop) |
| **T4** | `T4_TT_headline.pdf` | TensorTrust extract/hijack ASR + benign coherence (n=300/cell; coh n=50) | FoEk-RtR1, DefG-RtR3 |
| **T6** | `T6_auxiliary_benign.pdf` | BFCL + τ-bench + AlpacaEval/GSM8K/IFEval benign capability | FoEk-RtR3 (benign) |
| **T_GCG** | `T_GCG.pdf` | Discrete + LoRA-aware GCG; CE wall ~10.7 vs 0.74 | FoEk-RtR2, GHCC-RtR4 (adaptive) |
| **T_firewall** | `T_firewall.pdf` | vs PromptGuard + Sanitizer (static tie, adaptive crack) | DefG-RtR1, GHCC-Q4 |
| **tab:T_wasp** | `tab_T_wasp.pdf` | WASP (proxy) 0/239 across backbones ✅*caption fixed* | FoEk-RtR1/RtR2, GHCC-RtR3 |
| **tab:adaptive_tap_pair** | `tab_adaptive_tap_pair.pdf` | TAP/PAIR 0/64 (incl. 5× deep replay); base 36/48, Sanitizer 15/48 | FoEk-RtR2, DefG-RtR1, GHCC-RtR4/Q4 |
| t_ad_injection_styles | `t_ad_injection_styles.pdf` | AD injection-style matrix (held-out augmentations) | FoEk-RtR1 (OOD aug) |
| t_augmentation_recipe_shift | `t_augmentation_recipe_shift.pdf` | augmentation-mix ablation | (support) |
| t_author_attacks_full | `t_author_attacks_full.pdf` | custom test-time attacks | FoEk-RtR1 (OOD) |
| t_crossmodel_master | `t_crossmodel_master.pdf` | 5 backbones × 3 benchmarks | FoEk/GHCC (models) |
| t_crossmodel_persuite | `t_crossmodel_persuite.pdf` | AD per-suite cross-model | (support) |
| t_injecagent_full | `t_injecagent_full.pdf` | InjecAgent per-family, all defenses (Meta-SecAlign incl.) | DefG-Q3, GHCC-Q2 |
| t_layer_ablation | `t_layer_ablation.pdf` | layer-band sensitivity | FoEk/FSAe (layer Q) |
| t_melon_ad_e_aug, t_melon_attack_breakdown | `t_melon_*.pdf` | MELON comparator | (support) |
| t_stacking_matrix | `t_stacking_matrix.pdf` | overlay-stacking | (support) |
| **fig:react_jam_odile** | `fig_react_jam_odile.pdf` | ⚠️*authored, compiling* — ODILE **jams** in ReAct vs delimiter (literal `@9''9999…` token-soup) | DefG-RtR1 (ReAct robustness) |

## B. FIGURE exhibits → `pdf/figures/` (already standalone; copied in)

`loss_layer_range.pdf`, `loss_objective_comparison.pdf`, `loss_rank_scaling.pdf` (FSAe loss); `injection_heatmap.pdf`, `per_injection_asr.pdf` (per-injection ASR); `multiseed_variance.pdf`; `fig_trace_examples.pdf`, `mechanism_*.png`, `clean_capability.pdf`, `heatmap.png`, `odile_*swan.png`.

---

## C. Flags / pending
- **`tab_T_wasp.pdf` caption fixed** — "defense-ranking signal" removed, reframed as an agentic benchmark we run a faithful proxy of (per Memo 2026-05-29). **Paper body `05_results.tex:259,270` + `app:wasp` still have the phrase → camera-ready cleanup.**
- **`fig_react_jam_odile.pdf`** — freshly authored (NOT from any other draft); shows ODILE jamming with the literal 70B token-soup. **Needs Memo's eyes**: confirm backbone/token (Q3/8B form is `‽9‽‽9999…`) and the task scenario.
- **T6** rendered a complete caption despite a cosmetic "Dimension too large" LaTeX warning (long caption); page is normal.
- **Pending exhibits not yet rendered:** per-backbone Pareto figures (ungenerated); appendix-inline tables in `A_appendix_stub.tex` — `app:qwen3_next` (Qwen3-Next clear-wins: WASP/BFCL/AD-imp_instr), `app:loss_ablation` (Qwen3-8B canonical-vs-triplet), `app:comparator_8b`, `app:camel_open_weights`. Same extraction method applies on request.
- **No blue row-shading** (Memo declined).
