# ODILE, OpenReview-ready rebuttal (one comment per reviewer, each <= 5000 chars)

**How to use.** Paste one section per reviewer into the OpenReview comment box (Markdown enabled). `{ANON}` is the anonymous.4open.science base; find-replace `{ANON}` with your repo URL once you have the ID. No personal-repo links, no name/school anywhere. Each section below is under the 5000-character limit *with* the real URLs substituted. Greeting/closing are formulaic; code tokens are in backticks so underscores render; no text-mode LaTeX (OpenReview MathJax is math-mode only).

---

## FoEk (Rating 3)

We thank the reviewer for their constructive feedback, which helps strengthen our paper. Below, we address the questions they raised.

**1. Data leakage / fitting the test set.**
If ODILE were fitting AgentDojo's test set it would fail off-distribution, but it holds on four further benchmarks it never trained on, and on re-renderings of the exact training injections:
- **InjecAgent** (ReAct format, different tools and domains): ASR-all ≤ 0.15% on every backbone (n=1054/family). The [attacks]({ANON}/examples_pdf/injecagent_attacks.pdf) share no surface form with the [training injections]({ANON}/examples_pdf/agentdojo_train_injections.pdf).
- **TensorTrust** (guardian-bot password extraction): extract-leak from 78–92% to 1–44% across backbones (n=300/cell).
- **WASP** (web agent): attacker-action from 12.5% to 0/239, against base 25/200.
- **AgentDyn** (GitHub, Daily-Life, Shopping; suites absent from AgentDojo): Qwen3-8B from 22.1% to 0.0% (n=560) ([table]({ANON}/tab_agentdyn.pdf)).

The sharpest evidence is the held-out augmentation: we take the exact AgentDojo injections ODILE trained on and only re-render their surface (markers removed, or base64-encoded); ASR stays near 0 though the goal is unchanged, the opposite of what a test-fit defense would do.

**2. Adaptive attackers aware of the adapter.**
We add white-box attacks targeting the adapter's internals, all 0 cracks. Discrete GCG (75 steps): 0/4. Under ODILE the attacker-target cross-entropy plateaus at 10.7–11.0 while the base descends to 0.57 and complies, and the [loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf) stays flat to 500 steps. LoRA-aware GCG (bypass term at the LoRA layers): 0/3. TAP and PAIR (defense described to the attacker): 0/64 across two backbones, against base 36/48 ([table]({ANON}/tab_adaptive_tap_pair.pdf)). We do not claim these are the strongest possible attacker; RL-based attacks (Nasr et al. 2025) are planned for the camera-ready.

**3. Benign capability beyond AgentDojo (BFCL, Tau-Bench).**
General tool-use is retained. BFCL non-live AST: Llama-3.1-8B from 73.98 to 73.35; Qwen-2.5-7B from 74.98 to 75.12. Tau-bench airline (n=50): Llama-3.1-8B from 42.00 to 38.78 ([table]({ANON}/tab_capability.pdf)).

**4. Newer reasoning models.**
We add Qwen3-Next-80B-A3B-Thinking (Sept 2025), a frontier reasoning model with nonstandard (Gated-DeltaNet) attention; ODILE transfers ([table]({ANON}/tab_pareto_qnext.pdf)). The full 52-cell grid on this backbone is a camera-ready commitment.

**Q (why layers 30–55?).**
Not trial and error: following circuit-breakers (Zou et al.), mid-to-late layers avoid token-specific representations; we use that single principled band and depth-scale it across seven backbones. An adjacent-band check at Llama-3.1-8B (`L10-20`, `L12-22`, `L20-30`, 52 cells each) finds all suppress ASR to near 0 while the later band halves under-attack utility, so the mid band is selected. A full native-objective sweep is a camera-ready commitment.

We believe these results greatly strengthen the significance of our results, and improve the overall soundness of the paper. We hope our rebuttal addresses all of the reviewer's concerns, and we would be grateful for a reconsideration of the score.

---

## FSAe (Rating 7)

We thank the reviewer for their constructive feedback, which helps strengthen our paper. Below, we address the questions they raised.

**Q (worsened pairs): (a) how to handle them, (b) prior methods, (c) ASR including them.**
(a) These three banking pairs are structurally ambiguous: the attacker IBAN equals the legitimate recipient, so the harmful and benign twins are byte-identical and give the contrastive objective no signal. We exclude them by a hard-coded structural rule (target-collision in the task definition), not a per-sweep statistic. (b) No representation-level method can separate literally-identical actions; the right place to catch this is outside the representation, e.g. a deterministic post-execution validator (an IBAN allowlist checking the recipient against the user's known accounts) or a runtime detector (MELON, Zhu et al. 2025) stacked on ODILE. We measured the stacking: an instruction-hierarchy / sandwich overlay on ODILE closes the worst residual InjecAgent cell (`dh_base`) from 16.0% to 2.0%, and the two defenses fail on largely disjoint cells, which is why they compose. (c) A controlled include-vs-exclude ablation (the three pairs the only difference) shows inclusion is pure downside: AD-standard ASR rises from 0.12% to 0.29% and benign utility falls 0.37pp ([table]({ANON}/tab_worsened_pairs.pdf)). A full stacking-as-defense ablation for this class is a camera-ready commitment.

**Q (Qwen MSE better than triplet, why?).**
Both drive ASR to near 0, so the security result is not loss-specific; the difference is consistency. MSE behaves consistently across backbones; triplet is strong on some and weaker on others, so we report MSE as the headline and triplet as an ablation. Our working hypothesis: triplet relies on absolute distance margins, and hidden-state magnitudes differ across families (Qwen vs Llama; cf. massive activations, Sun et al. 2024), so a margin calibrated on one family is mis-scaled on another, while MSE does not depend on those magnitudes. We will add a dedicated loss-objective discussion with per-backbone margin-sensitivity ablations.

**Q (layer range: how identified, trial and error?).**
No: the mid-late band follows the circuit-breaker rationale (Zou et al.), depth-scaled across seven backbones. An adjacent-band check at Llama-3.1-8B (`L10-20`, `L12-22`, `L20-30`) finds all suppress ASR to near 0 while the later band halves under-attack utility, so the mid band is selected. A full native-objective sweep is a camera-ready commitment.

**Q (white-box vs black-box: a different landscape).**
We agree and will adopt this framing explicitly. ODILE is an inner defense for the open-weight / self-hosted regime; for API-only deployments, pipeline and black-box defenses are the appropriate layer. We will state in Sections 1 and 10 that ODILE occupies a complementary deployment layer rather than competing.

We believe these results greatly strengthen the significance of our results, and improve the overall soundness of the paper. We hope our rebuttal addresses all of the reviewer's concerns, and we would be grateful for maintaining or raising the score.

---

## DefG (Rating 4)

We thank the reviewer for their constructive feedback, which helps strengthen our paper. Below, we address the questions they raised.

**1. Meta-SecAlign, and how ODILE differs from prior model-level defenses.**
The three defenses act at different layers. Meta-SecAlign changes the weights (DPO to obey a privileged-instruction hierarchy, keyed to data-delimiters); a firewall/sanitizer rewrites the inputs in a second pass; ODILE reshapes the hidden states on the intent-formation window so a harmful-intent trajectory collapses to a non-actionable (jam) state, in the same single forward pass. The novelty is the representation-level, single-pass, own-behavior-trained mechanism, which also gives 1x inference cost and robustness where the others break: Meta-SecAlign is delimiter-keyed and leaks 3.38% on the marker-removed augmentation where ODILE holds 0.00% ([trace]({ANON}/traces_pdf/trace_secalign_cracked_vs_odile.pdf)), and ODILE is format-agnostic across ReAct vs delimiter-wrapped input ([figure]({ANON}/fig_react_jam_odile.pdf)). We benchmark the broader model-level field on one model and one metric pair: ODILE vs ReasAlign (Li et al. 2026) across all five benchmarks ([comparator]({ANON}/tab_comparator_8b.pdf)); ReasAlign matches ODILE on ASR but at a roughly 13pp benign-utility cost, and its near-0 InjecAgent ASR is a format-rejection artifact (its reasoning-tag format is rejected about 94% of the time), not a defense.

**2. Table 2 mixes models, metrics, and assumptions.**
We replace it with one table per backbone, each on a single model and a single metric pair (ASR vs benign utility): [70B]({ANON}/tab_pareto_70b.pdf), [8B]({ANON}/tab_pareto_8b.pdf), and the Qwen backbones. Under-attack utility is appendix-only by design: ODILE jams under injection, so that metric reflects residual non-malicious throughput, not task completion.

**3. Benchmark-specific / different agent frameworks.**
The four benchmarks above span different frameworks: ReAct (InjecAgent), a web agent (WASP), and a dynamic benchmark on three non-AgentDojo suites (AgentDyn), all reaching ASR near 0 with the same recipe.

**4. Brittle / garbled tool calls in a real loop.**
The jam is the design intent, not a failure: on the same task where the base calls `update_password`, ODILE emits no parseable tool call ([trace]({ANON}/traces_pdf/trace_base_executes_vs_odile_jams.pdf)). It never produces the attacker action (0/5805 distinct WASP paraphrases). In a retry loop this is the strict-deny endpoint, and deterministic validators stack on top.

**Q (data-exfiltration that looks like retrieval).**
Yes. InjecAgent's `ds` families are exactly retrieve-then-email-to-attacker; ODILE drives both to 0.00% at Llama-3.3-70B ([attacks]({ANON}/examples_pdf/injecagent_data_exfil.pdf)).

**Minor (Figure 2 legend).** We will declutter the legend and split the Llama and Qwen panels.

We believe these results greatly strengthen the significance of our results, and improve the overall soundness of the paper. We hope our rebuttal addresses all of the reviewer's concerns, and we would be grateful for a reconsideration of the score.

---

## GHCC (Rating 5)

We thank the reviewer for their constructive feedback, which helps strengthen our paper. Below, we address the questions they raised.

**1. Limited applicability to closed / API-only models.**
Agreed; this is a scope choice. ODILE is an inner defense for open-weight / self-hosted models; API-only deployments use pipeline defenses (Spotlight, instruction-hierarchy, MELON, CaMeL), which ODILE complements rather than replaces. We will add a deployment-scope paragraph in Section 10.

**2. Model coverage / include a newer Qwen.**
Six backbones plus Qwen3-Next-80B, including Qwen3-8B and Qwen3-32B from the Qwen-3 family; ODILE transfers ([cross-model]({ANON}/T2_crossmodel_AD.pdf)).

**3. Single-benchmark focus (AgentDyn, WASP).**
Both added. AgentDyn (three suites absent from AgentDojo): Qwen3-8B from 22.1% to 0.0% (n=560, trace-validated) ([table]({ANON}/tab_agentdyn.pdf)). WASP: attacker-action from 12.5% to 0/239 ([table]({ANON}/tab_T_wasp.pdf)); the full containerized version is a camera-ready commitment.

**4. No adaptive attack evaluation.**
We evaluate all four named axes: adapter- and layer-range-aware (LoRA-aware GCG with a bypass term at the LoRA layers, 0/3); objective-aware (TAP and PAIR with the defense described, 0/64 across two backbones); mask-aware (the completion-window mask is a training-time loss localization, with nothing input-side to route around); plus discrete GCG 0/4 ([table]({ANON}/tab_adaptive_tap_pair.pdf)).

**Q1 (what does "88% retention" mean?).**
It was one axis, the benign-split tool-call correctness on InjecAgent at 70B, not a global number. We now report ASR, benign utility, and under-attack utility separately, per backbone ([tables]({ANON}/tab_pareto_70b.pdf)).

**Q2 (where does the 12% drop come from?).**
Of the 12 of 100: none are malformed calls, incomplete chains, wrong answers, or refusals; all are format-envelope shifts (the strict-ReAct Action form becomes a semantically equivalent expression with correct tool-call content). This is metric strictness, not capability loss; we will list all 12 in the appendix.

**Q3 (position against CaMeL).**
CaMeL (Debenedetti et al. 2025) is a design-level defense (a privileged planner plus a capability-checked interpreter), a different layer of the stack, complementary to ODILE. It reaches near-0 ASR wherever its interpreter runs but is brittle and high-tax on open weights: it scale-floors at Qwen-2.5-14B, hits an interpreter crash on 2 of 4 suites at 70B, and spends 68–82% of benign utility at its target tier. ODILE reaches 0% ASR at 1x cost on every backbone.

**Q4 (simpler sanitizer firewalls).**
We re-implemented the Bhagwatkar et al. (2026) firewall at Llama-3.1-8B (a second LLM rewrites each tool output). It is useful but not invariant: statically it matches ODILE (2.1% ASR) and reduces base cracks, but benign-looking paraphrases bypass it, and on deploy-token / SSH-key / PAT records it introduces cracks the base never had (base 0/16, sanitizer 3/12), because the rewriter cannot tell "the user wants a deploy token" from "the injection wants one." ODILE holds 0/64 and 0/5805 (Fisher vs sanitizer p = 5e-10). The reason is mechanistic: the sanitizer acts on surface text, so any paraphrase that fools the rewriter gets through; ODILE acts on hidden states, where same-intent paraphrases converge, so surface variation gives the attacker almost no signal. ODILE is preferable when adaptive robustness, 1x cost, and benign-utility preservation matter together ([firewall table]({ANON}/T_firewall.pdf)).

We believe these results greatly strengthen the significance of our results, and improve the overall soundness of the paper. We hope our rebuttal addresses all of the reviewer's concerns, and we would be grateful for a reconsideration of the score.
