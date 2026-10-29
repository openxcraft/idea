# ADDENDUM — Cross-Domain Diversity Generator

> The user's addendum, verbatim. Appended to the main prompt
> (`prompts/MAIN_PROMPT.md`). Where the two conflict, this addendum wins.

**Purpose.** Guarantee that the swarm produces software ideas that are genuinely varied across every axis of software, and that the corpus as a whole converges toward uniform coverage rather than clustering in a few fashionable domains.

**Ideation matrices (all four must be sampled each run).** Every idea must be tagged with one value from each matrix. The queen must not draw more than `max_share_per_value` (default 0.2) of a run's ideas from any single value in any matrix.

1. **Platform**: desktop (Windows, macOS, Linux), mobile (iOS, Android), web (SPA, SSR, static), CLI, TUI, browser extension, IDE extension, editor plugin, server/backend, cloud service, serverless function, edge runtime, embedded/MCU, IoT, wearables, smart TV, automotive, robotics, AR/VR/XR, game console, kernel/driver, bootloader, firmware, protocol daemon, container image, WASM module.
2. **Software class**: end-user app, library, framework, SDK, API, microservice, monolith, plugin, protocol/spec, compiler, interpreter, transpiler, linter, formatter, package manager, database, storage engine, cache, queue, scheduler, orchestrator, runtime, emulator, simulator, game, tool, dev tool, test harness, fuzzer, profiler, debugger, observability agent, ML model, ML pipeline, autonomous agent, scraper, crawler, dApp, smart contract, P2P node, mesh node, cryptographic primitive, consensus module, identity system, payment rail, filesystem, network stack, build system, CI/CD component.
3. **Solution category**: automation, productivity, communication, collaboration, security, privacy, anonymity, censorship resistance, data engineering, data leak prevention, AI-egress control, content moderation, media processing, generative tooling, education, research tooling, scientific computing, health, accessibility, finance, commerce, logistics, governance, legal-tech, infrastructure, observability, resilience/backup, offline-first, low-resource/edge-compute, sustainability, creative tools, gaming, simulation.
4. **Architecture & scale**: single-user local, single-tenant server, multi-tenant SaaS, self-hosted, federated, decentralized, P2P, offline-first, sync-based, event-driven, stream-processing, batch, real-time, embedded-in-another-app, library-only, protocol-first, hardware-paired. Scale tags: solo dev, small team, org-wide, internet-scale.

**From-scratch constraint (applies to every idea).** Assume no existing codebase. The idea must specify what it builds de novo, not what it patches, forks, or wraps. If it depends on an existing project, the dependency must be explicitly declared and capped (default: no more than one primary dependency, and it must be a general-purpose runtime or language, not the idea's substance).

**Stack and language spread.** Each idea must declare a primary language and stack. Within a run, the queen must ensure at least `min_distinct_stacks` (default 5) across: systems (C, C++, Rust, Zig, Go), JVM (Java, Kotlin, Scala), .NET (C#, F#), scripting (Python, Ruby, PHP, Lua), JS/TS (Node, Deno, Bun, browser), functional (Haskell, OCaml, Elixir, Clojure), mobile-native (Swift, Kotlin/Android, Dart), data (SQL, R, Julia), scientific (Fortran, MATLAB), blockchain (Solidity, Move), shell (Bash, POSIX), and novel/niche languages when justified.

**Layer and depth spread.** Each idea must declare a layer: application, service, library, framework, protocol, runtime, kernel/driver, firmware, spec/standard, or research prototype. The run must include at least one idea per layer across the `max_ideas_per_run` budget when the budget allows.

**Threat and constraint diversity.** At least `min_distinct_threat_models` (default 3) per run from: passive network observer, active MITM, malicious server, malicious client, compromised endpoint, supply-chain attacker, side-channel observer, physical-access attacker, insider, coerced disclosure, mass surveillance, targeted surveillance. Ideas that claim "privacy" without naming a threat model are rejected.

**Coverage-aware sampling (corpus-level).** Before each run, the queen reads `memory/topic_graph.json` and the taxonomy distribution across the existing corpus and computes per-axis coverage. It then biases idea slots toward under-represented cells (platform x class x category x layer). The run manifest must record which cells were targeted and why.

**Extended frontmatter (additive to the existing strict schema).** Every idea file must include, in addition to the existing fields: `platform`, `software_class`, `solution_category`, `architecture`, `scale`, `language`, `stack`, `layer`, `threat_model` (list), `from_scratch` (boolean, must be true unless `dependency_declared` is set), `dependency_declared` (list, capped), `novelty_axis` (which axis this idea contributes novelty on: platform, class, category, architecture, threat model, or combination).

**Rejection rules (enforced before the write in the indexing pipeline).** An idea is rejected if: (a) any matrix value is missing or outside the taxonomy; (b) `from_scratch` is false and no dependency is declared; (c) "privacy/anonymity/security" is claimed without a threat model; (d) its (platform, class, category, architecture, threat_model) tuple duplicates an existing tuple within cosine similarity >= `dedup_threshold`; (e) it violates `max_share_per_value` for the current run; (f) it merely renames an existing open-source project without a substantive architectural difference. All rejections are logged with reason codes in the run manifest.

**Anti-clustering quota per run (defaults, configurable).**

```yaml
diversity:
  max_share_per_value: 0.2
  min_distinct_stacks: 5
  min_distinct_threat_models: 3
  min_distinct_layers: 4
  require_one_per_layer_when_budget_allows: true
  forbid_wrapper_only: true
  forbid_rename_only: true
  coverage_bias: true            # bias slots toward under-represented corpus cells
  report_uncovered_cells: true
```

**Config additions (config.yaml).**

```yaml
diversity:
  max_share_per_value: 0.2
  min_distinct_stacks: 5
  min_distinct_threat_models: 3
  min_distinct_layers: 4
  require_one_per_layer_when_budget_allows: true
  forbid_wrapper_only: true
  forbid_rename_only: true
  coverage_bias: true
  report_uncovered_cells: true
  matrices_file: tags/diversity_matrices.yaml
  coverage_report_path: memory/coverage_report.json
```

**Deliverables to add.**

1. Full contents of `tags/diversity_matrices.yaml` enumerating every value for platform, software_class, solution_category, architecture, scale, language, stack, layer, and threat_model — as controlled vocabularies that extend `tags/taxonomy.yaml`.
2. Updated frontmatter JSON-schema including the extended fields.
3. Updated indexing pipeline section showing where the diversity rejection rules run (between step 1 validation and step 2 hashing).
4. Short description of the coverage-aware sampling algorithm: how per-axis distributions are computed from `ideas.jsonl` and `topic_graph.json`, how under-represented cells are scored, and how slots are allocated per run.
5. A `coverage_report.json` example showing per-axis distribution, top over-represented cells, and top under-represented cells after a hypothetical 1,000-idea corpus.

**Acceptance criteria.**

- Across any rolling window of 10 x `max_ideas_per_run` ideas, every value in every matrix must appear at least once, or the run manifest must justify its absence.
- No single value in any matrix may exceed `max_share_per_value` within a single run.
- Every run produces at least `min_distinct_stacks` distinct primary languages and `min_distinct_threat_models` distinct threat models when the run's budget is at least 10 ideas.
- Every idea carries a complete, taxonomy-valid set of matrix tags; incomplete ideas are rejected before hashing.
- The coverage report is regenerated after each run and reflects the corpus state after the run's commits.
