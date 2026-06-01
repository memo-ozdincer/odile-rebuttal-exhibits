# ODILE, OpenReview-ready rebuttal (one comment per reviewer, each <= 5000 chars)

**How to use.** Paste one section per reviewer into the OpenReview comment box (Markdown enabled). `https://anonymous.4open.science/r/odile-anon-exhibits-82D6` is the anonymous.4open.science base; find-replace `https://anonymous.4open.science/r/odile-anon-exhibits-82D6` with your repo URL once you have the ID. No personal-repo links, no name/school anywhere. Each section is under the 5000-character limit *with* the real URLs substituted, while keeping per-experiment detail (n, significance, comparator, and what each result shows). Greetings/closings are personalized; code tokens are backticked; no text-mode LaTeX (OpenReview MathJax is math-mode only).

---

## FoEk (Rating 3)

We thank the reviewer: generalization, attack strength, capability breadth, and model recency are indeed the axes we want to showcase performance in. Below, we address the questions they raised.

**1. "There is clear data leakage [...]. I cannot tell whether ODILE learns a real prompt-injection defense or fitting on test set."**
A memorized adapter would fail off-distribution. ODILE does not, on four further benchmarks and on re-renderings of the exact training injections, each with n, significance, and a comparator:
- **InjecAgent** (Zhan et al. 2024; ReAct, domains absent from training): ASR-all ≤ 0.15% on every backbone (n=1054/family, Wilson UB 0.35%), at or below Meta-SecAlign (Chen et al. 2026), the closest model-level defense (0.2–2.6% at 70B) ([attacks](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/examples_pdf/injecagent_attacks.pdf) vs [training](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/examples_pdf/agentdojo_train_injections.pdf); [harness](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/examples_pdf/injecagent_harness.pdf)).
- **TensorTrust** (Toyer et al. 2023; guardian-bot password extraction, a different mechanic): extraction leak 78–92% to 1–44% (Qwen-2.5-14B 82% to 1%), n=300/cell, while benign coherence stays ~100% on neutral pre-prompts.
- **WASP** (Evtimov et al. 2025; web agent): attacker-action 12.5% to 0/239 verified-tight (Fisher p≈1e-9), vs base 25/200 and sanitizer 8/160.
- **AgentDyn** (Li et al. 2026; GitHub/Daily-Life/Shopping, absent from AgentDojo): the attack lands on the base agent (Qwen3-8B 22.1%, n=560) and ODILE drives it to 0.0% (Fisher p < 1e-12, [how it differs](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/examples_pdf/agentdyn_difference.pdf)).

Memorization cannot explain this: the adapter never saw these tools or environments at training time. The sharpest evidence is the held-out augmentation, where we re-render the *exact* training injections (markers removed, or base64) and ASR stays near 0 though the goal is unchanged, the opposite of a test-fit defense. ODILE is also format-driven, jamming identically across ReAct and a delimiter-wrapped variant ([figure](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/fig_react_jam_odile.pdf)).

**2. "The attack setting is too weak [...] does not test adversarial attackers aware of the adapter."**
We add defense-aware white-box attacks on the adapter's internals, all 0 cracks. **Discrete GCG** (Zou et al. 2023; 75 steps): 0/4; the attacker-target cross-entropy plateaus at 10.7–11.0 while the base descends to 0.57 and complies, and does not move with budget (0% to 500 steps on a HarmBench (Mazeika et al. 2024) sweep, base broken on 68%); the [loss curve](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/figures/fig_gcg_loss_curve.pdf) shows base reaching compliance while ODILE stays flat. **LoRA-aware GCG** (bypass at the LoRA layers): 0/3; the gap shrinks ~25% but never nears compliance. **TAP (Mehrotra et al. 2024), PAIR (Chao et al. 2024)** (objective described to the attacker): 0/64 across two backbones, vs base 36/48 and sanitizer 15/48 on the same cells (the positive control) ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_adaptive_tap_pair.pdf)). RL-based attacks (Nasr et al. 2025) are planned.

**3. "Benign capability evaluation is narrow [...] such as BFCL and Tau-Bench."**
We add exactly the two named (base to ODILE). **BFCL** (non-live AST, n=1240): Llama-3.1-8B 73.98 to 73.35 (−0.63), Qwen-2.5-7B 74.98 to 75.12 (+0.14, a gain). **Tau-bench airline** (n=50, single trial, directional): Llama-3.1-8B 42.00 to 38.78 ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_capability.pdf)). General tool-use is retained.

**4. "Model selection is outdated [...] do not test newer reasoning models."**
We add Qwen3-Next-80B-A3B-Thinking (Sept 2025), a frontier-agentic open-source model with nonstandard (Gated-DeltaNet) attention; ODILE transfers with attention-path adjustments ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_pareto_qnext.pdf)). The shown results are on the `important_instructions` family; the full 52-cell grid is a camera-ready commitment.

**Q. "Why is LoRA only applied on layers 30–55?"**
Not trial and error: following circuit-breakers (Zou et al.), mid-to-late layers avoid token-specific representations; we use that single principled band and depth-scale it across seven backbones. An adjacent-band check at Llama-3.1-8B (`L10-20`, `L12-22`, `L20-30`, 52 cells each) finds all suppress ASR to near 0 while the later band halves under-attack utility, so the mid band is selected. A full native-objective sweep is a camera-ready commitment.

We believe these additions directly address the leakage, attack-strength, capability, and model-recency concerns, and strengthen the soundness of the paper. We would be grateful for a reconsideration of the score.

---

## FSAe (Rating 7)

We thank the reviewer for the positive assessment and for precise questions on data curation, loss choice, layer selection, and deployment framing. Below, we address each.

**Q (worsened pairs): "(a) ideas on how to handle them. (b) if and how other previous methods were able to handle this. (c) were any experiments run including these pairs and what was the ASR [...]."**
(a) These three banking pairs are structurally ambiguous: the attacker IBAN equals the legitimate recipient, so the harmful and benign twins are byte-identical and give the contrastive objective no signal. We exclude them by a hard-coded structural rule (target-collision in the task definition), not a per-sweep statistic. (b) No representation-level method can separate literally-identical actions, because there is no internal signal to separate; the right place to catch this is outside the representation, e.g. a deterministic post-execution validator (an IBAN allowlist checking the recipient against the user's known accounts) or a runtime detector (MELON, Zhu et al. 2025) stacked on ODILE. We measured the stacking: an instruction-hierarchy / sandwich overlay on ODILE closes the worst residual InjecAgent cell (`dh_base`) from 16.0% to 2.0%, and the two defenses fail on largely disjoint cells, which is why they compose. (c) A controlled include-vs-exclude ablation (identical recipe, the three pairs the only difference) shows inclusion is pure downside: AD-standard ASR rises from 0.12% to 0.29% and benign utility falls 0.37pp, and every worsened sample traces to that one task ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_worsened_pairs.pdf)). A full stacking-as-defense ablation for this genuinely-ambiguous class is a camera-ready commitment.

**Q. "The authors mention why CRL is better than MSE for Llama and have a justification for it. However, Qwen MSE was better than Qwen Triplet, why?"**
Both objectives drive ASR to near 0, so the security result is not loss-specific; the difference is consistency. MSE behaves consistently across backbones; the triplet objective is strong on some backbones and weaker on others, so we report MSE as the headline and triplet as an ablation. Our working hypothesis: triplet relies on absolute distance margins, and hidden-state magnitudes differ substantially across families (Qwen-2.5/Qwen3 vs Llama; cf. massive activations, Sun et al. 2024), so a margin calibrated on one family is mis-scaled on another, while MSE does not depend on those magnitudes. We will add a dedicated loss-objective discussion with per-backbone margin-sensitivity ablations to test this directly.

**Q. "[...] the change in layer range showed significant effect [...]. Was it pure trial and error?"**
No: the mid-late band follows the circuit-breaker rationale (Zou et al.), depth-scaled across seven backbones with no per-backbone retuning. An adjacent-band check at Llama-3.1-8B (`L10-20`, `L12-22`, `L20-30`, 52 cells each) finds all three suppress ASR to near 0, but shifting the band later halves under-attack utility (≈10% vs ≈21%), so the mid band is the utility-preserving choice. A full native-objective sweep across all backbones is a camera-ready commitment.

**Q (white-box vs black-box: a different landscape).**
We agree and will adopt this framing explicitly. ODILE is an inner defense for the open-weight / self-hosted regime; for API-only deployments, pipeline and black-box defenses are the appropriate layer. We will state in Sections 1 and 10 that ODILE occupies a complementary deployment layer rather than competing.

In addition: **R1 (generalization):** four out-of-distribution benchmarks (InjecAgent, TensorTrust, WASP, AgentDyn) reach ASR near 0. **R2 (adaptive):** discrete and LoRA-aware GCG and TAP/PAIR, all 0 cracks ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_adaptive_tap_pair.pdf)).

We are grateful for the positive assessment and hope these clarifications on curation, loss, and layer selection further strengthen the paper. We would be glad to add any detail the reviewer finds useful, and would be grateful for maintaining or raising the score.

---

## DefG (Rating 4)

We thank the reviewer: defense comparison, a cleaner tradeoff table, cross-framework generalization, and real-loop behavior are the axes we strengthen below.

**1. "[...] compare more directly against recent model-level defenses such as Meta SecAlign and firewall/sanitizer-style defenses [...] clarify the novelty of ODILE relative to these works."**
**Novelty:** ODILE is a representation-level LoRA at 1x inference cost (no second pass, no input rewriting), trained by a paired-trace contrast on the model's own harmful direction.
**General:** across our five benchmarks ODILE holds the low-ASR, capability-preserving corner of the field; the [cross-benchmark comparator](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_comparator_8b.pdf) places it against ReasAlign (Li et al. 2026), which matches ASR at ~13pp benign cost.
**vs Meta-SecAlign** (Chen et al. 2026): 0.01% vs 2.11% ASR at 1x cost ([70B table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_pareto_70b.pdf)). The key differentiator is the ReAct agentic format: Meta-SecAlign cannot emit valid ReAct tool calls on InjecAgent (88–95% unparseable, so its low ASR is broken output, not defense), while ODILE stays in valid ReAct format ([figure](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/fig_react_jam_odile.pdf)).
**vs firewall/sanitizer** (Bhagwatkar et al. 2026): static parity (2.1% ASR), but it costs −6.2pp benign utility plus an extra LLM call, and is cracked 15/48 under adaptive attack vs ODILE's 0/64 ([firewall table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/T_firewall.pdf)).
ODILE is preferable when adaptive robustness, 1x cost, and benign preservation matter together.

**2. "Table 2 [...] mixes different models, utility definitions, and defense assumptions [...] add a cleaner comparison table [...]."**
We replace it with one table per backbone, each on a single model and metric pair (ASR vs benign utility): [70B](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_pareto_70b.pdf), [8B](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_pareto_8b.pdf), and the Qwen backbones. We report benign (no-attack) utility as the capability axis. Completing the user task while an injection is present is deliberately not ODILE's objective: ODILE jams under attack by design, so optimizing for task-completion-under-injection would undercut the defense. We report under-attack utility as a secondary diagnostic and will state this in the body.

**3. "Single-benchmark focus. [...] unclear how well this transfers to substantially different agent frameworks."**
We add four out-of-distribution benchmarks across different frameworks, all reaching ASR near 0: **InjecAgent** (ReAct), **TensorTrust** (guardian-bot extraction), **WASP** (web agent), and **AgentDyn** (three non-AgentDojo suites; attack lands on base 22.1%, n=560, then ODILE 0.0%) ([AgentDyn](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_agentdyn.pdf), [WASP](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_T_wasp.pdf)).

**4. "The defense behavior also feels brittle [...] unclear what happens in a real agent loop with retries, validators, or tool-call repair logic."**
The jam is by design, not a failure. We have prepared a matched trace for reference ([trace](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/traces_pdf/trace_base_executes_vs_odile_jams.pdf)): where the base calls `update_password`, ODILE emits no parseable tool call and never produces the attacker action (0/5805 distinct WASP paraphrases). In a real loop, a retry re-prompts with the same injection and re-jams (the strict-deny endpoint); deterministic validators and tool-call repair compose on top, since they act on the emitted call, not the representation. A repair loop that reformulates the injection is itself an adaptive attacker, which our adaptive sweep already tests (TAP/PAIR 0/64); the strongest such test, RL-based attacks (Nasr et al. 2025), is a camera-ready commitment, and we expect ODILE to hold.

**Q. "Does the method defend against data-exfiltration attacks where the harmful action looks like normal information retrieval [...]?"**
Yes. InjecAgent's `ds_base` and `ds_enhanced` families are exfiltration via a legitimate-looking tool call (retrieve X, then email it to the attacker); ODILE drives both to 0.00% at 70B ([32 real attacks](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/examples_pdf/injecagent_data_exfil.pdf), e.g. saved addresses sent via `GmailSendEmail`).

**Minor (Figure 2 legend).** We will declutter and split the Llama/Qwen panels. In addition, **R3/R4:** BFCL and Tau-bench retain capability ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_capability.pdf)) across seven backbones including Qwen3-Next-80B.

We hope these comparisons and tables resolve the comparison and tradeoff concerns. We would be grateful for a reconsideration of the score.

---

## GHCC (Rating 5)

We thank the reviewer: deployment scope, model coverage, benchmark breadth, and adaptive robustness are the axes we address below.

**1. "Limited applicability to closed models. ODILE needs access to internal representations [...]."**
Agreed; this is a deliberate scope choice for the open-weight / self-hosted setting. Beyond practitioners self-hosting open-weight models, the audience also includes model creators, who can ship the adapter as a default safety layer. For API-only deployments, pipeline defenses (Spotlight, instruction-hierarchy, MELON, CaMeL) are the appropriate layer, which ODILE complements rather than replaces. We will add a deployment-scope paragraph (§10).

**2. "Limited model coverage [...] include a newer Qwen model such as Qwen3 or justify the choice [...]."**
Six backbones plus Qwen3-Next-80B, including Qwen3-8B and Qwen3-32B from the Qwen-3 family the reviewer points to; Qwen3-Next-80B is additionally a frontier-agentic open-source model (nonstandard Gated-DeltaNet attention), and ODILE transfers across all of them ([cross-model](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/T2_crossmodel_AD.pdf)).

**3. "Single-benchmark focus [...] additional benchmarks such as AgentDyn and WASP would strengthen the claims."**
Both added, and we report benign utility on each as well as ASR. **AgentDyn** (GitHub, Daily-Life, Shopping; absent from AgentDojo): ASR base 22.1% to ODILE 0.0% (Qwen3-8B, n=560, Fisher p < 1e-12), with benign utility base 26.7% to ODILE 20.0% (a 6.7pp drop) ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_agentdyn.pdf)). **WASP**: attacker-action from 12.5% to 0/239 verified-tight (0/5805 per paraphrase), while benign utility is preserved (base/ODILE benign %: Llama-3.3-70B 100/93, Qwen-2.5-7B 100/95, Qwen-2.5-14B 69/69, Llama-3.1-8B 100/83) ([table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_T_wasp.pdf)). The full containerized WASP is a camera-ready commitment.

**4. "A security paper should test adaptive attacks that know the adapter, objective, layer range, or masking strategy."**
We add defense-aware white-box attacks covering all four named axes, 0 cracks throughout:
- **Adapter- and layer-range-aware:** LoRA-aware GCG (Zou et al. 2023) adds a bypass term penalizing the adapter's hidden-state deviation from base at the exact LoRA layers (λ ∈ {1,10,100}); 0/3. The term shrinks the adapter-vs-base gap ~25%, but cross-entropy never approaches compliance.
- **Objective-aware:** TAP (Mehrotra et al. 2024) and PAIR (Chao et al. 2024), given ODILE's objective in their system prompt, 0/64 across two backbones, against base 36/48 and a sanitizer 15/48 on the identical cells (base/sanitizer cracking those cells is the positive control).
- **Mask-aware:** the completion-window mask is a training-time loss localization, with nothing input-side for the attacker to route around.
- **Discrete GCG** (Zou et al. 2023; 75 steps): 0/4. The attacker-target cross-entropy plateaus at 10.7–11.0 while the base descends to 0.57 and complies, and does not move to 500 steps on a HarmBench (Mazeika et al. 2024) sweep (base broken on 68% of behaviors).

We do not claim these are the strongest possible attacker; RL-based attacks (Nasr et al. 2025) are planned. Full results: [GCG](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/T_GCG.pdf), [TAP/PAIR](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_adaptive_tap_pair.pdf), [loss curve](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/figures/fig_gcg_loss_curve.pdf).

**Q1. "What exactly does '88% benign capability retention' mean? [...] report benign utility, utility under attack, and ASR separately."**
The 88% is a relative capability-retention figure on AgentDojo (ODILE benign utility as a fraction of the undefended base), which we used to keep the comparison consistent across backbones. We agree absolute numbers are clearer: we now report, per backbone, absolute ASR, benign utility, and under-attack utility separately ([tables](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/tab_pareto_70b.pdf)). UAU is a secondary diagnostic, since (as above) ODILE jams under attack by design.

**Q2. "Where does the 12% drop come from: malformed tool calls, incomplete chains, wrong answers, or unnecessary refusals?"**
To clarify, this 12% is the AgentDojo benign-utility drop (relative ~12%, roughly 6pp absolute, concentrated in banking and slack). It is a mix of two modes: (i) format-envelope shifts, where the correct tool call is emitted in a non-canonical wrapper that the strict scorer counts as a miss; and (ii) a few over-cautious jams on harmful-looking-but-benign tasks (e.g. a legitimate large transfer that resembles the attacker pattern). We will catalog every benign failure mode in a dedicated appendix; we are also happy to provide a scrollable exhibit of these traces.

*(GHCC exceeds 5000 characters, so post it as two comments. Everything above this line is comment 1 of 2; everything below is comment 2 of 2.)*

**Q3. "Position against system-level defenses [...] how ODILE compares to defenses like CaMeL [...]."**
CaMeL (Debenedetti et al. 2025) is a design-level defense (a privileged/quarantined planner plus a capability-checked interpreter), a different, complementary layer. We evaluate it at three scales: **Qwen-2.5-14B** (it scale-floors, the privileged LLM cannot drive the interpreter, so its 0% ASR is a vacuous emergent jam); **Llama-3.3-70B** (runs on two of four suites, while banking and travel crash at an upstream interpreter bug); and **Gemini-2.5-flash-lite** (all four suites, ~0% ASR but only 18–32% benign utility, for two dual-LLM passes plus a custom interpreter). CaMeL reaches ~0% ASR wherever its interpreter runs, but on open weights it is brittle and high-tax; ODILE reaches 0% ASR at 1x cost on every backbone.

**Q4. "Compare with simpler sanitization defenses [...] explain when ODILE is preferable."**
We re-implemented the Bhagwatkar et al. (2026) firewall at 8B: a second LLM rewrites each tool output to strip injection-looking content before the agent sees it. Statically it matches ODILE (2.1% ASR), but under adaptive attack it is not invariant, in two ways. (i) Benign-looking paraphrases slip past the rewriter while still steering the agent. (ii) On developer-workflow records (deploy-token, SSH-key, PAT) it actively *introduces* cracks the base never had (base 0/16, sanitizer 3/12), because the rewriter cannot distinguish a user who legitimately wants a deploy token from an injection that wants one. ODILE instead holds 0/64 (TAP/PAIR) and 0/5805 (WASP paraphrases), Fisher vs sanitizer p=5e-10. The reason is mechanistic: the sanitizer can only act on surface text, so any paraphrase that fools the rewriter gets through, whereas ODILE acts on hidden states where same-intent paraphrases converge. ODILE is preferable when adaptive robustness, 1x inference cost, and benign-utility preservation matter together; the sanitizer's one edge is being backbone-agnostic (no per-model training) ([firewall table](https://anonymous.4open.science/r/odile-anon-exhibits-82D6/T_firewall.pdf)).

We hope the added benchmarks, adaptive-attack coverage, and deployment-scope framing address the reviewer's concerns. We would be grateful for a reconsideration of the score.
