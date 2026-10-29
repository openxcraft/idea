---
id: delta-room
title: "delta-room — a federated decision log that diffs what a team agreed from what happened"
slug: delta-room
summary: "A from-scratch Kotlin server, shipped as a self-hosted container image, that turns every team decision into a signed delta in an append-only room log and federates rooms between instances with an obligation-diff view."
status: accepted
created_at: "2026-10-29T21:03:00Z"
run_id: theme-work-20261029-2354
tags: [deliberation, audit-trails, provenance, transparency]
platform: container_image
software_class: protocol_spec
solution_category: collaboration
architecture: federated
scale: small_team
language: kotlin
stack: jvm
layer: protocol
threat_model: [active_mitm, targeted_surveillance]
from_scratch: true
dependency_declared: []
novelty_axis: architecture
---

## Problem

Every team makes dozens of decisions a week, and every one of them evaporates into chat scrollback within days. Six months later, "why did we pick this vendor", "who agreed to this API freeze", and "what did we trade away in that incident review" become archaeology projects with inconclusive findings. Meeting-notes tools store prose, not structure; decision registers in wikis rot because updating them is manual work nobody owns. The deeper loss is follow-through: decisions carry obligations with dates, and nothing on earth reminds the team that the agreed review never happened. Cloud decision platforms exist, but they hold the team's institutional memory hostage to a subscription and an account system.

## De-novo scope

Built completely from scratch: a decision-delta specification defining each decision as a structured, signed record with context, options considered, the choice, an owner, and a review-by date; a room replication protocol in which servers exchange signed deltas with causal ordering and anti-entropy digests; an obligation-diff engine that joins decisions carrying promised follow-ups against later completion events and renders the gap; and a self-hosted server with server-rendered rooms, shipped as a single container image. The protocol specification plus its reference implementation is the deliverable — a team can run it on a spare machine. No existing wiki, chat platform, or federation codebase is wrapped; the JVM is the general-purpose runtime, not the idea's substance.

## Architecture & components

Each room is an append-only log of signed deltas with a per-room hash chain, replicated server-to-server over authenticated connections using push-pull delta exchange plus periodic anti-entropy comparisons of log digests. Causal ordering comes from vector clocks scoped per room, so concurrent decisions from two federated instances merge deterministically. The obligation-diff engine is a pure query layer: it joins decision deltas with owner and review-at fields against completion events and produces an aging report of unfulfilled commitments. Unsigned or badly ordered deltas are quarantined, never silently dropped. Failure handling covers replication retries with exponential backoff, digest mismatch triggering full-log reconciliation, and room exports as verifiable statement bundles for archiving.

## Stack rationale

Kotlin on the JVM suits a protocol-first server: sealed classes and immutable data classes make the delta schema closed under extension in exactly the way a protocol specification wants, coroutines model the fan-out of federation exchanges without thread-pool sprawl, and the ecosystem's maturity for TLS and container packaging means operational hardening is straightforward. Server-side rendering keeps the client auditable and dependency-free, and a single fat container image with a bundled database file keeps self-hosting honest — one process, one volume, no orchestration required for a small team.

## Threat model

Two adversaries are named. An active man-in-the-middle on the federation links could alter, replay, or reorder deltas in transit — answered by requiring end-to-end signatures on every delta (servers verify authorship, not just transport), hash-chain continuity checks, and replay windows keyed to vector clocks. A targeted surveillance observer — a federation peer or network adversary mapping which teams deliberate what, and when — is answered by opaque room identifiers with no global directory, minimal membership metadata in replication envelopes, and a design where a curious peer learns only the rooms explicitly shared with it, never the existence or subject of others.

## Novelty statement

Novelty is on the architecture axis: a federated, signed decision-delta protocol with a built-in obligation diff, self-hostable as a single container. Cloud decision-log products centralize institutional memory behind accounts; chat platforms keep prose with no structure or follow-through; wikis rot. None of them federate, none carry causally ordered signed deltas, and none render the gap between what was agreed and what actually happened. The protocol, the replication design, and the obligation-diff join are introduced de novo — a substantive architectural difference, not a rename or wrapper of any existing project.
