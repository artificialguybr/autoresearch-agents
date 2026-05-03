---
name: autoresearch-agents
description: "Use this skill when the user wants to run, resume, or adapt the autoresearch-agents autonomous LLM training research workflow: multi-agent experiment planning, paper-inspired hypotheses, single-file train.py edits, fixed prepare.py evaluation, experiments.tsv logging, val_bpb optimization, or overnight GPU research loops."
metadata:
  short-description: Run autonomous LLM training experiments
---

# Autoresearch Agents

Use this skill to operate the `artificialguybr/autoresearch-agents` workflow: autonomous experiments against a small LLM training setup where the only objective is lowering `val_bpb` under the fixed evaluation in `prepare.py`.

## Core Contract

- Optimize `val_bpb`; lower is better.
- Edit only `train.py` during experiments.
- Treat `prepare.py` as immutable. It owns constants, data preparation, dataloading, tokenizer, and `evaluate_bpb`.
- Do not add dependencies or change `pyproject.toml`.
- Each experiment has a 5-minute training budget. Treat runs over 10 minutes wall clock as failed.
- Keep changes focused. Prefer simple changes unless a complex change earns a clear metric gain.
- Leave experiment state files uncommitted unless the repository instructions explicitly change.

## Repository Files

Read these first when present:

- `README.md` for project overview and setup.
- `orchestrator.md` for strategy, loop control, state files, and branch rules.
- `implementer.md` for the exact edit/run/log/keep/discard cycle.
- `brief.md` for current research direction and best-known facts.
- `todo.md` for session state.
- `experiments.tsv` and `papers.tsv` for compact searchable logs.
- `experiments/` and `papers/` only when the TSV line says more detail is needed.
- `crash_patterns.md` before approving risky ideas.
- `train.py` and `prepare.py` for code context. Only `train.py` is editable.

The current repository may mention `researcher.md`; verify whether it exists. If it is absent, handle the research role inline: propose hypotheses, search papers when useful, and write the hypothesis before implementation.

## Starting A Run

1. Confirm the target repo is clean enough for experiment commits.
2. Propose a run tag based on the date, then create `autoresearch/<tag>` from the main branch if it does not already exist.
3. Verify `~/.cache/autoresearch/` has data shards and tokenizer. If absent, tell the user to run `uv run prepare.py` and wait.
4. Initialize missing state files with headers/placeholders: `experiments.tsv`, `papers.tsv`, `brief.md`, `todo.md`, and `crash_patterns.md`.
5. Run the unmodified baseline first.

## Experiment Loop

For every experiment:

1. Read `brief.md`, recent rows from `experiments.tsv`, and `crash_patterns.md`.
2. Pick one focused hypothesis. Use `papers.tsv` or paper search for inspiration when the next direction is unclear.
3. Edit `train.py` only.
4. Commit the `train.py` change before running, using `exp NNN: short description`.
5. Run:

```bash
uv run train.py > run.log 2>&1
```

6. Extract:

```bash
grep "^val_bpb:\|^peak_vram_mb:\|^training_seconds:\|^mfu_percent:" run.log
```

7. If the run crashed, inspect `tail -n 50 run.log`. Fix only trivial implementation mistakes once; otherwise revert and log the crash.
8. Decide:
   - `keep`: lower `val_bpb`, or equal/better result with meaningful simplification.
   - `discard`: worse or equal result without a simplification win.
   - `crash`: failed run that should not be kept.
9. Log one TSV row in `experiments.tsv`:

```text
id	commit	tags	hypothesis	val_bpb	memory_gb	status	learned	paper_id	idea_source
```

10. Write `experiments/exp_NNN.md` when the result is surprising, the diff is complex, or the insight does not fit one TSV sentence.
11. Update `todo.md`; update `brief.md` every roughly 10 experiments or when direction shifts.

## Keep/Discard Discipline

- Keep the experiment commit only when it improves `val_bpb` or simplifies code without hurting the metric.
- For discards, reset the experiment commit created for that run. Do not reset unrelated user work.
- For crashes, record reusable lessons in `crash_patterns.md`.
- Weigh tiny improvements against complexity. A tiny gain from deleting code is more valuable than a tiny gain from a brittle patch.

## Reporting To The User

When summarizing a run, include:

- Experiment id and commit.
- `val_bpb`, memory GB, and status.
- What changed and what was learned.
- Whether the branch advanced or reverted.
- Any new crash pattern or direction change.

If the user asks for an autonomous/overnight run, continue the loop until interrupted, while keeping state in files so the run is recoverable.
