---
title: Error Code Standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Repository-wide error-code taxonomy and handling standard.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: core
---

# Error Code Standard

This standard defines how application and service errors should be represented, classified, and communicated.

## 1. Purpose

Error codes provide a stable contract between services, operators, and developers. They should be structured, searchable, and actionable.

## 2. Standard principles

- every business or service failure should have a machine-readable code
- error codes must be stable and semantically meaningful
- codes should identify category, action, and failure theme
- raw exception messages should not be the primary contract
- codes should support monitoring, alerting, and support workflows

## 3. Format

Recommended format:

`[DOMAIN]-[CATEGORY]-[SEQUENCE]`

Examples:

- `AUTH-INVALID-CREDENTIALS`
- `ORDER-VALIDATION-001`
- `PAYMENT-TIMEOUT-002`
- `SCAN-EXTERNAL-FAILURE`

The exact format can be adapted to the platform, but it must remain consistent and searchable.

## 4. Category taxonomy

Use a limited set of categories:

- `VALIDATION` — user or input data is invalid
- `AUTH` — authentication or authorization failure
- `NOT_FOUND` — requested resource is missing
- `CONFLICT` — duplicate or incompatible state
- `TIMEOUT` — dependency or request timeout
- `EXTERNAL` — upstream service or third-party dependency failure
- `INFRASTRUCTURE` — infrastructure or platform issue
- `BUSINESS` — domain rule violation or process failure

## 5. Error handling rules

- map technical exceptions to stable domain error codes
- include the human-readable message separately from the code
- preserve correlation IDs and trace context in every error record
- do not expose raw internal stack traces to external clients unless explicitly allowed
- log the real exception while returning a sanitized code to callers

## 6. Error payload contract

An error payload should include:

- errorCode
- message
- traceId
- requestId
- timestamp
- optional correlation or business identifiers

Example:

```json
{
  "errorCode": "PAYMENT-TIMEOUT-002",
  "message": "Payment gateway timed out while authorizing the order.",
  "traceId": "abc123",
  "requestId": "req-789",
  "timestamp": "2026-08-21T14:00:00Z"
}
```

## 7. Backend-specific requirements

For backend services:

- return structured codes from domain or application services
- keep codes stable across versions when possible
- use a centralized mapping layer for known exceptions
- avoid creating one-off ad hoc strings in each controller or handler

## 8. Frontend-specific requirements

For UI code:

- translate backend error codes into user-friendly messages
- keep raw codes in telemetry and logs
- avoid exposing internal implementation details to the user

## 9. Anti-patterns

- `Error: something bad happened`
- one-off IDs with no taxonomy
- mixing business errors and infrastructure faults in the same message
- returning plaintext exception messages from external dependencies without sanitization
- using different code names for the same failure across service boundaries

## 10. Review checklist

Before finalizing a change, ensure:

- the code is stable and meaningful
- the same failure uses the same code everywhere
- the code maps to a documented category
- the user-facing message is separate from the machine-readable code
- the code is logged and monitorable
