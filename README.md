# autoresearch

Fork of [karpathy/autoresearch](https://github.com/karpathy/autoresearch) for testing the `auto-research` Claude Code skill.

Uses the same setup: single-GPU, single-file (`train.py`) optimization with a fixed 5-minute time budget. The metric is **val_bpb** (validation bits per byte) — lower is better.

## Setup
**Requirements:** A NVIDIA GPU, Python 3.10+, [uv](https://docs.astral.sh/uv/).

```bash
uv sync
uv run prepare.py   # download data + train tokenizer (~2 min, one-time)
uv run train.py     # verify setup works (~5 min)
```

## Running autoresearch

### Prerequisites

You need [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed. Copy the contents of skill/ as follows:
- create a "auto-research" dir in ./claude/skills
- copy the SKILL.md into that directory
- copy the auto-research-agent.md into ./claude/agents/

### Caution
I allowed unrestricted Bash & Edit in .claude of this repo. Consider before running.

### Starting a run

In your repo, open Claude Code and type:

```
/auto-research
```

The skill starts with a **constraints phase** — it will ask you to define the parameters of the experiment before any code is touched. You need to provide:

| Constraint | What to enter for this repo |
|---|---|
| **Objective metric** (and direction) | `val_bpb`, lower is better |
| **How to extract the metric** | `grep "^val_bpb:" run.log` |
| **Run command** | `uv run train.py` |
| **Compute budget per run** | 5 minutes |
| **Modifiable files** | `train.py` |
| **Read-only files** | `prepare.py` |
| **Max iterations** | however many you want (default 20) |
| **Off-limits actions** | do not install new packages |

Once you confirm, the skill summarizes the constraints back to you and asks for a final go-ahead.

### What happens next

After confirmation the skill runs autonomously — you can walk away:

1. **Setup** — creates a `autoresearch/<tag>` branch, initializes `results.tsv` (experiment log) and `queue.md` (idea backlog).
2. **Understand** — reads `train.py`, identifies tunable levers (hyperparameters, architecture, etc.), and seeds `queue.md` with ~5 initial experiment ideas. You can add your own ideas at this point.
3. **Experiment loop** — repeats until max iterations:
   - Pops the top idea from `queue.md`
   - Delegates the experiment to a **worker subagent** that edits code, commits, runs `train.py`, extracts the metric, and logs the result to `results.tsv`
   - The main agent **verifies** the result (reads the diff, cross-checks the metric)
   - **Keeps** improvements (metric beats current best) or **reverts** via `git reset --hard` to the last good commit
   - Appends follow-on ideas to `queue.md` based on what it learned

Each experiment produces one commit and one row in `results.tsv`. The run log for the most recent experiment is in `run.log`.

### Architecture: orchestrator + worker subagent

The skill splits work across two context windows:

- **Orchestrator (main agent)** — thinks of ideas, manages the queue, delegates experiments, verifies results, decides keep/revert. This is the long-running context that maintains state across all iterations.
- **Worker subagent** — receives a single hypothesis, edits code, runs the experiment, logs the result, and reports back. Each worker invocation gets a fresh context window, so it doesn't accumulate context from prior experiments.

This split keeps the orchestrator's context window lean. In practice, ~44 iterations consumed ~164k/200k tokens of the main context — without the split, you'd blow through the context window much sooner.

### Tips

- **Unattended runs**: Grant broad tool permissions in your Claude Code settings so the skill doesn't block on permission prompts overnight.
- **Reviewing results**: `results.tsv` is the single source of truth. Each row has the commit hash, metric value, status (`keep`/`discard`/`crash`/`baseline`), and a description.
- **Steering mid-run**: Edit `queue.md` to inject ideas or reprioritize — the orchestrator pops from the top on each iteration.

## License

MIT
