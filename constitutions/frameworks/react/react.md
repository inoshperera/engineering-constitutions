---
id: framework.react
title: React
category: framework
version: 1.0.0
status: active
owners:
  - web-platform
applies_to:
  - react
depends_on:
  - language.typescript
  - universal.api-design
---

# React

## Purpose

Provide React-specific guidance for user-interface projects.

## Mandatory requirements

- Components MUST keep rendering behavior predictable and side effects explicit.
- Shared components MUST document props that affect accessibility or data loading.
- User-visible state MUST handle loading, empty, error, and success outcomes.
- Forms MUST validate user input and preserve meaningful error messages.
- Client code MUST NOT expose secrets or privileged tokens.

## Recommended practices

- Prefer composition over deep component inheritance patterns.
- Keep data fetching boundaries clear.
- Use accessible controls and semantic HTML before custom widgets.

## Verification

Reviewers check component state handling, accessibility, validation, data-fetching boundaries, and client-side secret exposure.
