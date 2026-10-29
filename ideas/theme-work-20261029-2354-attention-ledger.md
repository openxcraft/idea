---
id: attention-ledger
title: "attention-ledger — double-entry focus accounting for personal knowledge work"
slug: attention-ledger
summary: "A from-scratch Go desktop app that journals every window-focus change into a local append-only ledger and reconciles declared work intents against actual attention, producing weekly balance sheets of where focus really went."
status: accepted
created_at: "2026-10-29T20:55:00Z"
run_id: theme-work-20261029-2354
tags: [reconciliation, wal, audit-trails]
platform: desktop
software_class: end_user_app
solution_category: productivity
architecture: event_driven
scale: solo_dev
language: go
stack: systems
layer: application
threat_model: [compromised_endpoint]
from_scratch: true
dependency_declared: []
novelty_axis: combination
---

## Problem

Knowledge workers cannot answer the simplest question about their own day: where did my attention actually go. Manual timers fail because starting and stopping them is itself an act of attention, and honest self-report decays within hours. Screenshot-driven trackers are correctly rejected as workplace surveillance and are useless for personal use on sensitive work. Calendars record intentions, never reality, so weekly retrospectives are conducted on fiction. Without a ground-truth record of focus switching, every productivity decision — from tool choices to meeting policy — is made on vibes rather than evidence, and the cost of context switching stays permanently invisible.

## De-novo scope

Built completely from scratch: a focus-event daemon that subscribes to each platform's window-activation notification and writes timestamped events to an append-only log; a double-entry attention model where the user declares intents as budget envelopes and focus events debit them automatically; a reconciliation engine with user-editable matching rules that maps raw window titles onto intents; and a review UI that renders intent-versus-actual balance sheets, unclassified remainder lines, and switch-count rollups. Nothing is wrapped or forked from an existing tracker, and no dependency is declared beyond the Go runtime itself, which is the general-purpose platform rather than the idea's substance.

## Architecture & components

The core is a pure event-sourced pipeline: the daemon emits focus events, a write-ahead log persists them with checksums and batched fsync, and a nightly compaction job folds the WAL into immutable daily ledger pages with per-intent totals. The rule matcher is a deterministic, user-editable cascade evaluated at read time so reclassification never rewrites history. Reconciliation is a fold over daily pages producing double-entry rows: intended minutes on the credit side, observed minutes on the debit side, variance as the closing balance. Failure handling covers torn writes via checksum detection, clock-skew events marked rather than silently reordered, and a read-only fallback mode when the daemon cannot attach to the window manager.

## Stack rationale

Go is chosen deliberately: an always-on focus daemon must be a single static binary that installs without a runtime, must idle at negligible CPU and memory, and must talk to OS-level window APIs without heavyweight bindings. Go's standard library covers the daemon, the embedded review server, and the ledger compaction in one codebase, and its goroutine model maps cleanly onto the event loop of window notifications. The strict typing of the ledger schema matters at audit time: an event that fails to parse is a loud compile-time-style error, not a silent corrupted row.

## Threat model

The named adversary is a compromised endpoint: malware or a curious process on the same machine could read the raw ledger, and window titles are among the most sensitive data a machine produces — they leak client names, incident details, and personal browsing. The design answers this with at-rest encryption keyed to the OS keychain, an optional mode that stores only salted hashes of titles after compaction, and a hard architectural rule that the core daemon builds without any networking imports, so exfiltration requires a second, separately installed agent. Exports are explicit, user-triggered, and diffable in plain text.

## Novelty statement

Novelty is a combination of category and architecture: attention tracking treated as double-entry accounting, with intents as budget envelopes and observed focus as debits, reconciled weekly like a bookkeeper closes the books. Unlike existing event recorders such as ActivityWatch, which store raw events and stop there, the substance here is the reconciliation layer and the intent ledger it maintains; the raw event stream is merely its journal input. This is not a rename or wrapper of any existing project — the accounting model, the WAL-to-ledger compaction, and the network-free daemon build are introduced de novo.
