---
title: Backend Logging Standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Service-side application and infrastructure logging rules.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: supporting
---

# Backend Logging Standard

This section applies to application services, APIs, background jobs, and service-hosted components.

## Scope

- .NET services and platform components
- background worker jobs
- service-to-service calls
- persistence and infrastructure integrations

## Mandatory rules

1. Use structured logging with stable event names.
2. Every request or workflow must carry a traceId.
3. Use `ILogger` scopes to attach request context automatically.
4. Log business boundaries, not every internal method call.
5. Emit a summary log for each request or workflow.
6. Log exceptions with the exception object and relevant metadata.
7. Never log raw secrets, tokens, or regulated personal data.
8. Keep DEBUG logs off in production.

## Examples

```csharp
using var scope = logger.BeginScope(new Dictionary<string, object>
{
    ["traceId"] = traceId,
    ["userId"] = userId,
    ["tenantId"] = tenantId
});

logger.LogInformation("Order {OrderId} created for {UserId}", orderId, userId);
logger.LogError(ex, "Payment authorization failed for order {OrderId}", orderId);
```

## Required metadata

For each backend log event, include:

- timestamp
- level
- service
- environment
- traceId
- requestId
- userId (if available)
- durationMs (if relevant)
- result or errorCode

## Anti-patterns

- freeform string-only messages
- logging raw request payloads with secrets
- logging every loop iteration without value
- writing console-style logs in business code
- capturing stack traces without correlation
