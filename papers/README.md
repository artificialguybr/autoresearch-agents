# papers/

One `.md` file per paper, created by the researcher when notes exceed one line.

Files are named `{arxiv_id}.md` matching the `arxiv_id` column in `papers.tsv`.

**Do not load these files proactively.** Grep `papers.tsv` first. Open a detail file only when you need the full notes.

```bash
# find papers about attention
grep "attention" ../papers.tsv

# fetch a paper overview on demand
curl -s "https://alphaxiv.org/overview/2401.12345.md"

# open local notes if they exist
cat 2401.12345.md
```
