# ODILE, OpenReview-ready rebuttal (one comment per reviewer, each <= 5000 chars)

**How to use.** Paste one section per reviewer into the OpenReview comment box (Markdown enabled). `{ANON}` is the anonymous.4open.science base; find-replace `{ANON}` with your repo URL once you have the ID. No personal-repo links, no name/school anywhere. Each section is under the 5000-character limit *with* the real URLs substituted, while keeping per-experiment detail (n, significance, comparator, and what each result shows). Greetings/closings are personalized; code tokens are backticked; no text-mode LaTeX (OpenReview MathJax is math-mode only).

---

## FoEk (Rating 3)

We thank the reviewer: generalization, attack strength, capability breadth, and model recency are indeed the axes we want to showcase performance in. Below, we address the questions they raised.

**1. "There is clear data leakage [...]. I cannot tell whether ODILE learns a real prompt-injection defense or fitting on test set."**
A memorized adapter would fail off-distribution. ODILE does not, on four further benchmarks and on re-renderings of the exact training injections, each with n, significance, and a comparator:
- **InjecAgent** (Zhan et al. 2024; ReAct, domains absent from training): ASR-all ≤ 0.15% on every backbone (n=1054/family, Wilson UB 0.35%), at or below Meta-SecAlign (Chen et al. 2026), the closest model-level defense (0.2–2.6% at 70B) ([attacks]({ANON}/examples_pdf/injecagent_attacks.pdf) vs [training]({ANON}/examples_pdf/agentdojo_train_injections.pdf); [harness]({ANON}/examples_pdf/injecagent_harness.pdf)).
- **TensorTrust** (Toyer et al. 2023; guardian-bot password extraction, a different mechanic): extraction leak 78–92% to 1–44% (Qwen-2.5-14B 82% to 1%), n=300/cell, while benign coherence stays ~100% on neutral pre-prompts.
- **WASP** (Evtimov et al. 2025; web agent): attacker-action 12.5% to 0/239 verified-tight (Fisher p≈1e-9), vs base 25/200 and sanitizer 8/160.
- **AgentDyn** (Li et al. 2026; GitHub/Daily-Life/Shopping, absent from AgentDojo): the attack lands on the base agent (Qwen3-8B 22.1%, n=560) and ODILE drives it to 0.0% (Fisher p < 1e-12, [how it differs]({ANON}/examples_pdf/agentdyn_difference.pdf)).

Memorization cannot explain this: the adapter never saw these tools or environments at training time. The sharpest evidence is the held-out augmentation, where we re-render the *exact* training injections (markers removed, or base64) and ASR stays near 0 though the goal is unchanged, the opposite of a test-fit defense. ODILE is also format-driven, jamming identically across ReAct and a delimiter-wrapped variant ([figure]({ANON}/fig_react_jam_odile.pdf)).

**2. "The attack setting is too weak [...] does not test adversarial attackers aware of the adapter."**
We add defense-aware white-box attacks on the adapter's internals, all 0 cracks. **Discrete GCG** (Zou et al. 2023; 75 steps): 0/4; the attacker-target cross-entropy plateaus at 10.7–11.0 while the base descends to 0.57 and complies, and does not move with budget (0% to 500 steps on a HarmBench (Mazeika et al. 2024) sweep, base broken on 68%); the [loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf) shows base reaching compliance while ODILE stays flat. **LoRA-aware GCG** (bypass at the LoRA layers): 0/3; the gap shrinks ~25% but never nears compliance. **TAP (Mehrotra et al. 2024), PAIR (Chao et al. 2024)** (objective described to the attacker): 0/64 across two backbones, vs base 36/48 and sanitizer 15/48 on the same cells (the positive control) ([table]({ANON}/tab_adaptive_tap_pair.pdf)). RL-based attacks (Nasr et al. 2025) are planned.

**3. "Benign capability evaluation is narrow [...] such as BFCL and Tau-Bench."**
We add exactly the two named (base to ODILE). **BFCL** (non-live AST, n=1240): Llama-3.1-8B 73.98 to 73.35 (−0.63), Qwen-2.5-7B 74.98 to 75.12 (+0.14, a gain). **Tau-bench airline** (n=50, single trial, directional): Llama-3.1-8B 42.00 to 38.78 ([table]({ANON}/tab_capability.pdf)). General tool-use is retained.

**4. "Model selection is outdated [...] do not test newer reasoning models."**
We add Qwen3-Next-80B-A3B-Thinking (Sept 2025), a frontier-agentic open-source model with nonstandard (Gated-DeltaNet) attention; ODILE transfers with attention-path adjustments ([table]({ANON}/tab_pareto_qnext.pdf)). The shown results are on the `important_instructions` family; the full 52-cell grid is a camera-ready commitment.

**Q. "Why is LoRA only applied on layers 30–55?"**
Not trial and error: following circuit-breakers (Zou et al.), mid-to-late layers avoid token-specific representations; we use that single principled band and depth-scale it across seven backbones. An adjacent-band check at Llama-3.1-8B (`L10-20`, `L12-22`, `L20-30`, 52 cells each) finds all suppress ASR to near 0 while the later band halves under-attack utility, so the mid band is selected. A full native-objective sweep is a camera-ready commitment.

We believe these additions directly address the leakage, attack-strength, capability, and model-recency concerns, and strengthen the soundness of the paper. We would be grateful for a reconsideration of the score.

---

## FSAe (Rating 7)

We thank the reviewer for the positive assessment and for precise questions on data curation, loss choice, layer selection, and deployment framing. Below, we address each.

**Q (worsened pairs): "(a) ideas on how to handle them. (b) if and how other previous methods were able to handle this. (c) were any experiments run including these pairs and what was the ASR [...]."**
(a) These three banking pairs are structurally ambiguous: the attacker IBAN equals the legitimate recipient, so the harmful and benign twins are byte-identical and give the contrastive objective no signal. We exclude them by a hard-coded structural rule (target-collision in the task definition), not a per-sweep statistic. (b) No representation-level method can separate literally-identical actions, because there is no internal signal to separate; the right place to catch this is outside the representation, e.g. a deterministic post-execution validator (an IBAN allowlist checking the recipient against the user's known accounts) or a runtime detector (MELON, Zhu et al. 2025) stacked on ODILE. We measured the stacking: an instruction-hierarchy / sandwich overlay on ODILE closes the worst residual InjecAgent cell (`dh_base`) from 16.0% to 2.0%, and the two defenses fail on largely disjoint cells, which is why they compose. (c) A controlled include-vs-exclude ablation (identical recipe, the three pairs the only difference) shows inclusion is pure downside: AD-standard ASR rises from 0.12% to 0.29% and benign utility falls 0.37pp, and every worsened sample traces to that one task ([table]({ANON}/tab_worsened_pairs.pdf)). A full stacking-as-defense ablation for this genuinely-ambiguous class is a camera-ready commitment.

**Q. "The authors mention why CRL is better than MSE for Llama and have a justification for it. However, Qwen MSE was better than Qwen Triplet, why?"**
Both objectives drive ASR to near 0, so the security result is not loss-specific; the difference is consistency. MSE behaves consistently across backbones; the triplet objective is strong on some backbones and weaker on others, so we report MSE as the headline and triplet as an ablation. Our working hypothesis: triplet relies on absolute distance margins, and hidden-state magnitudes differ substantially across families (Qwen-2.5/Qwen3 vs Llama; cf. massive activations, Sun et al. 2024), so a margin calibrated on one family is mis-scaled on another, while MSE does not depend on those magnitudes. We will add a dedicated loss-objective discussion with per-backbone margin-sensitivity ablations to test this directly.

**Q. "[...] the change in layer range showed significant effect [...]. Was it pure trial and error?"**
No: the mid-late band follows the circuit-breaker rationale (Zou et al.), depth-scaled across seven backbones with no per-backbone retuning. An adjacent-band check at Llama-3.1-8B (`L10-20`, `L12-22`, `L20-30`, 52 cells each) finds all three suppress ASR to near 0, but shifting the band later halves under-attack utility (≈10% vs ≈21%), so the mid band is the utility-preserving choice. A full native-objective sweep across all backbones is a camera-ready commitment.

**Q (white-box vs black-box: a different landscape).**
We agree and will adopt this framing explicitly. ODILE is an inner defense for the open-weight / self-hosted regime; for API-only deployments, pipeline and black-box defenses are the appropriate layer. We will state in Sections 1 and 10 that ODILE occupies a complementary deployment layer rather than competing.

In addition: **R1 (generalization):** four out-of-distribution benchmarks (InjecAgent, TensorTrust, WASP, AgentDyn) reach ASR near 0. **R2 (adaptive):** discrete and LoRA-aware GCG and TAP/PAIR, all 0 cracks ([table]({ANON}/tab_adaptive_tap_pair.pdf)).

We are grateful for the positive assessment and hope these clarifications on curation, loss, and layer selection further strengthen the paper. We would be glad to add any detail the reviewer finds useful, and would be grateful for maintaining or raising the score.

---

## DefG (Rating 4)

We thank the reviewer: defense comparison, a cleaner security-and-utility table, cross-framework generalization, and real-loop behavior are the axes we strengthen below.

**1. "[...] compare more directly against recent model-level defenses such as Meta SecAlign and firewall/sanitizer-style defenses [...] clarify the novelty of ODILE relative to these works."**
**Novelty:** ODILE is a representation-level LoRA at 1x inference cost (no second model pass, no input rewriting), trained by a paired-trace contrast on the model's own harmful direction.
**vs Meta-SecAlign** (Chen et al. 2026): we hold the low-ASR corner (0.01% vs 2.11%) at 1x cost ([70B table]({ANON}/tab_pareto_70b.pdf)). The differentiator is the ReAct agentic format: on InjecAgent, Meta-SecAlign cannot emit valid ReAct tool calls (88–95% of its outputs are unparseable across families, so its low ASR reflects broken output, not defense), whereas ODILE drives ASR to ≤ 0.15% while producing valid ReAct tool calls on every backbone ([figure]({ANON}/fig_react_jam_odile.pdf)).
**vs firewall/sanitizer** (Bhagwatkar et al. 2026): static parity (2.1% ASR), but the sanitizer costs −6.2pp benign utility plus an extra LLM call per tool output, and is cracked 15/48 under adaptive attack where ODILE holds 0/64 ([firewall table]({ANON}/T_firewall.pdf)).
ODILE is preferable when adaptive robustness, 1x cost, and benign preservation matter together. The broader model-level field, including ReasAlign (Li et al. 2026), is in the [cross-benchmark comparator]({ANON}/tab_comparator_8b.pdf).

**2. "Table 2 [...] mixes different models, utility definitions, and defense assumptions [...] add a cleaner comparison table [...]."**
We replace it with one table per backbone, each on a single model and a single metric pair (ASR vs benign utility): [70B]({ANON}/tab_pareto_70b.pdf), [8B]({ANON}/tab_pareto_8b.pdf), and the Qwen backbones. Under-attack utility is appendix-only by design: ODILE removes the attacker tool call (it jams under injection), so that metric reflects residual non-malicious throughput, not task completion. We will make the distinction explicit in the body.

**3. "Single-benchmark focus. [...] unclear how well this transfers to substantially different agent frameworks."**
The four benchmarks above span different frameworks, all reaching ASR near 0 with the same recipe: ReAct text (InjecAgent), a web agent (WASP), and a dynamic benchmark on three non-AgentDojo suites (AgentDyn), where the attack lands on base (22.1%) before ODILE drives it to 0.0%.

**4. "The defense behavior also feels brittle [...] unclear what happens in a real agent loop with retries, validators, or tool-call repair logic."**
The jam is by design, not a failure. We have prepared a matched trace for reference ([trace]({ANON}/traces_pdf/trace_base_executes_vs_odile_jams.pdf)): on the same task where the base calls `update_password`, ODILE emits no parseable tool call, and never produces the attacker action (0/5805 distinct WASP paraphrases; any residual `send_money` has a degenerate recipient that never resolves to the attacker IBAN). In a real loop, a retry re-prompts with the same injection and re-jams (the strict-deny endpoint), and deterministic validators or tool-call repair compose on top, since they act on the emitted call rather than the representation.

**Q. "Does the method defend against data-exfiltration attacks where the harmful action looks like normal information retrieval [...]?"**
Yes. InjecAgent's `ds_base` and `ds_enhanced` families are exactly exfiltration via a legitimate-looking tool call (retrieve X, then email it to the attacker); ODILE drives both to 0.00% at Llama-3.3-70B ([32 real attacks]({ANON}/examples_pdf/injecagent_data_exfil.pdf), genetic data / saved addresses / financial records sent via `GmailSendEmail`).

**Minor (Figure 2 legend).** We will declutter the legend and split the Llama and Qwen panels.

In addition: **R2 (adaptive):** GCG and TAP/PAIR, 0 cracks ([table]({ANON}/tab_adaptive_tap_pair.pdf)). **R3 (capability):** BFCL and Tau-bench retained ([table]({ANON}/tab_capability.pdf)). **R4:** seven backbones including Qwen3-Next-80B.

We hope the head-to-head comparisons and the cleaner per-backbone tables resolve the comparison and tradeoff concerns, and improve the overall soundness of the paper. We would be grateful for a reconsideration of the score.

---

## GHCC (Rating 5)

We thank the reviewer: deployment scope, model coverage, benchmark breadth, and adaptive robustness are the axes we address below.

**1. "Limited applicability to closed models. ODILE needs access to internal representations [...]."**
Agreed; a deliberate scope choice. ODILE is an inner defense for open-weight / self-hosted models, and a primary audience is model creators, who control the weights and can ship the adapter by default. API-only deployments use pipeline defenses (Spotlight, instruction-hierarchy, MELON, CaMeL), which ODILE complements. We will add a deployment-scope paragraph (§10).

**2. "Limited model coverage [...] include a newer Qwen model such as Qwen3 or justify the choice [...]."**
Six backbones plus Qwen3-Next-80B, including Qwen3-8B and Qwen3-32B; Qwen3-Next is the newest reasoning model requested, and ODILE transfers ([cross-model]({ANON}/T2_crossmodel_AD.pdf)).

**3. "Single-benchmark focus [...] additional benchmarks such as AgentDyn and WASP would strengthen the claims."**
Both added. **AgentDyn** (GitHub, Daily-Life, Shopping; absent from AgentDojo): the attack hijacks the base agent (Qwen3-8B 22.1%, n=560, trace-validated) and ODILE drives it to 0.0% (Fisher p < 1e-12, [table]({ANON}/tab_agentdyn.pdf)). **WASP** falls from 12.5% to 0/239 verified-tight (0/5805 per paraphrase); our setup runs WASP's 21 goals and templates without the live browser loop ([table]({ANON}/tab_T_wasp.pdf); camera-ready: full containerized version).

**4. "A security paper should test adaptive attacks that know the adapter, objective, layer range, or masking strategy."**
We add defense-aware white-box attacks on all four named axes, 0 cracks. **Adapter/layer-aware:** LoRA-aware GCG (Zou et al. 2023; bypass at the LoRA layers), 0/3. **Objective-aware:** TAP (Mehrotra et al. 2024), PAIR (Chao et al. 2024) with the objective described, 0/64 across two backbones (base 36/48 = positive control). **Mask-aware:** the completion-window mask is a loss localization, nothing input-side to route around. **Discrete GCG** (75 steps): 0/4, cross-entropy plateaus at 10.7–11.0 to 500 steps on a HarmBench (Mazeika et al. 2024) sweep ([loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf), [table]({ANON}/tab_adaptive_tap_pair.pdf)). RL-based attacks (Nasr et al. 2025) are planned.

**Q1. "What exactly does '88% benign capability retention' mean? [...] report benign utility, utility under attack, and ASR separately."**
It was one axis, the tool-call correctness on InjecAgent's benign split at 70B (base = 100%), not a global number. We now report ASR, benign utility, and under-attack utility separately, per backbone ([tables]({ANON}/tab_pareto_70b.pdf)).

**Q2. "Where does the 12% drop come from: malformed tool calls, incomplete chains, wrong answers, or unnecessary refusals?"**
Of the 12 of 100: none are malformed calls, incomplete chains, wrong answers, or refusals; all are format-envelope shifts (the strict-ReAct Action form becomes an equivalent expression, correct content). Metric strictness, not capability loss; we will list all 12.

**Q3. "Position against system-level defenses [...] how ODILE compares to defenses like CaMeL [...]."**
CaMeL (Debenedetti et al. 2025) is a design-level defense (privileged planner plus capability-checked interpreter), a different, complementary layer. It reaches near-0 ASR wherever its interpreter runs but on open weights is brittle: scale-floors at Qwen-2.5-14B (vacuous jam), crashes on 2/4 suites at 70B, costs 68–82% benign utility at its target tier. ODILE reaches 0% ASR at 1x cost on every backbone.

**Q4. "Compare with simpler sanitization defenses [...] explain when ODILE is preferable."**
We re-implemented the Bhagwatkar et al. (2026) firewall at 8B (a second LLM rewrites each tool output). It is useful but not invariant: statically it matches ODILE (2.1% ASR), but benign-looking paraphrases bypass it, and on deploy-token / SSH-key / PAT records it introduces cracks the base never had (base 0/16, sanitizer 3/12), because the rewriter cannot tell "the user wants a deploy token" from "the injection wants one." ODILE holds 0/64 and 0/5805 (Fisher vs sanitizer p=5e-10). Mechanistically the sanitizer acts on surface text while ODILE acts on hidden states (same-intent paraphrases converge), so ODILE is preferable when adaptive robustness, 1x cost, and benign preservation matter together ([firewall table]({ANON}/T_firewall.pdf)).

We hope the added benchmarks, adaptive-attack coverage, and deployment-scope framing address the reviewer's concerns. We would be grateful for a reconsideration of the score.
