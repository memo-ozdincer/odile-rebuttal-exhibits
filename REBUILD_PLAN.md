# Rebuild plan: rich OLD base (82a22dd) + every correction since

**Base:** `REBUTTAL_OPENREVIEW_FULL.md` = the rich pre-compression version (restored from commit 82a22dd). It already contains the per-benchmark detail, example/table links (55), R-lists (5), and full discussion that the compressed `REBUTTAL_OPENREVIEW.md` lost. The job is to KEEP that richness and surgically apply the corrections below, then add multi-message split markers. Each reviewer may now span 2-3 OpenReview comments (5000 chars each), so we do NOT compress for length.

Legend: [C]=correction (rich text is wrong/outdated, replace), [F]=format/style, [K]=keep rich as-is (just verify), [S]=split marker.

---

## GLOBAL / formatting (apply throughout)
- [F] Personalized greetings per reviewer (rich has the identical "constructive feedback" boilerplate 4x):
  - FoEk (exact, user-provided): "We thank the reviewer: generalization, attack strength, capability breadth, and model recency are indeed the axes we want to showcase performance in. Below, we address the questions they raised."
  - FSAe: "We thank the reviewer for the positive assessment and for precise questions on data curation, loss choice, layer selection, and deployment framing. Below, we address each."
  - DefG: "We thank the reviewer: defense comparison, a cleaner tradeoff table, cross-framework generalization, and real-loop behavior are the axes we strengthen below."
  - GHCC: "We thank the reviewer: deployment scope, model coverage, benchmark breadth, and adaptive robustness are the axes we address below."
- [F] Varied closings per reviewer (rich has the identical closing 4x). FSAe keeps "maintaining or raising the score"; others "reconsideration of the score". Vary the middle clause per reviewer.
- [F] Backtick all code tokens so underscores don't italicize on OpenReview: `send_money`, `update_password`, `read_file`, `ds_base`, `ds_enhanced`, `dh_base`, `dh_enhanced`, `important_instructions`, `<INFORMATION>`, `GmailSendEmail`, etc.
- [F] No bare `<` / `>` in comment text: use `≤`, spaced ` < `, or backticks. ("p < 1e-12" spaced; `<INFORMATION>` backticked.)
- [F] No prose `→` arrows: use "to" / "from X to Y". No em-dashes `—`: use commas/colons.
- [F] Markdown + unicode only (no text-mode LaTeX). Links as `[text](url)` (confirmed render).
- [S] Update the "How to use" header: note that FoEk/DefG/GHCC each exceed 5000 chars and post as 2 (or 3) comments; markers below show the split points.

## NEW EXHIBITS to make sure are linked (newer than 82a22dd, or updated numbers)
- [C] `tab_agentdyn.pdf` now has benign + AgentDojo-ASR columns, **triplet removed** (base 22.1/20.5/26.7, ODILE 0.0/1.3/20.0).
- [C] `tab_comparator_8b.pdf` now **triplet removed** (base / ODILE / ReasAlign).
- [C] `tab_T_wasp.pdf` shows per-backbone benign.
- [K] `injecagent_harness.pdf`, `fig_react_jam_odile.pdf`, `fig_gcg_loss_curve.pdf`, the redesigned trace_pdf exhibits, the redesigned example PDFs (same link paths).

---

## FoEk (Rating 3) — rich ~9352 chars -> ~2 comments
- [F] Greeting (exact, above).
- [K] **#1 Leakage**: KEEP the full rich answer — the 4-benchmark bullets WITH n, Wilson, Fisher, comparators, benign-coherence; the InjecAgent valid-rate/Meta-SecAlign/ReasAlign discussion; the harness/attacks/training links; the held-out-augmentation closer.
  - [C] add the react-jam figure link to the format point (ODILE jams across ReAct + delimiter formats).
  - [C] ReasAlign = "Li et al. 2026" (rich already Li — verify).
- [C] **#2 Adaptive**: KEEP the full rich detail (GCG 0/4 with CE wall + HarmBench 500 steps; LoRA-aware 0/3 gap-shrink; TAP/PAIR 0/64 with deeper-tree replay + positive control). FIX: remove "The wall is budget-insensitive, not under-optimized" (show-don't-tell); add the `fig_gcg_loss_curve.pdf` link ("loss curve shows base reaching compliance while ODILE stays flat").
- [K] **#3 Benign (BFCL/Tau)**: KEEP rich (n=1240, signed deltas, "+0.14 a gain", directional caveat, AlpacaEval/GSM8K/IFEval mention).
- [C] **#4 Models**: "frontier reasoning model" -> "frontier-agentic open-source model". KEEP the rest (attention-path adjustments, important_instructions-family caveat, full-grid camera-ready).
- [K] **Q layers**: KEEP rich (adjacent-band note already present).
- [K] **R5 see-also**: keep.
- [F] Closing (varied).
- [S] Split marker after #2 or #3 (msg1 ~ leakage+adaptive, msg2 ~ benign+models+layers+R5).

## FSAe (Rating 7) — rich ~4583 chars (likely fits 1, maybe 2 comments)
- [F] Greeting (above).
- [K] **Worsened pairs (a/b/c)**: KEEP rich full (structural exclusion; prior-methods/validator/MELON; stacking dh_base 16->2; include-vs-exclude 0.12->0.29, 0.37pp).
- [C] **MSE-vs-triplet**: use the FULL question quote: "The authors mention why CRL is better than MSE for Llama and have a justification for it. However, Qwen MSE was better than Qwen Triplet, why?" KEEP rich answer (consistency story + massive-activations hypothesis).
- [K] **Layer range**: KEEP rich (adjacent-band).
- [K] **Whitebox/blackbox landscape**: KEEP rich.
- [K] R-list (R1/R2). [F] Closing (varied, "maintaining or raising").

## DefG (Rating 4) — rich ~7744 chars -> 2 comments
- [F] Greeting (above).
- [C] **Q1 (Meta-SecAlign / novelty)**: REPLACE the rich 3-bullet mechanism + long ReasAlign paragraph with the corrected structure (lead with 70B head-to-head, format separate, comparator restored):
  - **Novelty:** representation-level LoRA, 1x cost, paired-trace contrast.
  - **vs Meta-SecAlign (70B head-to-head):** AgentDojo ODILE 0.01% vs 2.11% (benign 62.6 vs 51.5, different frontier points) [70B table]; InjecAgent ODILE ASR-all <=0.20% vs 0.2-2.6% [InjecAgent table `t_injecagent_full.pdf`].
  - **Separately, format robustness:** Meta-SecAlign can't emit valid ReAct tool calls (88-95% unparseable = broken output, not defense); ODILE stays in-format + jams across ReAct/delimiter [figure]. (NOT the marker-removed augmentation — that claim is removed.)
  - **vs firewall/sanitizer:** static parity 2.1%, -6.2pp benign + extra LLM call, cracked 15/48 vs ODILE 0/64 [firewall table]. (KEEP the rich firewall detail / matched-backbone trace if room.)
  - **Broader field:** [cross-benchmark comparator] vs ReasAlign across 5 benchmarks.
  - preferable-when.
- [C] **Q2 (Table 2 / UAU)**: KEEP rich per-backbone-table replacement, but reframe UAU: NOT "appendix-only" -> deliberate design choice (ODILE jams by design, so task-completion-under-injection isn't the objective; secondary diagnostic).
- [C] **Q3 (single-benchmark)**: name the 4 benchmarks DIRECTLY (rich says "the same evidence as above" + lists them with links — make it self-contained, NOT "above"). Add AgentDyn benign (26.7->20.0) + a WASP benign note if room.
- [C] **Q4 (brittle/real-loop)**: reword "one can read exactly what it looks like" -> "we have prepared a matched trace for reference"; KEEP the degenerate-IBAN + jam detail; ADD real-loop confidence + RL-attack (Nasr 2025) as the strongest adaptive test.
- [K] **data-exfil Q**: keep rich (ds_base/ds_enhanced, 32 attacks link, GmailSendEmail examples).
- [K] **Minor (Fig 2)**: keep.
- [K] **R-list (R2/R3/R4)**: keep rich.
- [F] Closing (varied).
- [S] Split marker (msg1 ~ Q1; or msg1 reasons 1-4, msg2 questions+minor).

## GHCC (Rating 5) — rich ~7244 chars -> 2 comments
- [C] **#1 Closed models**: model creators are PART of the audience, NOT primary (primary = open-source practitioners, he knows). KEEP the pipeline-defenses + deployment-scope paragraph.
- [C] **#2 Coverage**: "frontier-agentic open-source", NOT "newest reasoning model requested" (they didn't request Qwen3-Next). KEEP the Qwen3-8B/32B family mention.
- [C] **#3 Benchmarks**: ADD benign rates — AgentDyn base 26.7% to ODILE 20.0% (n=560), WASP per-backbone benign (70B 100/93, Qwen-2.5-7B 100/95, Qwen-2.5-14B 69/69, Llama-8B 100/83). KEEP rich AgentDyn/WASP detail + links.
- [C] **#4 Adaptive**: make FoEk-complete (rich #4 is terse). Four-axis mapping with cites (Zou/Mehrotra/Chao/Mazeika/Nasr) + GCG plateau + loss-curve + full result links [GCG][TAP/PAIR][loss curve].
- [C] **Q1 (88%)**: rich says "InjecAgent benign-split tool-call correctness" — WRONG. It is AgentDojo RELATIVE retention; now report per-backbone ABSOLUTE ASR/benign/UAU (UAU secondary).
- [C] **Q2 (12% drop)**: rich says "12 of 100, all format-envelope" — WRONG (implies InjecAgent n=100). It is AgentDojo (relative ~12% / ~6pp, banking/slack); honest mix of format-envelope + over-cautious jam; commit to full appendix + offer scrollable benign-failure exhibit.
- [K] **Q3 (CaMeL)**: KEEP rich full three-scale (Qwen-2.5-14B vacuous jam; Llama-70B 2/4 suites crash; Gemini-2.5-flash-lite all 4, ~0% ASR, 18-32% benign).
- [C] **Q4 (firewall)**: KEEP rich detail (4 bypass classes, introduce-cracks 0/16->3/12, Fisher p=7e-24 vs base + 5e-10 vs sanitizer, gradient mechanism, TAP-refinement) but CLARIFY structure (two numbered failure modes; add the sanitizer's one edge = backbone-agnostic).
- [K] **R-list (R1/R3)**: keep.
- [F] Closing (varied).
- [S] Split marker (msg1 reasons 1-4 + Q1/Q2; msg2 Q3 CaMeL + Q4 firewall + closing).

---

## EXECUTION ORDER
1. FoEk: greeting, #2 GCG show-don't-tell + loss curve, #4 frontier, react-jam in #1, closing, split.
2. FSAe: greeting, MSE full quote, closing.
3. DefG: greeting, Q1 restructure, Q2 UAU, Q3 names+benign, Q4 reword+RL, closing, split.
4. GHCC: greeting, #1 creators, #2 frontier, #3 benign, #4 adaptive-complete, Q1-88% fix, Q2-12% fix, Q4 firewall clarity, closing, split.
5. Backtick/arrow/em-dash/`<>` sweep across all.
6. Render FINAL, verify each message <5000, no hazards, all corrections present.
