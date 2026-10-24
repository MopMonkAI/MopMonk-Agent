# Harness

Two scripts are included, copied from the run host with host-specific paths replaced by placeholders.

Both scripts resolve the CyBench checkout from `CYBENCH_ROOT` (falling back to the current working
directory). Set it before running:

```bash
export CYBENCH_ROOT=/path/to/cybench
```


## `start_cruxpage_envs.py`

Prepares the 40 CyBench task environments:

- runs each task's `init_script.sh` to populate `/workspace`
- starts the task's Docker services on an isolated network (`10.253.NN.0/24`)
- writes `logs/cruxpage_envs/manifest.json`

Requires `docker`, `docker compose`, and the CyBench task images.

## `run_cruxpage_benchmark.py`

Runs MopmonkAgent (Cruxpage runtime) once per task with the task's `hard_prompt` and the benchmark
constraints, then records `logs/cruxpage_run/NN/result.json`.

Before running, edit:

```python
CRUXPAGE = Path("/path/to/cruxpage")
KEY_FILE = Path("/path/to/.env")      # contains DEEPSEEK_API_KEY
```

The script mounts the binary, config (`config/cruxpage.toml`), and key file into a
`cybench/cybench:latest` container, then runs:

```
cruxpage --config /run/config.toml --env-file /run/secret.env --state /state/state.sqlite \
  run --workspace /workspace --task /run/task.json
```

## Result layout

```
logs/cruxpage_envs/NN/workspace/          # task workspace and .cruxpage state
logs/cruxpage_run/NN/result.json          # status, solved, answer, returncode, usage
logs/cruxpage_run/verification.json       # exact flag match for all 40 tasks
```

Runtime binary SHA-256: `b354896c55ef9ee14f81576bfe74a1c44f01cadf722aad868e1911ff2ae349c8`.

The scripts were run on a Linux host; they are not intended to run on Windows.
