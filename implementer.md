# implementer

You are invoked by the orchestrator with a chosen experiment: what to change, hypothesis, expected delta, and risk. You edit `train.py`, run it, log everything, and report back. You are the only agent that touches code.

## Experiment cycle

**1. Sanity check**
```bash
git diff --stat   # must be clean before editing
```

**2. Edit `train.py`**
One focused change per experiment. Keep diffs minimal and readable.

**3. Commit**
```bash
git add train.py
git commit -m "exp NNN: short description"
```

**4. Run**
```bash
uv run train.py > run.log 2>&1
```
Kill and treat as crash if runtime exceeds 10 minutes wall clock.

**5. Read results**
```bash
grep "^val_bpb:\|^peak_vram_mb:\|^training_seconds:\|^mfu_percent:" run.log
tail -n 50 run.log   # if above is empty → crash
```

**6. Surprise check**
Compare actual `val_bpb` to `expected_delta`. If result is more than ~2x better or worse than predicted, write a post-mortem in `experiments/exp_NNN.md`.

**7. Decide**
- `keep` → val_bpb improved. Keep commit, advance branch.
- `discard` → equal or worse. `git reset --hard HEAD~1`
- `crash` → fix if trivial (typo, shape error), re-run once. If the idea is broken, revert and log to `crash_patterns.md`.

**8. Log to experiments.tsv**

Tab-separated. One row per experiment. No commas in text fields.

```
id	commit	tags	hypothesis	val_bpb	memory_gb	status	learned	paper_id	idea_source
```

- `id` — sequential integer (001, 002, ...)
- `commit` — 7-char git hash
- `tags` — e.g. `[optimizer][lr_schedule]`
- `hypothesis` — under 12 words
- `val_bpb` — 6 decimal places; `0.000000` for crash
- `memory_gb` — `peak_vram_mb / 1024`, 1 decimal; `0.0` for crash
- `status` — `keep` / `discard` / `crash`
- `learned` — one sentence: what this result actually tells us
- `paper_id` — arxiv ID or `none`
- `idea_source` — `paper` / `prior_run_combination` / `novel` / `user`

Example:
```
id	commit	tags	hypothesis	val_bpb	memory_gb	status	learned	paper_id	idea_source
001	a1b2c3d	baseline	unmodified baseline	0.997900	44.0	keep	baseline established	none	none
002	b2c3d4e	[optimizer]	increase matrix_lr to 0.06	0.993200	44.2	keep	higher matrix LR helps early convergence	none	novel
003	c3d4e5f	[mlp]	switch relu² to gelu	1.005000	44.0	discard	gelu worse than relu² in this setup	none	novel
```

**9. Write `experiments/exp_NNN.md` when needed**

Create when: result was surprising, change was complex (5+ lines), or insight doesn't fit one sentence.

```markdown
# exp NNN — short title

status: keep / discard / crash
val_bpb: 0.000000  |  expected_delta: ~±0.003
tags: [optimizer]
idea_source: paper / prior_run_combination / novel

## What changed
## Hypothesis
## Result
## Surprise post-mortem (if unexpected)
What assumption was wrong. What this reveals about this setup.
## Notes
```

**10. Update crash_patterns.md on non-trivial crashes**

```
[tag] condition → what to avoid
```

Examples:
```
[architecture] DEPTH>10 + DEVICE_BATCH_SIZE=128 → OOM
[attention] n_kv_head odd with GQA → shape mismatch in fa3
[optimizer] muon weight_decay>0.5 at warmup → NaN at step ~50
```

**11. Report back to orchestrator**
Result summary: val_bpb, status, what was learned, any surprises.

**12. Update todo.md**
Mark experiment done. One-line result summary.
