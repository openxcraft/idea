---
id: carbon-cruise
title: "carbon-cruise — a kernel governor that schedules laptop work by grid carbon intensity"
slug: carbon-cruise
summary: "A from-scratch Linux cpufreq kernel module that shifts deferrable local work such as builds and backups into low-carbon electricity windows, exposing carbon budgets as a first-class per-process-group scheduling policy."
status: accepted
created_at: "2026-10-29T21:11:00Z"
run_id: theme-work-20261029-2354
tags: [energy-measurement, carbon-accounting, drivers, schedulers]
platform: kernel_driver
software_class: scheduler
solution_category: sustainability
architecture: real_time
scale: org_wide
language: c
stack: systems
layer: kernel_driver
threat_model: [supply_chain_attacker]
from_scratch: true
dependency_declared: []
novelty_axis: category
---

## Problem

A laptop's carbon footprint depends on when it draws power almost as much as how much it draws, yet every deferrable job a developer runs — the big compile, the disk backup, the model download, the index rebuild — fires the moment it is requested, regardless of whether the grid is coal-heavy or windy at that hour. Datacenters solved this with carbon-aware batch scheduling; laptops, which collectively draw gigawatts at the wall, have nothing but idle timers and userspace cron scripts with no enforcement teeth. The information (grid intensity forecasts) is public, and the actuator (CPU frequency and scheduling policy) sits in the kernel — but nothing connects the two, so the largest fleet of computing devices on earth runs carbon-blind.

## De-novo scope

Built completely from scratch: a Linux cpufreq governor module implementing a carbon policy with two process classes — immediate and deferrable — where deferrable cgroups are frequency-capped and delayed while forecast carbon intensity is above a user-set budget, then released into full performance during predicted low-carbon windows; a sysfs control plane exposing budget, class assignment, and the cached intensity curve; a tiny userspace feed daemon that fetches public grid forecasts and writes the curve into sysfs for the governor to consume; and per-cgroup joule accounting derived from platform energy counters so the savings the policy produces are measurable, not vibes. No existing governor is patched, no scheduler framework is wrapped, and the userspace feed is a data source adapter, not the idea's substance — the kernel module is.

## Architecture & components

The governor is a self-contained cpufreq policy driver: it reads the cached intensity curve and current class mix each sampling period, computes a frequency ceiling for deferrable work in real time, and hands immediate-class threads the platform default, so interactive latency never degrades. Deferral itself is cooperative-but-enforced — deferrable cgroups run at capped frequency and reduced timeslice share until the window opens, which delays completion without starving anything. The feed daemon is deliberately dumb and stateless: fetch, validate, write curve, exit — and the governor holds the last good curve with a staleness bound if the feed dies. Failure handling covers missing energy counters (accounting degrades to time-based estimates, flagged as such), malicious curve values rejected by sanity bounds, and an unload path that restores the default governor cleanly.

## Stack rationale

C is not a lifestyle choice here; it is the contract. cpufreq governors are in-kernel code, the interfaces they must satisfy are C ABI, and the hot path runs every sampling tick on machines that must not pay a runtime's overhead for a policy that exists precisely to save energy. Deterministic memory — no dynamic allocation in the sampling path, fixed tables for the curve cache — keeps the module auditable in the way kernel review demands, and the total line count stays small enough for a security reviewer to hold the whole policy in their head.

## Threat model

The named adversary is a supply-chain attacker: kernel modules execute at ring 0, so a compromised build or a trojaned feed value is a full-system compromise vector. The design answers this by minimizing what the module trusts — curve values from userspace are sanity-bounded and cannot command frequencies directly, only shift the deferral window — and by keeping the module small, allocation-free in hot paths, and signature-verifiable like any distributable kernel module. The feed daemon runs unprivileged with no write access beyond one sysfs file, so the high-privilege attack surface is the module's own parsing of a bounded table, reviewable in a single sitting.

## Novelty statement

Novelty is on the category axis: sustainability enters the corpus as an enforced scheduling property rather than a reporting afterthought, delivered at the kernel/driver layer, which no existing corpus idea occupies. Existing carbon-shifting tools are cron wrappers that only choose when to launch jobs, with no enforcement once a job runs, and datacenter-grade carbon schedulers live in cluster orchestrators far above the laptop. Placing the policy in the governor layer gives it authority over every deferrable thread on the machine and per-cgroup joule accounting to prove its own effect. The governor design, the two-class deferral policy, and the bounded-curve control plane are introduced de novo — a substantive architectural difference, not a rename or wrapper of any existing project.
