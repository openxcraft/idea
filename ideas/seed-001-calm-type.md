---
id: calm-type
title: "calm-type — a step-by-step type-inference visualizer for teaching Hindley-Milner"
slug: calm-type
summary: "A from-scratch offline web application implementing a small ML-like language whose unification engine visualizes every inference step, constraint, and substitution as students type."
status: accepted
created_at: "2026-10-29T17:14:00Z"
run_id: seed-001
tags: [visualization, step-by-step, repl, browsers]
platform: web
software_class: interpreter
solution_category: education
architecture: offline_first
scale: small_team
language: typescript
stack: js_ts
layer: runtime
threat_model: [malicious_server]
from_scratch: true
dependency_declared: []
novelty_axis: combination
---

## Problem

Type inference is the concept that most reliably breaks students entering functional programming: they see `fun x -> x x` fail and receive a one-line error with no story of why. Existing explainer sites show final results or fixed animations, but never expose the live constraint graph, the unification steps, or the substitution chain that produced the answer. Instructors cannot assign "watch inference happen on your own expression" homework because no artifact lets them. The result is ritual memorization of rules instead of an internalized algorithmic intuition.

## De-novo scope

Built completely from scratch: a hand-written lexer and recursive-descent parser for a small ML-like expression language with let-polymorphism; a constraint-generation pass that emits named constraints; a unification engine producing an explicit, replayable step list (occur-check, dereference, decompose, substitute); and an interactive canvas rendering the constraint graph, substitution stack, and type-scheme instantiation as draggable nodes. The app is a single offline-capable bundle with no server component beyond static hosting. No existing compiler, tutorial framework, or visualization library is wrapped. No dependency is declared beyond the browser JavaScript runtime itself, which is the general-purpose platform, not the substance.

## Architecture & components

The pipeline is pure functions from source text to an immutable event log: parse produces an AST with identity paths, constraint generation walks the AST and appends constraints, and the unifier reduces them while emitting typed steps that the renderer consumes. Every step is a serializable record with the exact AST region it touches, so clicking a step highlights code and vice versa. Infinite types, free-variable capture, and let-generalization each produce dedicated step kinds with plain-language commentary. A deterministic fuzz-driven example gallery ships in the bundle for offline use, and state is kept in the URL fragment so lessons can be shared without any backend.

## Stack rationale

TypeScript's own type system prototypes the algebra the app teaches, and discriminated unions make the step log impossible to construct incorrectly at compile time. The browser is the delivery target because classrooms range from Chromebooks to locked-down lab machines where nothing can be installed, and an offline-first single-page app needs no permissions or accounts. Strict immutability in the engine keeps step replay consistent, which arbitrary interactive mutation would not.

## Threat model

The named adversary is a malicious server: a compromised or coerced host could swap the shipped bundle to teach subtly wrong unification or inject tracker payloads. The defense is that the app is fully static and cache-offline after first load, all artifacts carry integrity metadata suitable for subresource-integrity pinning, and the build is reproducible from the public source so any tampered distribution can be diffed. No user data leaves the device because there is no telemetry, no accounts, and no network calls after boot, which also makes the tool safe for exams.

## Novelty statement

Novelty is a combination of category and layer: an educational artifact that is itself an interpreter layer, exposing inference internals live rather than as fixed media. The replayable, path-addressed step log and the clickable bidirectional linking between source and constraint graph are substantive architectural differences from any existing explainer site.
