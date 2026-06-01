# ODILE, OpenReview-ready rebuttal (full version)

**How to use.** Each reviewer's response is posted to OpenReview as one or more comments (5000-char limit per comment): **FoEk = 3 comments, DefG = 2, GHCC = 2, FSAe = 1**. The italic `*(... comment X of N ...)*` notes mark the split points — do NOT paste those notes; just start a new comment there. Replace `{ANON}` with your anonymous.4open.science repo URL once you have the ID (no personal-repo links, no name/school anywhere). Greetings/closings are personalized per reviewer; each reviewer keeps its per-concern answers plus a "see also" R-list. Code tokens are backticked so underscores render; no text-mode LaTeX (OpenReview MathJax is math-mode only).

---

## FoEk (Rating 3)

We thank the reviewer: generalization, attack strength, capability breadth, and model recency are indeed the axes we want to showcase performance in. Below, we address the questions they raised.

**1. "There is clear data leakage [...]. I cannot tell whether ODILE learns a real prompt-injection defense or fitting on test set."**
ODILE is trained on AgentDojo (Debenedetti et al. 2024) traces; a memorized adapter would fail on any distribution it did not train on, and we show it does not, on four further benchmarks and on held-out attack-augmentations, each with sample size, significance, a comparator, and a concrete demonstration that it is genuinely different.
- **InjecAgent** (Zhan et al. 2024; different benchmark, ReAct format, attack domains absent from training): ODILE drives ASR-all to ≤0.15% on every backbone (n=1054/family, Wilson upper bound 0.35% at Qwen-2.5-14B). The attacks share no surface form with training: compare the [30 direct-harm]({ANON}/examples_pdf/injecagent_attacks.pdf) and [32 data-stealing]({ANON}/examples_pdf/injecagent_data_exfil.pdf) attacks against [the AgentDojo injections ODILE trained on]({ANON}/examples_pdf/agentdojo_train_injections.pdf); a [harness walk-through]({ANON}/examples_pdf/injecagent_harness.pdf) shows where the injection hides and the base-vs-ODILE branches. Full numbers: [InjecAgent table]({ANON}/T3b_injecagent_crossmodel.pdf).
- **TensorTrust** (Toyer et al. 2023; a different mechanic, guardian-bot password extraction): extract-leak falls from 78–92% to 1–44% across backbones (Qwen-2.5-14B from 82 to 1), n=300/cell; benign coherence is preserved at ~100% on neutral pre-prompts. [91 real attacks]({ANON}/examples_pdf/tensortrust_attacks.pdf); full numbers in the [TensorTrust table]({ANON}/T4_TT_headline.pdf).
- **WASP** (Evtimov et al. 2025; web-agent injection): the attacker-action rate falls from 12.5% to 0/239 verified-tight (0/5805 per individual paraphrase attempt, Fisher p≈1e-9), against base 25/200 and a sanitizer at 8/160. [21 real attacker goals]({ANON}/examples_pdf/wasp_attacker_goals.pdf); full numbers in the [WASP table]({ANON}/tab_T_wasp.pdf). (Our setup omits the live browser loop; the full containerized version is a camera-ready commitment.)
- **AgentDyn** (Li et al. 2026; a dynamic benchmark on three suites absent from AgentDojo, namely GitHub, Daily-Life, and Shopping): on Qwen3-8B the attack genuinely lands on the base agent (22.1%, n=560) and ODILE drives it to 0.0% (Fisher p < 1e-12). [How it differs, with real tasks and goals]({ANON}/examples_pdf/agentdyn_difference.pdf); full numbers in the [AgentDyn table]({ANON}/tab_agentdyn.pdf).
- **AgentDojo attack-augmentations absent from training** (the same goal rendered with the injection markers removed, or base64-encoded): ASR remains ≈0 on surface forms the adapter never saw. [The augmented forms]({ANON}/examples_pdf/agentdojo_heldout_augmentations.pdf).

Test-set memorization cannot explain these results: the adapter never saw InjecAgent's tools, TensorTrust's extraction game, WASP's web environment, or AgentDyn's three suites at training time, and on AgentDyn the base model it is built on is hijacked 22% of the time, so the drop to ≈0 reflects a learned response to harmful intent, not recall. The strongest single piece of evidence is the held-out augmentation: we take the *exact* AgentDojo injections the adapter trained on and merely re-render their surface (markers removed, or base64-encoded), and ASR stays ≈0 even though the underlying goal is unchanged, which is the opposite of what a defense fit to the training strings would do. ODILE is also format-driven, not format-keyed: it jams identically across ReAct and a delimiter-wrapped variant of the same task ([figure]({ANON}/fig_react_jam_odile.pdf)).

*(FoEk is long; post it as 3 comments. End of comment 1 of 3. Comment 2 begins below.)*

**2. "The attack setting is too weak [...]. does not test adversarial attackers aware of the adapter."**
We add stronger, defense-aware dynamic attacks that target the adapter's internals. These are not the strongest conceivable adaptive attacker, but they directly probe the four axes one would expect a representation defense to be tested on:
- **Discrete GCG** (Zou et al. 2023; white-box, full weight, adapter, and gradient access): a 20-token adversarial suffix optimized for 75 steps at search-width 128. 0/4 crackable pairs. The attacker-target cross-entropy plateaus near 10.7–11.0 and never descends into the compliance regime, whereas on the same pairs the undefended base reaches 0.57 and complies, so the attack mechanism works and ODILE is what stops it. The plateau does not move with budget: on a HarmBench (Mazeika et al. 2024) text-generation GCG sweep, ODILE held 0% attack success even when the search was extended to 500 steps and to adaptive hidden-state targeting (base Llama-3.3-70B is broken on 68% of behaviors there), and the [loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf) shows the base descending to compliance while ODILE stays flat. [GCG table]({ANON}/T_GCG.pdf).
- **LoRA-aware GCG** (the loss adds a bypass term penalizing the adapter's hidden-state deviation from base at the exact LoRA layers): λ_bypass ∈ {1, 10, 100}, 50 steps. 0/3; the bypass term shrinks the adapter-vs-base gap by ~25% but cross-entropy never approaches compliance (from ~12.0 to ~11.9). This covers the adapter-aware and layer-range-aware axes.
- **TAP (Mehrotra et al. 2024) and PAIR (Chao et al. 2024)** (semantic attackers given ODILE's objective): 0/64 across two backbones, comprising 0/48 at the standard tree-search budget plus 0/16 when every surviving cell is re-attacked with a deeper tree (depth 5, branching 3, width 6, 2 parallel streams, against the standard depth 3, branching 2, width 4, 1 stream), against base 36/48 and a sanitizer firewall 15/48 on the identical cells. Base and sanitizer cracking those exact cells is the positive control. [TAP/PAIR table]({ANON}/tab_adaptive_tap_pair.pdf). This covers the objective-aware axis.

We do not claim these are the strongest possible adaptive attacker; in particular, we plan to evaluate reinforcement-learning-based attacks (Nasr et al. 2025, "The Attacker Moves Second", arXiv:2510.09023) after the rebuttal, alongside multi-suffix and transfer-ensemble GCG and a broader sweep across all backbones.

**3. "Benign capability evaluation is narrow [...] such as BFCL and Tau-Bench."**
We add exactly the two benchmarks named, reporting base and ODILE with the change in parentheses:
- **BFCL** (Patil et al. 2024; non-live AST, n=1240/cell): Llama-3.1-8B from 73.98 to 73.35 (−0.63); Qwen-2.5-7B from 74.98 to 75.12 (+0.14, a gain). General tool-call capability is retained.
- **τ-bench airline** (Yao et al. 2024; n=50 tasks, single trial, so directional): Llama-3.1-8B from 42.00 to 38.78 (−3.22).

Full per-backbone capability (with AlpacaEval, GSM8K, IFEval) is in the [capability table]({ANON}/tab_capability.pdf). Camera-ready: BFCL and τ-bench across all remaining backbones.

**4. "Model selection is outdated [...] do not test newer reasoning models."**
We add Qwen3-Next-80B-A3B-Thinking (Qwen Team 2025; September 2025), a frontier-agentic open-source model with nonstandard (Gated-DeltaNet) attention; ODILE transfers with the attention-path adjustments ([Qwen3-Next table]({ANON}/tab_pareto_qnext.pdf), [cross-model]({ANON}/T2_crossmodel_AD.pdf)). The Qwen3-Next results shown are on the `important_instructions` family; we commit to reporting the full 52-cell AgentDojo grid on Qwen3-Next across all four suites, together with its TensorTrust and AgentDyn cells, in the camera-ready.

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
(a) These three banking pairs are structurally ambiguous: the attacker IBAN equals the legitimate recipient, so the harmful and benign twins are byte-identical and give the contrastive objective no signal. We handle them by a hard-coded structural exclusion (target-collision in the task definition), not a per-sweep statistic. (b) No representation-level method can resolve a case where the harmful and benign actions are literally identical, because there is no internal signal to separate; the right place to catch it is outside the representation. Two complementary options apply: a deterministic post-execution validator (an IBAN allowlist that simply checks the recipient against the user's known accounts), or a runtime divergence detector such as MELON (Zhu et al. 2025) that re-runs the step without the suspected injection and flags a behavior change, stacked on top of ODILE. We measured this stacking: composing an instruction-hierarchy / sandwich overlay on ODILE closes the worst residual InjecAgent cell (`dh_base`) from 16.0% to 2.0%, and the failure geographies of the two defenses are largely disjoint (they do not fail on the same cells), which is why they compose. (c) A controlled include-versus-exclude ablation (identical recipe, the three pairs the only difference) shows inclusion is pure downside: AD-standard ASR rises by 0.18pp (0.12% to 0.29%) and benign utility falls by 0.37pp (negligible), and every worsened sample traces to that one task ([ablation table]({ANON}/tab_worsened_pairs.pdf)). We commit to a full stacking-as-defense ablation for this genuinely-ambiguous class in the camera-ready.

**Q. "The authors mention why CRL is better than MSE for Llama and have a justification for it. However, Qwen MSE was better than Qwen Triplet, why?"**
Both objectives drive ASR to ≈0, so the security result is not loss-specific; the difference is in consistency. The MSE objective behaves consistently across backbones, whereas the triplet objective is inconsistent, strong on some backbones and weaker on others, which is why we report MSE as the headline and triplet as an ablation. Our working hypothesis is that triplet relies on absolute distance margins, and the hidden-state magnitudes differ substantially across model families (Qwen-2.5/Qwen3 against Llama; cf. massive activations, Sun et al. 2024), so a margin calibrated on one family is mis-scaled on another, while MSE does not depend on those magnitudes in the same way. We agree this deserves a proper treatment and will add a dedicated discussion of the loss-objective comparison, with per-backbone margin-sensitivity ablations to test this hypothesis directly, to the paper.

**Q. "[...] the change in layer range showed significant effect [...]. Was it pure trial and error?"**
No: the mid-late band follows the circuit-breaker rationale (Zou et al.), depth-scaled and transfer-validated across seven backbones. We did run an adjacent-band check on Llama-3.1-8B (L10–20, L12–22, and L20–30, 52 cells each): all three suppress ASR to ≈0, but the later band halves under-attack utility (≈10% vs ≈21%), so the mid band is selected for utility preservation. A full native-objective sweep across all backbones is a camera-ready commitment.

**Q (whitebox/blackbox "different landscape").**
We agree and will adopt this framing explicitly. ODILE is an inner defense for the open-weight / self-hosted regime; for API-only deployments, pipeline and blackbox defenses are the appropriate layer. We will state in §1 and §10 that it occupies a different deployment layer, complementary rather than competing. We thank the reviewer for the framing.

In addition, we invite the reviewer to consult the rest of our supplementary rebuttal experiments:
- **R1 (Generalization):** four out-of-distribution benchmarks (InjecAgent, TensorTrust, WASP, AgentDyn) where ODILE drives ASR to ≈0.
- **R2 (Adaptive robustness):** discrete and LoRA-aware GCG and TAP/PAIR semantic attacks, all 0 cracks.
- **R4 (Model coverage):** seven backbones including the September-2025 frontier-agentic open-source model Qwen3-Next-80B.

We are grateful for the positive assessment and hope these clarifications on curation, loss, and layer selection further strengthen the paper. We would be glad to add any detail the reviewer finds useful, and would be grateful for maintaining or raising the score.

---

## DefG (Rating 4)

We thank the reviewer: defense comparison, a cleaner tradeoff table, cross-framework generalization, and real-loop behavior are the axes we strengthen below.

**1. "The paper should compare more directly against recent model-level defenses such as Meta SecAlign and against strong firewall/sanitizer-style defenses. Could the authors clarify the novelty of ODILE [...]?"**
**Novelty:** ODILE is a representation-level LoRA at 1x inference cost (no second model pass, no input rewriting), trained by a paired-trace contrast on the model's own harmful direction.
**vs Meta-SecAlign** (Chen et al. 2026), head-to-head at Llama-3.3-70B. On **AgentDojo**, ODILE holds 0.01% ASR vs Meta-SecAlign's 2.11% (≈200x lower) at 1x cost, while Meta-SecAlign retrains the full model and carries higher benign utility (62.6% vs 51.5%); the two sit at different points on the security-utility frontier ([70B table]({ANON}/tab_pareto_70b.pdf)). On **InjecAgent**, ODILE holds ASR-all ≤ 0.20% across all four families vs Meta-SecAlign's 0.2–2.6% ([InjecAgent table]({ANON}/t_injecagent_full.pdf)).
**Separately, format robustness:** ODILE is also robust to format shift where Meta-SecAlign is not. On InjecAgent's ReAct format Meta-SecAlign cannot emit valid tool calls (88–95% of its outputs are unparseable, so its low ASR is broken output, not defense), whereas ODILE stays in valid ReAct format and jams identically across ReAct and delimiter-wrapped formats ([figure]({ANON}/fig_react_jam_odile.pdf)).
**vs firewall/sanitizer** (Bhagwatkar et al. 2026): a second LLM rewrites each tool output. Static parity (2.1% ASR), but it costs −6.2pp benign utility plus an extra LLM call per output, and is cracked 15/48 under adaptive attack where ODILE holds 0/64 ([firewall table]({ANON}/T_firewall.pdf)).
**Broader field:** the [cross-benchmark comparator]({ANON}/tab_comparator_8b.pdf) places ODILE head-to-head with ReasAlign (Li et al. 2026) on all five benchmarks at once; ReasAlign matches ODILE on ASR but at ≈13pp AgentDojo benign cost (and on InjecAgent its near-zero ASR is the same format-rejection artifact, valid-rate 0.3% vs ODILE's 43.7%). We also benchmark adjacent stack layers so the comparison is not single-family: MELON (Zhu et al. 2025), a runtime detector, and CaMeL (Debenedetti et al. 2025), a design-level planner/interpreter (per-backbone Pareto tables in answer 2).
ODILE is preferable when adaptive robustness, 1x cost, and benign preservation matter together.

**2. "Table 2 is a bit hard to interpret. It mixes different models, utility definitions, and defense assumptions [...]." / "[...] add a cleaner comparison table where everything is on the same model and same utility metric."**
We replace Table 2 with one table per backbone, each on a single model with a single metric pair (ASR against benign utility): [70B]({ANON}/tab_pareto_70b.pdf), [8B]({ANON}/tab_pareto_8b.pdf), [Qwen-2.5-7B]({ANON}/tab_pareto_q25_7b.pdf), [Qwen-2.5-14B]({ANON}/tab_pareto_q25_14b.pdf), [Qwen3-32B]({ANON}/tab_pareto_q3_32b.pdf). On under-attack utility: we report benign (no-attack) utility as the capability axis. Completing the user task while an injection is present is deliberately not ODILE's objective: ODILE jams under attack by design (it removes the attacker tool call), so optimizing for task-completion-under-injection would undercut the defense. We therefore report under-attack utility as a secondary diagnostic, and will state this in the body.

**3. "Single-benchmark focus. The evaluation is centered on AgentDojo [...]."**
We add four out-of-distribution benchmarks across different agent frameworks, all reaching ASR ≈0 with the same recipe: InjecAgent (ReAct), TensorTrust (guardian-bot extraction), WASP (web agent), and AgentDyn (a dynamic benchmark on three non-AgentDojo suites). On AgentDyn the attack genuinely lands on the base agent (Qwen3-8B, 22.1% to 0.0% ASR, n=560) and benign utility holds (26.7% to 20.0%); WASP benign is largely preserved (e.g. Llama-3.3-70B 100/93, Qwen-2.5-7B 100/95). ([InjecAgent]({ANON}/T3b_injecagent_crossmodel.pdf), [TensorTrust]({ANON}/T4_TT_headline.pdf), [WASP]({ANON}/tab_T_wasp.pdf), [AgentDyn]({ANON}/tab_agentdyn.pdf)).

*(DefG exceeds 5000 characters; post it as 2 comments. End of comment 1 of 2. Comment 2 begins below.)*

**4. "The defense behavior also feels brittle [...]. it is unclear what happens in a real agent loop with retries, validators, or tool-call repair logic."**
The jam is the design intent, not a side effect. We have prepared a matched trace for reference: on the same task where the base model calls `update_password`, ODILE reads the file and emits token-soup with no parseable tool call ([trace]({ANON}/traces_pdf/trace_base_executes_vs_odile_jams.pdf)). In a real loop, the retry-on-error handler fires and the model is re-prompted with the injection rather than re-supplied (the strict-deny endpoint), and deterministic validators or tool-call repair compose on top, since they act on the emitted call rather than the representation. A repair loop that iteratively reformulates the injection is itself an adaptive attacker, which our adaptive sweep already tests (TAP/PAIR 0/64); the strongest such test, RL-based attacks (Nasr et al. 2025), is a camera-ready commitment, and we expect ODILE to hold. Critically, the jam never produces the attacker action: 0/5805 distinct WASP paraphrases (trace-validated), and any residual `send_money` has a degenerate recipient that never resolves to the attacker IBAN ([trace]({ANON}/traces_pdf/trace_secalign_cracked_vs_odile.pdf)).

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
Agreed; this is a deliberate scope choice for the open-weight / self-hosted setting. Beyond practitioners self-hosting open-weight models, the audience also includes model creators, who can ship the adapter as a default safety layer. For API-only deployments, pipeline defenses (Spotlight, instruction-hierarchy, MELON, CaMeL) are the appropriate layer, which ODILE complements rather than replaces. We will add an explicit deployment-scope paragraph in §10.

**2. "Limited model coverage. The evaluation uses mainly Llama-3.3-70B and Qwen-2.5-7B [...] include a newer Qwen model such as Qwen3 or justify the choice [...]."**
Six backbones plus Qwen3-Next-80B, including Qwen3-8B and Qwen3-32B from the Qwen-3 family the reviewer points to; Qwen3-Next-80B is additionally a frontier-agentic open-source model (nonstandard Gated-DeltaNet attention), and ODILE transfers across all of them ([cross-model]({ANON}/T2_crossmodel_AD.pdf), [Qwen3-Next]({ANON}/tab_pareto_qnext.pdf)).

**3. "Single-benchmark focus [...] additional benchmarks such as AgentDyn and WASP would strengthen the claims."**
We add both, and report benign utility on each as well as ASR. **AgentDyn** is a dynamic benchmark on three suites absent from AgentDojo (GitHub, Daily-Life, Shopping; [how it differs]({ANON}/examples_pdf/agentdyn_difference.pdf)), where the attack genuinely hijacks the base agent (Qwen3-8B, 22.1%, n=560, trace-validated) and ODILE drives it to 0.0% (Fisher p < 1e-12), with benign utility largely preserved (base 26.7% to ODILE 20.0%) ([table]({ANON}/tab_agentdyn.pdf)). **WASP** falls from 12.5% to 0/239 verified-tight (0/5805 per paraphrase attempt), while benign utility is preserved (base/ODILE benign %: Llama-3.3-70B 100/93, Qwen-2.5-7B 100/95, Qwen-2.5-14B 69/69, Llama-3.1-8B 100/83); our setup runs WASP's [21 attacker goals and templates]({ANON}/examples_pdf/wasp_attacker_goals.pdf) without the live browser loop, the full containerized version a camera-ready commitment ([table]({ANON}/tab_T_wasp.pdf)).

**4. "No adaptive attack evaluation [...] adaptive attacks that know the adapter, objective, layer range, or masking strategy."**
We add defense-aware white-box attacks covering all four named axes, 0 cracks throughout:
- **Adapter- and layer-range-aware:** LoRA-aware GCG (Zou et al. 2023) adds a bypass term penalizing the adapter's hidden-state deviation from base at the exact LoRA layers (λ ∈ {1,10,100}); 0/3. The term shrinks the adapter-vs-base gap ~25%, but cross-entropy never approaches compliance.
- **Objective-aware:** TAP (Mehrotra et al. 2024) and PAIR (Chao et al. 2024), given ODILE's objective in their system prompt, 0/64 across two backbones, against base 36/48 and a sanitizer 15/48 on the identical cells (the positive control).
- **Mask-aware:** the completion-window mask is a training-time loss localization, with nothing input-side for the attacker to route around.
- **Discrete GCG** (Zou et al. 2023; 75 steps): 0/4. The attacker-target cross-entropy plateaus at 10.7–11.0 while the base descends to 0.57 and complies, and does not move to 500 steps on a HarmBench (Mazeika et al. 2024) sweep (base broken on 68% of behaviors).

We do not claim these are the strongest possible attacker; RL-based attacks (Nasr et al. 2025) are planned. Full results: [GCG]({ANON}/T_GCG.pdf), [TAP/PAIR]({ANON}/tab_adaptive_tap_pair.pdf), [loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf).

**Q1. "What exactly does '88% benign capability retention' mean? [...] most security papers report benign utility, utility under attack, and ASR separately."**
The 88% is a relative capability-retention figure on AgentDojo (ODILE benign utility as a fraction of the undefended base), which we used to keep the comparison consistent across backbones. We agree absolute numbers are clearer: we now report, per backbone, absolute ASR, benign utility, and under-attack utility separately ([tables]({ANON}/tab_pareto_70b.pdf)). UAU is a secondary diagnostic since (as above) ODILE jams under attack by design.

*(GHCC exceeds 5000 characters; post it as 2 comments. End of comment 1 of 2. Comment 2 begins below.)*

**Q2. "Explain benign utility loss. Where does the 12% drop come from: malformed tool calls, incomplete chains, wrong answers, or unnecessary refusals?"**
To clarify, this 12% is the AgentDojo benign-utility drop (relative ~12%, roughly 6pp absolute, concentrated in banking and slack). It is a mix of two modes: (i) format-envelope shifts, where the correct tool call is emitted in a non-canonical wrapper that the strict scorer counts as a miss; and (ii) a few over-cautious jams on harmful-looking-but-benign tasks (e.g. a legitimate large transfer that resembles the attacker pattern). We will catalog every benign failure mode in a dedicated appendix; we are also happy to provide a scrollable exhibit of these traces.

**Q3. "Position against system-level defenses [...] how ODILE compares to defenses like CaMeL [...]."**
CaMeL (Debenedetti et al. 2025) is a design-level defense (a privileged/quarantined planner and a capability-checked interpreter), a different layer of the agent stack than ODILE's representation-level intervention, and the two are complementary. We evaluate CaMeL at three scales: Qwen-2.5-14B (it scale-floors, the privileged LLM cannot drive the interpreter, and its 0% ASR is a vacuous emergent jam); Llama-3.3-70B (it runs on two of four suites, while banking and travel crash at an upstream interpreter bug); and Gemini-2.5-flash-lite (all four suites, ~0% ASR but only 18–32% benign utility). CaMeL reaches ~0% ASR wherever its interpreter runs, but on open weights it is brittle and high-tax; ODILE reaches 0% ASR at 1× cost on every backbone.

**Q4. "Compare with simpler sanitization defenses [...] explain when ODILE is preferable to these simpler methods."**
We re-implemented the Bhagwatkar et al. (2026) firewall faithfully at Llama-3.1-8B: a second LLM rewrites every tool output to strip injection-looking content before the agent sees it. It is a legitimately useful defense, just not invariant. Statically it matches ODILE on ASR (2.1%), and under adaptive attack it does reduce base cracks (WASP 12.5% to 5.0%; AgentDojo TAP/PAIR 75% to 31%). But it is bypassed by paraphrases that look benign to the rewriter while still triggering the agent (four recurring classes: authority-pretextual, multi-step decomposition, benign-disguise narrative, and URL/code-embedded), and on some records it actively *introduces* cracks that the base never had: on the deploy-token, SSH-key, and personal-access-token records, base is 0/16 while the sanitizer is cracked 3/12, 1/12, and weakly, because those requests look like routine developer workflow and the rewriter has no signal to tell "the user wants a deploy token" from "the injection wants one". ODILE holds 0 across all of these: 0/64 on AgentDojo TAP/PAIR and 0/5805 distinct adaptive paraphrases on WASP (Fisher exact vs base p=7e-24; vs the sanitizer p=5e-10), with no bypass class observed.

The reason is mechanistic, and it is the core of our claim. The sanitizer operates on surface text, so an attacker has direct adversarial leverage: any paraphrase that fools the (smaller) rewriter LLM gets through, and the rewriter leaks which paraphrases it strips versus forwards, a gradient the attacker climbs. ODILE operates on hidden states: two paraphrases with different surface text but the same attacker intent converge to similar internal representations at the targeted layers, so surface variation gives the attacker almost no signal, which is why even TAP's iteratively-refined attempts with rejection feedback produced 0 cracks. ODILE is preferable when adaptive robustness, zero extra inference cost, and benign-utility preservation matter together, which is the realistic agent-deployment setting; the sanitizer's advantage is being backbone-agnostic (no per-model training). [firewall table]({ANON}/T_firewall.pdf), [adaptive table]({ANON}/tab_adaptive_tap_pair.pdf), [GCG loss curve]({ANON}/figures/fig_gcg_loss_curve.pdf).

In addition, we invite the reviewer to consult the rest of our supplementary rebuttal experiments:
- **R1 (Generalization):** InjecAgent and TensorTrust complete the four-benchmark out-of-distribution set ([InjecAgent]({ANON}/T3b_injecagent_crossmodel.pdf), [TensorTrust]({ANON}/T4_TT_headline.pdf)).
- **R3 (Capability):** BFCL and τ-bench confirm general tool-use is retained ([table]({ANON}/tab_capability.pdf)).

We hope the added benchmarks, the adaptive-attack coverage, and the deployment-scope framing address the reviewer's concerns. We would be grateful for a reconsideration of the score.
