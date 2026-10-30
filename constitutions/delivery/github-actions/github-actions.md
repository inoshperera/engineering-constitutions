---
id: platform.github-actions
title: GitHub Actions
category: platform
version: 1.0.0
status: active
owners:
  - engineering-enablement
applies_to:
  - github-actions
depends_on:
  - universal.security
  - universal.testing
---

# GitHub Actions

## Purpose

Provide GitHub Actions guidance for CI, release, and repository automation.

## Mandatory requirements

- Workflows MUST use least-privilege permissions.
- Third-party actions MUST be reviewed and pinned to stable versions.
- Secrets MUST be scoped only to jobs that require them.
- CI MUST run the validation commands documented by the project.
- Release workflows MUST require explicit triggers or protected branches.

## Recommended practices

- Cache dependencies only when cache keys are deterministic.
- Keep reusable workflows parameterized for consuming repositories.
- Separate validation from deployment jobs.

## Verification

Reviewers check workflow permissions, action versions, secret scope, trigger safety, and required validation jobs.
