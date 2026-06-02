# ODILE, OpenReview-ready rebuttal (full version)

**How to use.** Each reviewer's response is posted to OpenReview as one or more comments (5000-char limit per comment): **FoEk = 3 comments, DefG = 2, GHCC = 3, FSAe = 1**. The italic `*(... comment X of N ...)*` notes mark the split points — do NOT paste those notes; just start a new comment there. Replace `{ANON}` with your anonymous.4open.science repo URL once you have the ID (no personal-repo links, no name/school anywhere). Greetings/closings are personalized per reviewer; each reviewer keeps its per-concern answers plus a "see also" R-list. Code tokens are backticked so underscores render; no text-mode LaTeX (OpenReview MathJax is math-mode only). A shared **References** section at the end lists every cited work with a clickable link; post it as a brief final comment in each reviewer thread.

---

## FoEk (Rating 3)

We thank the reviewer: generalization, attack strength, capability breadth, and model recency are indeed the axes we want to showcase performance in. Below, we address the questions they raised.

**1. "There is clear data leakage [...]. I cannot tell whether ODILE learns a real prompt-injection defense or fitting on test set."**

ODILE trains on AgentDojo (Debenedetti et al. 2024); to show this is a learned defense and not test-set fitting, we add four further benchmarks and a held-out attack-augmentation set, none seen at training. A memorized adapter would fail on these; it does not, each reported with sample size, significance, a comparator, and a concrete demonstration that it is genuinely different.
- **InjecAgent** (Zhan et al. 2024; different benchmark, ReAct format, attack domains absent from training): ODILE drives ASR-all to ≤0.15% on every backbone (n=1054/family, Wilson upper bound 0.35% at Qwen-2.5-14B). The attacks share no surface form with training: compare the [30 direct-harm]({ANON}/examples_pdf/injecagent_attacks.pdf) and [32 data-stealing]({ANON}/examples_pdf/injecagent_data_exfil.pdf) attacks against [the AgentDojo injections ODILE trained on]({ANON}/examples_pdf/agentdojo_train_injections.pdf); a [harness walk-through]({ANON}/examples_pdf/injecagent_harness.pdf) shows where the injection hides and the base-vs-ODILE branches. General tool-use is retained (BFCL/τ-bench, answer 3). Full numbers: [InjecAgent table]({ANON}/T3b_injecagent_crossmodel.pdf).
- **TensorTrust** (Toyer et al. 2023; a different mechanic, guardian-bot password extraction): extract-leak falls from 78–92% to 1–44% across backbones (Qwen-2.5-14B from 82 to 1), n=300/cell; benign coherence is preserved at ~100% on neutral pre-prompts. [91 real attacks]({ANON}/examples_pdf/tensortrust_attacks.pdf); full numbers in the [TensorTrust table]({ANON}/T4_TT_headline.pdf).
- **WASP** (Evtimov et al. 2025; web-agent injection): the attacker-action rate falls from 12.5% to 0/239 verified-tight (0/5805 per individual paraphrase attempt, Fisher p≈1e-9), against base 25/200 and a sanitizer at 8/160, with benign utility preserved (base/ODILE, e.g. Llama-3.3-70B 100/93, Qwen-2.5-7B 100/95). [21 real attacker goals]({ANON}/examples_pdf/wasp_attacker_goals.pdf); full numbers in the [WASP table]({ANON}/tab_T_wasp.pdf). (Our setup omits the live browser loop; the full containerized version is a camera-ready commitment.)
- **AgentDyn** (Li et al. 2026; a dynamic benchmark on three suites absent from AgentDojo, namely GitHub, Daily-Life, and Shopping): on Qwen3-8B the attack genuinely lands on the base agent (22.1%, n=560) and ODILE drives it to 0.0% (Fisher p < 1e-12) while benign utility is largely preserved (26.7% to 20.0%); we ran this on Qwen3-8B for the rebuttal and will extend to all backbones for camera-ready. [How it differs, with real tasks and goals]({ANON}/examples_pdf/agentdyn_difference.pdf); full numbers in the [AgentDyn table]({ANON}/tab_agentdyn.pdf).
- **AgentDojo attack-augmentations absent from training** (the same goal rendered with the injection markers removed, or base64-encoded): ASR remains ≈0 on surface forms the adapter never saw. [The augmented forms]({ANON}/examples_pdf/agentdojo_heldout_augmentations.pdf).

On AgentDyn, the base model ODILE is built on is hijacked 22% of the time, so the drop to ≈0 reflects a learned response to harmful intent, not recall. The held-out augmentation isolates this: we take the *exact* AgentDojo injections the adapter trained on and merely re-render their surface (markers removed, or base64-encoded), and ASR stays ≈0 even though the underlying goal is unchanged, which is the opposite of what a defense fit to the training strings would do. On the defense comparison, ODILE also matches ReasAlign's near-zero ASR without its ~13pp benign-utility cost ([comparator]({ANON}/tab_comparator_8b.pdf)), and holds far lower ASR than Meta-SecAlign on AgentDojo and InjecAgent ([70B table]({ANON}/tab_pareto_70b.pdf)). These benchmarks will be completed across every backbone for the camera-ready release.

*(FoEk is long; post it as 3 comments. End of comment 1 of 3. Comment 2 begins below.)*

**2. "The attack setting is too weak [...]. does not test adversarial attackers aware of the adapter."**

We add stronger, defense-aware dynamic attacks that target the adapter's internals, directly probing the four axes one would expect a representation defense to be tested on:
- **Discrete GCG** (Zou et al. 2023; white-box, full weight, adapter, and gradient access): a 20-token adversarial suffix optimized for 75 steps at search-width 128. 0/4 crackable pairs. The attacker-target cross-entropy plateaus near 10.7–11.0 and never descends into the compliance regime, whereas on the same pairs the undefended base reaches 0.57 and complies, so the attack mechanism works and ODILE is what stops it. The plateau does not move with budget: on a HarmBench (Mazeika et al. 2024) text-generation GCG sweep, ODILE held 0% attack success even when the search was extended to 500 steps and to adaptive hidden-state targeting (base Llama-3.3-70B is broken on 68% of behaviors there), and the [loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf) shows the base descending to compliance while ODILE stays flat. [GCG table]({ANON}/T_GCG.pdf).
- **LoRA-aware GCG** (the loss adds a bypass term penalizing the adapter's hidden-state deviation from base at the exact LoRA layers): λ_bypass ∈ {1, 10, 100}, 50 steps. 0/3; the bypass term shrinks the adapter-vs-base gap by ~25% but cross-entropy never approaches compliance (from ~12.0 to ~11.9). This covers the adapter-aware and layer-range-aware axes.
- **TAP (Mehrotra et al. 2024) and PAIR (Chao et al. 2024)** (semantic attackers given ODILE's objective): 0/64 across two backbones, comprising 0/48 at the standard tree-search budget plus 0/16 when every surviving cell is re-attacked with a deeper tree (depth 5, branching 3, width 6, 2 parallel streams, against the standard depth 3, branching 2, width 4, 1 stream), against base 36/48 and a sanitizer firewall 15/48 on the identical cells. Base and sanitizer cracking those exact cells is the positive control. [TAP/PAIR table]({ANON}/tab_adaptive_tap_pair.pdf). This covers the objective-aware axis.

Another adaptive experiment we plan to run is a reinforcement-learning-based attack (Nasr et al. 2025, "The Attacker Moves Second", arXiv:2510.09023), alongside multi-suffix and transfer-ensemble GCG and a broader sweep across all backbones.

**3. "Benign capability evaluation is narrow [...] such as BFCL and Tau-Bench."**

We add exactly the two benchmarks named, reporting base and ODILE with the change in parentheses:
- **BFCL** (Patil et al. 2024; non-live AST, n=1240/cell): Llama-3.1-8B from 73.98 to 73.35 (−0.63); Qwen-2.5-7B from 74.98 to 75.12 (+0.14, a gain). General tool-call capability is retained.
- **τ-bench airline** (Yao et al. 2024; n=50 tasks, single trial, so directional): Llama-3.1-8B from 42.00 to 38.78 (−3.22).

Full per-backbone capability (with AlpacaEval, GSM8K, IFEval) is in the [capability table]({ANON}/tab_capability.pdf). Camera-ready: BFCL and τ-bench across all remaining backbones.

**4. "Model selection is outdated [...] do not test newer reasoning models."**

We add the Qwen-3 models, none of which were in the original submission: Qwen3-8B and Qwen3-32B from the Qwen-3 family the reviewer points to, and Qwen3-Next-80B-A3B-Thinking (Qwen Team 2025; September 2025), a frontier-agentic open-source model with nonstandard (Gated-DeltaNet) attention. ODILE transfers across all of them, with attention-path adjustments for Qwen3-Next ([Qwen3-Next table]({ANON}/tab_pareto_qnext.pdf), [cross-model]({ANON}/T2_crossmodel_AD.pdf)). The Qwen3-Next results shown are on the `important_instructions` family; we commit to reporting the full 52-cell AgentDojo grid on Qwen3-Next across all four suites, together with its TensorTrust and AgentDyn cells, in the camera-ready.

*(End of comment 2 of 3. Comment 3 begins below.)*

**Q. "Why is LoRA only applied on layers 30–55?"**

Not by trial and error: following circuit-breakers (Zou et al.), mid-to-late layers avoid token-specific representations; we use that single principled band and depth-scale it across all seven backbones with no per-backbone retuning, which is itself why the recipe transfers. An adjacent-band check on Llama-3.1-8B (windows L10–20, L12–22, and L20–30, 52 cells each) finds all three suppress ASR to ≈0 while shifting the band later halves under-attack utility (≈10% vs ≈21%), so the deployed mid band is the utility-preserving choice. A full native-objective layer-band sweep across all backbones is a camera-ready commitment.

In addition, we invite the reviewer to consult the rest of our supplementary rebuttal experiments:
- **R5 (Defense comparison):** matched-backbone traces vs Meta-SecAlign and a firewall/sanitizer, showing where they break and ODILE holds ([traces]({ANON}/traces_pdf/trace_secalign_cracked_vs_odile.pdf), [firewall table]({ANON}/T_firewall.pdf)).

We believe these additions directly address the leakage, attack-strength, capability, and model-recency concerns, and strengthen the soundness of the paper. We would be grateful for a reconsideration of the score.

---

## FSAe (Rating 7)

We thank the reviewer for the positive assessment and for precise questions on data curation, loss choice, layer selection, and deployment framing. Below, we address each.

**Q (worsened pairs). "(a) ideas on how to handle them. (b) if and how other previous methods were able to handle this. (c) were any experiments run including these pairs and what was the ASR [...]."**

- **(a) How we handle them.** The three banking pairs are structurally ambiguous: the only thing that differs between the harmful and benign twin is the recipient IBAN, which is hard to represent in intent space, so at the middle layers ODILE's contrastive objective operates on, the two converge to nearly the same representation and give it almost no signal to separate them. We exclude them by a hard-coded structural rule (target-collision in the task definition), not a per-sweep statistic.
- **(b) Prior methods.** No representation-level method can separate two literally identical actions; the right place to catch this is outside the representation. Two options stack on top of ODILE: a deterministic post-execution validator (an IBAN allowlist checking the recipient against the user's known accounts), or a runtime divergence detector such as MELON (Zhu et al. 2025). Stacking helps: an instruction-hierarchy overlay closes the worst residual InjecAgent cell (`dh_base`) from 16.0% to 2.0%, and the two defenses fail on disjoint cells.
- **(c) Experiments including the pairs.** A controlled include-vs-exclude ablation (identical recipe, the three pairs the only difference) shows inclusion is pure downside: AD-standard ASR rises 0.18pp (0.12% to 0.29%) and benign utility falls 0.37pp, and every worsened sample traces to that one task ([ablation table]({ANON}/tab_worsened_pairs.pdf)). Camera-ready: a full stacking-as-defense ablation for this ambiguous class.

**Q. "The authors mention why CRL is better than MSE for Llama and have a justification for it. However, Qwen MSE was better than Qwen Triplet, why?"**

Both objectives drive ASR to ≈0, so the result is not loss-specific; the difference is consistency. MSE behaves consistently across backbones, while triplet is strong on some and weaker on others, so we report them as an ablation. Our hypothesis: triplet relies on absolute distance margins, and hidden-state magnitudes differ across families (Qwen vs Llama; cf. massive activations, Sun et al. 2024), so a margin calibrated on one family is mis-scaled on another, whereas MSE does not depend on those magnitudes. Camera-ready: a dedicated loss-comparison discussion with per-backbone margin-sensitivity ablations to test this directly.

**Q. "[...] the change in layer range showed significant effect [...]. Was it pure trial and error?"**

No. The mid-late band follows the circuit-breaker rationale (Zou et al. 2024), depth-scaled and transfer-validated across seven backbones. We ran an adjacent-band check on Llama-3.1-8B (L10–20, L12–22, L20–30; 52 cells each): all three suppress ASR to ≈0, but the later band halves under-attack utility (≈10% vs ≈21%), so the mid band is selected for utility. Camera-ready: a full native-objective sweep across all backbones.

**Q (whitebox/blackbox "different landscape").**

We agree and will adopt the framing. ODILE is an inner defense for the open-weight / self-hosted regime; for API-only deployments, pipeline and blackbox defenses are the appropriate layer. We will state in §1 and §10 that it occupies a different, complementary deployment layer.

For reference, the rest of our supplementary experiments:
- **R1 (Generalization):** four out-of-distribution benchmarks (InjecAgent, TensorTrust, WASP, AgentDyn), ASR to ≈0.
- **R2 (Adaptive robustness):** discrete and LoRA-aware GCG plus TAP/PAIR, 0 cracks.
- **R4 (Model coverage):** seven backbones, including the frontier-agentic open-source Qwen3-Next-80B.

We are happy to add any further detail the reviewer finds useful, and hope these clarifications support maintaining or raising the score.

---

## DefG (Rating 4)

We thank the reviewer: defense comparison, a cleaner tradeoff table, cross-framework generalization, and real-loop behavior are the axes we strengthen below.

**1. "The paper should compare more directly against recent model-level defenses such as Meta SecAlign and against strong firewall/sanitizer-style defenses. Could the authors clarify the novelty of ODILE [...]?"**

ODILE's novelty is being a representation-level LoRA at 1x inference cost (no second model pass, no input rewriting), trained by a paired-trace contrast on the model's own harmful direction; empirically, it reaches the lowest ASR while preserving benign utility across four benchmarks. Against the defenses named:

- **vs Meta-SecAlign** (Chen et al. 2026), head-to-head at Llama-3.3-70B. On **AgentDojo**, ODILE holds 0.01% ASR vs Meta-SecAlign's 2.11% at 1x cost, while Meta-SecAlign retrains the full model and carries higher benign utility (62.6% vs 51.5%); they sit at different points on the security-utility frontier ([70B table]({ANON}/tab_pareto_70b.pdf)). We will add Meta-SecAlign comparisons on WASP and AgentDyn for the camera-ready.
- **Format robustness:** Meta-SecAlign does not transfer to InjecAgent's ReAct format: it cannot emit valid tool calls (88–95% of its outputs are unparseable, so its low ASR there is broken output, not defense), which is why a head-to-head ASR on InjecAgent would not be meaningful. ODILE stays in valid ReAct format and jams identically across ReAct and delimiter-wrapped formats ([figure]({ANON}/fig_react_jam_odile.pdf), [InjecAgent table]({ANON}/t_injecagent_full.pdf)).
- **vs firewall/sanitizer** (Bhagwatkar et al. 2026): a second LLM rewrites each tool output. Static parity (2.1% ASR), but it costs −6.2pp benign utility plus an extra LLM call per output, and is cracked 15/48 under adaptive attack where ODILE holds 0/64 ([firewall table]({ANON}/T_firewall.pdf)).
- **Broader field:** the [cross-benchmark comparator]({ANON}/tab_comparator_8b.pdf) places ODILE head-to-head with ReasAlign (Li et al. 2026) on all five benchmarks at once; ReasAlign matches ODILE on ASR but at ≈13pp AgentDojo benign cost and ≈4pp BFCL (and on InjecAgent its near-zero ASR is the same format-rejection artifact, valid-rate 0.3% vs ODILE's 43.7%). We also benchmark adjacent stack layers so the comparison is not single-family: MELON (Zhu et al. 2025), a runtime detector, and CaMeL (Debenedetti et al. 2025), a design-level planner/interpreter (per-backbone Pareto tables in answer 2).

ODILE is preferable when adaptive robustness, 1x cost, and benign preservation matter together.

**2. "Table 2 is a bit hard to interpret. It mixes different models, utility definitions, and defense assumptions [...]." / "[...] add a cleaner comparison table where everything is on the same model and same utility metric."**

We replace Table 2 with one table per backbone, each on a single model with a single metric pair (ASR against benign utility): [70B]({ANON}/tab_pareto_70b.pdf), [8B]({ANON}/tab_pareto_8b.pdf), [Qwen-2.5-7B]({ANON}/tab_pareto_q25_7b.pdf), [Qwen-2.5-14B]({ANON}/tab_pareto_q25_14b.pdf), [Qwen3-32B]({ANON}/tab_pareto_q3_32b.pdf). For camera-ready we will complete the full per-backbone tables and present them as security-vs-utility Pareto curves. On under-attack utility: we report benign (no-attack) utility as the capability axis. Completing the user task while an injection is present is deliberately not ODILE's objective: ODILE jams under attack by design (it never emits the attacker's tool call), so optimizing for task-completion-under-injection would undercut the defense. We therefore report under-attack utility for completeness, though it is not a primary metric for this paper, and we have made this framing explicit in the camera-ready copy.

*(DefG exceeds 5000 characters; post it as 2 comments. End of comment 1 of 2. Comment 2 begins below.)*

**3. "Single-benchmark focus. The evaluation is centered on AgentDojo [...]."**

We add four out-of-distribution benchmarks across different agent frameworks, all reaching ASR ≈0 with the same recipe: InjecAgent (ReAct), TensorTrust (guardian-bot extraction), WASP (web agent), and AgentDyn (a dynamic benchmark on three non-AgentDojo suites). On AgentDyn the attack genuinely lands on the base agent (Qwen3-8B, 22.1% to 0.0% ASR, n=560) and benign utility holds (26.7% to 20.0%); WASP benign is largely preserved (e.g. Llama-3.3-70B 100/93, Qwen-2.5-7B 100/95); TensorTrust coherence stays ~100% and general tool-use is retained (BFCL/τ-bench, R3). ([InjecAgent]({ANON}/T3b_injecagent_crossmodel.pdf), [TensorTrust]({ANON}/T4_TT_headline.pdf), [WASP]({ANON}/tab_T_wasp.pdf), [AgentDyn]({ANON}/tab_agentdyn.pdf)).

**4. "The defense behavior also feels brittle [...]. it is unclear what happens in a real agent loop with retries, validators, or tool-call repair logic."**

The worry is that the degenerate (token-soup) output could still trigger a harmful action; it does not, by design. Under attack ODILE emits no parseable tool call, so the harness has nothing to execute and the attacker action is never produced. On the same task where the base model calls `update_password`, ODILE reads the file and emits token-soup with no tool call ([trace]({ANON}/traces_pdf/trace_base_executes_vs_odile_jams.pdf)). For the three mechanisms the reviewer names:
- **Retries:** the retry-on-error handler re-prompts the model with the same injection (the strict-deny endpoint), so it jams again rather than complying.
- **Validators and tool-call repair:** both act on the emitted call, not on the representation, so they compose on top of ODILE rather than fighting it.
- **A repair loop that iteratively reformulates the injection** is itself an adaptive attacker, which is exactly the setting we run TAP and PAIR to probe (0/64).

The jam never produces the attacker action: 0/5805 distinct WASP paraphrases, trace-validated ([trace]({ANON}/traces_pdf/trace_secalign_cracked_vs_odile.pdf)).

**Q. "Does the method defend against data-exfiltration attacks where the harmful action looks like normal information retrieval [...]?"**

Yes. InjecAgent's `ds_base` and `ds_enhanced` families are exactly exfiltration via a legitimate-looking tool call (retrieve X, then email or send it to the attacker); we invite the reviewer to consult the [32 real data-stealing attacks]({ANON}/examples_pdf/injecagent_data_exfil.pdf) (genetic data, saved addresses, financial records sent via `GmailSendEmail`). ODILE drives both families to 0.00% at Llama-3.3-70B ([InjecAgent]({ANON}/T3b_injecagent_crossmodel.pdf)).

**Minor (Figure 2 legend).** We will declutter the legend and split the Llama and Qwen panels.

In addition, we invite the reviewer to consult the rest of our supplementary rebuttal experiments:
- **R2 (Adaptive robustness):** discrete and LoRA-aware GCG and TAP/PAIR semantic attacks, all 0 cracks ([GCG]({ANON}/T_GCG.pdf), [TAP/PAIR]({ANON}/tab_adaptive_tap_pair.pdf)).
- **R3 (Capability):** BFCL and τ-bench confirm general tool-use is retained ([table]({ANON}/tab_capability.pdf)).
- **R4 (Model coverage):** seven backbones including the September-2025 frontier-agentic open-source model Qwen3-Next-80B.

We hope the head-to-head comparisons and the cleaner per-backbone tables resolve the comparison and tradeoff concerns. We would be grateful for a reconsideration of the score.

---

## GHCC (Rating 5)

We thank the reviewer: deployment scope, model coverage, benchmark breadth, and adaptive robustness are the axes we address below.

**1. "Limited applicability to closed models. ODILE needs access to internal representations [...]."**

Agreed; this is a deliberate scope choice for the open-weight / self-hosted setting. Beyond researchers self-hosting open-weight models, the audience also includes model creators, who can ship the adapter as a default safety layer. For API-only deployments, pipeline defenses (Spotlight, instruction-hierarchy, MELON, CaMeL) are the appropriate layer, which ODILE complements rather than replaces. We will add an explicit deployment-scope paragraph in §10.

**2. "Limited model coverage. The evaluation uses mainly Llama-3.3-70B and Qwen-2.5-7B [...] include a newer Qwen model such as Qwen3 or justify the choice [...]."**

We have added the Qwen-3 models the reviewer points to. Coverage is now six backbones plus Qwen3-Next-80B: from the Qwen-3 family we added Qwen3-8B and Qwen3-32B, and we additionally added Qwen3-Next-80B, a frontier-agentic open-source model (nonstandard Gated-DeltaNet attention). ODILE transfers across all of them ([cross-model]({ANON}/T2_crossmodel_AD.pdf), [Qwen3-Next]({ANON}/tab_pareto_qnext.pdf)).

**3. "Single-benchmark focus [...] additional benchmarks such as AgentDyn and WASP would strengthen the claims."**

We add both, and report benign utility on each as well as ASR. **AgentDyn** is a dynamic benchmark on three suites absent from AgentDojo (GitHub, Daily-Life, Shopping; [how it differs]({ANON}/examples_pdf/agentdyn_difference.pdf)), where the attack genuinely hijacks the base agent (Qwen3-8B, 22.1%, n=560, trace-validated) and ODILE drives it to 0.0% (Fisher p < 1e-12), with benign utility largely preserved (base 26.7% to ODILE 20.0%); we ran AgentDyn on Qwen3-8B for the rebuttal and will extend to all backbones for camera-ready ([table]({ANON}/tab_agentdyn.pdf)). **WASP** falls from 12.5% to 0/239 verified-tight (0/5805 per paraphrase attempt), while benign utility is preserved (base/ODILE benign %: Llama-3.3-70B 100/93, Qwen-2.5-7B 100/95, Qwen-2.5-14B 69/69, Llama-3.1-8B 100/83); our setup runs WASP's [21 attacker goals and templates]({ANON}/examples_pdf/wasp_attacker_goals.pdf) without the live browser loop, the full containerized version a camera-ready commitment ([table]({ANON}/tab_T_wasp.pdf)).

**4. "No adaptive attack evaluation [...] adaptive attacks that know the adapter, objective, layer range, or masking strategy."**

We add defense-aware white-box attacks covering all four named axes, 0 cracks throughout:
- **Adapter- and layer-range-aware:** LoRA-aware GCG (Zou et al. 2023) adds a bypass term penalizing the adapter's hidden-state deviation from base at the exact LoRA layers (λ ∈ {1,10,100}); 0/3. The term shrinks the adapter-vs-base gap ~25%, but cross-entropy never approaches compliance.
- **Objective-aware:** TAP (Mehrotra et al. 2024) and PAIR (Chao et al. 2024), given ODILE's objective in their system prompt, 0/64 across two backbones, against base 36/48 and a sanitizer 15/48 on the identical cells (the positive control).
- **Mask-aware:** the completion-window mask is a training-time loss localization, with nothing input-side for the attacker to route around.
- **Discrete GCG** (Zou et al. 2023; 75 steps): 0/4. The attacker-target cross-entropy plateaus at 10.7–11.0 while the base descends to 0.57 and complies, and does not move to 500 steps on a HarmBench (Mazeika et al. 2024) sweep (base broken on 68% of behaviors).

Another adaptive experiment we plan to run is a reinforcement-learning-based attack (Nasr et al. 2025). Full results: [GCG]({ANON}/T_GCG.pdf), [TAP/PAIR]({ANON}/tab_adaptive_tap_pair.pdf), [loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf).

**Q1. "What exactly does '88% benign capability retention' mean? [...] most security papers report benign utility, utility under attack, and ASR separately."**

The 88% is a relative capability-retention figure on AgentDojo (ODILE benign utility as a fraction of the undefended base), which we used to keep the comparison consistent across backbones. We agree absolute numbers are clearer: we now report, per backbone, absolute ASR, benign utility, and under-attack utility separately ([tables]({ANON}/tab_pareto_70b.pdf)). UAU is not a primary metric for this work, since (as above) ODILE jams under attack by design.

*(GHCC is long; post it as 3 comments. End of comment 1 of 3. Comment 2 begins below.)*

**Q2. "Explain benign utility loss. Where does the 12% drop come from: malformed tool calls, incomplete chains, wrong answers, or unnecessary refusals?"**

To clarify, this 12% is the AgentDojo benign-utility drop (relative ~12%, roughly 6pp absolute, concentrated in banking and slack). It is a mix of two modes: (i) format-envelope shifts, where the correct tool call is emitted in a non-canonical wrapper that the strict scorer counts as a miss; and (ii) a few over-cautious jams on harmful-looking-but-benign tasks (e.g. a legitimate large transfer that resembles the attacker pattern). We will catalog every benign failure mode in a dedicated appendix; we are also happy to provide a scrollable exhibit of these traces.

**Q3. "Position against system-level defenses [...] how ODILE compares to defenses like CaMeL [...]."**

CaMeL (Debenedetti et al. 2025) is a design-level defense (a privileged planner plus a capability-checked restricted-Python interpreter), a different layer of the stack than ODILE's representation-level intervention, so the two are complementary. We could not get it to run on open weights for now: on Qwen-2.5-14B the privileged LLM cannot drive the interpreter (benign utility floors at 0%, undefended base 46%, so its 0% ASR is vacuous), and on Llama-3.3-70B only two of four AgentDojo suites complete because the interpreter cannot handle the format-string output the model emits. We therefore ran it on a frontier model, Gemini-2.5-flash-lite, where all four suites complete at ~0% ASR but benign utility is only 18–32% per suite (banking 28–31%, slack 17–29%, workspace 19–22%, travel 0%), well below the benign utility ODILE preserves. For camera-ready we will run CaMeL across all backbones and benchmarks on Gemini and other frontier models.

*(End of comment 2 of 3. Comment 3 begins below.)*

**Q4. "Compare with simpler sanitization defenses [...] explain when ODILE is preferable to these simpler methods."**

We re-implemented the Bhagwatkar et al. (2026) firewall faithfully at Llama-3.1-8B: a second LLM rewrites every tool output to strip injection-looking content before the agent sees it. It is a legitimately useful defense, just not invariant. Statically it matches ODILE on ASR (2.1%), and under adaptive attack it does reduce base cracks (WASP 12.5% to 5.0%; AgentDojo TAP/PAIR 75% to 31%). But it is bypassed by paraphrases that look benign to the rewriter while still triggering the agent (four recurring classes: authority-pretextual, multi-step decomposition, benign-disguise narrative, and URL/code-embedded), and on some records it actively *introduces* cracks that the base never had: on the deploy-token, SSH-key, and personal-access-token records, base is 0/16 while the sanitizer is cracked 3/12, 1/12, and weakly, because those requests look like routine developer workflow and the rewriter has no signal to tell "the user wants a deploy token" from "the injection wants one". ODILE holds 0 across all of these: 0/64 on AgentDojo TAP/PAIR and 0/5805 distinct adaptive paraphrases on WASP (Fisher exact vs base p=7e-24; vs the sanitizer p=5e-10), with no bypass class observed.

The reason is mechanistic. The sanitizer operates on surface text, so an attacker has direct adversarial leverage: any paraphrase that fools the (smaller) rewriter LLM gets through, and the rewriter leaks which paraphrases it strips versus forwards, a gradient the attacker climbs. ODILE operates on hidden states: two paraphrases with different surface text but the same attacker intent converge to similar internal representations at the targeted layers, so surface variation gives the attacker almost no signal, which is why even TAP's iteratively-refined attempts with rejection feedback produced 0 cracks. ODILE is preferable when adaptive robustness, zero extra inference cost, and benign-utility preservation matter together, which is the realistic agent-deployment setting; the sanitizer's advantage is being backbone-agnostic (no per-model training). [firewall table]({ANON}/T_firewall.pdf), [adaptive table]({ANON}/tab_adaptive_tap_pair.pdf), [GCG loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf).

In addition, we invite the reviewer to consult the rest of our supplementary rebuttal experiments:
- **R1 (Generalization):** InjecAgent and TensorTrust complete the four-benchmark out-of-distribution set ([InjecAgent]({ANON}/T3b_injecagent_crossmodel.pdf), [TensorTrust]({ANON}/T4_TT_headline.pdf)).
- **R3 (Capability):** BFCL and τ-bench confirm general tool-use is retained ([table]({ANON}/tab_capability.pdf)).

We hope the added benchmarks, the adaptive-attack coverage, and the deployment-scope framing address the reviewer's concerns. We would be grateful for a reconsideration of the score.

---

## References

*(Shared bibliography for every work cited across the four responses. Post it as a short final comment in each reviewer's thread so the citations are clickable.)*

- Bhagwatkar et al. (2026). Indirect Prompt Injections: Are Firewalls All You Need, or Stronger Benchmarks? [arXiv:2510.05244](https://arxiv.org/abs/2510.05244).
- Chao et al. (2024). Jailbreaking Black Box Large Language Models in Twenty Queries (PAIR). [arXiv:2310.08419](https://arxiv.org/abs/2310.08419).
- Chen et al. (2026). Meta SecAlign: A Secure Foundation LLM Against Prompt Injection Attacks. [arXiv:2507.02735](https://arxiv.org/abs/2507.02735).
- Debenedetti et al. (2024). AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents. NeurIPS 2024. [arXiv:2406.13352](https://arxiv.org/abs/2406.13352).
- Debenedetti et al. (2025). Defeating Prompt Injections by Design (CaMeL). [arXiv:2503.18813](https://arxiv.org/abs/2503.18813).
- Evtimov et al. (2025). WASP: Benchmarking Web Agent Security Against Prompt Injection Attacks. [arXiv:2504.18575](https://arxiv.org/abs/2504.18575).
- Hines et al. (2024). Defending Against Indirect Prompt Injection Attacks With Spotlighting. [arXiv:2403.14720](https://arxiv.org/abs/2403.14720).
- Li et al. (2026). AgentDyn: A Dynamic Open-Ended Benchmark for Evaluating Prompt Injection Attacks of Real-World Agent Security Systems. [arXiv:2602.03117](https://arxiv.org/abs/2602.03117).
- Li et al. (2026). ReasAlign: Reasoning Enhanced Safety Alignment against Prompt Injection Attack. [arXiv:2601.10173](https://arxiv.org/abs/2601.10173).
- Mazeika et al. (2024). HarmBench: A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal. [arXiv:2402.04249](https://arxiv.org/abs/2402.04249).
- Mehrotra et al. (2024). Tree of Attacks: Jailbreaking Black-Box LLMs Automatically (TAP). [arXiv:2312.02119](https://arxiv.org/abs/2312.02119).
- Nasr et al. (2025). The Attacker Moves Second: Stronger Adaptive Attacks Bypass Defenses Against LLM Jailbreaks and Prompt Injections. [arXiv:2510.09023](https://arxiv.org/abs/2510.09023).
- Patil et al. (2024). Berkeley Function Calling Leaderboard (BFCL). [gorilla.cs.berkeley.edu](https://gorilla.cs.berkeley.edu/leaderboard.html).
- Sun et al. (2024). Massive Activations in Large Language Models. [arXiv:2402.17762](https://arxiv.org/abs/2402.17762).
- Toyer et al. (2023). Tensor Trust: Interpretable Prompt Injection Attacks from an Online Game. [arXiv:2311.01011](https://arxiv.org/abs/2311.01011).
- Wallace et al. (2024). The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions. [arXiv:2404.13208](https://arxiv.org/abs/2404.13208).
- Yao et al. (2024). τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains. [arXiv:2406.12045](https://arxiv.org/abs/2406.12045).
- Zhan et al. (2024). InjecAgent: Benchmarking Indirect Prompt Injections in Tool-Integrated Large Language Model Agents. Findings of ACL 2024. [aclanthology.org/2024.findings-acl.624](https://aclanthology.org/2024.findings-acl.624/).
- Zhu et al. (2025). MELON: Provable Indirect Prompt Injection Defense via Masked Re-Execution and Tool Comparison. [arXiv:2502.05174](https://arxiv.org/abs/2502.05174).
- Zou et al. (2023). Universal and Transferable Adversarial Attacks on Aligned Language Models (GCG). [arXiv:2307.15043](https://arxiv.org/abs/2307.15043).
- Zou et al. (2024). Improving Alignment and Robustness with Circuit Breakers. [arXiv:2406.04313](https://arxiv.org/abs/2406.04313).
