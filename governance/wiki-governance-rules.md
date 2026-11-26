---
title: Wiki Retrieval, Conflict Resolution, and Update Rules
category: concepts
tags: [wiki, navigation, conventions, architecture, visibility/public]
sources: [distilled-engineering-standards]
summary: Operational rules for progressive wiki retrieval, source-of-truth conflict resolution, and anti-drift maintenance in an AI-assisted Obsidian Wiki.
created: 2026-11-24T15:54:00+08:00
updated: 2026-11-24T15:54:00+08:00
base_confidence: 0.95
lifecycle: reviewed
lifecycle_changed: 2026-11-24
tier: core
provenance:
  extracted: 1.00
  inferred: 0.00
  ambiguous: 0.00
relationships:
  - target: "[[governance/progressive-loading]]"
    type: extends
---

# Wiki Retrieval, Conflict Resolution, and Update Rules

This page defines the operational contract for reading from and writing to an AI-assisted Obsidian Wiki (following the `llm-wiki` pattern). It complements [[governance/progressive-loading|Progressive Knowledge Loading]] with concrete precedence, anti-drift, and indexing guardrails.

## 1. Retrieval Protocol (Read Order)

Always move from cheapest index primitives to full page reads; never blindly read full folders:

1. **Recent context**: Read `hot.md` first for the latest 3 major updates, active threads, and core invariants.
2. **Graph index pass**: Query the compiled wiki graph index first; if a single-line summary answers the question, stop without opening full bodies.
3. **Full inventory scan**: Check `index.md` (covers 100% of substantive pages with 1-line summaries).
4. **Semantic / keyword search**: Run local search (such as `qmd` or targeted frontmatter/section `grep`) over the vault.
5. **Bounded body read**: Open at most the top 3 matching pages, preferring `tier: core` > `tier: supporting` and skipping `tier: peripheral` or `lifecycle: archived` unless specifically tracing history.

## 2. Source-of-Truth Precedence & Conflict Resolution

When two pages in the vault disagree, resolve conflicts strictly in this priority order:

| Priority | Scope | Authority Rule |
| --- | --- | --- |
| 1 (Highest) | Canonical distilled architecture pages & ADRs | Authoritative truth for current contracts, threading, and boundaries. |
| 2 | Engineering standards (`tier: core` practice guides over `tier: supporting` sub-guides) | Authoritative rules for coding, concurrency, async, performance, naming, logging, and error handling. |
| 3 | External reference evidence & samples | Empirical reference only; never overrides canonical project contracts. |
| 4 (Lowest / Historical) | `lifecycle: archived` / `tier: peripheral` historical mirrors and superseded plans | Historical traceability only. **Must not** be cited as current behavior; follow `superseded_by` to the canonical page. |

## 3. Update & Write Guardrails (Anti-Drift)

Every ingest, capture, or sync write must obey these invariants:

1. **Single-line YAML `title:` and `summary:`**: Avoid YAML folded multiline scalars (`title: >-` or `summary: >-`) when lightweight graph parsers expect single-line values. Keep `summary:` concise (1–2 sentences, **≤ 200 characters**).
2. **Required classification metadata**: Every substantive page must set explicit `tier:` (`core` | `supporting` | `peripheral`), `lifecycle:`, `base_confidence:`, `sources:`, and typed `relationships:` (`uses`, `extends`, `implements`, `derived_from`, `contradicts`, `replaces`, `related_to`) when connections are clear.
3. **No dual-maintenance of raw mirrors**: Update only the canonical distilled page(s). Leave archived mirrors untouched so drift does not spread.
4. **Canonical Obsidian wikilinks**: Link pages using consistent vault-root Obsidian wikilinks so Obsidian Graph View and orphan checks stay intact.
5. **Visibility discipline**: Tag reusable public-safe knowledge with `visibility/public`, and keep proprietary or company-specific pages explicitly isolated in private vaults so public exports stay zero-leak.
6. **Hard size caps for hot/log files**:
   - Keep `hot.md` **≤ 800 words** (last 3 `Recent Activity` items + active threads + core invariants).
   - Archive older `log.md` entries when logs grow large so search indexes stay high-signal.
7. **Post-write verification**: After writing pages, update `index.md` and `hot.md`, then refresh local search embeddings if configured.
