---
id: language.shell
title: Shell
category: language
version: 1.0.0
status: active
owners:
  - platform-operations
applies_to:
  - shell
depends_on:
  - universal.code-quality
  - universal.security
---

# Shell

## Purpose

Provide shell scripting guidance for automation and operational tasks.

## Mandatory requirements

- Shell scripts MUST declare their interpreter.
- Scripts MUST fail clearly on unexpected errors.
- Inputs and paths MUST be quoted safely.
- Destructive commands MUST require explicit review and safeguards.
- Secrets MUST NOT be printed to logs.

## Recommended practices

- Prefer small scripts with named functions.
- Use ShellCheck where available.
- Move complex logic to a general-purpose language when maintainability suffers.

## Verification

Reviewers check quoting, error handling, interpreter choice, destructive actions, and log safety.
