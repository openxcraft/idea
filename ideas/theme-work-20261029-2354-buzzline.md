---
id: buzzline
title: "buzzline — a haptic presence vocabulary for small teams over direct BLE"
slug: buzzline
summary: "A from-scratch Swift SDK and wearable app that encodes short teammate-presence nudges into a shared haptic vocabulary exchanged phone-to-phone over BLE, with no cloud relay, no message log, and no location data."
status: accepted
created_at: "2026-10-29T21:09:00Z"
run_id: theme-work-20261029-2354
tags: [ble, p2p, ios, delay-tolerant]
platform: wearables
software_class: sdk
solution_category: communication
architecture: p2p
scale: org_wide
language: swift
stack: mobile_native
layer: framework
threat_model: [passive_network_observer]
from_scratch: true
dependency_declared: []
novelty_axis: combination
---

## Problem

Teammates working apart need a cheap way to signal presence — "I'm here if you need me", "I'm in flow, don't interrupt", "need a hand in five minutes" — without the heavyweight ceremony of a message. Every existing channel is text-first: a message demands composition, interrupts the recipient with a screen, and leaves a logged artifact in someone's datacenter. Status indicators in chat apps answer the opposite question and quietly become surveillance instruments that employees learn to resent and defeat. What is missing is the digital equivalent of a knock on the doorframe: a gesture that carries intent, costs both parties almost nothing, and leaves no trace.

## De-novo scope

Built completely from scratch: a haptic intent vocabulary of a dozen grammatical elements — greeting, availability, need-a-hand, in-flow, heads-up, acknowledgment, celebration — each with urgency and duration encodings rendered as distinct vibration patterns; a BLE transport protocol that frames intent messages into advertising payloads exchanged directly between devices with no relay, no pairing ceremony, and no server; a policy state machine governing do-not-disturb windows, repetition backoff, and per-contact permissions; and the SDK plus a companion watch app implementing the vocabulary on the wrist. The SDK is the deliverable so other client teams can adopt the protocol; the Apple platforms are the first binding. No messaging SDK, chat platform, or mesh library is wrapped; CoreBluetooth and CoreHaptics are platform primitives, not the idea's substance.

## Architecture & components

The SDK core is a pure encoding-decoding layer over the vocabulary with the policy machine as its only stateful component, so the same core could back future bindings on other platforms. Transport uses BLE advertising frames carrying encrypted, rotating payloads; no GATT connection is required for the common case, which keeps the exchange asymmetric-cheap: the sender bursts, receivers decode passively. The policy machine resolves conflicts — an in-flow window suppresses all but urgent intents, and repeated nudges back off exponentially to make nagging structurally expensive. Delivery is best-effort and delay-tolerant by design: presence nudges are perishable, so a nudge that arrives late is silently dropped rather than queued. Failure modes cover BLE unavailability with a clear offline state, replay attempts rejected by nonces, and congestion handled by advertising-interval backoff.

## Stack rationale

Swift is chosen because the haptic vocabulary's natural home is the wrist, and watchOS exposes haptics and BLE with the energy discipline a always-nearby wearable demands: background advertising modes, hardware haptic control, and low-power pairing primitives. The vocabulary core is pure value types and small state machines, which Swift expresses without ceremony, and the SDK form factor lets the protocol live above any single app decision. An energy-hungry cross-platform framework would undermine the ambient, always-available property that makes presence signaling work at all.

## Threat model

The named adversary is a passive network observer: anyone with a BLE sniffer in range can log advertising traffic, and presence metadata — who nudges whom, how often, and when — is exactly the data a stalker or an overreaching employer would want. The design answers this at the protocol level: payloads carry rotating pseudonymous device addresses with no stable identifiers, frames are encrypted under pair keys exchanged out-of-band when teammates link, intents contain no message text and no timestamps beyond coarse buckets, and the absence of any server means no central log of interactions exists to seize. A sniffer learns only that some devices exchanged short encrypted bursts.

## Novelty statement

Novelty is a combination of platform and architecture: presence communication as a peer-to-peer haptic protocol delivered through an SDK for wearables, with no cloud, no message store, and no status surveillance. Messaging apps centralize text conversation and its logs; presence indicators in chat platforms report to employers; existing BLE beacon schemes broadcast context, not intentional teammate signals. The vocabulary grammar, the advertising-frame protocol, and the perishable best-effort delivery model are introduced de novo — a substantive architectural difference rather than a rename or wrapper of any existing project.
