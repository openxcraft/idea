---
id: envdock
title: "envdock — a machine-checked specification for reproducible development environments"
slug: envdock
summary: "A from-scratch Python toolchain that declares a team's pinned development environment in one manifest and conformance-checks any laptop or CI runner against it, emitting machine-readable drift reports for fleets."
status: accepted
created_at: "2026-10-29T21:13:00Z"
run_id: theme-work-20261029-2354
tags: [provisioning, ci, reproducibility, build]
platform: cli
software_class: protocol_spec
solution_category: infrastructure
architecture: batch
scale: internet_scale
language: python
stack: scripting
layer: spec_standard
threat_model: [insider]
from_scratch: true
dependency_declared: []
novelty_axis: category
---

## Problem

"It works on my machine" persists not because people are careless but because the definition of a working environment is scattered across a README paragraph, three dotfiles, a Dockerfile that CI outgrew, and tribal knowledge in two senior heads. Drift is invisible until it breaks, and when it breaks, the debugging session reconstructs the environment by archaeology. Existing tools attack provisioning — building a fresh, hermetic environment from a lockfile — which solves new machines but says nothing about the heterogeneous laptops and runners a real team already has. Nobody can answer "is this machine still conformant with what the team declared six months ago", so environments rot silently between incident reports.

## De-novo scope

Built completely from scratch: the envdock manifest specification — a versioned, declarative document pinning toolchain versions, expected environment variables, service dependencies with port and liveness expectations, file existence and checksum expectations, and platform variants for the same logical environment; a conformance checker that compiles a manifest into a probe plan of pure, sandboxed probe functions (version-string capture, checksum verification, port probing), executes them in parallel with timeouts, and emits both a human diff view and a machine-readable drift report carrying the manifest hash and probe versions; and a batch mode that fans the check across a CI matrix or fleet inventory and aggregates one fleet-level drift report. The deliverable is the specification plus its reference checker. No configuration management framework, package manager, or container tool is wrapped — envdock checks existing machines rather than building new ones, which is precisely the unoccupied ground.

## Architecture & components

The checker is a three-stage batch pipeline: manifest compilation validates the document and resolves platform variants into a concrete probe plan; parallel execution runs probes as isolated subprocess calls with per-probe timeouts and captured exit codes; reporting folds results into findings keyed by manifest clause, so every failure names the exact expectation that was violated. Probes are pure functions with declared capabilities — read-only filesystem access, subprocess execution, socket connect — making the checker's own attack surface explicit and auditable. Exit codes encode conformance level so CI can gate on drift thresholds. Failure handling covers manifest version mismatch (explicit error, never best-effort guessing), probe crashes (reported as inconclusive with captured stderr rather than skipped silently), and partial fleets (the aggregate report marks machines that could not be reached instead of averaging over absence).

## Stack rationale

Python fits a tool whose job is shelling out to arbitrary toolchains and parsing their version strings: it is preinstalled on effectively every developer machine and CI runner the checker must inspect, so conformance checking begins with zero installation friction — the checker itself must not be the hardest thing to install. The standard library covers subprocess management, hashing, JSON emission, and parallel execution, keeping the reference implementation dependency-free and its own supply chain empty, which matters for a tool that runs everywhere and inspects everything.

## Threat model

The named adversary is an insider: a team member with repository access who quietly edits the manifest to mask accumulating drift, or smuggles weakened expectations past review to make a pet environment "conformant". The design answers this with an audit trail — every drift report embeds the manifest hash and probe version it was produced with, so historical reports can be re-verified against the manifest history in version control, and quiet retroactive edits surface as mismatches between recorded hashes and history. Significant-manifest changes require a fresh report run on a reference machine before CI accepts the change, making weakening the spec an explicit, reviewed act rather than a silent edit.

## Novelty statement

Novelty is on the category axis: infrastructure treated as a conformance-checking problem over existing machines rather than a provisioning problem over new ones. Declarative environment tools build hermetic worlds but say nothing about the laptops a team already owns; containerization sidesteps the question by abandoning the machine. The manifest specification, the compiled probe-plan model, and the hash-anchored drift audit trail are introduced de novo — a substantive architectural difference, not a rename or wrapper of any existing project.
