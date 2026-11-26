---
title: Operations and Alerting Standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Operations and alerting standard.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
---

# Operations and Alerting Standard

This section defines how logs feed monitoring, dashboards, and incident response.

## Core principles

- Summary logs drive alerting and metrics.
- Detail logs support investigation.
- ERROR and CRITICAL events should be classified and routed to monitoring.
- WARN should be monitored for trend analysis, but not necessarily hard-alerted.

## Alert mapping

- DEBUG: no alerting
- INFO: informational only
- WARN: trend/monitoring, not necessarily critical
- ERROR: actionable operational failure
- CRITICAL: immediate incident response

## Required operational indicators

- service name
- environment
- timestamp
- traceId
- result
- errorCode
- durationMs

## Dashboard expectations

Logging should support:

- request success/failure rate
- p95 and p99 latency
- dependency failure counts
- retry counts
- top business error events

## Alert-quality requirements

A log event should be alertable only when:

- it is actionable
- it has enough context for triage
- it is stable enough to aggregate
- it includes correlation identifiers
