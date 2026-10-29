# Runbook — Daily Themed Run: Productivity / Learning / Work / Team (Task ID: SWARM-THEME-WORK)

Executed daily 09:00 Europe/Moscow. Identical to `docs/runbook_scheduled_run.md`
in every pipeline step, quota, rejection rule, and publishing channel — with
one difference: the run's `solution_category` slots are themed.

## Theme mapping (user intent -> controlled vocabulary)

| Theme word | solution_category targets |
|---|---|
| productivity | `productivity` |
| learning     | `education` (adjacent: `research_tooling`) |
| work         | `productivity`, `collaboration`, `communication`, `automation` |
| team         | `collaboration`, `communication` |

Priority order: `productivity`, `education`, `collaboration`,
`communication`, `automation`.

## Themed-run rules

1. Theme applies to `solution_category` ONLY. Every other axis (platform,
   software_class, architecture, scale, layer, stack, language,
   threat_model) stays coverage-biased as usual.
2. The per-run cap `max_share_per_value` (0.2 -> max 2 ideas per category
   at n=10) is pipeline-enforced and NEVER exceeded for the theme's sake.
   Fill themed slots first (up to 8 of 10 at 2 per category); remaining
   slots follow normal coverage-aware sampling and may land anywhere the
   cap allows (including themed categories with headroom).
3. If the pipeline rejects a themed idea for a quota violation, redirect
   the replacement to a themed category with remaining headroom; if none,
   fall back to a normal coverage-biased cell.
4. Use run-id `theme-work-YYYYMMDD-HHMM` (Europe/Moscow) so manifests,
   commits, and digests carry the theme.
5. All SWARM-RUN rules apply unchanged: from-scratch only, frontmatter
   per `schema/idea_frontmatter.schema.json`, six body sections >= 25
   words each, >= 5 distinct stacks, >= 3 distinct threat models, >= 4
   distinct layers, rejection rules (a)-(f), credentials only via
   `/home/z/my-project/secrets` + GIT_ASKPASS, retry-once-then-log per
   channel.
6. The digest posted to Discord + Telegram must open with:
   "themed run: productivity / learning / work / team".
