---
id: language.typescript
title: TypeScript
category: language
version: 1.0.0
status: active
owners:
  - web-platform
applies_to:
  - typescript
depends_on:
  - language.javascript
---

# TypeScript

## Purpose

Provide TypeScript-specific guidance for typed JavaScript projects.

## Mandatory requirements

- Type checking MUST run in CI.
- New code MUST avoid `any` unless the reason is documented locally.
- Public types MUST describe stable contracts.
- Unsafe type assertions MUST be minimized and reviewed.

## Recommended practices

- Enable strict compiler options where feasible.
- Prefer domain types over loosely shaped objects.
- Keep generated types separate from manually maintained types.

## Verification

Reviewers check compiler settings, type-check output, use of unsafe assertions, and public type definitions.
