# Coverage-aware sampling algorithm

The queen uses this algorithm (addendum deliverable 4) to decide which
cells each run should fill. It is implemented inside
`scripts/swarm_index.py` (`plan_slots()`), which also emits the slot plan
into the run manifest.

## 1. Computing per-axis distributions

From `ideas.jsonl` (one line per accepted idea, carrying every matrix tag):

- For each matrix `M` and each value `v` in the controlled vocabulary:
  `count[M][v]` = number of corpus ideas tagged `v`;
  `share[M][v] = count[M][v] / |corpus|`.
- Values with zero ideas are counted too — they are the coverage frontier.

From `memory/topic_graph.json` (nodes = ideas, edges = tag-Jaccard >= 0.3):

- `neighborhood[v]` = set of ideas within 2 hops of any idea tagged `v`.
  A value whose neighborhood spans many domains is considered
  well-integrated; an isolated value is "covered but clustered" and gets a
  small extra deficit bonus so the swarm preferentially adds ideas that
  also connect it to NEW domains.

## 2. Scoring under-represented cells

A **cell** is a (platform x class x category x layer) 4-tuple — the axes
the addendum names for coverage bias. For each cell `c` observed (or
plausible, since most cells are empty early on):

```
axis_deficit(M, v) = max(0, uniform(M) - share[M][v])
        where uniform(M) = 1 / |values(M)|

cell_deficit(c) = sum over the 4 axes of axis_deficit of the cell's values
                + 0.5 x (1 - cell_count(c) / max_cell_count)      # emptiness
                + 0.1 x isolation_penalty(neighborhoods)          # clustering
```

Empty cells inherit their axes' deficits, so early runs naturally spread
across the matrix product instead of drilling one axis. Over-represented
values contribute zero deficit (the max(0, ...) clamp), so they never
attract slots.

## 3. Slot allocation per run

```
1. n = min(max_ideas_per_run, remaining budget for the rolling window)
2. Sort candidate cells by cell_deficit descending (ties: fewer total
   corpus ideas first, then random for variety).
3. Take the top n cells as TARGET cells; cap repeats of any single axis
   value so the plan itself respects max_share_per_value (with the
   balanced_relaxation cap for small matrices).
4. Each slot = one target cell + free choice on the remaining axes,
   subject to run-level quotas (>= 5 stacks, >= 3 threat models,
   >= 4 layers when budget >= 10).
5. The manifest records, for each slot: the targeted cell, its deficit
   score, and one sentence of rationale.
```

## 4. Reporting

After the run commits, `memory/coverage_report.json` is rebuilt
(`docs/indexing_pipeline.md` step 6): per-axis counts + shares, entropy per
axis, top over-represented and top under-represented cells, uncovered
values per matrix, and the rolling-window status over the last
`10 x max_ideas_per_run` ideas. See
`memory/coverage_report.example.json` for the shape of the report after a
hypothetical 1,000-idea corpus.
