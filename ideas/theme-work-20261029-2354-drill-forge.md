---
id: drill-forge
title: "drill-forge — a spaced-repetition engine whose cards are typed programs, not static text"
slug: drill-forge
summary: "A from-scratch TypeScript study framework where every flashcard is a small parameterized expression evaluated fresh at review time, paired with a de-novo decay scheduler and CRDT deck sync across offline-capable devices."
status: accepted
created_at: "2026-10-29T20:59:00Z"
run_id: theme-work-20261029-2354
tags: [tutoring, assessment, step-by-step, offline-sync]
platform: web
software_class: framework
solution_category: education
architecture: sync_based
scale: small_team
language: typescript
stack: js_ts
layer: library
threat_model: [malicious_client]
from_scratch: true
dependency_declared: []
novelty_axis: architecture
---

## Problem

Spaced repetition works, but its card model is stuck in the index-card era: a card is static text, so learners memorize the card rather than the skill. A learner who can answer "12×13 = 156" on a static card may still fail to multiply any other pair, and conjugation drills, code-tracing exercises, and unit conversions all need fresh instances every session. The dominant existing tools bolt on randomization through fragile add-ons that inject scripts into a review UI never designed for evaluation. Worse, decks are locked inside cloud accounts, so a study group or classroom cannot own its material. The result is a memorization tool that systematically fails at the exact moment skills must generalize.

## De-novo scope

Built completely from scratch: a typed card language in which a card is a parameterized expression — generators draw values from declared ranges, predicates constrain them, and renderers present question and answer templates; a deterministic evaluator sandbox that executes card programs with no network, storage, or clock capabilities; a de-novo scheduler implementing per-card decay estimation over log-scale memory strength, updated on each grade; a review session engine that instantiates cards at review time so two sessions never see the same numbers; and a CRDT operation-log sync layer so decks and scheduling state merge across devices without a central authority. The deliverable is a framework-layer library with an offline-capable web front end. No existing SRS codebase, sandbox library, or CRDT package is wrapped; the browser runtime is the platform, not the substance.

## Architecture & components

The pipeline runs card source through compile, evaluate, schedule, and render stages. The compiler type-checks card programs against the framework's interfaces so a malformed card fails at authoring time, never mid-review. The evaluator executes compiled programs inside a strict capability-free environment where determinism is enforced by seeding from a session nonce, making every generated instance reproducible for debugging. The scheduler owns a per-card memory-strength vector updated by bounded exponential decay between reviews; grades feed a simple regression over log-strength rather than fixed intervals. Sync serializes deck edits and grade events as an operation log with causal ordering; concurrent merges converge without a server because operations commute by construction. Failure modes — a card program that throws, a sync conflict on scheduling state — degrade to quarantining the card or deferring to the most conservative memory estimate.

## Stack rationale

TypeScript is the right fit because the framework's core promise is that card programs are checked against an interface, and its type system expresses exactly that contract between deck authors and the review engine. The browser is the delivery platform since learners study on locked-down machines where nothing can be installed; a service worker makes sessions work offline. A single language covers the evaluator, the web worker that isolates execution from the UI thread, and the sync layer, keeping the typed boundary between authored cards and engine internals in one enforceable place.

## Threat model

The named adversary is a malicious client: because decks sync peer-to-peer without a trusted server, a hostile or compromised device in a study group can attempt two attacks. First, forged review grades sent to farm streaks or corrupt shared scheduling state — answered by requiring grades to be signed by the generating session and by anomaly checks that cap implausible grade rates during merge. Second, hostile card programs embedded in a shared deck attempting to escape the sandbox — answered by the evaluator's capability-free design: no network, no storage access, no wall clock, and deterministic seeded randomness, so even a malicious card has nothing to exfiltrate to and nothing to observe.

## Novelty statement

Novelty is on the architecture axis: a sync-based study framework where cards are evaluated programs rather than stored text, with offline peer-to-peer deck replication instead of cloud accounts. Existing tools like Anki are event-recording databases with static cards and server-locked sync; add-ons that inject randomization run outside any capability boundary. The architectural difference is substantive — typed card compilation, a capability-free deterministic evaluator, and CRDT-based convergence without a central service — and is introduced de novo rather than as a wrapper or rename of any existing project.
