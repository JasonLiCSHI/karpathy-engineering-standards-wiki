---
title: Configuration-Driven Layout
category: concepts
tags: [configuration, layout, ui, visibility/public]
sources: [distilled-engineering-standards]
created: 2026-07-14T15:52:02+08:00
updated: 2026-08-18T11:36:41+08:00
summary: Configuration-driven layout stores UI-independent content placement in configuration while leaving rendering details to the page.
provenance:
  extracted: 0.68
  inferred: 0.3
  ambiguous: 0.02
base_confidence: 0.58
lifecycle: draft
lifecycle_changed: 2026-07-14
tier: supporting
---

# Configuration-Driven Layout

Configuration-driven layout stores UI-independent page placement and content metadata in configuration instead of runtime parameter objects.

## What It Is

In a configuration-driven parameter UI, XML or JSON configuration defines category identity, tab identity,
and explicit category/tab/parameter order. Category and tab `order` are required;
parameter `order` is optional and defaults to zero. Runtime parameter objects focus on
values and domain interaction, while parent-owned ordered collections expose content
sequence to the page.

The key rule is negative: configuration does not describe parameter row/column
coordinates or UI rendering details such as width, height, column width, row
height, row span, column span, header style, spacing, or responsive behavior.
Those belong to the page and design system.

Favorites are stricter than layout metadata: XML declares a favorites tab but does not persist which columns are favorited. The service tracks favorite columns at runtime and rejects persistent `favorite` attributes in configuration.

## Related

- [[architecture/clean-architecture]]
