---
id: language.kotlin
title: Kotlin
category: language
version: 1.0.0
status: active
owners:
  - kotlin-platform
applies_to:
  - kotlin
depends_on:
  - universal.code-quality
  - universal.testing
  - universal.dependency-management
---

# Kotlin

## Purpose

Provide Kotlin-specific engineering guidance.

## Mandatory requirements

- Kotlin projects MUST declare supported Kotlin and JVM or Android targets.
- Nullability MUST be represented in types rather than deferred to runtime checks.
- Coroutines MUST define explicit scopes and cancellation behavior.
- Interoperability with Java APIs MUST handle platform types carefully.

## Recommended practices

- Prefer data classes for immutable value carriers.
- Keep extension functions close to the domain they serve.
- Use linting and formatting in CI.

## Verification

Reviewers check target versions, nullability, coroutine scopes, Java interop, and lint output.
