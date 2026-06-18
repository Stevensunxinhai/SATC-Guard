# Selected Paper Runs

These **12 directories** under `results/runs/` are the final runs referenced by `paper/common_subset_metrics.json` and the camera-ready paper. All use `protocol = strict_online` and are scored on the shared **867-utterance global paired subset**.

## Direct baselines

| Paper condition | Run ID | Directory | Prompt | Hypotheses |
|---|---|---|---|---|
| GPT-4o Direct | `gpt4o_direct` | `gpt4o_direct_20260616T101550Z` | v1.0.0 | 869 |
| Qwen Direct | `qwen_direct` | `qwen_direct_20260616T130251Z` | v1.0.0 | 868 |
| MiMo Direct | `mimo_direct` | `mimo_direct_20260616T173746Z` | v1.0.0 | 869 |

## Transcript Repair (v1 ablation)

Harness mode: `satc_guard` · Config: `configs/experiment.yaml` · Prompt: v1.0.0

| Backbone | Run ID | Directory | Hypotheses |
|---|---|---|---|
| GPT-4o | `gpt4o_guard` | `gpt4o_guard_20260616T180118Z` | 869 |
| Qwen | `qwen_guard` | `qwen_guard_20260616T132905Z` | 868 |
| MiMo | `mimo_guard` | `mimo_guard_20260616T192252Z` | 869 |

## Conservative Merge (v2 ablation)

Harness mode: `satc_guard_v2` · Config: `configs/experiment_v2.yaml` · Prompt: v2.0.0

| Backbone | Run ID | Directory | Hypotheses |
|---|---|---|---|
| GPT-4o | `gpt4o_guard_v2` | `gpt4o_guard_v2_20260617T113307Z` | 869 |
| Qwen | `qwen_guard_v2` | `qwen_guard_v2_20260617T124222Z` | 868 |
| MiMo | `mimo_guard_v2` | `mimo_guard_v2_20260617T160958Z` | 869 |

## SATC-Guard (main method, v2.1)

Harness mode: `satc_guard_v2_1` · Config: `configs/experiment_v2_1.yaml` · Prompt: v2.1.0 · Harness: v2.1.0

| Backbone | Run ID | Directory | Hypotheses |
|---|---|---|---|
| GPT-4o | `gpt4o_guard_v2_1` | `gpt4o_guard_v2_1_20260617T161816Z` | 869 |
| Qwen | `qwen_guard_v2_1` | `qwen_guard_v2_1_20260617T184340Z` | 868 |
| MiMo | `mimo_guard_v2_1` | `mimo_guard_v2_1_20260617T221816Z` | 869 |

## Shared subset metrics (867 utterances)

Rounded headline numbers from `paper/common_subset_metrics.json`:

### Direct

| Backbone | WER | CER | Ent-F1 |
|---|---|---|---|
| GPT-4o-transcribe | 0.465 | 0.357 | 0.426 |
| Qwen3.5-Omni-Plus | 0.361 | 0.257 | 0.421 |
| MiMo-V2.5-ASR | 0.572 | 0.413 | 0.380 |

### SATC-Guard (main)

| Backbone | WER | CER | Ent-F1 |
|---|---|---|---|
| GPT-4o + SATC-Guard | 0.465 | 0.357 | 0.418 |
| Qwen3.5-Omni + SATC-Guard | 0.357 | 0.253 | 0.409 |
| MiMo-V2.5-ASR + SATC-Guard | 0.568 | 0.409 | 0.377 |

## Verification checklist

For each run directory, confirm:

1. `run_manifest.json` exists with expected `run_id`, `prompt_version`, and `protocol: strict_online`.
2. Guard runs (v2 / v2.1) include `config_snapshot.yaml` and `harness_version` in the manifest.
3. `hypotheses.jsonl` is present (868–869 lines depending on backbone; Qwen runs omit the provider-rejected utterance).
4. `metrics.json` matches the bundled snapshot (regenerate with `scripts/03_run_eval.py` if needed).

To recompute all paired-subset metrics:

```bash
python paper/compute_common_subset_results.py
```

Expected: `"common_subset_size": 867`, `"subset": "global_paired"`.
