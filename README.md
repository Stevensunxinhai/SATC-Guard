# SATC-Guard

**SATC-Guard** is a context-aware, two-pass ASR harness for air-traffic control communications. It runs a direct speech-to-text pass, then applies constrained entity decoding, consistency checks, and optional audio-grounded repair before exposing transcripts for downstream use.

This folder is a self-contained release bundle for reproducing the reported experiments on the ATCO2 1-hour gold evaluation subset. It includes the harness source code, frozen prompt bundles, experiment configs, segment manifest metadata, paper scoring scripts, and the **12 final run directories** behind the camera-ready tables. Raw ATCO2 audio is **not** redistributed — you must obtain the dataset separately under its license.

## Paper-facing conditions

| Condition | Internal harness | Prompt version | Config |
|---|---|---|---|
| **Direct** | `direct` | v1.0.0 (`baseline_a.txt`) | `configs/experiment.yaml` |
| **Transcript Repair** | `satc_guard` (v1) | v1.0.0 (`pass2.txt`) | `configs/experiment.yaml` |
| **Conservative Merge** | `satc_guard_v2` | v2.0.0 | `configs/experiment_v2.yaml` |
| **SATC-Guard** (main) | `satc_guard_v2_1` | v2.1.0 | `configs/experiment_v2_1.yaml` |

Each backbone (GPT-4o-transcribe, Qwen3.5-Omni-Plus, MiMo-V2.5-ASR) has one run per condition. See [RUNS.md](RUNS.md) for the exact run directories and internal run IDs.

## Shared evaluation subset

All paper tables use a **global paired subset** of **867 utterances** (`subset: global_paired` in `paper/common_subset_metrics.json`):

- Manifest size: **869** segments (ATCO2 1h gold eval subset)
- Excluded utterance: `atco2_test-set-v3_en_LZIB_STEFANIK_Tower_118_3MHz_20210425_120446-B!000000-000459` (Qwen/DashScope provider-side content inspection rejection)
- Protocol: **`strict_online`** (context from `.info` + rolling memory only)

Bundled run artifacts already include full-run hypotheses; the scoring script re-evaluates the shared 867 subset for table generation.

## Quick start

```bash
# From this directory (opensource/)
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux/macOS
pip install -e .

cp .env.example .env            # fill in API keys and local ATCO2 paths
```

Step-by-step reproduction: [REPRODUCE.md](REPRODUCE.md).

## Regenerate paper metrics and tables

```bash
python paper/compute_common_subset_results.py
```

Expected top-level fields in `paper/common_subset_metrics.json`:

- `"subset": "global_paired"`
- `"common_subset_size": 867`

LaTeX tables are written to `paper/tables/`. The camera-ready manuscript source is `paper/main.tex`.

## API credentials and data licensing

### Required environment variables

Copy `.env.example` to `.env` and set:

| Variable | Purpose |
|---|---|
| `OPENAI_API_KEY`, `OPENAI_BASE_URL` | GPT-4o direct + SATC-Guard passes |
| `DASHSCOPE_API_KEY`, `DASHSCOPE_BASE_URL` | Qwen3.5-Omni-Plus |
| `MIMO_API_KEY`, `MIMO_BASE_URL` | MiMo-V2.5-ASR |
| `ATCO2_DATA_ROOT`, `ATCO2_WAV_DIR` | Local path to licensed ATCO2 corpus |

Model IDs and decoding defaults are in `configs/models.yaml`.

### Dataset

The **ATCO2 ASR dataset** must be obtained separately under its license. This release includes:

- `data/manifests/segments_v1.jsonl` — segment metadata and gold references
- `data/manifests/splits/recording_split_v1.json` — recording-level dev/eval split (seed 42)

It does **not** include WAV files, `.xml`/`.info` source trees, or sliced audio cache. After placing ATCO2 locally, run `scripts/00_prepare_manifest.py` (optional refresh) and `scripts/01_slice_audio.py` before inference.

## Bundled run artifacts

Each selected run under `results/runs/` includes:

- `run_manifest.json` — run ID, prompt/harness version, protocol, seed
- `config_snapshot.yaml` — frozen config (when captured)
- `hypotheses.jsonl` — model outputs with per-segment audit trails
- `metrics.json` — full-run evaluation metrics
- `errors.jsonl` — provider/API failures (when present)

Intermediate checkpoints (`metrics_partial.json`) and non-final/smoke runs are excluded.

## Repository layout

```
opensource/
├── README.md              ← this file
├── RUNS.md                ← run ID → paper condition mapping
├── REPRODUCE.md           ← full pipeline steps
├── pyproject.toml
├── .env.example
├── src/atc_agent/         ← harness, models, eval
├── scripts/               ← 00–03 reproducibility scripts
├── configs/               ← experiment YAML + frozen prompts
├── data/manifests/        ← segment manifest + split (no audio)
├── paper/                 ← scoring script, metrics JSON, main.tex, tables
└── results/runs/          ← 12 final paper runs
```

## Limitations

- Re-running inference requires live API access and incurs provider cost; bundled hypotheses allow offline metric recomputation.
- Exact numeric reproduction may vary slightly if providers change model behavior; configs, prompts, and decoding settings are fixed in this bundle.
- One manifest utterance is permanently excluded from the shared 867 subset due to a Qwen provider rejection (documented in metrics JSON).
