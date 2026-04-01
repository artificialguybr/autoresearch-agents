# research brief

_Rewritten by the orchestrator every ~10 experiments or when direction shifts. ANCHOR never changes. Everything else is rewritten — old detail fades naturally._

---

## ANCHOR (never edit after first run)

goal: minimize `val_bpb` | metric: lower is better, vocab-size-independent
constraint: only `train.py`, 5-min budget, no new packages, fixed eval in `prepare.py`

---

## current direction
[What we're exploring and why. 2–3 sentences.]

## findings
_Specific, recent, experiment-level facts._
- [e.g. "DEPTH 8→10 gave +0.3% MFU, no bpb gain — exp 007"]

## patterns
_What seems consistent across multiple findings. Hedged._
- [e.g. "LR changes below 10% don't move bpb meaningfully here"]

## principles
_What we now believe about this specific hardware+dataset+architecture. Slow to change._
- [e.g. "bpb improvements below 0.001 may be noise — verify before keeping"]

## best result
val_bpb: — | experiment: — | key change: —

## category counts
[optimizer]:0  [lr_schedule]:0  [architecture]:0  [attention]:0  [regularization]:0
[mlp]:0  [positional]:0  [normalization]:0  [init]:0  [batch]:0
[value_embed]:0  [window]:0  [depth]:0  [width]:0

## drift check (every ~10 experiments)
last checked at exp #: —
- still optimizing val_bpb? —
- recent experiments coherent with direction? —
- same category 5+ runs without a keep? → if yes, pivot

## unexplored territory
[Categories or ideas not yet tried.]

## dead ends
[What failed and why. Be specific enough to avoid re-trying the same thing.]
