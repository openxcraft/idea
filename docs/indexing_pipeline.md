# Indexing pipeline

Implemented by `scripts/swarm_index.py`. One idea flows through the steps
below; anything that fails a gate is **rejected before hashing** and logged
in the run manifest with a reason code.

```
step 0  load config.yaml + tags/diversity_matrices.yaml + corpus memory
        |
step 1  VALIDATION
        - frontmatter parses and matches schema/idea_frontmatter.schema.json
        - all required base + extended fields present
        - tags >= 3 and all exist in tags/taxonomy.yaml
        - body contains the required sections with substantive content
        - wrapper-only / rename-only heuristics (config: forbid_wrapper_only,
          forbid_rename_only) — title/summary/name reused from a known OSS
          project without a stated architectural difference => reject (f)
        |
step 1.5  DIVERSITY GATE            <-- the addendum's rejection rules run HERE,
        - (a) any matrix value missing or outside the taxonomy => reject
        - (b) from_scratch == false and dependency_declared empty => reject
        - (c) solution_category in {security, privacy, anonymity,
              censorship_resistance} and threat_model empty => reject
        - (e) this idea's matrix values would push the run over
              max_share_per_value (after balanced_relaxation) => reject
        - run-level minimums checked after the batch: min_distinct_stacks,
              min_distinct_threat_models, min_distinct_layers
        |
step 2  HASHING
        - sha256 over canonical serialization (id, title, summary, matrix
          tuple, body text normalized) => corpus-stable content hash
        |
step 3  DEDUP
        - (d) cosine similarity of the (platform, class, category,
              architecture, threat_model) tuple + TF-IDF of title+summary
              against every accepted corpus idea >= dedup_threshold
              (0.85) => reject
        |
step 4  INDEX APPEND
        - one JSON line appended to ideas.jsonl (hash, path, all matrix
          tags, cell key, run_id, created_at)
        |
step 5  TOPIC GRAPH UPDATE
        - memory/topic_graph.json: node per idea, edges by tag Jaccard >= 0.3
        |
step 6  COVERAGE REPORT REGENERATION
        - memory/coverage_report.json rebuilt from ideas.jsonl
        - reflects corpus state after this run's commits
        |
step 7  MANIFEST WRITE
        - runs/<run-id>_manifest.json: run metadata, targeted coverage cells
          + rationale, accepted ideas, rejected ideas with reason codes
          (a)-(f), quota reconciliation notes, rolling-window status
        |
step 8  PUBLISH (external, scripts/swarm_push.py)
        - commit + push remotes github + gitlab, then Discord + Telegram
        - push results are written back into the manifest
```

## Reason codes

| Code | Meaning |
|------|---------|
| (a)  | Matrix value missing or outside taxonomy |
| (b)  | from_scratch false with no declared dependency |
| (c)  | Privacy/security claim without a named threat model |
| (d)  | Duplicate of an existing tuple (cosine >= dedup_threshold) |
| (e)  | Violates max_share_per_value for the current run |
| (f)  | Wrapper-only or rename-only of an existing project |

## Quota reconciliation (balanced_relaxation)

A run of n ideas cannot satisfy `max_share_per_value = 0.2` on a matrix with
fewer than `n x 0.2` placeable slots: e.g. the `scale` matrix has 4 values,
so at most `floor(10 x 0.2) x 4 = 8` of 10 ideas can be placed without
exceeding the cap. For exactly those matrices, the per-value cap is raised
to `ceil(n / values)` — the most balanced distribution possible — and the
deviation is recorded in the run manifest under
`quota_reconciliation`. All other matrices enforce the strict 0.2 cap.

## Audit mode

`python3 scripts/swarm_index.py --audit-only` re-runs steps 5-7 against the
full corpus (no new ideas), regenerating the coverage report and a
rolling-window audit: across the last `10 x max_ideas_per_run` accepted
ideas, every value in every matrix must appear at least once, or the audit
must justify its absence. Results are written to `runs/audit-<date>.md`.
