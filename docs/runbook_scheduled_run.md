# Runbook — Scheduled Swarm Diversity Run (Task ID: SWARM-RUN)

Executed every 6 hours by the scheduler. Work in `/home/z/my-project/swarm`
(git repo; remotes: `github` = https://github.com/openxcraft/idea,
`gitlab` = https://gitlab.com/opnxcode/idea). Run-id: `run-YYYYMMDD-HHMM`
(Europe/Moscow).

## Procedure

1. **Read the spec before writing anything:**
   - `prompts/MAIN_PROMPT.md` (base spec)
   - `prompts/ADDENDUM_diversity.md` (MANDATORY diversity rules)
   - `config.yaml` (quotas), `tags/diversity_matrices.yaml` (vocabularies)
2. **Load corpus state:** `ideas.jsonl`, `memory/topic_graph.json`,
   `memory/coverage_report.json`.
3. **Coverage-aware slot planning:** bias slots toward under-represented
   (platform x class x category x layer) cells from the coverage report
   (`top_under_represented`, `rolling_window.gaps`); avoid values already
   near their per-run cap.
4. **Generate `max_ideas_per_run` (10) NEW from-scratch ideas**, each at
   `ideas/<run-id>-<slug>.md`:
   - COMPLETE frontmatter per `schema/idea_frontmatter.schema.json`:
     id, title, slug, summary (>=60 chars), status: accepted, created_at
     (ISO-8601 UTC), run_id, tags (>=3 from `tags/taxonomy.yaml`), and the
     matrix fields platform, software_class, solution_category,
     architecture, scale, language, stack (must match the language's stack
     family), layer, threat_model (list, >=1), from_scratch: true,
     dependency_declared (max 1, general-purpose runtime/toolchain only),
     novelty_axis.
   - Body: exactly six sections, each >=25 words: `## Problem`,
     `## De-novo scope`, `## Architecture & components`, `## Stack rationale`,
     `## Threat model`, `## Novelty statement`.
   - Quotas (pipeline-enforced): max 0.2 share per matrix value per run
     (balanced_relaxation only for the 4-value `scale` matrix — at 10 ideas:
     max 2 per scale value); >=5 distinct stacks, >=3 distinct threat models,
     >=4 distinct layers; never reuse a (platform, class, category,
     architecture, threat_model) tuple already in `ideas.jsonl`; no wrappers
     or renames of existing OSS projects; privacy/security/anonymity/
     censorship-resistance claims MUST name a threat model.
5. **Index:**
   `python3 /home/z/my-project/scripts/swarm_index.py --run-id <run-id>`
   Validates, applies rejection rules (a)-(f) with reason codes, hashes,
   dedups (tuple cosine >= 0.85), appends `ideas.jsonl`, updates topic graph
   + `memory/coverage_report.json`, writes `runs/<run-id>_manifest.json`.
   If ideas are rejected, write replacements (new cells, same quotas) and
   re-run until the batch passes or the 10-idea budget is spent.
6. **Publish:**
   `python3 /home/z/my-project/scripts/swarm_push.py --run-id <run-id>`
   Commits, pushes GitHub + GitLab, posts the run digest to the Discord
   webhook and Telegram topic group, writes push results into the manifest.

## Security (absolute rules)

- Credentials live ONLY in `/home/z/my-project/secrets/credentials.json`
  (0600). NEVER print, echo, or commit token values; NEVER stage anything
  under `/home/z/my-project/secrets`. The scripts inject credentials via
  GIT_ASKPASS and run a pre-commit secret scan — do not bypass it.
- If a channel fails (e.g. GitHub/GitLab 403): retry once, log in the
  manifest, continue. Never delete generated ideas because of push
  failures; commits stay local and push automatically on a later run once
  permissions are fixed.

## Finish

Report: ideas accepted/rejected (reason codes), diversity minimums,
corpus size, coverage delta, push status per channel.
