---
title: Frontend error codes
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Frontend error-code structure and conventions.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
---

# Frontend error codes

This section defines how UI errors are classified and surfaced.

## Rules

- frontend should not invent unrelated internal codes
- backend error codes should remain the source of truth
- map backend codes to display-friendly user messages
- keep telemetry separate from user-facing wording

## Examples

- `AUTH-INVALID-CREDENTIALS` -> "Your session has expired"
- `PAYMENT-TIMEOUT-002` -> "The payment request timed out. Please retry."
