# Runbook — Weekly Coverage Audit (Task ID: SWARM-AUDIT)

Executed Sundays 20:00 Europe/Moscow.

1. `python3 /home/z/my-project/scripts/swarm_index.py --audit-only`
   Regenerates `memory/coverage_report.json` against the full corpus and
   writes `runs/audit-<date>.md` with the rolling-window check: across the
   last 10 x max_ideas_per_run accepted ideas, every value in every matrix
   must appear at least once, or the audit must justify its absence.
2. `python3 /home/z/my-project/scripts/swarm_push.py --audit`
   Commits the refreshed coverage report + audit file, pushes to GitHub
   (openxcraft/idea) and GitLab (wellonn/idea), and posts the coverage
   digest (per-axis entropy, uncovered values, window gaps) to the Discord
   webhook and the Telegram topic group.
3. Security: never print/commit anything from `/home/z/my-project/secrets`;
   the push script handles credentials via GIT_ASKPASS.
4. If pushes fail (e.g. 403): retry once, log it, continue — the audit file
   still exists locally and in the chat digests.
5. Finish by reporting: corpus size, per-axis entropy, uncovered values in
   the rolling window, and which specific (platform x class x category x
   layer) cells the next generation runs must target.
