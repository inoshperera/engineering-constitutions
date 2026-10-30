---
id: universal.documentation
title: Documentation
category: universal
version: 1.0.0
status: active
owners:
  - documentation-team
applies_to:
  - all
depends_on:
  - universal.engineering-principles
---

# Documentation

## Purpose

Ensure project knowledge remains discoverable and current.

## Mandatory requirements

- Each project MUST have a README that explains purpose, setup, validation, and ownership.
- Significant architecture decisions MUST be documented with context and trade-offs.
- Public APIs, configuration, upgrade steps, and migrations MUST be documented.
- Documentation updates MUST be included with implementation changes that alter behavior.
- PRs MUST link updated specifications, API contracts, setup instructions, or release guidance where affected, or record why no update is needed.
- Comments SHOULD explain why code exists rather than restating what code does.

## Recommended practices

- Keep operational runbooks close to the services they describe.
- Link decisions to issues, incidents, or design reviews when helpful.
- Prefer examples that can be copied and run.

## Verification

Reviewers verify that changed behavior, configuration, APIs, and migration steps are reflected in documentation in the same change, or that the recorded reason for no update holds.
