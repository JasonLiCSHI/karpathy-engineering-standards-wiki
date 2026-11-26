---
title: Progressive Knowledge Loading
category: concepts
tags: [wiki, navigation, visibility/public]
sources: [distilled-engineering-standards]
summary: Defines the L0, L1, and L2+ reading depths used to retrieve wiki knowledge efficiently without confusing them with the folder hierarchy.
provenance:
  extracted: 1.00
  inferred: 0.00
  ambiguous: 0.00
base_confidence: 0.95
lifecycle: reviewed
lifecycle_changed: 2026-08-21
tier: core
created: 2026-08-21T16:04:13+08:00
updated: 2026-08-21T16:04:13+08:00
---

# Progressive Knowledge Loading

Use reading depth to retrieve only the amount of knowledge needed for the
current decision. Reading depth is independent of the `L0-Project/L1-...`
folder hierarchy: folders classify knowledge; loading levels control how much
of it is read.

## L0 - Summary

One sentence that lets a reader quickly decide whether a page, topic, or
subtree is relevant.

Every maintained page provides this through its frontmatter `summary`.

## L1 - Overview

The core facts, boundaries, decisions, and usage scenarios needed for planning
or choosing a direction. Read this before opening detailed evidence or source
material.

Section `README` pages and the opening sections of maintained pages provide L1.

## L2 - Detail

The complete operational detail, evidence, constraints, edge cases, and source
links needed to implement, debug, review, or validate a decision.

## L3+ - Optional Depth

Add deeper layers only when they improve retrieval:

- **L3** - implementation-specific flows, component breakdowns, or source
  evidence;
- **L4+** - specialized cases such as test fixtures, migration notes, protocol
  variants, or historical decision records.

Do not add depth merely to mirror folders or source files. Each additional
level must reduce the cost of finding or understanding the relevant knowledge.

## Reading rule

Start at L0, open L1 only when the topic is relevant, then retrieve L2+ only for the active task. Link related pages so readers can deepen one concern without loading unrelated material. For Source-of-Truth conflict precedence and update guardrails, see [[governance/wiki-governance-rules|Wiki Retrieval, Conflict Resolution, and Update Rules]].
