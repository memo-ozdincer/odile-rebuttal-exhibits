# Injection examples - real attack data (rendered + raw)

Each attack list is available as a **rendered PDF** (skimmable, highlighted) and the **raw benchmark JSON** (authentic structure). Pulled verbatim from the benchmarks; the held-out benchmarks share no surface form, domain, or tooling with ODILE's AgentDojo training injections.

| List | Rendered PDF | Raw JSON |
|---|---|---|
| InjecAgent attacks (124 unique) | [PDF](examples_pdf/injecagent_attacks.pdf) | [ds_base](examples_json/injecagent_ds_base.json) / [ds_enh](examples_json/injecagent_ds_enhanced.json) / [dh_base](examples_json/injecagent_dh_base.json) / [dh_enh](examples_json/injecagent_dh_enhanced.json) |
| InjecAgent data-exfil (ds) | [PDF](examples_pdf/injecagent_data_exfil.pdf) | (subset of ds JSON above) |
| TensorTrust attacks | [PDF](examples_pdf/tensortrust_attacks.pdf) | - |
| AgentDyn difference + tasks/goals | [PDF](examples_pdf/agentdyn_difference.pdf) | [JSON](examples_json/agentdyn_tasks_and_goals.json) |
| AgentDojo training injections | [PDF](examples_pdf/agentdojo_train_injections.pdf) | - |
| AgentDojo held-out augmentations | [PDF](examples_pdf/agentdojo_heldout_augmentations.pdf) | - |
| WASP attacker goals (21) | [PDF](examples_pdf/wasp_attacker_goals.pdf) | - |

Traces (base/comparator cracks vs ODILE jams, verbatim messages, color-coded) are in [traces_pdf/](traces_pdf/).
