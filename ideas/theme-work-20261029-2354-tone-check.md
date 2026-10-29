---
id: tone-check
title: "tone-check — a local linter for escalation patterns in team messages"
slug: tone-check
summary: "A from-scratch JavaScript browser extension that lints outgoing team messages for blame framing, passive aggression, and late-night send pressure entirely inside the tab, with zero network permissions and no keystrokes stored."
status: accepted
created_at: "2026-10-29T21:07:00Z"
run_id: theme-work-20261029-2354
tags: [nlp, linting, extensions, summarization]
platform: browser_extension
software_class: linter
solution_category: communication
architecture: embedded_in_another_app
scale: org_wide
language: javascript
stack: js_ts
layer: library
threat_model: [compromised_endpoint]
from_scratch: true
dependency_declared: []
novelty_axis: class
---

## Problem

Written async communication is where remote teams lose the most goodwill, because text strips the tone signals that keep disagreement civil in person. A draft written at 11pm reads angrier than intended; "you always break the build" assigns blame where the situation caused it; sarcasm lands as hostility across cultures. The commercial answer — cloud "tone AI" assistants — demands piping every draft, including messages about layoffs and incidents, through a third party, which most security-conscious organizations correctly refuse. The result is that the people who most need a tone check before hitting send are the least likely to have one, and everyone else develops a superstition about re-reading messages that no one actually performs under pressure.

## De-novo scope

Built completely from scratch: an escalation grammar implemented as a real parser over message text, with pattern classes for blame framing, absolutes and generalizations, sarcasm markers, dismissive acknowledgments, and imperative stacking, each carrying severity and confidence; a deterministic reframing suggester that proposes template rewrites ("you always miss reviews" toward "the last three PRs needed review round-trips — can we look at the pattern together"); a send-time advisor applying local-clock heuristics for late-night sending pressure; and the extension shell that underlines spans in the compose box and renders finding cards. The linter core is a library with a clean API so other shells — a mail client add-in, a CLI for drafted announcements — can embed it. No ML service, no third-party NLP package, and no existing grammar tool is wrapped; the browser extension platform is the runtime, not the substance.

## Architecture & components

The content script injects a capture hook into the mail web app's compose box, feeds draft text through the parser, and renders findings as underlines with severity badges; every finding carries the matched span, the rule identifier, and a one-line rationale, so suggestions are explainable rather than magical. Rules compose from small matcher primitives — token sequences, punctuation density, sentence-initial pronouns, clause structure — making the grammar itself reviewable in a single afternoon, which is the point. Findings are ephemeral by design: nothing is written to durable storage, drafts are processed in memory and discarded, and the options page holds only rule toggles. Failure handling degrades to silence — a page structure the parser cannot recognize disables the linter rather than guessing.

## Stack rationale

Plain JavaScript, no build step, is a deliberate choice: extension review is a security question, and a shipped artifact a security reviewer can read line-by-line in one sitting is a feature, not a nostalgia statement. The extension APIs are JavaScript-native; span-level underline rendering needs direct DOM access that heavier frameworks only obscure; and a dependency-free artifact means the supply chain that could compromise a tone advisor — of all things — is empty by construction.

## Threat model

The named adversary is a compromised endpoint sibling: another extension, desktop app, or keylogger with access to the same browsing session or machine, seeking to harvest draft text. The design minimizes what there is to steal rather than pretending to prevent compromise: zero network permissions declared in the manifest (provably no exfiltration path from this extension), findings and draft text held only in volatile memory, no analytics, no storage of messages, and a permission surface limited to the mail hosts themselves. A fully compromised operating system defeats any in-browser tool — the honest goal is to add zero new attack surface while still catching the escalation patterns that damage teams.

## Novelty statement

Novelty is on the class axis: a linter — with the discipline of a linter, rule identifiers, severities, explainable findings, suppressible rules — applied to human communication instead of code. Cloud tone assistants ingest every draft into someone else's datacenter and return unexplainable verdicts; grammar checkers fix correctness, not escalation. The deterministic, span-addressed, network-free design with an ephemeral in-memory draft model is a substantive architectural difference from every existing approach, introduced de novo rather than as a rename or wrapper of an existing project.
