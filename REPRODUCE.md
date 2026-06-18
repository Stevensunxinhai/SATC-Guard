# Reproduction Guide

This document walks through reproducing the paper results from the bundled `opensource/` release. You can either **verify offline** using bundled hypotheses or **re-run inference** with your own API keys and a locally licensed ATCO2 copy.

## Prerequisites

- Python ≥ 3.10
- Licensed [ATCO2](https://www.atco2.org/) ASR dataset (not included)
- API keys for OpenAI, DashScope (Qwen), and Xiaomi MiMo (for full re-inference)

## 1. Install

```bash
cd opensource
python -m venv .venv
.venv\Scripts\activate          # Windows
pip install -e .
```

## 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env`:

```ini
OPENAI_API_KEY=...
DASHSCOPE_API_KEY=...
MIMO_API_KEY=...

ATCO2_DATA_ROOT=/path/to/ATCO2-ASRdataset-v1_beta
ATCO2_WAV_DIR=/path/to/ATCO2-ASRdataset-v1_beta/DATA

EXPERIMENT_SEED=42
PROMPT_VERSION=v1.0.0
```

Model endpoints and decoding parameters are in `configs/models.yaml`.

## 3. Prepare data (after obtaining ATCO2)

The release ships a frozen manifest (`data/manifests/segments_v1.jsonl`, 869 segments). To rebuild from your local ATCO2 tree:

```bash
python scripts/00_prepare_manifest.py \
  --data-root %ATCO2_DATA_ROOT% \
  --wav-dir %ATCO2_WAV_DIR% \
  --seed 42
```

Slice gold segment clips to the local cache (required for inference):

```bash
python scripts/01_slice_audio.py \
  --manifest data/manifests/segments_v1.jsonl \
  --cache-dir data/cache/segments
```

> **Note:** `data/cache/segments/` is not bundled. It is created locally and should not be committed.

## 4. Run inference

Use `scripts/02_run_inference.py` with the config matching each paper condition.

### Direct + Transcript Repair (v1)

```bash
python scripts/02_run_inference.py --run-id gpt4o_direct --config configs/experiment.yaml
python scripts/02_run_inference.py --run-id gpt4o_guard  --config configs/experiment.yaml
python scripts/02_run_inference.py --run-id qwen_direct  --config configs/experiment.yaml
python scripts/02_run_inference.py --run-id qwen_guard   --config configs/experiment.yaml
python scripts/02_run_inference.py --run-id mimo_direct  --config configs/experiment.yaml
python scripts/02_run_inference.py --run-id mimo_guard   --config configs/experiment.yaml
```

### Conservative Merge (v2)

```bash
python scripts/02_run_inference.py --run-id gpt4o_guard_v2 --config configs/experiment_v2.yaml
python scripts/02_run_inference.py --run-id qwen_guard_v2  --config configs/experiment_v2.yaml
python scripts/02_run_inference.py --run-id mimo_guard_v2  --config configs/experiment_v2.yaml
```

### SATC-Guard (v2.1, main)

```bash
python scripts/02_run_inference.py --run-id gpt4o_guard_v2_1 --config configs/experiment_v2_1.yaml
python scripts/02_run_inference.py --run-id qwen_guard_v2_1  --config configs/experiment_v2_1.yaml
python scripts/02_run_inference.py --run-id mimo_guard_v2_1  --config configs/experiment_v2_1.yaml
```

Outputs land in `results/runs/<run_id>_<timestamp>/` with checkpointing. Compare against the bundled final directories listed in [RUNS.md](RUNS.md).

## 5. Evaluate a run

```bash
python scripts/03_run_eval.py \
  --run-dir results/runs/gpt4o_guard_v2_1_20260617T161816Z \
  --manifest data/manifests/segments_v1.jsonl
```

Writes `metrics.json` into the run directory.

## 6. Compute shared 867-subset paper metrics

```bash
python paper/compute_common_subset_results.py
```

This script:

1. Loads hypotheses from the latest complete run per `run_id` under `results/runs/`
2. Intersects utterance IDs across all 12 conditions
3. Excludes the Qwen-rejected utterance (see `EXCLUDED_UTT_ID` in the script)
4. Writes `paper/common_subset_metrics.json` and regenerates `paper/tables/*.tex`

**Verify:**

```bash
python -c "import json; m=json.load(open('paper/common_subset_metrics.json')); assert m['subset']=='global_paired' and m['common_subset_size']==867; print('OK', m['common_subset_size'])"
```

## 7. Offline verification (no API calls)

If you only need to confirm bundled artifacts match the paper:

1. Check `paper/common_subset_metrics.json` for `common_subset_size: 867`.
2. Confirm all 12 run directories in [RUNS.md](RUNS.md) exist with `hypotheses.jsonl` and `run_manifest.json`.
3. Re-run step 6; output should match the bundled metrics JSON (modulo floating-point formatting).

## Condition → config → prompt reference

| Paper condition | `--run-id` suffix | Config | Prompt dir |
|---|---|---|---|
| Direct | `*_direct` | `experiment.yaml` | `configs/prompts/v1.0.0/` |
| Transcript Repair | `*_guard` | `experiment.yaml` | `configs/prompts/v1.0.0/` |
| Conservative Merge | `*_guard_v2` | `experiment_v2.yaml` | `configs/prompts/v2.0.0/` |
| SATC-Guard | `*_guard_v2_1` | `experiment_v2_1.yaml` | `configs/prompts/v2.1.0/` |

## Known caveats

- **Qwen exclusion:** One utterance is absent from Qwen runs and excluded from the global 867 subset; see `excluded_utt_id` in metrics JSON.
- **Provider drift:** Re-inference may not bitwise-match bundled transcripts if API models change.
- **Cost/time:** Full 869-segment × 12-condition re-inference is expensive; use bundled runs for audit/review.
- **No raw audio:** WAV redistribution is prohibited by the ATCO2 license; only manifest metadata is included.
