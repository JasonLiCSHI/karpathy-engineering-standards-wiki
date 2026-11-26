---
title: Frontend Logging Standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Frontend logging rules.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
---

# Frontend Logging Standard

This section applies to UI, route-level workflows, browser-facing logic, and client-side diagnostics.

## Scope

- Angular or web client code
- client actions and route transitions
- user-triggered workflows
- browser-side diagnostics that are useful for triage

## Mandatory rules

1. Keep frontend logs focused on user actions and significant UI state changes.
2. Use request correlation IDs when available from backend responses.
3. Log user-triggered failures with actionable context.
4. Avoid noisy logs for every render cycle or every keystroke.
5. Do not log secrets, session tokens, or personal identifiers in full.
6. Prefer structured fields over plain message strings.

## Recommended fields

- timestamp
- level
- component or route
- user action
- traceId (if available)
- result
- errorCode

## Examples

```ts
logger.info('UserLoginAttempt', { userId, tenantId, result: 'success' });
logger.warn('PatientSearchFailed', { queryLength, reason: 'timeout' });
```

## Anti-patterns

- `console.log` for production behavior
- logging full input payloads from user forms
- logging tokens or passwords
- repeated logs from render loops
- unstructured and unsearchable UI telemetry
