---
title: Logging Standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Repository-wide standard for application and service logging.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: core
---

# Logging Standard

This document defines the project logging standard for backend, service, and application components. It follows the principle that logs are structured events, not plain-text output, and is compatible with .NET `ILogger`, Serilog, NLog, OpenTelemetry, and similar observability stacks.

## 1. Objectives

Logs must serve three goals at the same time:

- Operations: determine service health, detect failures, and trigger alerts.
- Development: help localize and explain issues quickly.
- Audit: answer who did what, when, and with what result.

## 2. Log layering

Do not rely on a single log stream. Logs are split into two layers.

### 2.1 Summary Log

A Summary Log is a single, stable, business-level record for a complete request or workflow.

Use it for:

- monitoring
- alerting
- SLA measurement
- dashboards and trends

Example:

```json
{
  "event": "CreateOrder",
  "traceId": "abc123",
  "userId": "u001",
  "durationMs": 35,
  "result": "Success"
}
```

Rules:

- one summary log per request or business action when possible
- stable event names
- long retention
- searchable and dashboard-friendly

### 2.2 Detail Log

A Detail Log records critical steps inside a workflow, such as downstream calls, retries, validations, and state transitions.

Use it for:

- troubleshooting
- root-cause analysis
- replaying request flow

Rules:

- more verbose than summary logs
- valuable for short-term investigations
- not intended for long-term operational dashboards

## 3. Log levels

Use a unified five-level standard.

### DEBUG

For development diagnostics only.

Include:

- parameters
- return values
- execution path
- context debug values

Should be off in production.

### INFO

Business and lifecycle events that are expected and useful.

Examples:

- order created
- payment succeeded
- service started
- task completed

### WARN

Expected but noteworthy conditions that do not stop the system.

Examples:

- cache expired
- invalid input supplied
- retry succeeded after transient failure

### ERROR

Business or dependency failures that require investigation.

Examples:

- database connection failure
- payment failure
- third-party API timeout

This level should be visible to monitoring and alerting as appropriate.

### CRITICAL

System-level failures that can impact core service reliability or data integrity.

Examples:

- service startup failure
- essential dependency unavailable
- out-of-memory condition
- data corruption

This level must trigger alerting.

## 4. Structured logging principle

Logs must be structured events, not free-form strings.

Avoid:

```text
Order create success user=1 amount=100
```

Prefer:

```json
{
  "event": "OrderCreated",
  "userId": 1,
  "amount": 100
}
```

In .NET, prefer message templates with `ILogger`:

```csharp
logger.LogInformation(
    "Order {OrderId} created for {UserId}",
    orderId,
    userId);
```

This preserves structured data fields instead of only a text string.

## 5. Required log fields

Every log should include standard metadata. The exact names can vary by implementation, but the semantics should be consistent.

### 5.1 Base fields

- timestamp
- level
- service
- environment
- version

### 5.2 Trace fields

All request-scoped logs must include:

- traceId
- spanId
- requestId

### 5.3 User fields

When applicable:

- userId
- tenantId
- clientIp

### 5.4 Business fields

Add business keys when relevant:

- orderId
- paymentId
- taskId
- jobId

### 5.5 Result fields

- result
- errorCode
- durationMs

## 6. Event naming

Use a consistent event name pattern:

`Verb + Object + Result`

Examples:

- `UserLoginSucceeded`
- `UserLoginFailed`
- `OrderCreated`
- `OrderCancelled`
- `PaymentAuthorized`
- `InventoryReserved`

Avoid vague names such as:

- start creating order
- order processing finished
- executed here

The event name should be explicit, stable, and easy to aggregate.

## 7. Exception logging

Never call `LogError(ex.Message)` without the exception object.

Bad:

```csharp
logger.LogError(ex.Message);
```

Good:

```csharp
logger.LogError(
    ex,
    "Payment authorization failed");
```

Exception logs must include:

- ExceptionType
- Message
- StackTrace
- TraceId
- relevant business parameters

## 8. Scope usage

Use logging scopes to carry request context automatically.

```csharp
using (logger.BeginScope(new Dictionary<string, object>
{
    ["traceId"] = traceId,
    ["userId"] = userId,
    ["tenantId"] = tenantId
}))
{
    logger.LogInformation("Processing order {OrderId}", orderId);
}
```

This allows downstream logs to inherit the same context without repeating fields every time.

## 9. Sensitive data handling

Never log secrets or regulated identifiers in full.

Never log:

- Password
- Token
- AccessKey
- RefreshToken
- national ID numbers
- full credit card numbers
- full phone numbers

Use masking or hash-based redaction instead:

- `138****1234`
- `6222********8888`

If a field is sensitive, do not write raw values even in debug logs.

## 10. Observability and OpenTelemetry

Logs should be structured, searchable, aggregatable, and traceable.

This is especially important when a system sends logs to:

- ELK
- Datadog
- Azure Monitor
- OpenTelemetry collectors
- Grafana

A log pipeline should be able to:

- search by traceId
- group by service or event name
- correlate logs with telemetry spans and metrics
- detect anomalies through aggregates

.NET `ILogger` is designed for this approach and integrates naturally with OpenTelemetry.

## 11. Recommended default rules

These are the condensed company policies:

1. Only structured logs are allowed.
2. Every request must have a traceId.
3. Separate Summary Log and Detail Log.
4. DEBUG is for development only.
5. WARN is not an alert by default.
6. ERROR must be actionable and traceable.
7. CRITICAL must trigger alerting.
8. Logs must not impact business performance.
9. Sensitive data must never be logged in raw form.
10. Event names must describe business meaning, not ad hoc narration.

## 12. Tooling alignment

This standard is intended to work with:

- .NET / `ILogger`
- Serilog
- NLog
- Log4Net
- Logback
- SLF4J
- Zap
- OpenTelemetry

## 13. Practical implementation guidance

When implementing logging in code:

- log at the boundary of the business action
- include correlation IDs in scopes
- log failures with exceptions, not only messages
- include a stable event name and key identifiers
- do not dump payloads with secrets
- omit noisy debug logs from production paths

## 14. Summary

The core rule is simple:

> Log structure, not strings.

The system should support monitoring, troubleshooting, and audit with structured, searchable, traceable events that can be fed into modern observability platforms without forcing each component to invent its own log format.
