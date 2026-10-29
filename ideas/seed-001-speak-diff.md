---
id: speak-diff
title: "speak-diff — a narrating CLI that makes code review audible for blind developers"
slug: speak-diff
summary: "A from-scratch C# command-line tool that parses unified diffs into semantic units and narrates them through the OS speech pipeline with review-specific pacing, voice earcons, and structural cues."
status: accepted
created_at: "2026-10-29T17:18:00Z"
run_id: seed-001
tags: [narration, screen-reader, diffing, hardening]
platform: cli
software_class: tool
solution_category: accessibility
architecture: single_user_local
scale: solo_dev
language: csharp
stack: dotnet
layer: application
threat_model: [supply_chain_attacker]
from_scratch: true
dependency_declared: []
novelty_axis: category
---

## Problem

Blind and low-vision developers experience code review through screen readers reading raw unified diffs line by line, which flattens the review's most important structure: which function changed, whether a hunk adds or removes, and how hunks relate. Listening to a 600-line diff means reconstructing file geometry from memory, and existing TUIs assume a visual two-column mental model. No tool today speaks a diff the way a sighted colleague would summarize it over a voice call: "in the auth middleware, they added a retry counter here, and removed the timeout check there."

## De-novo scope

Built from scratch: a unified-diff parser into a typed hunk model with file, function-context inference, and change classification (add, remove, modify, move heuristic); a narration planner that turns hunks into spoken prose with configurable verbosity; an audio-profile engine that distinguishes voice pitch and speed per change type so additions and deletions are recognizable without a spoken label; and a human-readable summary mode with counts by file and by change class. No screen reader is wrapped, and no existing diff tool is forked. No dependency is declared beyond the .NET runtime and the operating system's built-in speech synthesis interface, both general-purpose platforms rather than the substance.

## Architecture & components

The pipeline is three pure stages connected by immutable records: parse produces hunks with inferred function context using lightweight brace and indentation heuristics; plan orders narration by review importance (API changes first, tests second, formatting last) with a per-file preface; and render drives the OS speech pipeline while streaming transcript lines to stdout for deaf-blind users of braille displays. Interactive controls (pause, skip file, replay hunk, jump to next function) are single-keystroke so no visual scan is needed to find position. Failure handling is graceful: malformed diff chunks are announced as such and skipped rather than silently dropped, and the tool never exits mid-utterance, always finishing the current hunk first.

## Stack rationale

C# on .NET is the pragmatic choice because the two platforms where this tool is most needed, Windows with Narrator/NVDA and macOS via VoiceOver, are both reachable through .NET's runtime with per-OS speech adapters kept behind one interface. Span-based parsing keeps the diff reader allocation-light for large reviews, and discriminated unions via records model hunk kinds with compile-time exhaustiveness, which prevents the classic bug of narrating a removal as an addition. Single-file publish produces a self-contained binary that needs no installation rights on locked corporate machines.

## Threat model

The named adversary is a supply-chain attacker: accessibility tools are installed with unusual trust and run against arbitrary repository content, so a poisoned build or a hijacked dependency would execute with the developer's rights while also hearing every proprietary diff. The defense is a minimal dependency policy with zero third-party packages, reproducible builds with pinned toolchain versions and published checksums, and a hard rule that diff content is treated as untrusted input: no shell-outs, no format-string execution, and strict Unicode sanitization before any text reaches the speech layer. Residual risk is documented: OS speech APIs are outside the tool's trust boundary, so the tool strips control characters and never passes markup through.

## Novelty statement

The novelty is on the category axis: treating audible review as a first-class accessibility product rather than a screen-reader bolt-on. The semantic narration planner, earcon-like audio profiles, and braille-transcript mirroring are substantive architectural differences from generic diff viewers with TTS enabled, none of which structure spoken output for review comprehension.
