# anox idea swarm — corpus of diverse from-scratch software ideas

This repository is the **living corpus** of a software-idea swarm ("the
Queen"). Scheduled runs generate diversity-balanced, from-scratch software
ideas, index them, and publish everything to two git remotes and two chat
channels automatically.

- GitHub mirror: https://github.com/openxcraft/idea
- GitLab home:   https://gitlab.com/wellonn/idea
- Digests are posted after every run to a Discord channel and a Telegram
  topic group.

## Layout

| Path | What it is |
|------|------------|
| `config.yaml` | Run budget, quotas, remotes, digest mode |
| `tags/taxonomy.yaml` | Base topic taxonomy |
| `tags/diversity_matrices.yaml` | Controlled vocabularies for all matrices |
| `schema/idea_frontmatter.schema.json` | Strict frontmatter schema (base + extended) |
| `prompts/MAIN_PROMPT.md` | Base spec for the Queen |
| `prompts/ADDENDUM_diversity.md` | Cross-Domain Diversity Generator addendum |
| `docs/indexing_pipeline.md` | Pipeline + rejection rules (a)-(f) |
| `docs/coverage_sampling.md` | Coverage-aware slot allocation algorithm |
| `ideas/` | One markdown file per idea |
| `ideas.jsonl` | Master index (one JSON line per accepted idea) |
| `memory/topic_graph.json` | Corpus topic graph |
| `memory/coverage_report.json` | Regenerated after every run |
| `runs/` | Per-run manifests (accepts, rejects with reason codes, push results) |

## How a run works

1. The Queen reads the prompts, config, and corpus memory.
2. Coverage-aware sampling picks under-represented
   (platform x class x category x layer) cells as slot targets.
3. Up to `max_ideas_per_run` ideas are written with complete, valid matrix
   tags (platform, class, category, architecture, scale, language, stack,
   layer, threat model, from-scratch, novelty axis).
4. The indexing pipeline validates, applies diversity rejection rules,
   hashes, dedups, appends `ideas.jsonl`, updates the topic graph and the
   coverage report, and writes a run manifest.
5. Everything is committed and pushed to both remotes, then a digest is
   posted to Discord and Telegram.

## Security

Credentials never live in this repository. The pipeline reads them from a
0600-permission store outside the repo and injects them into git via
`GIT_ASKPASS`. A pre-push secret scan aborts any commit that contains
token-like material.
