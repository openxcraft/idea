---
id: pair-pulse
title: "pair-pulse — blind pairing rotations that cross-pollinate team knowledge"
slug: pair-pulse
summary: "A from-scratch F# serverless API that computes fair rotating pairing schedules for team learning sessions from affinity sketches, so the host can learn pairings without ever holding readable skill data."
status: accepted
created_at: "2026-10-29T21:05:00Z"
run_id: theme-work-20261029-2354
tags: [serverless, orchestration, background-tasks]
platform: serverless_function
software_class: api
solution_category: collaboration
scale: small_team
language: fsharp
stack: dotnet
layer: service
threat_model: [malicious_server]
novelty_axis: platform
architecture: single_tenant_server
from_scratch: true
dependency_declared: []
---

## Problem

Teams that pair-program, rotate mob seats, or run cross-training sessions face the same quiet failure: pairing drifts into cliques. People schedule with the colleagues they already know, skill silos harden, and the junior who most needs exposure to the payments domain keeps pairing with the same front-end friend. At ten-plus people, manual fairness is combinatorially hopeless, and the obvious fix — feeding a scheduling tool everyone's skill matrix — means shipping an HR-grade dossier to a SaaS just to answer "who pairs with whom on Tuesday". The data cost of the solution exceeds the value of the rotation, so most teams simply give up and rotate alphabetically, which is fair and useless.

## De-novo scope

Built completely from scratch: a rotation solver that maximizes pairwise skill-distance under calendar and availability constraints, expressed as balanced round-robin construction with a diversity objective; a blind-input protocol where each member's client computes a local affinity sketch — a compact, non-reversible digest of their skill tags — and submits only the sketch plus availability; a scheduled trigger that assembles commitments, solves, and publishes the week's pairings to the team chat webhook; and a client-side verification routine that lets any member re-derive the optimal pairing from public commitments and check that the host executed the solver honestly. No scheduling SDK, no HR platform, no matching library is wrapped; the .NET runtime is the general-purpose platform, not the substance.

## Architecture & components

The function pipeline is stateless by construction: a timer trigger fetches the current commitment set from object storage, validates each member's sketch envelope for well-formedness, runs the deterministic solver, and posts results. The solver treats pairing as balanced assignment — availability windows partition the week into slots, sketches feed a pairwise overlap score that the objective minimizes, and fairness is enforced by rotation memory carried inside the commitment set rather than server state. Because the solver is deterministic and its inputs are public commitments, every client can recompute the expected output and compare; a mismatch is a loud, provable signal of host misbehavior. Failure handling covers missing envelopes by excluding members with an explicit note, solver infeasibility by relaxing the diversity objective before the fairness constraint, and idempotent re-runs that produce identical pairings for identical inputs.

## Stack rationale

F# fits a pure, deterministic compute kernel unusually well: the rotation domain falls out naturally as discriminated unions and pipelines, immutability makes the "same inputs, same output, provable by clients" guarantee a property of the language rather than a discipline, and first-class serverless hosting keeps cold-start cost tolerable. Expressing the solver as a composition of total functions means the client-side verification routine can share the exact same code compiled for both runtimes, which is the difference between trusting the host and checking it.

## Threat model

The named adversary is a malicious server: the function host or its operator, curious about the team's internal skill distribution, or subtly rigging pairings to keep certain people apart. The rigging attack is answered by the deterministic solver plus public commitments — every client re-derives the expected pairing and detects deviation. The surveillance attack is answered by the sketch protocol: the host sees only non-reversible digests of skill tags and availability windows, never the underlying matrix, and the stateless design keeps nothing worth subpoenaing at rest. The worst a compromised host can do is deny service, which the team notices immediately.

## Novelty statement

Novelty is on the platform axis: collaboration scheduling computed in ephemeral serverless functions over blinded inputs, with client-side verifiability of the host's honesty. Existing rotation bots ingest readable skill and calendar data into a persistent service; fairness checkers exist only inside HR platforms. Neither minimizes what the host can learn nor makes the computation checkable by participants. The sketch protocol, the stateless commitment-driven solver, and the shared verification routine are introduced de novo and constitute a substantive architectural difference, not a rename or wrapper of an existing project.
