---
id: apprentice-log
title: "apprentice-log — offline competency evidence for vocational apprentices"
slug: apprentice-log
summary: "A from-scratch Swift mobile app that lets apprentices capture signed competency evidence in connectivity-poor workshops and export a verifiable mastery portfolio without any cloud account or monthly fee."
status: accepted
created_at: "2026-10-29T21:01:00Z"
run_id: theme-work-20261029-2354
tags: [offline-sync, background-tasks, assessment, provenance]
platform: mobile
software_class: end_user_app
solution_category: education
architecture: offline_first
scale: solo_dev
language: swift
stack: mobile_native
layer: application
threat_model: [physical_access_attacker]
from_scratch: true
dependency_declared: []
novelty_axis: platform
---

## Problem

Vocational training happens in the worst possible places for software: welding bays, hospital wards, teaching kitchens, and factory floors, where signal is absent and gloves are on. Competency sign-off therefore lives on paper logbooks that get soaked, lost, and disputed years later, while cloud e-portfolio platforms demand accounts, connectivity, and subscriptions that apprentices and small trade schools cannot justify. The consequence is that an apprentice's real evidence — the weld passed, the procedure performed under supervision, the hours accumulated — evaporates exactly when it is needed for certification, job applications, or credit transfer between institutions.

## De-novo scope

Built completely from scratch: an evidence capture model unifying photographs, structured checklists, durations, and free-text observations into a single local record format; a witness-signing protocol where a supervisor confirms evidence on the apprentice's device using a key held on the supervisor's own phone, exchanged via a camera-scanned pairing code; a competency graph mapping each evidence item to curriculum nodes with attainment thresholds; an append-only local store with per-day compaction; and a portfolio exporter that emits a chain of signed statements verifiable offline by any third party. The app uses no portfolio service, no cloud identity, and no third-party SDK — the iOS platform with its camera, secure storage, and crypto primitives is the runtime rather than the idea's substance.

## Architecture & components

The app is organized as a capture pipeline feeding an evidence ledger. Capture runs as a background task that collects media and checklist state into a draft, then commits atomically to the append-only store. Signing is asynchronous by design: the apprentice's device queues unsigned evidence, and when a supervisor is physically present, a short-range exchange presents a summary for review and receives a signature over the record's hash. The competency graph is evaluated incrementally so progress indicators update as evidence lands. Export walks the ledger and emits a linked sequence of signed statements with a root digest. Failure handling covers interrupted capture via draft journals, signature-queue persistence across restarts, and storage-pressure migration that moves sealed evidence days from media-heavy to digest-only representations without breaking the chain.

## Stack rationale

Swift on iOS is chosen because the target population carries phones, not laptops, into workshops, and the required capabilities — camera capture, secure enclave key storage, biometric gating, and low-power background operation — are platform primitives that a native app can invoke directly and audit. The witness-signing protocol needs hardware-backed key material to be meaningful, and mobile platforms are the only consumer devices where that is universally present. A cross-platform webview shell would forfeit both the offline capture reliability and the secure key storage that the idea's trust model depends on.

## Threat model

The named adversary is a physical-access attacker: a phone left on a bench, stolen, or briefly borrowed in a workshop. The design answers this with biometric gating before any portfolio view, evidence encryption at rest keyed to the secure enclave, and auto-lock that requires re-authentication after a short absence. A stolen device must yield nothing editable — signatures live under keys the attacker cannot extract, so historical evidence cannot be forged or backdated from the device alone. Exports remain verifiable offline by any employer using only public material, meaning a lost phone destroys availability, never integrity.

## Novelty statement

Novelty is on the platform axis: verifiable competency evidence captured on mobile in genuinely offline, glove-friendly environments, where the supervisor's signature is collected device-to-device rather than through a portal. Cloud e-portfolio platforms are connectivity-first and account-based; paper logbooks have no integrity guarantees at all. Neither offers physical-presence signing with offline-verifiable export chains. This is a from-scratch architecture — the evidence ledger, the pairing-based witness protocol, and the chain-verified portfolio export are introduced de novo and are not a wrapper or rename of any existing project.
