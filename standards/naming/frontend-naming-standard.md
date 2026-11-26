---
title: Frontend naming standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Naming rules for frontend code.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
---

# Frontend naming standard

This section defines naming expectations for web UI and Angular code.

## Rules

- components and services use PascalCase
- files use kebab-case
- route names and folders reflect the feature area
- UI state names read as positive conditions
- avoid one-letter variable names outside trivial loops

## Examples

- `PatientSearchService`
- `ExposureStatusComponent`
- `customer-search.service.ts`
- `isReady`

## Avoid

- `comp1`
- `tmpData`
- `doThing`
- generic names in shared UI modules
