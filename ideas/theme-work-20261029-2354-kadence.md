---
id: kadence
title: "kadence — a compiler that type-checks your workday before you live it"
slug: kadence
summary: "A from-scratch OCaml TUI that compiles a day-plan written in a small constraint language into a concrete time-blocked schedule, reporting overlaps, starved deep-work, and recovery violations as compile errors with source spans."
status: accepted
created_at: "2026-10-29T20:57:00Z"
run_id: theme-work-20261029-2354
tags: [parsing, linting, curriculum]
platform: tui
software_class: compiler
solution_category: productivity
architecture: single_user_local
scale: solo_dev
language: ocaml
stack: functional
layer: framework
threat_model: [supply_chain_attacker]
from_scratch: true
dependency_declared: []
novelty_axis: class
---

## Problem

Calendars are passive containers: they hold what you drag into them and never complain when the day is impossible. The damage surfaces only in lived time — a lunch eaten during a call, deep work scheduled at the post-lunch energy trough, back-to-back meetings with zero recovery margin, a deadline colliding with school pickup. Auto-scheduling SaaS products attack this with opaque cloud heuristics and account lock-in, so the plan itself cannot be reviewed, versioned, or trusted. What is missing is a planner that refuses to compile an infeasible day the way a compiler refuses broken source: loudly, specifically, and before execution begins.

## De-novo scope

Built completely from scratch: a lexer and recursive-descent parser for a small day-plan language where fixed events, elastic tasks with durations and priorities, energy windows, commute blocks, and recovery rules are first-class declarations; an elaboration pass that turns them into interval variables; a constraint solver performing unification and priority-ordered backtracking over those variables; a diagnostics engine that emits typed errors carrying source spans — overlap at line 12, deep-work starved below its declared minimum, recovery invariant violated after a three-meeting chain; and a terminal UI that renders the compiled schedule with inline errors and writes plain ICS output for interop. No existing calendar library, solver package, or planner application is wrapped; the only runtime is the OCaml toolchain itself.

## Architecture & components

The pipeline is staged and purely functional: source text parses to an AST with identity paths, elaborates to a constraint graph, solves to either a layout or a minimal unsatisfiable core, and renders. Diagnostics are algebraic data values carrying spans, severity, and suggested fixes, so the TUI can underline the offending plan line exactly as an IDE underlines a type error. Determinism is a design invariant — the same plan always compiles to the same schedule — which makes plans diffable in version control. Failure handling includes unsatisfiable-core extraction that names the smallest conflicting set of declarations, and a watch mode that recompiles on save incrementally by reusing the elaborated graph.

## Stack rationale

OCaml fits because the entire application is a compiler, and its strengths are exactly compiler-shaped: algebraic data types make diagnostics impossible to construct incorrectly, pattern matching keeps the elaboration and solving passes total and exhaustively checked, and native compilation yields a fast single binary that starts instantly in a terminal. Interval arithmetic and unification are natural in a functional core with no hidden state, and the deterministic semantics that the idea promises are far easier to guarantee in a language without ambient mutation. A garbage-collected scripting language would make the solver's determinism claims harder to audit.

## Threat model

The named adversary is a supply-chain attacker: a compromised package from the language's ecosystem could exfiltrate the user's calendar content, which reveals meetings, health appointments, and personal obligations. The design answers this by declaring zero external dependencies — the standard library covers parsing, solving, terminal rendering, and ICS emission — so the dependency graph is empty by construction and trivially auditable. Builds are reproducible bytecode or static binaries from a pinned toolchain, and an audit flag prints the complete dependency closure, which is expected to remain the compiler and nothing else.

## Novelty statement

Novelty is on the class axis: this is a compiler whose target artifact is a human schedule, treating the day-plan as a typed program with checked invariants rather than a document to render. Cloud auto-planners compute schedules with hidden heuristics and no diagnostics; calendar tools draw whatever is dragged in. Neither offers compile-time errors with source spans, unsatisfiable-core reporting, or deterministic diffable output. The architectural difference from existing planners is substantive and structural — elaboration to interval variables, a unification-based solver, and spanned diagnostics — not a rename or thin wrapper of any existing project.
