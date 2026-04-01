# autoresearch-agents

A personalized fork of [karpathy/autoresearch](https://github.com/karpathy/autoresearch) with multi-agent autonomous research capabilities.

The original idea: give an AI agent a small but real LLM training setup and let it experiment autonomously overnight. This fork keeps that philosophy intact, same fixed 5-minute time budget, same `val_bpb` metric, same single-file editing, but adds a multi-agent org structure, selective context loading, paper-driven ideation, and a richer but lean experiment log.

**What changed:**
- `program.md` split into role-specific agent files: `orchestrator.md`, `researcher.md`, `implementer.md`
- `results.tsv` replaced by `experiments.tsv` with richer columns + `experiments/` folder for on-demand detail
- `papers.tsv` for tracking paper-to-experiment connections + `papers/` folder for notes
- `brief.md` — a living research state document, rewritten periodically by the orchestrator
- `todo.md` — session-level task state, updated step by step

**What did NOT change:**
- `prepare.py` — untouched, fixed evaluation harness
- `train.py` — still the only file agents edit
- `pyproject.toml` — no new dependencies
- 5-minute time budget, `val_bpb` metric, single GPU

---

## How it works

Three agents, each with a clear role:

- **Orchestrator** — reads `brief.md`, decides research direction, invokes researcher and implementer, updates `brief.md` when direction shifts
- **Researcher** — proposes hypotheses, searches papers, writes the hypothesis before each experiment
- **Implementer** — edits `train.py`, runs the experiment, reads results, logs everything

Context is loaded selectively. Agents grep `experiments.tsv` and `papers.tsv` first. They only open detail files (`experiments/exp_NNN.md`, `papers/paper_ID.md`) when they need more than the summary line.

---

## Quick start

**Requirements:** A single NVIDIA GPU (tested on H100), Python 3.10+, [uv](https://docs.astral.sh/uv/).

```bash
# 1. Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. Install dependencies
uv sync

# 3. Download data and train tokenizer (one-time, ~2 min)
uv run prepare.py

# 4. Manually verify setup
uv run train.py
```

## Running the agents

Spin up Claude Code (or any agent) in this repo, then prompt:

```
Read orchestrator.md and let's kick off a new autoresearch run.
```

The orchestrator will set up the branch, read the role files, and begin the loop.

---

## Project structure

```
prepare.py          — constants, data prep, eval (do not modify)
train.py            — model + training loop (agents modify this)
orchestrator.md     — orchestrator agent rules
researcher.md       — researcher agent rules
implementer.md      — implementer agent rules
brief.md            — living research state (orchestrator rewrites)
todo.md             — current session task state
experiments.tsv     — experiment log, one row per run
experiments/        — detail files per experiment (on-demand)
papers.tsv          — paper log
papers/             — detail notes per paper (on-demand)
analysis.ipynb      — visualization of results
pyproject.toml      — dependencies
```

---

## Design principles

- **Selective context.** Agents never load all experiments or all papers. They grep first, open detail files only when needed.
- **Hypothesis before run.** The researcher writes what it expects *before* the implementer runs anything.
- **Short, dense logs.** Every TSV row is one line. Detail lives in files opened on demand.
- **brief.md forgets naturally.** When the orchestrator rewrites the brief, old detail fades. Only what matters now survives.
- **Papers as inspiration, not scripts.** The researcher searches arxiv/HuggingFace papers before each experiment but is equally encouraged to invent novel ideas.
- **Never stop.** Once the loop starts, agents run until manually interrupted.

---

## Platform support

Same as upstream — single NVIDIA GPU required. See the [original README](https://github.com/karpathy/autoresearch) for MacOS/Windows/AMD forks.

## License

MIT
