---
id: language.javascript
title: JavaScript
category: language
version: 1.0.0
status: active
owners:
  - web-platform
applies_to:
  - javascript
depends_on:
  - universal.code-quality
  - universal.testing
  - universal.dependency-management
---

# JavaScript

## Purpose

Provide JavaScript-specific engineering guidance.

## Mandatory requirements

- Projects MUST declare supported Node.js or browser targets.
- Package manager and lock files MUST be committed.
- Asynchronous failures MUST be handled explicitly.
- Runtime configuration MUST be validated before use.
- Browser code MUST avoid unsafe DOM injection.

## Recommended practices

- Prefer small modules with clear side-effect boundaries.
- Use linting and formatting in CI.
- Keep dependency updates reviewable.

## Verification

Reviewers check lock files, runtime targets, async error handling, config validation, and DOM safety.
