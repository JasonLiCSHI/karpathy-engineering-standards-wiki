---
title: Backend naming standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Naming rules for service and backend code.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
---

# Backend naming standard

This section defines naming expectations for service and domain code.

## Rules

- Use PascalCase for public types.
- Use camelCase for local variables and parameters.
- Use domain names and business terms where possible.
- Use event names aligned with operation semantics.
- Keep namespace and folder names aligned with feature boundaries.

## Examples

- `OrderService`
- `PatientRepository`
- `CreateScanExecution`
- `ProtocolSelectionResult`

## Avoid

- `Service1`
- `ProcessData`
- `HandleThing`
- inconsistent names across similar concepts
