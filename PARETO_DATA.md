# Per-backbone Pareto data — for Memo to plot (2026-05-29)

**Pareto axes:** x = AgentDojo ASR (%, ↓ better), y = AgentDojo **benign** utility (%, ↑ better) — the straightforward ASR/BU pair (NOT under-attack utility).

**Rules applied (per Memo):**
- Every defense we've ever mentioned = a row on **every** backbone; **blank = we don't have that cell** (no guessing about applicability).
- **Triplet excluded** (ablation; FSAe gets its own triplet table — see bottom).
- Numbers are copied verbatim from our tables; **source/grid noted per cell** because the grids differ (full 52-cell vs comparator-sweep vs banking-subset). **Keep each Pareto on ONE grid** — don't mix a 52-cell base with a comparator-sweep comparator.

⚠️ **Decisions for you:** (a) **Llama-3.1-8B** has 3 candidate grids — pick one (I recommend the comparator-sweep grid, since that's the one the comparators live on). (b) **Qwen3-8B** AD-std ASR is unmeasured (only benign util exists) → no x-value yet. (c) **Qwen3-Next** only has the `important_instructions` family, not the 52-cell grid → its own mini-Pareto or fold into a note.

---

## 1. Llama-3.3-70B  — GOLD, self-consistent (T1: full AD-std, n=8,177; benign n=97)
| Defense | ASR % | Benign util % | note |
|---|---|---|---|
| No defense | 14.04 | 59.9 | |
| **ODILE (nullify)** | **0.01** | **51.5** | headline |
| Meta-SecAlign-70B | 2.11 | 62.6 | full-model DPO |
| Meta-SecAlign + Sandwich | 2.44 | 63.4 | stack |
| MELON detector | 0.24 | 48.6 | 2× inf |
| Instruction Hierarchy | 15.45 | 59.4 | overlay |
| Spotlight | 11.96 | 60.0 | overlay |
| Repeat-Prompt | 13.89 | 51.4 | overlay |
| DeBERTa classifier | 4.59 | 59.2 | detector |
| PromptGuard | — | — | (only run at 8B) |
| Sanitizer (firewall) | — | — | (only run at 8B; "not run at 70B") |
| PromptArmor / TaskShield / Perplexity / ReasAlign | — | — | (only in 8B comparator field) |
| CaMeL | partial | partial | only 2/4 suites run (workspace 0%ASR/0.90ben, slack 0%/0.50); banking+travel interpreter-crash. No 52-cell aggregate. |

## 2. Llama-3.1-8B — ⚠️ THREE GRIDS, pick one
**Grid B = comparator-sweep (RECOMMENDED for the comparator Pareto; appendix says "these cells back the per-backbone comparator Paretos"). PROVISIONAL per `app:comparator_8b` \todo.**
| Defense | ASR % | Benign util % | note |
|---|---|---|---|
| No defense | 16 | 26 | comparator-grid base |
| **ODILE (nullify)** | **0.9** | **29** | headline |
| ReasAlign | 0.4 | 13 | rep-class comparator |
| Spotlight-encode | 0.2 | ~15 | provisional |
| Spotlight-datamark | 3.9 | ~19 | provisional |
| Spotlight-delimiting | 8.8 | (weak) | no real protection at scale |
| PromptArmor | 0.2 | 19 | provisional |
| TaskShield | 0.2 | 23.8 | provisional |
| Perplexity filter | 1.0 | 12.7 | provisional |

**Grid C = firewall (banking / `important_instructions` only, n=144 attack / 16 benign) — different grid, do NOT mix with B:**
| Defense | ASR % | Benign util % |
|---|---|---|
| No defense | 11.1 | 25.0 |
| PromptGuard detector | 11.8 | 25.0 |
| PromptGuard (t=0.5) | 10.4 | 25.0 |
| Sanitizer (rewriter) | 2.1 | 18.8 |
| **ODILE** | **2.1** | **25.0** |

**Grid A = full 52-cell (matches the other Qwen backbones' grid): base 8.7 / 20.3 · ODILE 0.2 / 20.2.** (Use this if you want 8B consistent with the cross-model panel rather than the comparator field.)

## 3. Qwen-2.5-7B — (T2, full 52-cell, n=8,177; benign n=629)
| Defense | ASR % | Benign util % |
|---|---|---|
| No defense | 9.6 | 35.5 |
| **ODILE** | **0.1** | **35.6** |
| (all comparators) | — | — | (not run on this backbone) |

## 4. Qwen-2.5-14B — (T2, full 52-cell)
| Defense | ASR % | Benign util % | note |
|---|---|---|---|
| No defense | 8.3 | 46.1 | |
| **ODILE** | **0.0** | **41.8** | |
| CaMeL | 0.0 | 0.0 | **vacuous** — scale-floor, no tool calls either direction (not a real defense point; plot with an asterisk or omit) |
| (other comparators) | — | — | |

## 5. Qwen3-8B — ⚠️ AD-std ASR UNMEASURED (only benign util)
| Defense | ASR % | Benign util % | note |
|---|---|---|---|
| No defense | *(pending)* | 21.75 | 52-cell ASR not run; smoke (banking/direct) = 0% both |
| **ODILE** | *(pending)* | 18.25 | |
→ No Pareto x-value until the 52-cell ASR is run. (We DO have IA, TT, GSM8K/IFEval/AlpacaEval for Qwen3-8B if you want a non-AD Pareto.)

## 6. Qwen3-32B — (T2, full 52-cell)
| Defense | ASR % | Benign util % |
|---|---|---|
| No defense | 8.1 | 35.3 |
| **ODILE** | **0.5** | **37.5** |
| (comparators) | — | — |

## 7. Qwen3-Next-80B — ⚠️ `important_instructions` family only (NOT 52-cell; not comparable to the above)
| Suite | base ASR % | ODILE ASR % | base benign | ODILE benign |
|---|---|---|---|---|
| banking | 87.5 | 3.1 | 0.81 | 0.81 |
| slack | 78.6 | 7.1 | 0.86 | 0.71 |
| travel | 60.0 | 0.0 | 0.80 | 0.75 |
| workspace | 16.3 | 2.5 | 0.72 | 0.675 |
→ Suite-mean ≈ base 60.6% → ODILE 3.2% ASR; benign ~0.80 → ~0.74. Own mini-panel, or a footnote. No comparators.

---

## Triplet (FSAe-only direct table — NOT a Pareto row)
At Llama-3.1-8B (comparator grid): **ODILE-triplet 0.2% ASR / 32% benign** (vs nullify 0.9/29) — triplet wins AD-std + BFCL (101% retain) + TT-extract (8% leak) here, but is the inconsistent ablation elsewhere (over-jams IA valid-rate, etc.). Give FSAe a small base / nullify / triplet / ReasAlign table; keep it out of the headline Paretos.

## Provenance
T1 `tables/T1_70B_headline.tex` · T2 `tables/T2_crossmodel_AD.tex` · firewall `tables/T_firewall.tex` · 8B comparator `A_appendix_stub.tex:642` (`tab:comparator_8b_xbench`) + field paragraph `:662` (PROVISIONAL) · CaMeL 70B `A_appendix_stub.tex:370` · Qwen3-Next `A_appendix_stub.tex:873`.
