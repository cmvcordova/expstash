# experimentStash

Config-centric experiment reproducibility for Hydra-based tools.

## Entry Points

```bash
# Add a tool (submodule + config copy + meta.yaml registration)
python3 scripts/add_tool <name> <repo_url>

# Run experiments — always invoke tools directly
cd tools/<tool>
uv run python -m <tool>.main <hydra overrides>

# Freeze for reproduction
python3 scripts/snapshot_experiment <tool> <experiment> --tag <tag> --commit
```

## Core Concept

```
snapshot = flattened_config + tool_commit_pin + git_tag
```

Two scripts, two modes:

| Script | What it does |
|--------|-------------|
| `add_tool` | Clone submodule, copy base configs, register in meta.yaml |
| `snapshot_experiment` | Flatten config, pin commit SHA, create git tag |

## Config Composition

Three layers (highest to lowest precedence):

| Layer | Where |
|-------|-------|
| CLI overrides | `uv run python -m tool.main key=val` |
| Experiment configs | `configs/<tool>/experiment/*.yaml` |
| Base configs (read-only) | `configs/<tool>/` (copied from tool) |

## Rules

- **Invoke tools directly** — `uv run python -m <tool>.main`, not `run_experiment`.
- **Copied configs are read-only** — never edit files in `configs/<tool>/` (except `experiment/`).
- **Hydra group overrides** — `experiment=name` not `--config-name=experiment/name`.
- **Never `pip install`** — always `uv sync` / `uv run`.

## Key Files

| File | What it does |
|------|-------------|
| `configs/meta.yaml` | Tool registry (name, path, entrypoint) |
| `scripts/add_tool` | Register tool + copy base configs |
| `scripts/snapshot_experiment` | Freeze config + pin commit + tag |

## Tool Compatibility

Tools must use Hydra with no hardcoded config path:

```python
@hydra.main(config_path=None, config_name=None, version_base=None)
```
