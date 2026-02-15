<div align="center">

<pre>
  configs/&lt;tool&gt;/         tools/&lt;tool&gt;/
  ┌──────────────┐       ┌──────────────┐
  │ experiment/  │  ←──  │ git submodule │
  │ snapshots/   │       │ (pinned SHA)  │
  └──────────────┘       └──────────────┘

    e x p e r i m e n t S t a s h

  config + commit pin = exact reproduction
</pre>

</div>

---

## quickstart

```bash
# add a tool (clones as submodule, copies configs, registers in meta.yaml)
python3 scripts/add_tool mytool https://github.com/org/mytool

# run experiments — invoke the tool directly, never through a bridge script
cd tools/mytool
uv run python -m mytool.main experiment=my_sweep

# freeze for reproduction
python3 scripts/snapshot_experiment mytool my_sweep --tag camera-ready --commit

# reproduce anytime
git checkout snapshot/camera-ready
git submodule update --init
cd tools/mytool && uv run python -m mytool.main experiment=my_sweep
```

---

## architecture

```
experimentStash/
├── configs/
│   ├── meta.yaml              # tool registry
│   └── <tool>/
│       ├── experiment/        # your sweep configs
│       └── snapshots/<tag>/   # frozen configs (fully resolved)
├── tools/
│   └── <tool>/                # git submodule (pinned commit)
├── scripts/
│   ├── add_tool               # register tool + copy base configs
│   └── snapshot_experiment    # freeze config + pin commit + tag
└── outputs/                   # experiment results
```

Three layers for config composition (highest to lowest precedence):

| Layer | Where | When |
|-------|-------|------|
| CLI overrides | `uv run python -m tool.main key=val` | always available |
| Experiment configs | `configs/<tool>/experiment/*.yaml` | sweep definitions |
| Base configs (read-only) | `configs/<tool>/` | copied from tool, never edited |

---

## two modes

| Mode | Script | What it does |
|------|--------|-------------|
| **Copy** | `add_tool` | Clones submodule, copies base configs for reference |
| **Freeze** | `snapshot_experiment` | Flattens config, pins commit, creates git tag |

---

## snapshot

```bash
python3 scripts/snapshot_experiment mytool my_exp --tag camera-ready --commit
```

1. Resolves all Hydra defaults and interpolations
2. Pins tool submodule to exact commit SHA
3. Creates `snapshot/camera-ready` git tag
4. Commits everything atomically

---

## rules

- **Invoke tools directly** — `uv run python -m <tool>.main`, not `run_experiment`.
- **Copied configs are read-only** — never edit base configs in `configs/<tool>/`.
- **Customize via CLI or experiment YAML** — `experiment=name` not `--config-name`.
- **Never `pip install`** — always `uv sync` / `uv run`.

---

## tool compatibility

Tools must accept CLI config path override:

```python
# compatible
@hydra.main(config_path=None, config_name=None, version_base=None)
```

---

<br>

<p align="center">
<sub>MIT License &middot; <a href="https://github.com/cmvcordova">cmvcordova</a></sub>
</p>
