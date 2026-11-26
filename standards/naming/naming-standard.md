---
title: Naming Standard
category: conventions
tags: [conventions, visibility/public]
sources: [distilled-engineering-standards]
summary: Repository-wide naming standard for code, APIs, logs, and business identifiers.
created: 2026-09-08T14:05:23+08:00
updated: 2026-09-08T14:05:23+08:00
base_confidence: 0.6
lifecycle: draft
lifecycle_changed: 2026-09-08T14:11:24+08:00
tier: core
---

# Naming Standard

This document defines the naming conventions for code, identifiers, events, and operational objects so the repository remains readable, consistent, and searchable.

## 1. Core principles

- names should describe intent, not implementation detail
- prefer domain language over technical jargon when possible
- use consistent terms across services, modules, and UI code
- avoid generic names such as `data`, `info`, `thing`, `helper`
- keep names stable for APIs, events, and user-facing fields

## 2. General naming rules

### 2.1 PascalCase for types and public symbols

Use PascalCase for:

- classes
- interfaces
- enums
- components
- services
- DTOs and models

Examples:

- `PatientStudyService`
- `OrderValidationResult`
- `ExposureStatusComponent`

### 2.2 camelCase for variables and locals

Use camelCase for:

- local variables
- function arguments
- private fields
- method parameters

Examples:

- `patientId`
- `isReady`
- `retryCount`

### 2.3 UPPER_SNAKE_CASE for constants

Use UPPER_SNAKE_CASE for constants and static configuration keys.

Examples:

- `DEFAULT_TIMEOUT_MS`
- `MAX_RETRY_COUNT`

### 2.4 Avoid ambiguous abbreviations

Prefer readable names over cryptic short names.

Preferred:

- `patientId`
- `studyInstanceUid`
- `authorizationToken`

Avoid:

- `pid` unless the domain absolutely requires it
- `x` or `tmp` for meaningful state
- non-standard abbreviations in shared APIs

## 3. Domain naming

Names should reflect the business language and existing CT model terminology.

Examples:

- `Study`
- `Customer`
- `ScanExecution`
- `OrderRepository`

Do not mix implementation language with business language unless the domain is explicitly technical.

## 4. Event and log naming

Event names should be explicit and stable.

Preferred pattern:

`Verb + Object + Result`

Examples:

- `OrderCreated`
- `PaymentAuthorized`
- `UserLoginFailed`
- `ScanCompleted`

Avoid vague names such as:

- `HandleOrder`
- `ProcessDone`
- `DoStuff`

## 5. Method naming

Methods should describe an action in terms of the domain or behavior.

Preferred:

- `CreateOrder()`
- `ValidatePatientStudy()`
- `GetActiveProtocol()`

Avoid:

- `DoIt()`
- `Stuff()`
- `Handle()` without context

## 6. Boolean naming

Use positive, readable condition names.

Preferred:

- `IsReady`
- `HasUnsavedChanges`
- `CanStartAcquisition`

Avoid:

- `NotReady`
- `IsNotActive`
- `CanNotStartAcquisition`

## 7. Frontend naming

For Angular and UI code:

- use PascalCase for components and services
- use kebab-case file names
- use descriptive route and feature names
- keep folder names aligned with the feature or route they represent

Examples:

- `customer-search.service.ts`
- `exposure-status.component.ts`
- `PatientSearchService`

## 8. Backend naming

For backend and service code:

- prefer feature folders over technical folders when business grouping is clear
- use domain terminology in class names and namespace structure
- avoid naming low-level classes using UI or workflow labels that don't match business meaning
- be consistent with the repository's CT domain vocabulary

## 9. Naming review checklist

Before finishing a change, confirm the following:

- names describe behavior or meaning, not incidental implementation
- abbreviations are used consistently and are domain-appropriate
- the same concept uses the same name everywhere
- events and logs use stable vocabulary
- public API names are understandable without reading implementation
