# Injection examples — real, verbatim from the benchmarks

Direct attack strings pulled straight from each benchmark (not curated/paraphrased), so a reviewer can scroll and see the diversity first-hand — and that the held-out benchmarks share no surface form, domain, or tooling with ODILE's AgentDojo training injections.

- **[InjecAgent attacks](examples/injecagent_attacks.md)** — 124 unique attacker instructions across 2,108 cases (direct-harm + data-stealing; e-commerce, IoT, email, finance-app, smart-home domains).
- **[TensorTrust attacks](examples/tensortrust_attacks.md)** — 91 extraction + 39 hijack prompts (the guardian-bot password game — a different mechanic entirely).
- **[AgentDojo training injections](examples/agentdojo_train_injections.md)** — the 26 exact injection strings ODILE actually trained on (banking / slack / travel / workspace).

Scroll the first two against the third: different domains, different tools, different phrasings. ODILE reaches **≤0.15% InjecAgent ASR** and reduces **TensorTrust extract-leak to 1–44%** with none of these strings in its training corpus.
