---
id: fragment-trace
title: "fragment-trace — no_std heap-fragmentation profiler for bare-metal Rust"
slug: fragment-trace
summary: "A from-scratch no_std instrumentation library that records every allocator event on Cortex-M MCUs into a lock-free SRAM ring buffer and reconstructs a fragmentation timeline on the host."
status: accepted
created_at: "2026-10-29T17:10:00Z"
run_id: seed-001
tags: [mcu, profiling, memory-management, rtos, power]
platform: embedded_mcu
software_class: profiler
solution_category: observability
architecture: library_only
scale: solo_dev
language: rust
stack: systems
layer: library
threat_model: [side_channel_observer]
from_scratch: true
dependency_declared: []
novelty_axis: platform
---

## Problem

Embedded Rust applications on microcontrollers with 8-256 KiB of RAM die silently from heap fragmentation: after days of packet handling, allocation fails even though free memory exists, missed real-time deadlines follow, and post-mortem core dumps show nothing because the heap state is gone. Host-side profilers such as valgrind or heaptrack cannot run on-device, and JTAG memory snapshots destroy the temporal dimension of the failure. Firmware teams currently debug fragmentation with printf guesswork, which changes timing and often hides the bug.

## De-novo scope

Everything is built from scratch: a `no_std` Rust crate exposing a `GlobalAlloc` wrapper that hooks the global allocator, a compact binary event encoding (pointer, size class, operation, monotonic timestamp) fitting in 8 bytes, a single-producer single-consumer ring buffer placed in a reserved SRAM bank via a linker symbol, and a host-side decoder in the same workspace that renders a fragmentation timeline. No existing profiling project is wrapped or forked. The single declared dependency is the Rust toolchain itself, which is the general-purpose language runtime and not the substance of the idea.

## Architecture & components

The allocator wrapper emits events from any context including interrupts into the SPSC ring buffer with zero allocation and bounded worst-case time, using atomics that degrade gracefully on single-core MCUs. The linker script reserves a configurable bank (default 4 KiB) for the buffer, and a boot-time constant seeds the producer-consumer indices from flash to survive watchdog resets. The host decoder consumes a raw SRAM dump over any existing debug channel, rebuilds every allocation and free in order, computes free-block counts, largest-fitting block, and fragmentation entropy per time bucket, and renders an ASCII timeline plus a worst-ten-sites report. Failure handling is explicit: buffer overrun sets an overflow flag in the header rather than corrupting older events.

## Stack rationale

Rust is the only mainstream systems language whose `no_std` ecosystem, `GlobalAlloc` trait, and zero-cost abstractions make an interrupt-safe allocator wrapper feasible without an OS. Const generics size the buffer per chip family at compile time, and `unsafe` is quarantined behind the ring-buffer module with safety proofs documented. The decoder lives in the same crate family to keep the event format and the rendering logic version-locked, which avoids the schema drift that plagues separate tool ecosystems.

## Threat model

The named adversary is a side-channel observer: co-resident firmware or a privileged task that can read shared SRAM and infer application behavior from allocation patterns, such as request timing or payload-size fingerprints. The design answers it by compacting events to fixed-size bins, optionally XOR-masking the ring with a boot-random key, and allowing the profiler to be compiled out of release builds with a cargo feature so production firmware leaks nothing. Residual risk is documented: mask reuse across resets is detectable by a patient observer, so the mask rekeys whenever an entropy source is available.

## Novelty statement

The novelty is on the platform axis: temporal heap-fragmentation profiling executed on-MCU as a drop-in library, rather than host emulation or static analysis. Existing profilers assume an OS and virtual memory; this one treats the physical SRAM layout and interrupt constraints of bare-metal targets as first-class design inputs, and the watchdog-resilient ring header is an architectural difference, not a rename of any host profiler.
