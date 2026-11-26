---
title: Backend error codes
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Backend error-code structure and conventions.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
---

# Backend error codes

This section defines error-code conventions for service and domain layers.

## Standard pattern

Use a stable, readable code with a domain prefix and a category.

Examples:

- `ORDER-VALIDATION-001`
- `PAYMENT-TIMEOUT-002`
- `AUTH-INVALID-CREDENTIALS`

## Rules

- codes must be deterministic and documented
- keep domain names consistent with business terms
- use a category from the standard taxonomy
- log both the code and the underlying exception details
