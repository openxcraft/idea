---
id: quorum-mesh
title: "quorum-mesh — store-carry-forward gossip daemon for off-internet neighborhoods"
slug: quorum-mesh
summary: "A from-scratch Elixir daemon that carries signed, encrypted message bundles between phones and Linux routers over BLE and Wi-Fi Direct, syncing a delay-tolerant local web when the internet is unavailable or censored."
status: accepted
created_at: "2026-10-29T17:12:00Z"
run_id: seed-001
tags: [p2p, gossip, delay-tolerant, ble, traffic-analysis-resistance]
platform: protocol_daemon
software_class: p2p_node
solution_category: censorship_resistance
architecture: p2p
scale: internet_scale
language: elixir
stack: functional
layer: protocol
threat_model: [passive_network_observer, active_mitm, mass_surveillance]
from_scratch: true
dependency_declared: []
novelty_axis: architecture
---

## Problem

During blackouts, natural disasters, protest shutdowns, and rural dead zones, people physically clustered within a few hundred meters cannot exchange messages because every mainstream messenger assumes a working route to a data center. Existing mesh chat apps require every node to be online simultaneously and drain batteries with continuous scanning, while delay-tolerant networking research never produced a phone-friendly daemon with usable defaults. The missing artifact is a gossip protocol where messages ride physical mobility and opportunistic contacts instead of live end-to-end paths.

## De-novo scope

The daemon is built from scratch in its entirety: the wire protocol (framing, versioning, capability negotiation), an epidemic-routing bundle format with per-neighbor rate limits, a content-addressed store with expiry and quota, an Ed25519 signing and X25519 key-agreement layer, and an anti-entropy set-reconciliation handshake based on Bloom-filter sketches. Nothing is wrapped or forked from existing mesh stacks. The one declared dependency is the BEAM runtime, which is a general-purpose language platform, not the substance of the idea.

## Architecture & components

Each device runs a bundle router with three layers: a transport pool that discovers neighbors over BLE advertisements and Wi-Fi Direct groups with exponential-scan backoff; a storage layer holding bundles as content-addressed objects with per-topic quotas and TTL-based eviction; and a policy layer applying destination, interest, and transitivity rules per topic. Contact opportunities trigger reconciliation: peers exchange sketch digests, compute the bundle difference set, and transfer only missing objects with per-contact bandwidth caps. The VM's supervision trees isolate driver crashes, and bundle validity is checked end-to-end via signatures and topic-epoch counters, so a compromised relay cannot inject undetected content.

## Stack rationale

Elixir on the BEAM suits thousands of concurrent contacts, retries, and timers without per-connection OS threads, and supervision trees give the daemon self-healing behavior that censorship-response tools need under adversarial radio conditions. Binary pattern matching keeps the wire-protocol codec concise and testable, while hot-reloadable modules allow upgrading the reconciliation logic in the field without dropping neighbor state. Functional immutability also makes the content store trivially auditable and crash-consistent.

## Threat model

Three adversaries are named. A passive network observer should learn nothing about social structure: payloads are sealed to recipient keys, topic names are hash-milled, and padding plus cover-traffic bundles keep contact sizes uniform. An active man-in-the-middle cannot forge or replay bundles because each carries an Ed25519 signature over the topic epoch, and mutual X25519 agreement encrypts every transfer. Mass surveillance is blunted by the offline topology itself: no central servers exist to subpoena, and the reconciliation handshake reveals only sketch digests rather than stored content. Residual limits are documented honestly: a global adversary correlating physical encounters still observes contact graphs, which the design reduces but cannot erase.

## Novelty statement

The novelty is on the architecture axis: the protocol-first design treats human mobility as the transport, combining store-carry-forward routing, set-reconciliation handshakes, and BEAM supervision in one daemon. Unlike always-on mesh chats, it assumes disjoint connectivity windows as the normal case, which is a substantive architectural difference rather than a rename of any existing messenger.
