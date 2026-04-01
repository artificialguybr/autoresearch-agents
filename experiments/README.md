# experiments/

One `.md` file per experiment, created by the implementer when the result warrants more than one line.

Files are named `exp_NNN.md` matching the `id` column in `experiments.tsv`.

**Do not load these files proactively.** Grep `experiments.tsv` first. Open a detail file only when you need more context than the summary row provides.

```bash
# find experiments about attention
grep "attention" ../experiments.tsv

# then open a specific one if needed
cat exp_042.md
```
