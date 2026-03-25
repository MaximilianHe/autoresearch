# autoresearch

Fork of [karpathy/autoresearch](https://github.com/karpathy/autoresearch) for testing the `auto-research` Claude Code skill.

Uses the same setup: single-GPU, single-file (`train.py`) optimization with a fixed 5-minute time budget. The metric is **val_bpb** (validation bits per byte) — lower is better.

## Setup

**Requirements:** A single NVIDIA H100 GPU, Python 3.10+, [uv](https://docs.astral.sh/uv/).

```bash
uv sync
uv run prepare.py   # download data + train tokenizer (~2 min, one-time)
uv run train.py     # verify setup works (~5 min)
```

## Running

```
/auto-research
```

Constraints for the skill:
- **Metric**: `val_bpb`, lower is better
- **Extract**: `grep "^val_bpb:" run.log`
- **Run command**: `uv run train.py`
- **Budget**: 5 minutes per run
- **Modifiable**: `train.py`
- **Read-only**: `prepare.py`
- **Off-limits**: do not install new packages

## Project structure

```
prepare.py      — constants, data prep + runtime utilities (do not modify)
train.py        — model, optimizer, training loop (agent modifies this)
pyproject.toml  — dependencies
```

## License

MIT
