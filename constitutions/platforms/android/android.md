---
id: framework.android
title: Android
category: framework
version: 1.0.0
status: active
owners:
  - mobile-platform
applies_to:
  - android
depends_on:
  - language.kotlin
  - universal.security
  - universal.testing
---

# Android

## Purpose

Provide Android-specific guidance for mobile applications.

## Mandatory requirements

- Android projects MUST declare supported SDK levels.
- Sensitive data MUST use platform-appropriate secure storage.
- Runtime permissions MUST be requested only when needed and explained in context.
- UI changes MUST consider accessibility and localization.
- Background work MUST respect platform lifecycle constraints.

## Recommended practices

- Keep presentation state separate from platform APIs where practical.
- Use instrumentation tests for critical flows.
- Monitor startup and rendering performance for user-facing changes.

## Verification

Reviewers check SDK targets, storage choices, permissions, accessibility, lifecycle behavior, and mobile test coverage.
