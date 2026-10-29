---
id: coldstash
title: "coldstash — a crypto-shredding cold-storage engine with per-tenant deletion proofs"
slug: coldstash
summary: "A from-scratch Python storage engine that writes immutable, per-tenant-keyed chunks to cold object storage and makes deletion a verifiable act of key destruction rather than an unlink promise."
status: accepted
created_at: "2026-10-29T17:16:00Z"
run_id: seed-001
tags: [storage-engines, wal, crypto-shredding, key-management, provenance]
platform: server_backend
software_class: storage_engine
solution_category: data_engineering
architecture: event_driven
scale: org_wide
language: python
stack: scripting
layer: service
threat_model: [insider, coerced_disclosure]
from_scratch: true
dependency_declared: []
novelty_axis: threat_model
---

## Problem

Organizations keep years of append-only archives (compliance logs, sensor history, medical records) in cheap object storage, and when a retention window closes or a data subject invokes erasure, the only honest answer they can give is that someone ran a delete job. Bit rot, replicated caches, backups, and judicial hold make unlink-style deletion unprovable, and legal teams cannot answer "is it really gone?" with evidence. Existing cold-storage engines offer immutability in abundance but treat deletion as an operational ceremony with no cryptographic residue.

## De-novo scope

Everything is new: a chunked, content-addressed storage format where each chunk is sealed under a per-tenant data-encryption-key (DEK) derived through a versioned KDF tree; an event-sourced metadata journal (the WAL) mapping tenants, retention classes, and key epochs to chunk manifests; a key-ephest engine that rotates DEKs on schedule and destroys expired epochs by writing threshold-sharded key funerals; and an auditor CLI that, given a tenant ID, produces a deletion attestation proving every chunk is sealed under a destroyed epoch. No existing database or backup tool is wrapped or forked. No dependency is declared beyond the Python runtime and its standard library, which is a general-purpose platform, not the substance.

## Architecture & components

Ingest events land in a local WAL, a packer seals records into chunks of configurable size, each chunk is encrypted with a fresh DEK, and the manifest journal records the binding between chunk, tenant, retention class, and key epoch. Deletion is redefined: the engine schedules a key funeral that threshold-destroys the DEK material across shard files and records the funeral event, leaving ciphertext chunks in place as meaningless bytes until space reclamation. The auditor walks the journal, verifies every sealed chunk references a destroyed epoch, and emits a signed attestation. Failure handling is ledger-first: crashed funerals resume idempotently, and a half-destroyed epoch is modeled as pending, never as gone.

## Stack rationale

Python suits a service whose hot path is I/O-shape rather than CPU-shape: chunk sealing is dominated by storage latency, and the standard library's hashlib, hmac, and ssl provide auditable primitives without opaque binary extensions. A pure-Python engine keeps the deletion logic reviewable line by line, which matters because the product's value is precisely that a regulator can read the mechanism. Maturity of Python's testing and packaging ecosystem makes the attestation tooling easy to ship to auditors on any platform.

## Threat model

Two adversaries are named. An insider with full storage access can read ciphertext but cannot resurrect destroyed-key epochs because DEK material exists only as threshold shards, and funeral events are externally witnessed in an append-only log the insider cannot silently rewrite without detection. Coerced disclosure is bounded by design: the engine cannot surrender plaintexts it never kept, so a compelled handover of storage and metadata yields only shards and sealed chunks, and the attestation machinery documents exactly what a legal order can and cannot recover. Residual risk is explicit: keys destroyed only after a policy delay remain live until the funeral completes, and the journal shows that window honestly.

## Novelty statement

The novelty is on the threat-model axis: deletion as a first-class, cryptographically attested state transition with auditor-facing proofs, rather than an unlink side effect. The key-funeral ledger and the per-tenant deletion attestation are substantive architectural differences from every existing cold-storage engine, none of which treat "prove it is gone" as a queryable feature.
