# orchestrator

You run the autonomous research loop. You own strategy, direction, and decisions. You invoke subagents by asking them to read their respective `.md` files.

## Constraints (single source of truth — subagents follow these)

- Only `train.py` is editable. `prepare.py` is immutable — it contains the fixed eval, dataloader, tokenizer, and training constants.
- No new packages. Only what's in `pyproject.toml`.
- Ground truth metric: `val_bpb` (lower is better, vocab-size-independent).
- 5-minute wall-clock training budget per experiment (excluding startup/compilation).
- Do not commit `experiments.tsv`, `papers.tsv`, `brief.md`, `todo.md`, `crash_patterns.md` — leave untracked by git.

## Simplicity criterion

All else being equal, simpler is better. Weigh complexity cost against improvement magnitude on every keep decision:
- Tiny improvement + added complexity → probably not worth it
- Tiny improvement + deleted code → keep
- No improvement + simpler code → keep

## Setup (new run)

1. Agree on a run tag (e.g. `apr1`). Branch `autoresearch/<tag>` must not exist.
2. `git checkout -b autoresearch/<tag>`
3. Read `prepare.py` (skim) and `train.py` (skim).
4. Verify `~/.cache/autoresearch/` has data shards and tokenizer. If not, tell the user to run `uv run prepare.py` and wait.
5. Initialize: `experiments.tsv` (header only), `papers.tsv` (header only), `crash_patterns.md` (empty), `brief.md`, `todo.md`.
6. Confirm setup with user. First experiment is always the unmodified baseline.

## The loop

LOOP FOREVER — never pause to ask if you should continue. The user may be asleep. Run until manually interrupted.

1. Read `brief.md` — direction, principles, drift check.
2. Invoke **researcher** (→ `researcher.md`): give it the current direction, best result so far, and exhausted categories. Ask it to search and return ideas with reasoning.
3. Researcher returns ideas. You review, pick the best candidate, decide what to test next.
4. Invoke **implementer** (→ `implementer.md`): give it the chosen experiment — what to change, the hypothesis, expected delta, and risk.
5. Implementer returns result. Update `brief.md` category counts and `todo.md`.
6. Every ~10 experiments: fill drift check in `brief.md`. Reorient if anything is off.
7. When direction shifts or ~10 experiments pass: rewrite `brief.md` (except ANCHOR). Compress findings → patterns → principles.

## Plateau & balance

- 5+ consecutive discard/crash → change direction. Tell researcher to explore a different category.
- One category dominating 5+ runs without a keep → push toward unexplored territory.
- If stuck: think harder — combine previous near-misses, try more radical architectural changes.

## Multi-GPU

One branch per GPU (`autoresearch/<tag>-gpu0`, etc.). Assign roles: one GPU explores radically, one refines. Every ~5 experiments, sync: read the other branch's `brief.md`, pull new findings, patterns, or principles into your own.
